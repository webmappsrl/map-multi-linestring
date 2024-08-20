<template>
  <PanelItem :index="index" :field="field">
    <template #value>
       <p v-if="!field.geojson">Geometria non disponibile</p>
      <wm-map-multi-linestring :field="field" :edit=false v-else></wm-map-multi-linestring>
    </template>
  </PanelItem>
</template>

<script>
export default {
  props: ['index', 'field'],
}
</script>
