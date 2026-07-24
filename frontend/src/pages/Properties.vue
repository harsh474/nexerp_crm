<template>
  <LayoutHeader>
    <template #left-header>
      <ViewBreadcrumbs v-model="viewControls" routeName="Properties" />
    </template>
    <template #right-header>
      <Button
        variant="solid"
        :label="__('Create')"
        iconLeft="plus"
        @click="showPropertyModal = true"
      />
    </template>
  </LayoutHeader>
  <ViewControls
    ref="viewControls"
    v-model="properties"
    v-model:loadMore="loadMore"
    v-model:resizeColumn="triggerResize"
    v-model:updatedPageCount="updatedPageCount"
    doctype="RE Property Unit"
    :options="{
      allowedViews: ['list'],
    }"
  />
  <PropertiesListView
    v-if="properties.data && rows.length"
    v-model="properties.data.page_length_count"
    :rows="rows"
    :columns="columns"
    :options="{
      showTooltip: false,
      resizeColumn: true,
      rowCount: properties.data.row_count,
      totalCount: properties.data.total_count,
    }"
    @loadMore="() => loadMore++"
    @columnWidthUpdated="() => triggerResize++"
    @updatePageCount="(count) => (updatedPageCount = count)"
  />
  <EmptyState
    v-else-if="properties.data && !rows.length"
    name="Properties"
  />
  <PropertyModal v-if="showPropertyModal" v-model="showPropertyModal" />
</template>

<script setup>
import ViewBreadcrumbs from '@/components/ViewBreadcrumbs.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import PropertiesListView from '@/components/ListViews/PropertiesListView.vue'
import EmptyState from '@/components/ListViews/EmptyState.vue'
import PropertyModal from '@/components/Modals/PropertyModal.vue'
import ViewControls from '@/components/ViewControls.vue'
import { getMeta } from '@/stores/meta'
import { ref, computed } from 'vue'

const { getFormattedCurrency } = getMeta('RE Property Unit')

const statusColor = {
  Available: 'green',
  Held: 'amber',
  Booked: 'blue',
  Sold: 'gray',
  Blocked: 'red',
}

const viewControls = ref(null)
const showPropertyModal = ref(false)

// properties data is loaded in the ViewControls component
const properties = ref({})
const loadMore = ref(1)
const triggerResize = ref(1)
const updatedPageCount = ref(20)

const rows = computed(() => {
  if (!properties.value?.data?.data) return []
  return parseRows(properties.value?.data.data)
})

const columns = computed(() => properties.value?.data?.columns || [])

function parseRows(rows) {
  return rows.map((unit) => {
    let _rows = {}
    properties.value.data.rows.forEach((row) => {
      if (row === 'status') {
        _rows[row] = {
          label: unit.status,
          value: unit.status,
          color: statusColor[unit.status] || 'gray',
        }
      } else if (row === 'price') {
        _rows[row] = getFormattedCurrency('price', unit)
      } else {
        _rows[row] = unit[row]
      }
    })
    return _rows
  })
}
</script>
