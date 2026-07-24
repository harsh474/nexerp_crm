<template>
  <FadedScrollableDiv class="flex flex-col h-full overflow-y-auto px-3 pb-3 sm:px-10 sm:pb-5">
    <div
      v-if="visits.loading"
      class="flex flex-1 flex-col items-center justify-center gap-3 pt-10 text-2xl-medium text-ink-gray-4"
    >
      <LoadingIndicator class="h-6 w-6" />
      <span>{{ __('Loading...') }}</span>
    </div>
    <div
      v-else-if="!visits.data?.length"
      class="flex flex-1 items-center justify-center pt-10 text-base text-ink-gray-4"
    >
      {{ __('No site visits for this property yet') }}
    </div>
    <div v-else class="flex flex-col divide-y divide-outline-gray-modals pt-3">
      <div
        v-for="visit in visits.data"
        :key="visit.name"
        class="flex cursor-pointer items-center justify-between gap-3 rounded px-2 py-3 hover:bg-surface-gray-2"
        @click="
          router.push({ name: 'SiteVisitDetail', params: { visitId: visit.name } })
        "
      >
        <div class="flex min-w-0 flex-col gap-0.5">
          <div class="truncate text-base font-medium text-ink-gray-9">
            {{ formatDate(visit.scheduled_on) }}
          </div>
          <div class="truncate text-sm text-ink-gray-5">
            {{ visit.reference_name }}
            · {{ visit.reference_doctype === 'CRM Deal' ? __('Deal') : __('Lead') }}
          </div>
        </div>
        <Badge
          variant="subtle"
          size="sm"
          :theme="statusColor[visit.status] || 'gray'"
          :label="visit.status"
        />
      </div>
    </div>
  </FadedScrollableDiv>
</template>

<script setup>
import FadedScrollableDiv from '@/components/FadedScrollableDiv.vue'
import LoadingIndicator from '@/components/Icons/LoadingIndicator.vue'
import { formatDate } from '@/utils'
import { createResource } from 'frappe-ui'
import { useRouter } from 'vue-router'

const props = defineProps({
  unitId: { type: String, required: true },
})

const router = useRouter()

const statusColor = {
  Scheduled: 'blue',
  Completed: 'green',
  'No-show': 'amber',
  Cancelled: 'red',
}

const visits = createResource({
  url: 'frappe.client.get_list',
  cache: ['PropertyUnitSiteVisits', props.unitId],
  params: {
    doctype: 'RE Site Visit',
    filters: { property_unit: props.unitId },
    fields: ['name', 'reference_doctype', 'reference_name', 'scheduled_on', 'status'],
    order_by: 'scheduled_on desc',
    limit_page_length: 0,
  },
  auto: true,
})
</script>
