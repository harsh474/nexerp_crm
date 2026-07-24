<template>
  <LayoutHeader>
    <template #left-header>
      <Breadcrumbs :items="breadcrumbs" />
    </template>
    <template v-if="!errorTitle" #right-header>
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
        <PropertyUnitSiteVisitsTab v-if="tab.name === 'Site Visits'" :unitId="unitId" />
        <Activities
          v-else
          v-model:tabIndex="tabIndex"
          doctype="RE Property Unit"
          :docname="unitId"
          :tabs="tabs"
        />
      </template>
    </Tabs>
    <Resizer side="right" class="flex flex-col justify-between border-l">
      <div
        class="flex h-[45px] cursor-copy items-center border-b px-5 py-2.5 text-lg-medium text-ink-gray-9"
        @click="copyToClipboard(unitId)"
      >
        {{ __(unitId) }}
      </div>
      <div class="flex flex-col gap-1 border-b p-5">
        <div class="truncate text-3xl-medium text-ink-gray-9">
          {{ doc.unit_number }}
        </div>
        <div class="truncate text-base text-ink-gray-5">
          {{ doc.project }}
        </div>
      </div>
      <div
        v-if="sections.data"
        class="flex flex-1 flex-col overflow-hidden"
      >
        <SidePanelLayout
          :sections="sections.data"
          doctype="RE Property Unit"
          :docname="unitId"
          @reload="sections.reload"
        />
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Booking') }}
          </div>
          <div class="w-[60%] truncate text-base">
            <router-link
              v-if="booking.data?.[0]"
              :to="{ name: 'BookingDetail', params: { bookingId: booking.data[0].name } }"
              class="text-ink-blue-3 hover:underline"
            >
              {{ booking.data[0].name }}
            </router-link>
            <span v-else class="text-ink-gray-4">{{ __('No booking yet') }}</span>
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
import PropertyUnitSiteVisitsTab from '@/components/PropertyUnitSiteVisitsTab.vue'
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
import { useRouter } from 'vue-router'

const props = defineProps({
  unitId: { type: String, required: true },
})

const router = useRouter()
const { brand } = getSettings()

const statusColor = {
  Available: 'green',
  Held: 'amber',
  Booked: 'blue',
  Sold: 'gray',
  Blocked: 'red',
}

const errorTitle = ref('')
const errorMessage = ref('')

const { document, error } = useDocument('RE Property Unit', props.unitId)

const doc = computed(() => document.doc || {})

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
  params: { doctype: 'RE Property Unit' },
  cache: ['SidePanelSections', 'RE Property Unit'],
  auto: true,
  transform: (_sections) => {
    _sections.forEach((section) => {
      section.columns?.[0]?.fields?.forEach((field) => {
        if (field.fieldname === 'current_deal') {
          field.link = (dealName) =>
            router.push({ name: 'Deal', params: { dealId: dealName } })
        } else if (field.fieldname === 'project') {
          field.link = (projectName) =>
            router.push({
              name: 'ProjectDetail',
              params: { projectId: projectName },
            })
        }
      })
    })
    return _sections
  },
})

const booking = createResource({
  url: 'frappe.client.get_list',
  cache: ['PropertyUnitBooking', props.unitId],
  params: {
    doctype: 'RE Booking',
    filters: { property_unit: props.unitId },
    fields: ['name'],
    limit_page_length: 1,
  },
  auto: true,
})

const breadcrumbs = computed(() => [
  { label: __('Properties'), route: { name: 'Properties' } },
  {
    label: doc.value.unit_number || props.unitId,
    route: { name: 'PropertyDetail', params: { unitId: props.unitId } },
  },
])

usePageMeta(() => {
  return {
    title: doc.value.unit_number || props.unitId,
    icon: brand.favicon,
  }
})

const tabIndex = ref(0)

const tabs = computed(() => [
  { name: 'Site Visits', label: __('Site Visits'), icon: ListIcon },
  { name: 'Activity', label: __('Activity'), icon: ActivityIcon },
  { name: 'Comments', label: __('Comments'), icon: CommentIcon },
  { name: 'Tasks', label: __('Tasks'), icon: TaskIcon },
  { name: 'Notes', label: __('Notes'), icon: NoteIcon },
  { name: 'Attachments', label: __('Attachments'), icon: AttachmentIcon },
])
</script>
