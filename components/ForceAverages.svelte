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

  export let circles

  let width = 1000
  let height = 1000
  let r = 4.5

  $: xScale = scalePoint()
    .domain(["average", "covid"])
    .range([300, width - 200])

  $: simulation1 = forceSimulation(
    circles.filter((d) => d.fatality === "average")
  )
    .force("center", forceCenter(width - 600, height / 2))
    .force("charge", forceManyBody().strength(0.4))
    .force("collide", forceCollide().radius(6))
    .alpha(1)
    .restart()

  $: simulation2 = forceSimulation(
    circles.filter((d) => d.fatality === "covid")
  )
    .force("center", forceCenter(width - 300, height / 2))
    .force("charge", forceManyBody().strength(0.4))
    .force("collide", forceCollide().radius(6))
    .alpha(1)
    .restart()

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

  $: for (let i = 0; i < circles.length; ++i) simulation.tick()

  // $: console.log(circles)
</script>

<svg {width} {height}>
  {#each circles as circle}
    <circle cx={circle.x} cy={circle.y} fill="plum" {r} />
  {/each}
</svg>
