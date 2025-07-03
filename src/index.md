# VIA Explorer

<div class="grid grid-cols-4">
	<div class="card">
		<h2>Average delay</h2>
		${Math.round(d3.mean(arrival_times, d => d.difference_s) / 60)}
	</div>
</div>

```js
const stations_raw = await FileAttachment('./data/viarail.ca/gtfs/stops.txt').csv()
const stations_with_arrivals = await FileAttachment('./data/viarail.ca/times/station-codes.csv').csv()

const stations = stations_raw
	.filter(d => stations_with_arrivals.some(s => d.stop_code === s.stop_code))
```

```js
const stations_search = view(Inputs.search(
	stations,
	{
		columns: ["stop_code", "stop_name"]
	}
))
```

```js
const station_oi = view(Inputs.table(stations_search, {
	sort: "stop_code",
	multiple: false,
	width: {
		stop_code: 100
	}
}))
```


```js
const arrival_times = FileAttachment('./data/viarail.ca/times/train-times.parquet').parquet()
```

```js
station_oi === null ? html`<p>Pick a station to see the chart.</p>` : Plot.plot({
	title: `Distribution of arrival punctuality at ${station_oi.stop_name} (${station_oi.stop_code})`,
	y: {
		percent: true
	},
	x: {
		transform: d => d / 60,
		label: "Minutes (early to late)"
	},
	fy: {
		transform: d => d.toString(),
		label: "Year"
	},
	marks: [
		Plot.rectY(
			[...arrival_times]
				.filter(d => d.stop_code === station_oi.stop_code),
			Plot.binX({y: "proportion-facet", domain: [-30 * 60, 240 * 60]}, {x: {interval: 15 * 60, value: "difference_s"}, fy: "arrival_year", tip: true})
		)
	]
})
```

```js
station_oi === null ? html`<p>Pick a station to see the chart.</p>` : Plot.plot({
	title: `Distribution of arrival punctuality at ${station_oi.stop_name} (${station_oi.stop_code})`,
	y: {
		percent: true,
		transform: d => Math.round(d * 1000) / 10
	},
	x: {
		label: "Minutes (early to late)",
		domain: punctuality_bins
	},
	fy: {
		transform: d => d.toString(),
		label: "Year"
	},
	marks: [
		Plot.barY(
			[...arrival_times]
				.filter(d => d.stop_code === station_oi.stop_code)
				.map(bin_arrival_punctuality),
			Plot.groupX({y: "proportion-facet"}, {x: "punctuality", tip: true, fy: "arrival_year"})
		)
	]
})
```

```js
const punctuality_bins = ["-16 or earlier", "-15 to 0", "0 to 15", "16 to 60", "61 to 240", "241 or later"]

const bin_arrival_punctuality = (row) => {
	const newRow = row
	const arrival_difference = row.difference_s / 60

	let punctuality = undefined

	if (arrival_difference < -15) {
		punctuality = "-16 or earlier"
	} else if (arrival_difference < 0) {
		punctuality = "-15 to 0"
	} else if (arrival_difference <= 15) {
		punctuality = "0 to 15"
	} else if (arrival_difference <= 60) {
		punctuality = "16 to 60"
	} else if (arrival_difference <= 240) {
		punctuality = "61 to 240"
	} else if (arrival_difference > 240) {
		punctuality = "241 or later"
	}

	return {
		...newRow,
		punctuality
	}
}
```
