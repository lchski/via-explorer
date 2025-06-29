# VIA Explorer

<div class="grid grid-cols-4">
	<div class="card">
		<h2>Average delay</h2>
		${Math.round(d3.mean(arrival_times, d => d.difference_s) / 60)}
	</div>
</div>


```js
const arrival_times = FileAttachment('./data/viarail.ca/times/train-times.parquet').parquet()
```

```js
Plot.plot({
	y: {
		percent: true
	},
	x: {
		transform: d => d / 60,
		label: "Minutes"
	},
	fy: {
		transform: d => d.toString()
	},
	marks: [
		Plot.rectY(
			[...arrival_times]
				.filter(d => d.stop_code === "TRTO"),
			Plot.binX({y: "proportion-facet", domain: [-14400, 14400]}, {x: "difference_s", fy: "arrival_year"})
		)
	]
})
```
