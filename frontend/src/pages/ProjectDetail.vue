<template>
  <LayoutHeader>
    <template #left-header>
      <Breadcrumbs :items="breadcrumbs" />
    </template>
    <template v-if="!errorTitle" #right-header>
      <Badge
        v-if="doc.ownership_type"
        variant="subtle"
        size="lg"
        :theme="ownershipColor[doc.ownership_type] || 'gray'"
        :label="doc.ownership_type"
      />
      <Badge
        v-if="doc.status"
        variant="subtle"
        size="lg"
        :theme="statusColor[doc.status] || 'gray'"
        :label="doc.status"
      />
    </template>
  </LayoutHeader>
  <div v-if="doc.name" class="flex h-full overflow-hidden">
    <Tabs
      v-model="tabIndex"
      as="div"
      :tabs="tabs"
      class="flex flex-1 overflow-hidden flex-col [&_[role='tab']]:px-0 [&_[role='tab']]:shrink-0 [&_[role='tablist']]:px-5 [&_[role='tablist']::-webkit-scrollbar]:h-0 [&_[role='tablist']]:min-h-[45px] [&_[role='tablist']]:gap-7.5"
    >
      <template #tab-panel="{ tab }">
        <ProjectUnitsTab v-if="tab.name === 'Units'" :projectId="projectId" />
        <Activities
          v-else
          v-model:tabIndex="tabIndex"
          doctype="RE Project"
          :docname="projectId"
          :tabs="tabs"
        />
      </template>
    </Tabs>
    <Resizer side="right" class="flex flex-col justify-between border-l">
      <div
        class="flex h-[45px] cursor-copy items-center border-b px-5 py-2.5 text-lg-medium text-ink-gray-9"
        @click="copyToClipboard(projectId)"
      >
        {{ __(projectId) }}
      </div>
      <div class="flex flex-col gap-1 border-b p-5">
        <div class="truncate text-3xl-medium text-ink-gray-9">
          {{ doc.project_name }}
        </div>
        <div class="truncate text-base text-ink-gray-5">
          {{ doc.city }}
        </div>
      </div>
      <div class="flex flex-1 flex-col overflow-y-auto py-2">
        <div v-if="sections.data" class="flex flex-col">
          <SidePanelLayout
            :sections="sections.data"
            doctype="RE Project"
            :docname="projectId"
            @reload="sections.reload"
          />
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Total Units') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ unitCounts.data?.total ?? '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Available Units') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ unitCounts.data?.available ?? '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Builder') }}
          </div>
          <div class="w-[60%] truncate text-base">
            <router-link
              v-if="doc.builder"
              :to="{ name: 'Organization', params: { organizationId: doc.builder } }"
              class="text-ink-blue-3 hover:underline"
            >
              {{ doc.builder }}
            </router-link>
            <span v-else class="text-ink-gray-4">
              {{ __('Own inventory — no external builder') }}
            </span>
          </div>
        </div>
        <div class="field flex items-start gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate pt-1 text-sm text-ink-gray-5"
          >
            {{ __('Amenities') }}
          </div>
          <div class="flex w-[60%] flex-wrap gap-1.5">
            <Badge
              v-for="amenity in amenities"
              :key="amenity"
              variant="outline"
              size="md"
              :label="amenity"
            />
            <span v-if="!amenities.length" class="text-base text-ink-gray-4">
              -
            </span>
          </div>
        </div>
      </div>
    </Resizer>
  </div>
  <ErrorPage
    v-else-if="errorTitle"
    :errorTitle="errorTitle"
    :errorMessage="errorMessage"
  />
</template>

<script setup>
import ErrorPage from '@/components/ErrorPage.vue'
import Resizer from '@/components/Resizer.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import SidePanelLayout from '@/components/SidePanelLayout.vue'
import Activities from '@/components/Activities/Activities.vue'
import ProjectUnitsTab from '@/components/ProjectUnitsTab.vue'
import ListIcon from '@/components/Icons/ListIcon.vue'
import ActivityIcon from '@/components/Icons/ActivityIcon.vue'
import CommentIcon from '@/components/Icons/CommentIcon.vue'
import TaskIcon from '@/components/Icons/TaskIcon.vue'
import NoteIcon from '@/components/Icons/NoteIcon.vue'
import AttachmentIcon from '@/components/Icons/AttachmentIcon.vue'
import { copyToClipboard } from '@/utils'
import { getSettings } from '@/stores/settings'
import { useDocument } from '@/data/document'
import { Badge, Tabs, Breadcrumbs, createResource, usePageMeta } from 'frappe-ui'
import { computed, ref, watch } from 'vue'

const props = defineProps({
  projectId: { type: String, required: true },
})

const { brand } = getSettings()

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

const errorTitle = ref('')
const errorMessage = ref('')

const { document, error } = useDocument('RE Project', props.projectId)

const doc = computed(() => document.doc || {})

const amenities = computed(() =>
  (doc.value.amenities || []).map((row) => row.amenity).filter(Boolean),
)

watch(
  error,
  (err) => {
    if (err) {
      errorTitle.value = __(
        err.exc_type == 'DoesNotExistError'
          ? 'Document Not Found'
          : 'Error Occurred',
      )
      errorMessage.value = __(err.messages?.[0] || 'An Error Occurred')
    } else {
      errorTitle.value = ''
      errorMessage.value = ''
    }
  },
)

const sections = createResource({
  url: 'crm.fcrm.doctype.crm_fields_layout.crm_fields_layout.get_sidepanel_sections',
  params: { doctype: 'RE Project' },
  cache: ['SidePanelSections', 'RE Project'],
  auto: true,
})

const unitCounts = createResource({
  url: 'nexerp_realestate.api.get_project_unit_counts',
  cache: ['ProjectUnitCounts', props.projectId],
  params: { projects: [props.projectId] },
  auto: true,
  transform: (data) => data[props.projectId],
})

const breadcrumbs = computed(() => [
  { label: __('Projects'), route: { name: 'Projects' } },
  {
    label: doc.value.project_name || props.projectId,
    route: { name: 'ProjectDetail', params: { projectId: props.projectId } },
  },
])

usePageMeta(() => {
  return {
    title: doc.value.project_name || props.projectId,
    icon: brand.favicon,
  }
})

const tabIndex = ref(0)

const tabs = computed(() => [
  { name: 'Units', label: __('Units'), icon: ListIcon },
  { name: 'Activity', label: __('Activity'), icon: ActivityIcon },
  { name: 'Comments', label: __('Comments'), icon: CommentIcon },
  { name: 'Tasks', label: __('Tasks'), icon: TaskIcon },
  { name: 'Notes', label: __('Notes'), icon: NoteIcon },
  { name: 'Attachments', label: __('Attachments'), icon: AttachmentIcon },
])
</script>
