<script>
  import { onMount } from "svelte"
  import { json, csv, timeParse } from "d3"
  import ForceAverages from "../components/ForceAverages.svelte"
  import Monthly from "../components/Monthly.svelte"
  import States from "../components/States.svelte"
  import StatesCanvas from "../components/StatesCanvas.svelte"
  import Card from "../components/Card.svelte"
  import Carousel from "svelte-carousel"

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

    const statesData = await csv("data/summary2.csv", (d) => {
      return {
        ...d,
        year: +d.year,
        fatalities: +d.fatalities,
        covidFatalities: +d.covidFatalities,
        fatalitiesNonCovid: +d.fatalitiesNonCovid,
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

  let stories = [
    "Worker died in fall from roof",
    "Worker fatally struck by pickup truck",
    "Worker fatally injured while training horse",
    "Worker fatally crushed when mower rolled over.",
  ]
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
  <!-- <div class="monthly">
    <Monthly {monthly} {blue} />
  </div> -->
  <div class="carousel-container">
    <div class="carousel">
      <Carousel let:showPrevPage let:showNextPage>
        <div
          slot="prev"
          on:click={showPrevPage}
          class="custom-arrow custom-arrow-prev"
        >
          <i class="arrow arrow-left" />
        </div>
        {#each stories as story}
          <Card {story} />
        {/each}
        <div
          slot="next"
          on:click={showNextPage}
          class="custom-arrow custom-arrow-next"
        >
          <i class="arrow arrow-right" />
        </div>
      </Carousel>
    </div>
  </div>
  <div class="monthly">
    <States {states} {statesSorted} {blue} />
  </div>
  <div class="monthly2">
    <StatesCanvas {states} {statesSorted} {blue} />
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

  .carousel-container {
    display: flex;
    justify-content: center;
  }

  .carousel {
    width: 500px;
    border: 1px solid rgba(88, 88, 88, 0.81);
    padding: 20px 0;
    margin-bottom: 50px;
    /* box-shadow: 20px 20px rgba(0, 0, 0, 0.15); */
    box-shadow: rgba(117, 117, 117, 0.15) 2.5px 2.5px 3.2px;
    /* box-shadow: rgba(0, 0, 0, 0.15) 2.6px 2.6px 3.5px; */
  }

  .custom-arrow {
    margin: auto 10px;
    width: 25px;
    height: 25px;
    border-radius: 50%;
    background-color: white;
    border: 1px solid grey;
    transition: opacity 100ms ease;
    cursor: pointer;
    box-shadow: rgba(121, 121, 121, 0.15) 1px 1px 1px;
  }

  .arrow {
    border: solid rgba(153, 153, 153, 0.858);
    border-width: 0 2px 2px 0;
    display: inline-block;
    padding: 2px;
  }

  .arrow-left {
    transform: rotate(135deg);
    -webkit-transform: rotate(135deg);
  }

  .arrow-right {
    transform: rotate(-45deg);
    -webkit-transform: rotate(-45deg);
  }
  .monthly,
  .monthly2 {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
</style>
