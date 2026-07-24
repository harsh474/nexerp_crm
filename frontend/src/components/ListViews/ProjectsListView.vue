<template>
  <ListView
    :class="$attrs.class"
    :columns="columns"
    :rows="rows"
    :options="{
      getRowRoute: (row) => ({
        name: 'ProjectDetail',
        params: { projectId: row.name },
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
    <ListRows v-slot="{ column, item }" :rows="rows" doctype="RE Project">
      <ListRowItem :item="item" :align="column.align" class="overflow-hidden">
        <template #default="{ label }">
          <div v-if="['ownership_type', 'status'].includes(column.key)">
            <Badge
              v-if="item.value"
              variant="subtle"
              :theme="item.color"
              size="md"
              :label="item.value"
            />
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
