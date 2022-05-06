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
  export let blue

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

  $: console.log(states)
</script>

<div class="buttons-container">
  <button on:click={() => (varFiltering = "covidFatalities")}>covid</button>
  <button on:click={() => (varFiltering = "fatalitiesNonCovid")}
    >non-covid</button
  >
  <button on:click={() => (varFiltering = "fatalities")}>all</button>
</div>
<svg width="1000" {height}>
  {#each states as state}
    <circle
      cx={xScale(state.date)}
      cy={yScale(state.state)}
      r={scaleCircle(state[varFiltering])}
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
  svg {
    display: block;
  }
  circle {
    transition: r 0.5s ease-out;
  }
  .buttons-container {
    display: flex;
    gap: 5px;
  }

  button {
    /* cursor: pointer;
    align-items: center;
    background-color: #ffffff;
    border: 1px solid rgba(0, 0, 0, 0.1);
    border-radius: 0.25rem;
    box-shadow: rgba(0, 0, 0, 0.02) 0 1px 3px 0;
    box-sizing: border-box;
    color: rgba(0, 0, 0, 0.85);
    cursor: pointer;
    display: inline-flex;
    font-family: system-ui, -apple-system, system-ui, "Helvetica Neue",
      Helvetica, Arial, sans-serif;
    font-size: 16px;
    font-weight: 600;
    justify-content: center;
    line-height: 1.25;
    margin: 0;
    min-height: 3rem;
    padding: calc(0.875rem - 1px) calc(1.5rem - 1px);
    position: relative;
    text-decoration: none;
    transition: all 250ms;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
    vertical-align: baseline;
    width: auto; */
    border: 1px solid rgba(88, 88, 88, 0.81);
    box-shadow: rgba(117, 117, 117, 0.15) 2.5px 2.5px 3.2px;
    background-color: #ffffff;
    cursor: pointer;
  }

  button:hover,
  button:focus {
    box-shadow: rgba(64, 64, 64, 0.15) 3px 3px 3.25x;
  }

  button:hover {
    transform: translateY(-2px);
  }

  button:active {
    box-shadow: rgba(75, 75, 75, 0.15) 3px 3px 3.25x;
    transform: translateY(0);
  }
</style>
