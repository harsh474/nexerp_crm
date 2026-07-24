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
      <template #tab-panel>
        <Activities
          v-model:tabIndex="tabIndex"
          doctype="RE Site Visit"
          :docname="visitId"
          :tabs="tabs"
        />
      </template>
    </Tabs>
    <Resizer side="right" class="flex flex-col justify-between border-l">
      <div
        class="flex h-[45px] cursor-copy items-center border-b px-5 py-2.5 text-lg-medium text-ink-gray-9"
        @click="copyToClipboard(visitId)"
      >
        {{ __(visitId) }}
      </div>
      <div class="flex flex-col gap-1 border-b p-5">
        <div class="truncate text-3xl-medium text-ink-gray-9">
          {{ __('Visit') }} · {{ doc.property_unit }}
        </div>
        <div class="truncate text-base text-ink-gray-5">
          {{ formatDate(doc.scheduled_on) }}
        </div>
      </div>
      <div class="flex flex-1 flex-col overflow-y-auto py-2">
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Property Unit') }}
          </div>
          <div class="w-[60%] truncate text-base">
            <router-link
              v-if="doc.property_unit"
              :to="{ name: 'PropertyDetail', params: { unitId: doc.property_unit } }"
              class="text-ink-blue-3 hover:underline"
            >
              {{ doc.property_unit }}
            </router-link>
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ referenceLabel }}
          </div>
          <div class="w-[60%] truncate text-base">
            <router-link
              v-if="doc.reference_name"
              :to="referenceRoute"
              class="text-ink-blue-3 hover:underline"
            >
              {{ doc.reference_name }}
            </router-link>
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Scheduled On') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ formatDate(doc.scheduled_on) || '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Sales Person') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ salesPersonName || '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Feedback Rating') }}
          </div>
          <div class="w-[60%] flex items-center gap-2">
            <template v-if="doc.feedback_rating">
              <RatingInput :value="Number(doc.feedback_rating) / 5" :max="5" :disabled="true" />
              <span class="text-sm text-ink-gray-5">
                {{ doc.feedback_rating }}/5
              </span>
            </template>
            <span v-else class="text-base text-ink-gray-4">-</span>
          </div>
        </div>
        <div class="field flex items-start gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate pt-1 text-sm text-ink-gray-5"
          >
            {{ __('Feedback Notes') }}
          </div>
          <div class="w-[60%] text-base text-ink-gray-8">
            {{ doc.feedback_notes || '-' }}
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
import Activities from '@/components/Activities/Activities.vue'
import RatingInput from '@/components/Controls/RatingInput.vue'
import ActivityIcon from '@/components/Icons/ActivityIcon.vue'
import CommentIcon from '@/components/Icons/CommentIcon.vue'
import { copyToClipboard, formatDate } from '@/utils'
import { getSettings } from '@/stores/settings'
import { usersStore } from '@/stores/users'
import { useDocument } from '@/data/document'
import { Badge, Tabs, Breadcrumbs, usePageMeta } from 'frappe-ui'
import { computed, ref, watch } from 'vue'

const props = defineProps({
  visitId: { type: String, required: true },
})

const { brand } = getSettings()
const { getUser } = usersStore()

const statusColor = {
  Scheduled: 'blue',
  Completed: 'green',
  'No-show': 'amber',
  Cancelled: 'red',
}

const errorTitle = ref('')
const errorMessage = ref('')

const { document, error } = useDocument('RE Site Visit', props.visitId)

const doc = computed(() => document.doc || {})

const salesPersonName = computed(() =>
  doc.value.sales_person ? getUser(doc.value.sales_person)?.full_name : '',
)

const referenceLabel = computed(() =>
  doc.value.reference_doctype === 'CRM Deal' ? __('Deal') : __('Lead'),
)

const referenceRoute = computed(() => {
  if (doc.value.reference_doctype === 'CRM Deal') {
    return { name: 'Deal', params: { dealId: doc.value.reference_name } }
  }
  return { name: 'Lead', params: { leadId: doc.value.reference_name } }
})

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

const breadcrumbs = computed(() => [
  { label: __('Site Visits'), route: { name: 'SiteVisits' } },
  {
    label: `${__('Visit')} · ${doc.value.property_unit || props.visitId}`,
    route: { name: 'SiteVisitDetail', params: { visitId: props.visitId } },
  },
])

usePageMeta(() => {
  return {
    title: `${__('Visit')} · ${doc.value.property_unit || props.visitId}`,
    icon: brand.favicon,
  }
})

const tabIndex = ref(0)

const tabs = computed(() => [
  { name: 'Activity', label: __('Activity'), icon: ActivityIcon },
  { name: 'Comments', label: __('Comments'), icon: CommentIcon },
])
</script>
