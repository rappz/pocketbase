<script>
  import Icon from "@iconify/svelte";
  export let width;
  export let height;
  export let icon;
  export let tooltip;
  export let clazz;
</script>

<div class="{clazz} group">
  <Icon {icon} {width} {height} />
  <span class="nav-tooltip group-hover:scale-100">{tooltip}</span>
</div>
