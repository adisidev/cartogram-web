<script setup lang="ts">
import { reactive } from 'vue'

import type CartMap from '../../common/lib/cartMap'
import type { MapHandlers } from '../../common/lib/interface'
import type { DataTable } from '../lib/interface'

import { useCartogramStore } from '../stores/cartogram'
const store = useCartogramStore()

const props = defineProps<{
  maps: MapHandlers
  map: CartMap
}>()

const state = reactive({
  data: {
    fields: [],
    items: {}
  } as DataTable
})

const emit = defineEmits<{
  (e: 'change', data: DataTable): void
}>()

function generateTable() {
  state.data.fields = []
  state.data.items = {}
  if (!props.map || !props.map.versions) return

  // Header
  state.data.fields = [
    { key: 'area', label: props.maps[store.currentMapName].region_identifier, editable: false },
    { key: 'color', label: 'Color', editable: true, type: 'color' }
  ]
  for (const [versionKey, version] of Object.entries(props.map.versions)) {
    if (versionKey === '0-base') continue
    state.data.fields.push({
      key: versionKey,
      label: version.name + ' (' + version.unit + ')',
      editable: true,
      type: 'number',
      headerEditable: true
    })
  }

  // Content
  for (const [regionKey, region] of Object.entries(props.map.regions)) {
    var data: any = [region.name, props.map.colors[regionKey]]
    for (const [versionKey, version] of Object.entries(region.versions)) {
      if (versionKey === '0-base') continue
      data.push(version.value)
    }
    state.data.items[regionKey] = data
  }
}

function updateCartogram() {
  emit('change', state.data)
}
</script>

<template>
  <!-- Button trigger modal -->
  <button
    v-bind:class="{ disabled: store.isLoading }"
    class="btn btn-primary me-2"
    data-bs-toggle="modal"
    data-bs-target="#editModal"
    title="Edit data"
    v-on:click="generateTable()"
  >
    <i class="far fa-edit"></i>
  </button>

  <!-- Modal -->
  <div
    class="modal fade"
    id="editModal"
    tabindex="-1"
    aria-labelledby="editModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-dialog-scrollable">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5" id="shareModalLabel">
            Update {{ props.maps[store.currentMapName].name }}
          </h1>
        </div>
        <div class="modal-body">
          <table>
            <tr class="bg-light">
              <th v-for="(field, index) in state.data.fields" class="p-1">
                <span v-if="!field.headerEditable">{{ field.label }}</span>
                <input
                  v-else
                  type="text"
                  v-bind:id="'input-h-' + field.key"
                  v-model="field.label"
                />
              </th>
            </tr>
            <tr v-for="(row, rIndex) in state.data.items">
              <td v-for="(cell, cIndex) in row" class="p-1">
                <span v-if="!state.data.fields[cIndex].editable">{{ cell }}</span>
                <input
                  v-else
                  v-bind:id="'input-' + rIndex + '-' + cIndex"
                  v-model="state.data.items[rIndex][cIndex]"
                  :type="state.data.fields[cIndex].type"
                />
              </td>
            </tr>
          </table>
        </div>
        <div class="modal-footer modal-footer--sticky bg-white">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
          <button
            type="button"
            class="btn btn-primary"
            data-bs-dismiss="modal"
            v-on:click="updateCartogram"
          >
            Save changes
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
input {
  border: 0;
  box-sizing: border-box;
  display: block;
}

.modal-footer--sticky {
  position: sticky;
  bottom: 0;
  background-color: inherit;
  z-index: 1055;
}
</style>
../common/lib/cartMap
