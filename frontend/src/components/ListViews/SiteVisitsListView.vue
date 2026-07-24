<template>
  <ListView
    :class="$attrs.class"
    :columns="columns"
    :rows="rows"
    :options="{
      getRowRoute: (row) => ({
        name: 'SiteVisitDetail',
        params: { visitId: row.name },
      }),
      showTooltip: options.showTooltip,
      resizeColumn: options.resizeColumn,
    }"
    row-key="name"
  >
    <ListHeader
      class="sm:mx-5 mx-3"
      @columnWidthUpdated="emit('columnWidthUpdated')"
    >
      <ListHeaderItem
        v-for="column in columns"
        :key="column.key"
        :item="column"
        @columnWidthUpdated="emit('columnWidthUpdated', column)"
      />
    </ListHeader>
    <ListRows v-slot="{ column, item }" :rows="rows" doctype="RE Site Visit">
      <ListRowItem :item="item" :align="column.align" class="overflow-hidden">
        <template #default="{ label }">
          <div v-if="column.key === 'property_unit'">
            <router-link
              v-if="item.value"
              :to="{ name: 'PropertyDetail', params: { unitId: item.value } }"
              class="truncate text-base text-ink-blue-3 hover:underline"
              @click.stop
            >
              {{ item.value }}
            </router-link>
          </div>
          <div v-else-if="column.key === 'status'">
            <Badge
              v-if="item.value"
              variant="subtle"
              :theme="item.color"
              size="md"
              :label="item.value"
            />
          </div>
          <div v-else-if="column.key === 'feedback'">
            <RatingInput
              v-if="item.rating"
              :value="item.rating / 5"
              :max="5"
              :disabled="true"
            />
            <span v-else class="text-base text-ink-gray-4">—</span>
          </div>
          <div v-else-if="label" class="truncate text-base">
            {{ label }}
          </div>
          <div v-else class="truncate text-base text-ink-gray-4">-</div>
        </template>
      </ListRowItem>
    </ListRows>
  </ListView>
  <ListFooter
    v-if="pageLengthCount"
    v-model="pageLengthCount"
    class="border-t sm:px-5 px-3 py-2"
    :options="{
      rowCount: options.rowCount,
      totalCount: options.totalCount,
    }"
    @loadMore="emit('loadMore')"
  />
</template>

<script setup>
import ListRows from '@/components/ListViews/ListRows.vue'
import RatingInput from '@/components/Controls/RatingInput.vue'
import {
  ListView,
  ListHeader,
  ListHeaderItem,
  ListRowItem,
  ListFooter,
  Badge,
} from 'frappe-ui'
import { watch } from 'vue'

defineProps({
  rows: { type: Array, required: true },
  columns: { type: Array, required: true },
  options: {
    type: Object,
    default: () => ({
      showTooltip: false,
      resizeColumn: true,
      totalCount: 0,
      rowCount: 0,
    }),
  },
})

const emit = defineEmits(['loadMore', 'updatePageCount', 'columnWidthUpdated'])

const pageLengthCount = defineModel({ type: Number })

watch(pageLengthCount, (val, old_value) => {
  if (val === old_value) return
  emit('updatePageCount', val)
})
</script>
