<template>
  <LayoutHeader>
    <template #left-header>
      <ViewBreadcrumbs v-model="viewControls" routeName="Projects" />
    </template>
    <template #right-header>
      <Button
        variant="solid"
        :label="__('Create')"
        iconLeft="plus"
        @click="showProjectModal = true"
      />
    </template>
  </LayoutHeader>
  <ViewControls
    ref="viewControls"
    v-model="projects"
    v-model:loadMore="loadMore"
    v-model:resizeColumn="triggerResize"
    v-model:updatedPageCount="updatedPageCount"
    doctype="RE Project"
    :options="{
      allowedViews: ['list'],
    }"
  />
  <ProjectsListView
    v-if="projects.data && rows.length"
    v-model="projects.data.page_length_count"
    :rows="rows"
    :columns="columns"
    :options="{
      showTooltip: false,
      resizeColumn: true,
      rowCount: projects.data.row_count,
      totalCount: projects.data.total_count,
    }"
    @loadMore="() => loadMore++"
    @columnWidthUpdated="() => triggerResize++"
    @updatePageCount="(count) => (updatedPageCount = count)"
  />
  <EmptyState v-else-if="projects.data && !rows.length" name="Projects" />
  <ProjectModal v-if="showProjectModal" v-model="showProjectModal" />
</template>

<script setup>
import ViewBreadcrumbs from '@/components/ViewBreadcrumbs.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import ProjectsListView from '@/components/ListViews/ProjectsListView.vue'
import EmptyState from '@/components/ListViews/EmptyState.vue'
import ProjectModal from '@/components/Modals/ProjectModal.vue'
import ViewControls from '@/components/ViewControls.vue'
import { createResource } from 'frappe-ui'
import { ref, computed, watch } from 'vue'

const ownershipColor = {
  'Own Inventory': 'blue',
  'Third-Party': 'violet',
}

const statusColor = {
  Planning: 'gray',
  'Under Construction': 'amber',
  'Ready to Move': 'blue',
  Completed: 'green',
}

const viewControls = ref(null)
const showProjectModal = ref(false)

// projects data is loaded in the ViewControls component
const projects = ref({})
const loadMore = ref(1)
const triggerResize = ref(1)
const updatedPageCount = ref(20)

const unitCounts = createResource({
  url: 'nexerp_realestate.api.get_project_unit_counts',
})

const projectNames = computed(() =>
  (projects.value?.data?.data || []).map((project) => project.name),
)

watch(
  projectNames,
  (names) => {
    if (!names.length) return
    unitCounts.submit({ projects: names })
  },
  { immediate: true },
)

const rows = computed(() => {
  if (!projects.value?.data?.data) return []
  return parseRows(projects.value?.data.data)
})

const columns = computed(() => {
  let _columns = projects.value?.data?.columns || []
  if (!_columns.length) return []
  return [
    ..._columns,
    { label: 'Available / Total Units', type: 'Data', key: 'unit_counts', width: '10rem' },
  ]
})

function parseRows(rows) {
  return rows.map((project) => {
    let _rows = {}
    projects.value.data.rows.forEach((row) => {
      if (row === 'ownership_type') {
        _rows[row] = {
          label: project.ownership_type,
          value: project.ownership_type,
          color: ownershipColor[project.ownership_type] || 'gray',
        }
      } else if (row === 'status') {
        _rows[row] = {
          label: project.status,
          value: project.status,
          color: statusColor[project.status] || 'gray',
        }
      } else if (row === 'builder') {
        _rows[row] = project.builder || '—'
      } else {
        _rows[row] = project[row]
      }
    })
    let counts = unitCounts.data?.[project.name]
    _rows.unit_counts = counts ? `${counts.available} / ${counts.total}` : '- / -'
    return _rows
  })
}
</script>
