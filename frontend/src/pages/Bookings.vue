<template>
  <LayoutHeader>
    <template #left-header>
      <ViewBreadcrumbs v-model="viewControls" routeName="Bookings" />
    </template>
  </LayoutHeader>
  <ViewControls
    ref="viewControls"
    v-model="bookings"
    v-model:loadMore="loadMore"
    v-model:resizeColumn="triggerResize"
    v-model:updatedPageCount="updatedPageCount"
    doctype="RE Booking"
    :options="{
      allowedViews: ['list'],
    }"
  />
  <BookingsListView
    v-if="bookings.data && rows.length"
    v-model="bookings.data.page_length_count"
    :rows="rows"
    :columns="columns"
    :options="{
      showTooltip: false,
      resizeColumn: true,
      rowCount: bookings.data.row_count,
      totalCount: bookings.data.total_count,
    }"
    @loadMore="() => loadMore++"
    @columnWidthUpdated="() => triggerResize++"
    @updatePageCount="(count) => (updatedPageCount = count)"
  />
  <EmptyState v-else-if="bookings.data && !rows.length" name="Bookings" />
</template>

<script setup>
import ViewBreadcrumbs from '@/components/ViewBreadcrumbs.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import BookingsListView from '@/components/ListViews/BookingsListView.vue'
import EmptyState from '@/components/ListViews/EmptyState.vue'
import ViewControls from '@/components/ViewControls.vue'
import { createResource } from 'frappe-ui'
import { ref, computed, watch } from 'vue'

const statusColor = {
  Confirmed: 'blue',
  Cancelled: 'red',
  'Converted to Sale': 'green',
}

const viewControls = ref(null)

// bookings data is loaded in the ViewControls component
const bookings = ref({})
const loadMore = ref(1)
const triggerResize = ref(1)
const updatedPageCount = ref(20)

const milestoneCounts = createResource({
  url: 'nexerp_realestate.api.get_booking_milestone_counts',
})

const bookingNames = computed(() =>
  (bookings.value?.data?.data || []).map((booking) => booking.name),
)

watch(
  bookingNames,
  (names) => {
    if (!names.length) return
    milestoneCounts.submit({ bookings: names })
  },
  { immediate: true },
)

const rows = computed(() => {
  if (!bookings.value?.data?.data) return []
  return parseRows(bookings.value?.data.data)
})

const columns = computed(() => {
  let _columns = bookings.value?.data?.columns || []
  if (!_columns.length) return []
  return [
    ..._columns,
    { label: 'Milestones Paid', type: 'Data', key: 'milestones_paid', width: '9rem' },
  ]
})

function parseRows(rows) {
  return rows.map((booking) => {
    let _rows = {}
    bookings.value.data.rows.forEach((row) => {
      if (row === 'property_unit' || row === 'deal') {
        _rows[row] = { label: booking[row], value: booking[row] }
      } else if (row === 'status') {
        _rows[row] = {
          label: booking.status,
          value: booking.status,
          color: statusColor[booking.status] || 'gray',
        }
      } else {
        _rows[row] = booking[row]
      }
    })
    let counts = milestoneCounts.data?.[booking.name]
    _rows.milestones_paid = counts ? `${counts.received} / ${counts.total}` : '- / -'
    return _rows
  })
}
</script>
