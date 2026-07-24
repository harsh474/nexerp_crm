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
        <PaymentScheduleStepper
          v-if="tab.name === 'Payment Schedule'"
          :rows="doc.payment_schedule || []"
        />
        <BookingCoApplicantsTab
          v-else-if="tab.name === 'Co-Applicants'"
          :rows="doc.co_applicants || []"
        />
        <Activities
          v-else
          v-model:tabIndex="tabIndex"
          doctype="RE Booking"
          :docname="bookingId"
          :tabs="tabs"
        />
      </template>
    </Tabs>
    <Resizer side="right" class="flex flex-col justify-between border-l">
      <div
        class="flex h-[45px] cursor-copy items-center border-b px-5 py-2.5 text-lg-medium text-ink-gray-9"
        @click="copyToClipboard(bookingId)"
      >
        {{ __(bookingId) }}
      </div>
      <div class="flex flex-col gap-1 border-b p-5">
        <div class="truncate text-3xl-medium text-ink-gray-9">
          {{ __('Booking') }} · {{ doc.property_unit }}
        </div>
        <div class="truncate text-base text-ink-gray-5">
          {{ formatDate(doc.booking_date, '', true) }}
        </div>
      </div>
      <div class="flex flex-1 flex-col overflow-y-auto py-2">
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Booking Date') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ formatDate(doc.booking_date, '', true) || '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Token Amount') }}
          </div>
          <div class="w-[60%] truncate text-base text-ink-gray-8">
            {{ formattedTokenAmount || '-' }}
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Status') }}
          </div>
          <div class="w-[60%]">
            <Badge
              v-if="doc.status"
              variant="subtle"
              size="md"
              :theme="statusColor[doc.status] || 'gray'"
              :label="doc.status"
            />
          </div>
        </div>
        <div class="field flex items-center gap-2 px-5 py-2 leading-5">
          <div
            class="w-[40%] min-w-24 shrink-0 truncate text-sm text-ink-gray-5"
          >
            {{ __('Deal') }}
          </div>
          <div class="w-[60%] truncate text-base">
            <router-link
              v-if="doc.deal"
              :to="{ name: 'Deal', params: { dealId: doc.deal } }"
              class="text-ink-blue-3 hover:underline"
            >
              {{ doc.deal }}
            </router-link>
          </div>
        </div>
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
import PaymentScheduleStepper from '@/components/PaymentScheduleStepper.vue'
import BookingCoApplicantsTab from '@/components/BookingCoApplicantsTab.vue'
import ListChecksIcon from '~icons/lucide/list-checks'
import UsersIcon from '~icons/lucide/users'
import ActivityIcon from '@/components/Icons/ActivityIcon.vue'
import CommentIcon from '@/components/Icons/CommentIcon.vue'
import { copyToClipboard, formatDate } from '@/utils'
import { getSettings } from '@/stores/settings'
import { getMeta } from '@/stores/meta'
import { useDocument } from '@/data/document'
import { Badge, Tabs, Breadcrumbs, usePageMeta } from 'frappe-ui'
import { computed, ref, watch } from 'vue'

const props = defineProps({
  bookingId: { type: String, required: true },
})

const { brand } = getSettings()
const { getFormattedCurrency } = getMeta('RE Booking')

const statusColor = {
  Confirmed: 'blue',
  Cancelled: 'red',
  'Converted to Sale': 'green',
}

const errorTitle = ref('')
const errorMessage = ref('')

const { document, error } = useDocument('RE Booking', props.bookingId)

const doc = computed(() => document.doc || {})

const formattedTokenAmount = computed(() => {
  if (!doc.value.name) return ''
  return getFormattedCurrency('token_amount', doc.value)
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
  { label: __('Bookings'), route: { name: 'Bookings' } },
  {
    label: `${__('Booking')} · ${doc.value.property_unit || props.bookingId}`,
    route: { name: 'BookingDetail', params: { bookingId: props.bookingId } },
  },
])

usePageMeta(() => {
  return {
    title: `${__('Booking')} · ${doc.value.property_unit || props.bookingId}`,
    icon: brand.favicon,
  }
})

const tabIndex = ref(0)

const tabs = computed(() => [
  { name: 'Payment Schedule', label: __('Payment Schedule'), icon: ListChecksIcon },
  { name: 'Co-Applicants', label: __('Co-Applicants'), icon: UsersIcon },
  { name: 'Activity', label: __('Activity'), icon: ActivityIcon },
  { name: 'Comments', label: __('Comments'), icon: CommentIcon },
])
</script>
