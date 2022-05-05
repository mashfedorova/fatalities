<script>
  import {
    forceSimulation,
    forceCollide,
    forceX,
    forceY,
    scalePoint,
    forceManyBody,
    forceCenter,
    scaleOrdinal,
  } from "d3"
  import { afterUpdate } from "svelte"

  export let circles

  // $: console.log(circles)
  // export let offset
  export let width = 200

  $: width < 500 ? 500 : width

  let height
  let r = 5

  // $: xScale = scalePoint()
  //   .domain(["average", "covid"])
  //   .range([300, width - 200])

  // afterUpdate(() => {
  //   circles = circles
  // })

  // $: simulation = forceSimulation(circles)
  //   .force("center", forceCenter(width / 2, height / 2))
  //   .force("charge", forceManyBody().strength(0.4))
  //   .force("collide", forceCollide().radius(6))
  //   .alpha(1)
  //   .restart()

  //   $: simulation = forceSimulation(circles)
  // .force(
  //   "x",
  //   forceX()
  //     .x((d) => xScale(d.fatality))
  //     .strength(0.5)
  // )
  // .force("y", forceY(500).strength(0.5))
  // .force("collide", forceCollide(5).strength(0.3).iterations(1))
  // .alphaDecay(0.1)
  // .alpha(1)

  // $: console.log(circles)

  // $: for (let i = 0; i < circles.length; ++i) simulation.tick()
</script>

{#each circles as circle}
  <circle cx={circle.x} cy={circle.y} fill="plum" {r} />
{/each}

<style>
</style>
