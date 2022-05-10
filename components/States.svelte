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

  import { fade } from "svelte/transition"
  import Tooltip from "./Tooltip.svelte"

  export let states
  export let statesSorted
  // export let blue

  let width = 1000
  let height = 1500
  let varFiltering = "fatalities"

  let margin = {
    left: 50,
    top: 20,
    right: 20,
    bottom: 50,
  }

  $: xScale = scaleTime()
    .domain([new Date("2017-01-01 00:00:00"), new Date("2021-12-01 00:00:00")])
    .range([margin.left, width - margin.right])

  $: tickStates = map(statesSorted, (d) => d.state)

  $: yScale = scaleBand()
    .domain(tickStates)
    .range([margin.top, height - margin.bottom])

  $: scaleCircle = scaleSqrt()
    .domain(extent(states, (d) => d.fatalities))
    .range([3, 25])

  $: ticksDates = xScale.ticks(5)

  // $: console.log(states)

  //tooltips
  let hoveredPoint = null

  $: quadtreeFun = quadtree()
    .x((d) => xScale(d.date))
    .y((d) => yScale(d.state))
    .addAll(states)
</script>

<div class="buttons-container">
  <button on:click={() => (varFiltering = "covidFatalities")}>covid</button>
  <button on:click={() => (varFiltering = "fatalitiesNonCovid")}
    >non-covid</button
  >
  <button on:click={() => (varFiltering = "fatalities")}>all</button>
</div>
<svg {width} {height}>
  {#each states as state}
    <circle
      out:fade
      cx={xScale(state.date)}
      cy={yScale(state.state)}
      r={scaleCircle(state[varFiltering])}
      fill={hoveredPoint === state
        ? "rgb(224, 212, 148)"
        : "rgb(153, 192, 233)"}
      stroke={hoveredPoint === state ? "black" : "none"}
      opacity={varFiltering === "covidFatalities" && !state.covidFatalities
        ? 0
        : 0.6}
    />
  {/each}
  {#each ticksDates as tick}
    <text x={xScale(tick)} y={height}>{timeFormat("%b %y")(tick)}</text>
  {/each}
  {#each tickStates as tick}
    <text x="0" y={yScale(tick)}>{tick}</text>
  {/each}

  <rect
    x={margin.right}
    width={width - margin.top}
    height={height - margin.bottom}
    fill="transparent"
    on:mousemove={(e) => {
      const pos = pointer(e)
      const x = pos[0]
      const y = pos[1]
      const closestPoint = quadtreeFun.find(x, y)
      if (!closestPoint) return

      // don't highlight if too far away
      const hoveredPointPosition = [
        xScale(closestPoint.date),
        yScale(closestPoint.state),
      ]
      // a^2 + b^2 = c^2
      const distance = Math.sqrt(
        (x - hoveredPointPosition[0]) ** 2 + (y - hoveredPointPosition[1]) ** 2
      )
      if (distance < 15) {
        hoveredPoint = closestPoint
      } else {
        hoveredPoint = null
      }
    }}
  />
</svg>

{#if hoveredPoint}
  <Tooltip
    x={margin.left + 70 + xScale(hoveredPoint.date)}
    y={yScale(hoveredPoint.state) + 785}
  >
    <strong>
      {hoveredPoint.date}
    </strong>
    <div>
      {hoveredPoint.state}
    </div>
  </Tooltip>
{/if}

<style>
  svg {
    display: block;
  }
  circle {
    transition: r 0.5s ease-out, opacity 0.5s;
  }
  .buttons-container {
    display: flex;
    gap: 5px;
  }

  button {
    border: 1px solid rgba(88, 88, 88, 0.81);
    box-shadow: rgba(117, 117, 117, 0.15) 2.5px 2.5px 3.2px;
    background-color: #ffffff;
    cursor: pointer;
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
</style>
