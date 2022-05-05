<script>
  import { scaleSqrt, scaleTime, extent, timeFormat } from "d3"
  import { onMount, afterUpdate } from "svelte"
  export let monthly

  // $: console.log(monthly)

  let lineWidth = 700
  let r = 5

  // $: scale = scaleTime()
  //   .domain([new Date("2017-01-01 00:00:00"), new Date("2021-12-01 00:00:00")])
  //   .range([0, pathLength])

  let path = `m110 80 ${lineWidth},0 0,80 -${lineWidth},0 0,80 ${lineWidth}, 0 0,80 -${lineWidth},0 0, 80 ${lineWidth},0`

  // $: pathLength = path.getTotalLength()
  // $: pathNode = path.node()

  let pathLength = ""
  let myPath = ""

  let rAccessor = (d) => d.fatalities

  let xScale = function () {}

  $: extentFatalities = extent(monthly, rAccessor)

  $: scaleRad = scaleSqrt().domain(extentFatalities).range([0, 30])

  afterUpdate(() => {
    myPath = document.getElementById("path")
    // if (myPath) {
    pathLength = myPath.getTotalLength()
    // }

    // console.log(pathLength)

    xScale = scaleTime()
      .domain([
        new Date("2017-01-01 00:00:00"),
        new Date("2021-12-01 00:00:00"),
      ])
      .range([0, pathLength])

    // console.log(scale(monthly[0]))
    // console.log(scale(monthly[1]?.date || {}))
  })

  // $: console.log("path", myPath)
  // $: console.log("path2", pathLength)

  $: dataCalc = monthly.map((d) => {
    return {
      ...d,
      xPos: myPath.getPointAtLength(xScale(d.date)).x,
      yPos: myPath.getPointAtLength(xScale(d.date)).y,
      r: scaleRad(d.fatalities),
    }
  })

  // $: console.log(dataCalc)
</script>

<svg width="1000" height="700">
  <path id="path" d={path} stroke="black" fill="none" />
  {#each dataCalc as d}
    <circle cx={d.xPos} cy={d.yPos} r={d.r} fill="plum" opacity="0.5" />
    <text
      fill="grey"
      x={d.xPos}
      y={d.yPos + 20}
      text-anchor="middle"
      dominant-baseline="hanging"
    >
      {timeFormat("%b %y")(d.date)}
      <!-- {d.month.includes("Jan")
        ? timeFormat("%b %Y")(d.date)
        : timeFormat("%b")(d.date)} -->
    </text>
  {/each}
</svg>
