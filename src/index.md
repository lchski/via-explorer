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
[...arrival_times]
```

```js
Plot.plot({
    height: 20000,
    y: {
        tickFormat: "s"
    },
    x: {
        transform: (d) => d / 60,
        label: "Minutes"
    },
    marks: [
        Plot.rectY(
            arrival_times,
            Plot.binX({y: "count", domain: [-14400, 14400]}, {x: "difference_s", fy: "stop_code"})
        )
    ]
})
```
