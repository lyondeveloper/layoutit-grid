<template>
  <span class="token selector">.<CssCodeAreaName :area="area" /></span><span> {</span>
  <template v-if="area.grid">
    <CssDecl property="display">grid</CssDecl>
    <CssDecl v-if="area.grid.autoFlow !== 'row'" property="grid-auto-flow">{{ area.grid.autoFlow }}</CssDecl>
    <CssDecl v-if="area.grid.col.auto !== 'initial'" property="grid-auto-columns">{{ area.grid.col.auto }}</CssDecl>
    <CssDecl v-if="area.grid.row.auto !== 'initial'" property="grid-auto-rows">{{ area.grid.row.auto }}</CssDecl>
    <CssDecl property="grid-template-columns">
      <CssCodeTemplateTracks :grid="area.grid" type="col" :repeat="options.repeat" />
    </CssDecl>
    <CssDecl property="grid-template-rows">
      <CssCodeTemplateTracks :grid="area.grid" type="row" :repeat="options.repeat" />
    </CssDecl>
    <CssDecl property="gap">
      <CssCodeGap :grid="area.grid" type="row" /> <CssCodeGap :grid="area.grid" type="col" />
    </CssDecl>
    <CssDecl v-if="includeTemplateAreas" property="grid-template-areas">
      {{ '\n    ' }}<CssCodeTemplateAreas :area="area" :options="options" />
    </CssDecl>
    <CssDecl v-if="area.grid.justifyContent !== 'initial'" property="justify-content">{{
      area.grid.justifyContent
    }}</CssDecl>
    <CssDecl v-if="area.grid.alignContent !== 'initial'" property="align-content">{{ area.grid.alignContent }}</CssDecl>
    <CssDecl v-if="area.grid.justifyItems !== 'initial'" property="justify-items">{{ area.grid.justifyItems }}</CssDecl>
    <CssDecl v-if="area.grid.alignItems !== 'initial'" property="align-items">{{ area.grid.alignItems }}</CssDecl>
  </template>
  <template v-if="area !== mainArea">
    <CssDecl v-if="area.justifySelf !== 'initial'" property="justify-self">{{ area.justifySelf }}</CssDecl>
    <CssDecl v-if="area.alignSelf !== 'initial'" property="align-self">{{ area.alignSelf }}</CssDecl>
    <template v-if="gridArea">
      <CssDecl property="grid-area" :ident="singleLine ? ' ' : '\n  '"
        ><CssCodeGridArea :area="area" :options="options"
      /></CssDecl>
    </template>
    <CssDecl v-if="area.width !== 'initial'" property="width">{{ area.width }}</CssDecl>
    <CssDecl v-if="area.height !== 'initial'" property="height">{{ area.height }}</CssDecl>
    <CssDecl v-if="area.margin !== '0'" property="margin">{{ area.margin }}</CssDecl>
    <CssDecl v-if="area.padding !== '0'" property="padding">{{ area.padding }}</CssDecl> </template
  >{{ (singleLine ? '' : '\n') + '}' }}
  <template v-for="(a, i) in areasToInclude" :key="`area-${area.name}-${i}`"
    >{{ '\n' }}<CssCodeArea :area="a" :options="options"
  /></template>
</template>

<script setup="props">
export { default as CssCodeTemplateTracks } from './CssCodeTemplateTracks.vue'
export { default as CssCodeAreaName } from './CssCodeAreaName.vue'
export { default as CssCodeGridArea } from './CssCodeGridArea.vue'
export { default as CssCodeTemplateAreas } from './CssCodeTemplateAreas.vue'
export { default as CssCodeGap } from './CssCodeGap.vue'
export { default as CssDecl } from './CssCodeDeclaration.vue'

import { computed } from 'vue'
export {
  namedTemplateColumns,
  namedTemplateRows,
  gridTemplateAreas,
  getGridAreaWithNamedLines,
  areaIsSingleLineInCSS,
} from '../../utils.js'
export { mainArea, getGridRegion } from '../../store.js'

export default {
  name: 'CssCodeArea',
  props: {
    area: { type: Object, required: true },
    options: { type: Object, required: true },
  },
}

export const cssAreaName = computed(() => props.area.name)

function getGridTemplateAreas(area) {
  return area.display === 'grid' ? gridTemplateAreas(area, '\n    ') : undefined
}

export const templateAreas = computed(() => getGridTemplateAreas(props.area))

export const includeTemplateAreas = computed(() => props.options.templateAreas && templateAreas.value !== undefined)

export const gridArea = computed(() => {
  const gridRegion = getGridRegion(props.area) // TODO: span
  if (!gridRegion) {
    return undefined
  }
  const { parent } = props.area
  if (parent) {
    return props.options.templateAreas && getGridTemplateAreas(parent)
      ? cssAreaName.value
      : getGridAreaWithNamedLines(props.area, parent.grid)
  } else {
    return getGridAreaWithNamedLines(props.area)
  }
})

export const singleLine = computed(() => areaIsSingleLineInCSS(props.area))

export const areasToInclude = computed(() =>
  props.area.children.filter((area) => !areaIsSingleLineInCSS(area) && area.gridArea === 'auto')
)
</script>

<style scoped lang="scss"></style>
