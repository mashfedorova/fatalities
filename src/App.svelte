<script>
  import { onMount } from "svelte"
  import { json, csv } from "d3"
  import ForceAverages from "../components/ForceAverages.svelte"

  let circles = []

  onMount(async () => {
    const dataCircles = await csv(
      "data/forceCirclesFatalitiesAverages.csv",
      (d) => {
        return {
          ...d,
        }
      }
    )
    circles = dataCircles
  })
</script>

<div class="charts">
  <h1>Hello!</h1>
  <ForceAverages {circles} />
</div>

<style>
  /* .charts {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  } */
</style>
