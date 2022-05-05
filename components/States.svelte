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
  } from "d3"

  export let states
  export let statesSorted
  let width = 1000
  let height = 1500
  export let blue

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
    .range([3, 30])

  $: ticksDates = xScale.ticks(5)

  $: console.log(statesSorted)
</script>

<svg width="1000" {height}>
  {#each states as state}
    <circle
      cx={xScale(state.date)}
      cy={yScale(state.state)}
      r={scaleCircle(state.fatalities)}
      fill={blue}
      opacity="0.5"
    />
  {/each}
  {#each ticksDates as tick}
    <text x={xScale(tick)} y={height}>{timeFormat("%b %y")(tick)}</text>
  {/each}
  {#each tickStates as tick}
    <text x="0" y={yScale(tick)}>{tick}</text>
  {/each}
</svg>

<style>
</style>
