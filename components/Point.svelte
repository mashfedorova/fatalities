<script>
  import { Layer } from "svelte-canvas"
  import { tweened } from "svelte/motion"
  import { cubicOut } from "svelte/easing"
  import { interpolateLab } from "d3-interpolate"
  // export let fill = "rgb(188, 74, 60)"
  export let r = 5
  export let x
  export let y
  export let hoverItem
  export let fill

  $: console.log("hover", hoverItem)

  const _r = tweened(r, { duration: 600, easing: cubicOut })
  $: _r.set(r)

  const _color = tweened(fill, {
    duration: 100,
    interpolate: interpolateLab,
  })
  $: _color.set(fill)

  $: render = ({ context }) => {
    context.beginPath()
    context.arc(x, y, $_r, 0, 2 * Math.PI)
    context.fillStyle = $_color
    context.fill()
    context.stroke()
    context.strokeStyle = "#e0e1e2"
  }
</script>

<Layer {render} />

<style>
</style>
