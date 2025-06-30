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
			Plot.binX({y: "proportion-facet", domain: [-14400, 14400]}, {x: "difference_s", fy: "arrival_year"})
		)
	]
})
```
