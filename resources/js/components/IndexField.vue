<template>
  <wm-map-multi-linestring :edit=false class="index-view" :field="field"></wm-map-multi-linestring>
</template>
<script>
export default {
  props: ['field'],
}
</script>
