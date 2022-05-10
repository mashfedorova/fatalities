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

  import { LayerCake, Svg, WebGL, Html } from "layercake"

  import { Canvas } from "svelte-canvas"
  import { Layer } from "svelte-canvas"
  import Point from "./Point.svelte"
  import QuadTree from "./QuadTree.svelte"

  import { fade } from "svelte/transition"

  export let states
  export let statesSorted

  let hoverItem = null
  // export let blue

  let width = 1000
  let height = 1200
  let varFiltering = "fatalities"

  let margin = {
    left: 50,
    top: 20,
    right: 20,
    bottom: 50,
  }

  // $: console.log(hoverItem)

  $: xScale = scaleTime()
    .domain([new Date("2017-01-01 00:00:00"), new Date("2021-12-01 00:00:00")])
    .range([0, width])

  $: tickStates = map(statesSorted, (d) => d.state)

  $: yScale = scaleBand()
    .domain(tickStates)
    .range([0, height + 23])
  $: yScaleTooltip = scaleBand().domain(tickStates).range([height, 0])

  $: xKey = "myX"
  $: yKey = "myY"

  $: scaleCircle = scaleSqrt()
    .domain(extent(states, (d) => d.fatalities))
    .range([3, 25])

  $: ticksDates = xScale.ticks(5)

  // $: console.log(states)

  // $: console.log(yScale("CA")) //53.57

  let hoveredItem = null
  //tooltips
  let hoveredPoint = null

  // $: console.log(hoveredItem)

  // $: quadtreeFun = quadtree()
  //   .x((d) => xScale(d.date))
  //   .y((d) => yScale(d.state))
  //   .addAll(states)

  $: statesCalc = states.map((d) => {
    // const { x = 0, y = 0 } = d
    return {
      ...d,
      myX: xScale(d.date),
      myY: yScaleTooltip(d.state),
      r: scaleCircle(d[varFiltering]),
      yVar: yScale(d.state),
      // xTransformed: x + d.myX,
      // yTransformed: y + d.myY,
    }
  })

  // $: console.log(statesCalc)

  $: quadtreeFun = quadtree()
    .x((d) => xScale(d.date))
    .y((d) => yScale(d.state))
    .addAll(states)

  function handleMousemove(e) {
    // const { offsetX: x, offsetY: y } = e
    // const closest = minBy(
    //   states.map((d) => {
    //     return {
    //       ...d,
    //       diff: Math.hypot(x - d.myX, y - d.myY),
    //     }
    //   }),
    //   "diff"
    // )
    // if (closest && closest.diff < closest.radius * 1.5) {
    //   if (hoverItem && hoverItem.id === closest.id) return
    //   hoverItem = null
    //   hoverItem = { ...closest }
    //   return
    // }
    // hoverItem = null
    // console.log("item", e)

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
    if (distance < 50) {
      hoveredPoint = closestPoint
    } else {
      hoveredPoint = null
    }
    console.log("point", hoveredPoint)
  }

  // $: console.log(statesCalc)
</script>

<div class="buttons-container">
  <button on:click={() => (varFiltering = "covidFatalities")}>covid</button>
  <button on:click={() => (varFiltering = "fatalitiesNonCovid")}
    >non-covid</button
  >
  <button on:click={() => (varFiltering = "fatalities")}>all</button>
</div>
<div class="chart-container">
  <LayerCake x={xKey} y={yKey} data={statesCalc}>
    <Canvas {width} {height} bind:hoveredPoint on:mousemove={handleMousemove}>
      {#each states as state}
        <Point
          x={xScale(state.date)}
          y={yScale(state.state)}
          r={scaleCircle(state[varFiltering])}
          fill={varFiltering === "covidFatalities" && !state.covidFatalities
            ? "white"
            : hoveredPoint === state
            ? "rgb(84, 114, 145)"
            : "rgb(153, 192, 233)"}
          stroke={hoveredPoint === state ? "black" : "white"}
          {hoveredPoint}
        />
      {/each}
    </Canvas>
    {#if hoveredPoint}
      <Tooltip x={xScale(hoveredPoint.date)} y={yScale(hoveredPoint.state)}>
        <strong>
          {hoveredPoint.date}
        </strong>
        <div>
          {hoveredPoint.state}
        </div>
        <div>
          {hoveredPoint[varFiltering]}
        </div>
      </Tooltip>
    {/if}
  </LayerCake>
</div>

<style>
  /* svg {
    display: block;
  }
  circle {
    transition: r 0.5s ease-out, opacity 0.5s;
  } */
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
