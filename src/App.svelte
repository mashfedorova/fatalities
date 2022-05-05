<script>
  import { onMount } from "svelte"
  import { json, csv, timeParse } from "d3"
  import ForceAverages from "../components/ForceAverages.svelte"
  import Monthly from "../components/Monthly.svelte"
  import States from "../components/States.svelte"

  let width = document.body.clientWidth
  let circles = []
  let monthly = []
  let circlesCovid = []
  let states = []
  let statesUniq = []
  let statesSorted = []
  let offset
  let height = 500
  let blue = "#99C0E9"

  function resize() {
    width = document.body.clientWidth
  }

  $: widthChart = 600

  onMount(resize)

  onMount(async () => {
    // const dataCircles = await csv(
    //   "data/forceCirclesFatalitiesAverages.csv",
    //   (d) => {
    //     return {
    //       ...d,
    //     }
    //   }
    // )
    // circles = dataCircles

    const dataCirclesCovid = await json("data/fatCovid.json", (d) => {
      return {
        ...d,
      }
    })
    circlesCovid = dataCirclesCovid

    const dataCircles = await json("data/fatAv.json", (d) => {
      return {
        ...d,
      }
    })
    circles = dataCircles

    const monthlyData = await csv("data/montlyFatalities.csv", (d) => {
      return {
        ...d,
        year: +d.year,
        fatalities: +d.fatalities,
        date: timeParse("%b.%y")(d.month),
      }
    })
    monthly = monthlyData

    const statesData = await csv("data/montlyFatalitiesState.csv", (d) => {
      return {
        ...d,
        year: +d.year,
        fatalities: +d.fatalities,
        monthNum: +d.monthNum,
        date: timeParse("%b.%y")(d.month),
      }
    })
    states = statesData

    const sortedStatesData = await csv("data/statesSorted.csv", (d) => {
      return {
        ...d,
      }
    })
    statesSorted = sortedStatesData
  })

  // $: console.log(circles)
</script>

<svelte:window on:resize={resize} />

<div class="charts">
  <div class="circles" bind:this={width}>
    <svg width={widthChart} height="500">
      <g transform="translate(-100,-255)">
        <ForceAverages {circles} offset={0} width={widthChart} {blue} />
      </g>
      <text y={height - 120} x="55">105 fatal injuries</text>
      <text y={height - 100} x="38">happened on average </text>
      <text y={height - 80} x="58">each month of</text>
      <text y={height - 60} x="35">2017, 2018, and 2019</text>
      <g transform="translate(200,-255)">
        <ForceAverages
          circles={circlesCovid}
          offset={0}
          width={widthChart}
          {blue}
        />
      </g>
      <text y={height - 120} x="390">216 fatal injuries</text>
      <text y={height - 100} x="369">happened on average </text>
      <text y={height - 80} x="375">each month of 2020</text>
      <text y={height - 60} x="360">largerly due to COVID-19</text>
    </svg>
  </div>
  <div class="monthly">
    <Monthly {monthly} {blue} />
  </div>
  <div class="monthly">
    <States {states} {statesSorted} {blue} />
  </div>
</div>

<style>
  .circles {
    display: flex;
    justify-content: center;
  }
  svg {
    display: block;
    overflow: visible;
  }
  .charts {
    overflow-x: hidden;
    text-align: center;
    padding: 1em;
    max-width: 1200px;
    margin: 0 auto;
  }
</style>
