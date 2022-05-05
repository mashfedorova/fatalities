<script>
  import { onMount } from "svelte"
  import { json, csv } from "d3"
  import ForceAverages from "../components/ForceAverages.svelte"

  let width = document.body.clientWidth
  let circles = []
  let circlesCovid = []
  let offset
  let height = 500

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
  })

  $: console.log(circles)
</script>

<svelte:window on:resize={resize} />

<div class="charts">
  <div class="circles" bind:this={width}>
    <svg width={widthChart} height="500">
      <g transform="translate(-100,-255)">
        <ForceAverages {circles} offset={0} width={widthChart} />
      </g>
      <text y={height - 120} x="55">105 fatal injuries</text>
      <text y={height - 100} x="38">happened on average </text>
      <text y={height - 80} x="58">each month of</text>
      <text y={height - 60} x="35">2017, 2018, and 2019</text>
      <g transform="translate(200,-255)">
        <ForceAverages circles={circlesCovid} offset={0} width={widthChart} />
      </g>
      <text y={height - 120} x="390">216 fatal injuries</text>
      <text y={height - 100} x="369">happened on average </text>
      <text y={height - 80} x="375">each month of 2020</text>
      <text y={height - 60} x="360">largerly due to COVID-19</text>
    </svg>
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
