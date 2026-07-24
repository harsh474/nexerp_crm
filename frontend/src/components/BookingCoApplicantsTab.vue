<template>
  <FadedScrollableDiv class="flex flex-col h-full overflow-y-auto px-3 pb-3 sm:px-10 sm:pb-5">
    <div
      v-if="!rows.length"
      class="flex flex-1 items-center justify-center pt-10 text-base text-ink-gray-4"
    >
      {{ __('No co-applicants added yet') }}
    </div>
    <div v-else class="flex flex-col divide-y divide-outline-gray-modals pt-3">
      <div
        v-for="row in rows"
        :key="row.name"
        class="flex items-center justify-between gap-3 px-2 py-3"
      >
        <router-link
          :to="{ name: 'Contact', params: { contactId: row.contact } }"
          class="truncate text-base text-ink-blue-3 hover:underline"
        >
          {{ row.contact }}
        </router-link>
        <div class="shrink-0 text-sm text-ink-gray-6">
          {{ row.share_percent }}% {{ __('share') }}
        </div>
      </div>
    </div>
  </FadedScrollableDiv>
</template>

<script setup>
import FadedScrollableDiv from '@/components/FadedScrollableDiv.vue'

defineProps({
  rows: { type: Array, default: () => [] },
})
</script>
