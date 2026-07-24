<template>
  <Dialog v-model:open="show" :size="'3xl'">
    <template #body>
      <div class="bg-surface-elevation-2 px-4 pb-6 pt-5 sm:px-6">
        <div class="mb-5 flex items-center justify-between">
          <div>
            <h3 class="text-3xl-semibold leading-6 text-ink-gray-9">
              {{ __('Create Property Unit') }}
            </h3>
          </div>
          <div class="flex items-center gap-1">
            <Button
              variant="ghost"
              class="w-7"
              icon="lucide-x"
              @click="show = false"
            />
          </div>
        </div>
        <div>
          <FieldLayout
            v-if="tabs.data?.length"
            :tabs="tabs.data"
            :data="propertyUnit.doc"
            doctype="RE Property Unit"
          />
          <ErrorMessage v-if="error" class="mt-4" :message="__(error)" />
        </div>
      </div>
      <div class="px-4 pb-7 pt-4 sm:px-6">
        <div class="flex flex-row-reverse gap-2">
          <Button
            variant="solid"
            :label="__('Create')"
            :loading="isCreating"
            @click="createPropertyUnit"
          />
        </div>
      </div>
    </template>
  </Dialog>
</template>

<script setup>
import FieldLayout from '@/components/FieldLayout/FieldLayout.vue'
import { useDocument } from '@/data/document'
import { createResource } from 'frappe-ui'
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const props = defineProps({
  defaults: { type: Object, default: () => ({}) },
})

const show = defineModel({ type: Boolean })
const router = useRouter()
const error = ref(null)
const isCreating = ref(false)

const { document: propertyUnit } = useDocument('RE Property Unit')

const tabs = createResource({
  url: 'crm.fcrm.doctype.crm_fields_layout.crm_fields_layout.get_fields_layout',
  cache: ['QuickEntry', 'RE Property Unit'],
  params: { doctype: 'RE Property Unit', type: 'Quick Entry' },
  auto: true,
})

function createPropertyUnit() {
  createResource({
    url: 'frappe.client.insert',
    params: { doc: { doctype: 'RE Property Unit', ...propertyUnit.doc } },
    auto: true,
    validate() {
      error.value = null
      if (!propertyUnit.doc.unit_number) {
        error.value = __('Unit Number is required')
        return error.value
      }
      if (!propertyUnit.doc.project) {
        error.value = __('Project is required')
        return error.value
      }
      if (!propertyUnit.doc.price) {
        error.value = __('Price is required')
        return error.value
      }
      isCreating.value = true
    },
    onSuccess(doc) {
      isCreating.value = false
      show.value = false
      router.push({ name: 'PropertyDetail', params: { unitId: doc.name } })
    },
    onError(err) {
      isCreating.value = false
      if (!err.messages) {
        error.value = err.message
        return
      }
      error.value = err.messages.join('\n')
    },
  })
}

onMounted(() => {
  Object.assign(propertyUnit.doc, props.defaults)
  if (!propertyUnit.doc.status) {
    propertyUnit.doc.status = 'Available'
  }
})
</script>
