<script>
  import { getContext } from "svelte"
  import { quadtree } from "d3"
  // export let data, xGet, yGet, width, height

  const { data, xGet, yGet, width, height } = getContext("LayerCake")

  $: console.log("data", data)

  let visible = false
  let found = {}
  let e = {}

  export let x = "x"
  export let y = "y"

  export let searchRadius = undefined

  export let dataset = undefined

  $: xGetter = $xGet
  $: yGetter = $yGet

  function findItem(evt) {
    e = evt

    const xLayerKey = `layer${x.toUpperCase()}`
    const yLayerKey = `layer${y.toUpperCase()}`

    found = finder.find(evt[xLayerKey], evt[yLayerKey], searchRadius) || {}
    visible = Object.keys(found).length > 0
    // console.log(found)
  }

  $: finder = quadtree()
    .extent([
      [-1, -1],
      [$width + 1, $height + 1],
    ])
    .x(xGetter)
    .y(yGetter)
    .addAll(dataset || $data)
</script>

<div
  class="bg"
  on:mousemove={findItem}
  on:mouseout={() => (visible = false)}
  on:blur={() => (visible = false)}
/>
<slot x={xGetter(found) || 0} y={yGetter(found) || 0} {found} {visible} {e} />

<style>
  .bg {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }
</style>
