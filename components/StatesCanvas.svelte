<script>
  import {
    scaleLinear,
    scaleOrdinal,
    scalePoint,
    scaleBand,
    scaleTime,
    scaleSqrt,
    map,
    extent,
    timeFormat,
    pointer,
    quadtree,
  } from "d3"

  import Tooltip from "./Tooltip.svelte"

  import { LayerCake } from "layercake"

  import { Canvas } from "svelte-canvas"
  import Point from "./Point.svelte"

  export let states
  export let statesSorted

  export let widthChart
  $: width =
    widthChart >= 1000
      ? 1000
      : widthChart < 1000 && widthChart > 600
      ? 600
      : 300
  let height = 1200
  let varFiltering = "fatalities"

  let margin

  $: width >= 1000
    ? (margin = {
        left: 120,
        top: 60,
        right: 30,
        bottom: 30,
      })
    : (margin = {
        left: 50,
        top: 40,
        right: 30,
        bottom: 20,
      })

  $: xScale = scaleTime()
    .domain([new Date("2017-01-01 00:00:00"), new Date("2021-12-01 00:00:00")])
    .range([margin.left, width - margin.right])

  $: tickStates = map(statesSorted, (d) => d.state)

  $: yScale = scaleBand()
    .domain(tickStates)
    .range([margin.top, height - margin.bottom])
  $: yScaleTooltip = scaleBand().domain(tickStates).range([height, 0])

  $: xKey = "myX"
  $: yKey = "myY"

  $: scaleCircle = scaleSqrt()
    .domain(extent(states, (d) => d.fatalities))
    .range([3, 25])

  $: scaleCircleSmall = scaleSqrt()
    .domain(extent(states, (d) => d.fatalities))
    .range([2, 15])

  $: ticksDates = xScale.ticks(5)

  //tooltips
  let hoveredPoint = null

  $: quadtreeFun = quadtree()
    .x((d) => xScale(d.date))
    .y((d) => yScale(d.state))
    .addAll(states)

  function handleMousemove(e) {
    const pos = pointer(e)
    const x = pos[0]
    const y = pos[1]
    const closestPoint = quadtreeFun.find(x, y)
    if (!closestPoint) return

    const hoveredPointPosition = [
      xScale(closestPoint.date),
      yScale(closestPoint.state),
    ]

    const distance = Math.sqrt(
      (x - hoveredPointPosition[0]) ** 2 + (y - hoveredPointPosition[1]) ** 2
    )
    if (distance < 10) {
      hoveredPoint = closestPoint
    } else {
      hoveredPoint = null
    }
  }
</script>

<div class="buttons-container">
  <p class="filter-text">Filter by fatalities:</p>
  <button on:click={() => (varFiltering = "covidFatalities")}>covid</button>
  <button on:click={() => (varFiltering = "fatalitiesNonCovid")}
    >non-covid</button
  >
  <button on:click={() => (varFiltering = "fatalities")}>all</button>
</div>
<div class="chart-container">
  <svg {width} {height}>
    {#each ticksDates as tick}
      <text x={xScale(tick)} y={height - 15} text-anchor="middle"
        >{timeFormat("%b %y")(tick)}</text
      >
    {/each}
    {#each ticksDates as tick}
      <text x={xScale(tick)} y={margin.top / 2} text-anchor="middle"
        >{timeFormat("%b %y")(tick)}</text
      >
    {/each}
    {#each statesSorted as tick}
      <text x="0" y={yScale(tick.state) + 1}
        >{width >= 1000 ? tick.stateName : tick.state}</text
      >
    {/each}
  </svg>
  <LayerCake x={xKey} y={yKey} data={states}>
    <Canvas {width} {height} on:mousemove={handleMousemove}>
      {#each states as state}
        <Point
          x={xScale(state.date)}
          y={yScale(state.state)}
          r={width >= 600
            ? scaleCircle(state[varFiltering])
            : scaleCircleSmall(state[varFiltering])}
          fill={varFiltering === "covidFatalities" && !state.covidFatalities
            ? "white"
            : varFiltering === "non-covid" && !state.fatalitiesNonCovid
            ? "white"
            : hoveredPoint === state
            ? "rgb(84, 114, 145)"
            : "rgb(153, 192, 233)"}
        />
      {/each}
    </Canvas>
    {#if hoveredPoint}
      <Tooltip x={xScale(hoveredPoint.date)} y={yScale(hoveredPoint.state)}>
        <strong>
          {timeFormat("%b %Y")(hoveredPoint.date)}
        </strong>
        <div>
          {hoveredPoint[varFiltering]}
          {hoveredPoint[varFiltering] > 1 ? "fatalities" : "fatality"}
        </div>
      </Tooltip>
    {/if}
  </LayerCake>
</div>

<style>
  svg {
    display: block;
    position: absolute;
  }

  /* circle {
    transition: r 0.5s ease-out, opacity 0.5s;
  }  */
  .buttons-container {
    display: flex;
    gap: 5px;
    margin-bottom: 15px;
  }

  button {
    border: 1px solid rgba(88, 88, 88, 0.81);
    box-shadow: rgba(117, 117, 117, 0.15) 2.5px 2.5px 3.2px;
    background-color: #ffffff;
    cursor: pointer;
    padding: 0.1em 0.6em;
  }

  .filter-text {
    text-align: center;
    margin-right: 10px;
  }

  /* button:hover,
  button:focus {
    box-shadow: rgba(64, 64, 64, 0.15) 3px 3px 3.25x;
  }

  button:hover {
    transform: translateY(-2px);
  }

  button:active {
    box-shadow: rgba(75, 75, 75, 0.15) 3px 3px 3.25x;
    transform: translateY(0);
  } */
  /*
  .chart-container {
    width: 1000px;
    height: 1200px;
  } */

  /* .circle {
    position: absolute;
    border-radius: 50%;
    background-color: rgba(171, 0, 214);
    transform: translate(-50%, -50%);
    pointer-events: none;
     width: 15px;
    height: 15px;
  } */
</style>
