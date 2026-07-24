<template>
  <div class="flex flex-col px-3 pb-3 pt-4 sm:px-10 sm:pb-5">
    <div
      v-if="!rows.length"
      class="flex flex-1 items-center justify-center pt-10 text-base text-ink-gray-4"
    >
      {{ __('No payment schedule added yet') }}
    </div>
    <div v-for="(row, idx) in rows" v-else :key="row.name" class="flex gap-4">
      <div class="flex flex-col items-center">
        <div
          class="mt-1 h-3 w-3 shrink-0 rounded-full"
          :class="dotClass(row.status)"
        />
        <div
          v-if="idx !== rows.length - 1"
          class="my-1 w-px flex-1 bg-outline-gray-2"
        />
      </div>
      <div class="flex-1 pb-7">
        <div class="flex items-center justify-between gap-3">
          <div class="text-base font-medium text-ink-gray-9">
            {{ row.milestone_name }}
          </div>
          <Badge
            variant="subtle"
            size="sm"
            :theme="statusTheme[row.status] || 'gray'"
            :label="row.status"
          />
        </div>
        <div class="mt-1 flex items-center gap-2 text-sm text-ink-gray-5">
          <span>{{ formatDate(row.due_date, '', true) }}</span>
          <span>·</span>
          <span>{{ getFormattedCurrency('amount', row) }}</span>
          <span v-if="row.percent">·</span>
          <span v-if="row.percent">{{ row.percent }}%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { getMeta } from '@/stores/meta'
import { formatDate } from '@/utils'
import { Badge } from 'frappe-ui'

defineProps({
  rows: { type: Array, default: () => [] },
})

const { getFormattedCurrency } = getMeta('RE Payment Schedule Row')

// dot color communicates status; the connecting line stays neutral
const statusTheme = {
  Received: 'green',
  Pending: 'amber',
  Overdue: 'red',
}

const dotColor = {
  Received: 'bg-green-600',
  Pending: 'bg-amber-500',
  Overdue: 'bg-red-500',
}

function dotClass(status) {
  return dotColor[status] || 'bg-gray-400'
}
</script>
