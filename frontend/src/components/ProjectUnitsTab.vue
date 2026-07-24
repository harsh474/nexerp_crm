<template>
  <FadedScrollableDiv class="flex flex-col h-full overflow-y-auto px-3 pb-3 sm:px-10 sm:pb-5">
    <div
      v-if="units.loading"
      class="flex flex-1 flex-col items-center justify-center gap-3 pt-10 text-2xl-medium text-ink-gray-4"
    >
      <LoadingIndicator class="h-6 w-6" />
      <span>{{ __('Loading...') }}</span>
    </div>
    <div
      v-else-if="!units.data?.length"
      class="flex flex-1 items-center justify-center pt-10 text-base text-ink-gray-4"
    >
      {{ __('No units in this project yet') }}
    </div>
    <div v-else class="flex flex-col divide-y divide-outline-gray-modals pt-3">
      <div
        v-for="unit in units.data"
        :key="unit.name"
        class="flex cursor-pointer items-center justify-between gap-3 rounded px-2 py-3 hover:bg-surface-gray-2"
        @click="
          router.push({ name: 'PropertyDetail', params: { unitId: unit.name } })
        "
      >
        <div class="flex min-w-0 flex-col gap-0.5">
          <div class="truncate text-base font-medium text-ink-gray-9">
            {{ unit.unit_number }}
          </div>
          <div class="truncate text-sm text-ink-gray-5">
            {{ unit.property_type || '-' }}
          </div>
        </div>
        <div class="flex shrink-0 items-center gap-3">
          <div class="text-sm text-ink-gray-7">
            {{ getFormattedCurrency('price', unit) }}
          </div>
          <Badge
            variant="subtle"
            size="sm"
            :theme="statusColor[unit.status] || 'gray'"
            :label="unit.status"
          />
        </div>
      </div>
    </div>
  </FadedScrollableDiv>
</template>

<script setup>
import FadedScrollableDiv from '@/components/FadedScrollableDiv.vue'
import LoadingIndicator from '@/components/Icons/LoadingIndicator.vue'
import { getMeta } from '@/stores/meta'
import { createResource } from 'frappe-ui'
import { useRouter } from 'vue-router'

const props = defineProps({
  projectId: { type: String, required: true },
})

const router = useRouter()
const { getFormattedCurrency } = getMeta('RE Property Unit')

const statusColor = {
  Available: 'green',
  Held: 'amber',
  Booked: 'blue',
  Sold: 'gray',
  Blocked: 'red',
}

const units = createResource({
  url: 'frappe.client.get_list',
  cache: ['ProjectUnits', props.projectId],
  params: {
    doctype: 'RE Property Unit',
    filters: { project: props.projectId },
    fields: ['name', 'unit_number', 'property_type', 'price', 'status'],
    order_by: 'unit_number asc',
    limit_page_length: 0,
  },
  auto: true,
})
</script>
