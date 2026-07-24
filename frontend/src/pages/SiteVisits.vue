<template>
  <LayoutHeader>
    <template #left-header>
      <ViewBreadcrumbs v-model="viewControls" routeName="Site Visits" />
    </template>
    <template #right-header>
      <Button
        variant="solid"
        :label="__('Create')"
        iconLeft="plus"
        @click="showSiteVisitModal = true"
      />
    </template>
  </LayoutHeader>
  <ViewControls
    ref="viewControls"
    v-model="visits"
    v-model:loadMore="loadMore"
    v-model:resizeColumn="triggerResize"
    v-model:updatedPageCount="updatedPageCount"
    doctype="RE Site Visit"
    :options="{
      allowedViews: ['list'],
    }"
  />
  <SiteVisitsListView
    v-if="visits.data && rows.length"
    v-model="visits.data.page_length_count"
    :rows="rows"
    :columns="columns"
    :options="{
      showTooltip: false,
      resizeColumn: true,
      rowCount: visits.data.row_count,
      totalCount: visits.data.total_count,
    }"
    @loadMore="() => loadMore++"
    @columnWidthUpdated="() => triggerResize++"
    @updatePageCount="(count) => (updatedPageCount = count)"
  />
  <EmptyState v-else-if="visits.data && !rows.length" name="Site Visits" />
  <SiteVisitModal v-if="showSiteVisitModal" v-model="showSiteVisitModal" />
</template>

<script setup>
import ViewBreadcrumbs from '@/components/ViewBreadcrumbs.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import SiteVisitsListView from '@/components/ListViews/SiteVisitsListView.vue'
import EmptyState from '@/components/ListViews/EmptyState.vue'
import SiteVisitModal from '@/components/Modals/SiteVisitModal.vue'
import ViewControls from '@/components/ViewControls.vue'
import { formatDate } from '@/utils'
import { ref, computed } from 'vue'

const statusColor = {
  Scheduled: 'blue',
  Completed: 'green',
  'No-show': 'amber',
  Cancelled: 'red',
}

const viewControls = ref(null)
const showSiteVisitModal = ref(false)

// visits data is loaded in the ViewControls component
const visits = ref({})
const loadMore = ref(1)
const triggerResize = ref(1)
const updatedPageCount = ref(20)

const rows = computed(() => {
  if (!visits.value?.data?.data) return []
  return parseRows(visits.value?.data.data)
})

const columns = computed(() => {
  let _columns = visits.value?.data?.columns || []
  if (!_columns.length) return []
  // insert Reference right after Property Unit, and append Feedback at the end
  let _withReference = [
    _columns[0],
    { label: 'Reference', type: 'Data', key: 'reference', width: '14rem' },
    ..._columns.slice(1),
  ]
  return [
    ..._withReference,
    { label: 'Feedback', type: 'Data', key: 'feedback', width: '8rem' },
  ]
})

function parseRows(rows) {
  return rows.map((visit) => {
    let _rows = {}
    visits.value.data.rows.forEach((row) => {
      if (row === 'status') {
        _rows[row] = {
          label: visit.status,
          value: visit.status,
          color: statusColor[visit.status] || 'gray',
        }
      } else if (row === 'scheduled_on') {
        _rows[row] = formatDate(visit.scheduled_on)
      } else if (row === 'property_unit') {
        _rows[row] = { label: visit.property_unit, value: visit.property_unit }
      } else if (['reference_doctype', 'reference_name', 'feedback_rating'].includes(row)) {
        // handled below, not shown as their own columns
      } else {
        _rows[row] = visit[row]
      }
    })
    let refLabel = visit.reference_doctype === 'CRM Deal' ? 'Deal' : 'Lead'
    _rows.reference = `${visit.reference_name} · ${refLabel}`
    _rows.feedback = { rating: visit.feedback_rating ? Number(visit.feedback_rating) : 0 }
    return _rows
  })
}
</script>
