<template>
  <el-dialog
    :title="`${$t('views.log.selectDataset')}/${$t('common.fileUpload.document')}`"
    v-model="dialogVisible"
    width="500"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
  >
    <el-form
      ref="formRef"
      :model="form"
      label-position="top"
      require-asterisk-position="right"
      :rules="rules"
      @submit.prevent
    >
      <el-form-item :label="$t('views.log.selectDataset')" prop="dataset_id">
        <el-select
          v-model="form.dataset_id"
          filterable
          :placeholder="$t('views.log.selectDatasetPlaceholder')"
          :loading="optionLoading"
          @change="changeDataset"
        >
          <el-option v-for="item in datasetList" :key="item.id" :label="item.name" :value="item.id">
            <span class="flex align-center">
              <AppAvatar
                v-if="!item.dataset_id && item.type === '1'"
                class="mr-12 avatar-purple"
                shape="square"
                :size="24"
              >
                <img src="@/assets/icon_web.svg" style="width: 58%" alt="" />
              </AppAvatar>
              <AppAvatar
                v-else-if="!item.dataset_id && item.type === '2'"
                class="mr-12 avatar-purple"
                shape="square"
                :size="24"
                style="background: none"
              >
                <img src="@/assets/logo_lark.svg" style="width: 100%" alt="" />
              </AppAvatar>
              <AppAvatar
                v-else-if="!item.dataset_id && item.type === '0'"
                class="mr-12 avatar-blue"
                shape="square"
                :size="24"
              >
                <img src="@/assets/icon_document.svg" style="width: 58%" alt="" />
              </AppAvatar>
              {{ item.name }}
            </span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label="$t('views.log.saveToDocument')" prop="document_id">
        <el-select
          v-model="form.document_id"
          filterable
          :placeholder="$t('views.log.documentPlaceholder')"
          :loading="optionLoading"
        >
          <el-option
            v-for="item in documentList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          >
            {{ item.name }}
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click.prevent="dialogVisible = false"> {{ $t('common.cancel') }} </el-button>
        <el-button type="primary" @click="submitForm(formRef)" :loading="loading">
          {{ $t('views.document.setting.migration') }}
        </el-button>
      </span>
    </template>
  </el-dialog>
</template>
<script setup lang="ts">
import { ref, watch, reactive } from 'vue'
import { useRoute } from 'vue-router'
import type { FormInstance, FormRules } from 'element-plus'
import paragraphApi from '@/api/paragraph'
import useStore from '@/stores'
import { t } from '@/locales'
const { dataset, document } = useStore()

const route = useRoute()
const {
  params: { id, documentId }
} = route as any

const emit = defineEmits(['refresh'])
const formRef = ref()

const dialogVisible = ref<boolean>(false)
const loading = ref(false)

const form = ref<any>({
  dataset_id: '',
  document_id: ''
})

const rules = reactive<FormRules>({
  dataset_id: [
    { required: true, message: t('views.log.selectDatasetPlaceholder'), trigger: 'change' }
  ],
  document_id: [{ required: true, message: t('views.log.documentPlaceholder'), trigger: 'change' }]
})

const datasetList = ref<any[]>([])
const documentList = ref<any[]>([])
const optionLoading = ref(false)
const paragraphList = ref<string[]>([])

watch(dialogVisible, (bool) => {
  if (!bool) {
    form.value = {
      dataset_id: '',
      document_id: ''
    }
    datasetList.value = []
    documentList.value = []
    paragraphList.value = []
    formRef.value?.clearValidate()
  }
})

function changeDataset(id: string) {
  form.value.document_id = ''
  getDocument(id)
}

function getDocument(id: string) {
  document.asyncGetAllDocument(id, loading).then((res: any) => {
    documentList.value = res.data?.filter((v: any) => v.id !== documentId)
  })
}

function getDataset() {
  dataset.asyncGetAllDataset(loading).then((res: any) => {
    datasetList.value = res.data
  })
}

const open = (list: any) => {
  paragraphList.value = list
  getDataset()
  formRef.value?.clearValidate()
  dialogVisible.value = true
}
const submitForm = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      paragraphApi
        .putMigrateMulParagraph(
          id,
          documentId,
          form.value.dataset_id,
          form.value.document_id,
          paragraphList.value,
          loading
        )
        .then(() => {
          emit('refresh')
          dialogVisible.value = false
        })
    }
  })
}

defineExpose({ open })
</script>
<style lang="scss" scoped></style>
