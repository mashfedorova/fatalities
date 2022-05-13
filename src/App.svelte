<script>
  import { onMount } from "svelte"
  import { json, csv, timeParse } from "d3"
  import ForceAverages from "../components/ForceAverages.svelte"
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

  let stories = [
    "Worker fatally struck by tree debris, in Minnesota",
    "Worker died in fall from telecommunications tower, in Illinois",
    "Worker fatally struck by trailer backing up, in Texas",
    "Worker fatally injured in fall from ladder, in Tennessee",
    "Worker died in fall from roof, in Washington",
    "Worker fatally crushed while servicing elevator, in Florida",
    "Worker died when tree fell on truck, California",
    "Worker electrocuted after contacting transformer, in California",
    "Worker died after being struck by vehicle at construction site, in Texas",
    "Worker electrocuted after contacting power lines, in Texas",
    "Worker died in vehicle collision, in Iowa",
    "Worker died after being struck by bulldozer., in  Hawaii",
  ]
</script>

<svelte:window on:resize={resize} />
<div class="charts">
  <h1>Work related fatalities</h1>
  <p class="intro">
    Between 2017 and 2021, at least 8,000 people were killed in work-related
    incidents in the United States. This means that every day, approximately
    four people are fatally injured at their workplaces.
  </p>
  <p class="intro">
    While some industries are inherently more dangerous than others, a
    work-related fatality can occur in any setting and in a number of ways.
  </p>
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
  <p class="intro">
    In a typical year, around 1,300 people are fatally injured at U.S.
    workplaces. However, due to the COVID-19 pandemic, there were 2,600
    work-related fatalities in 2020, nearly twice as many as during other years.
  </p>
  <h3 class="force-title">
    The average number of work-related fatalities each month of every year
    between 2017 and 2021
  </h3>
  <div class="circles" bind:this={width}>
    <svg width={widthChart} height="500">
      {#if width < 500}
        <g transform="translate(-40,-350)">
          <ForceAverages {circles} width={widthChart} />
        </g>
        <g transform="translate(78,-50)">
          <text y={height - 120} x="55">105 fatal injuries</text>
          <text y={height - 100} x="38">happened on average </text>
          <text y={height - 80} x="58">each month of</text>
          <text y={height - 60} x="35">2017, 2018, and 2019</text>
        </g>
        <g transform="translate(130,-350)">
          <ForceAverages circles={circlesCovid} width={widthChart} />
        </g>
        <g transform="translate(-59,-60)">
          <text y={height - 120} x="390">216 fatal injuries</text>
          <text y={height - 100} x="369">happened on average </text>
          <text y={height - 80} x="375">each month of 2020</text>
          <text y={height - 60} x="360">largerly due to COVID-19</text>
        </g>
      {/if}
      {#if width >= 500}
        <g transform="translate(-100,-360)">
          <ForceAverages {circles} width={widthChart} />
        </g>
        <g transform="translate(0,-70)">
          <text y={height - 120} x="75">105 fatal injuries</text>
          <text y={height - 100} x="55">happened on average</text>
          <text y={height - 80} x="79">each month of</text>
          <text y={height - 60} x="57">2017, 2018, and 2019</text>
        </g>
        <g transform="translate(200,-360)">
          <ForceAverages circles={circlesCovid} width={widthChart} />
        </g>
        <g transform="translate(0,-70)">
          <text y={height - 120} x="400">216 fatal injuries</text>
          <text y={height - 100} x="385">happened on average </text>
          <text y={height - 80} x="389">each month of 2020</text>
          <text y={height - 60} x="375">largerly due to COVID-19</text>
        </g>
      {/if}
    </svg>
  </div>

  <p class="para">
    The highest number of work-related fatalities was recorded in April 2020,
    when the COVID-19 pandemic hit the United States. The number of
    COVID-related fatalities remained relatively high in 2020, with
    approximately 100 people dying from COVID-19 at their workplaces, amounting
    to more than half of all work-related fatalities in 2020.
  </p>
  <p class="para para-last">
    The visualization below allows exploring the absolute number of work-related
    fatalities by state, month, and year, as well as filtering by COVID-19 and
    non COVID-19 related fatalities. While looking at the number of fatalities,
    it is important to keep in mind that the absolute numbers reflect the
    population of the States. While having experienced the highest number of
    work-related fatalities, California and Taxes are also the two most populous
    states in the U.S.
  </p>
  <h3>
    Number of work-related fatalities by state and month between 2017 and 2017
  </h3>
  <div class="monthly2" bind:this={width}>
    <StatesCanvas {states} {statesSorted} widthChart={width} />
  </div>
  <p class="intro" />
  <p class="intro last">
    The visualizations above provide an overview of the work-related fatalities
    that occurred under Federal OSHA and State Plan jurisdiction for cases that
    have been closed or citations issued on or after January 1, 2017. The full
    dataset is available on the United States Department of Labor <a
      href="https://www.osha.gov/fatalities"
      target="_blank">website</a
    >.These numbers reflect only those that have been reported to the DOL, so
    the real totals are likely much higher.
  </p>
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
    margin-top: 40px;
    box-shadow: rgba(117, 117, 117, 0.15) 2.5px 2.5px 3.2px;
  }

  @media only screen and (max-width: 550px) {
    .carousel {
      width: 380px;
    }
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

  .monthly2 {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  a {
    color: #84a8cf;
  }

  a:hover {
    color: #597491;
  }

  h1,
  h3 {
    color: #99c0e9;
    margin-bottom: 40px;
  }

  .intro,
  .para {
    width: 70%;
    margin: 0 auto;
    margin-top: 10px;
  }

  .force-title {
    margin-top: 50px;
  }

  .para-last {
    margin-bottom: 50px;
  }

  .last {
    margin-bottom: 50px;
    margin-top: 50px;
  }
</style>
