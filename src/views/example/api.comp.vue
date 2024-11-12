<script lang="ts" setup>
import { Http } from '@/utils/http'

export interface RequestParams {
  ttId: number
}
const props = defineProps<{
  title?: string
  size?: number
  params?: RequestParams
}>()

const drawer = defineModel<boolean>('modelValue', { required: true })

const title = computed(() => props.title ?? '图像采集')
const size = computed(() => props.size ?? 500)

const loading = ref(true)

interface Data {
  teacherDueNums: number
  studentDueNums: number
  personNums: number

  dtime: string
  urlRoot: string
  ossOrgPicUrl: string
  ossAiPersonPic: string
}
const data = ref<Data[]>([])

async function fetchData(newParams: RequestParams) {
  try {
    data.value = (await Http.RetryRequest<Data[]>({
      url: '/boxImageData/getItems',
      data: newParams,
    })).data
  }
  catch (error) {
    console.error(`查询课时ID: ${newParams.ttId} 的上课照片失败\n ${error}`)
  }
  finally {
    console.log('请求结束')

    loading.value = false
  }
}

function getCompleteUrl(urlRoot: string, url: string): string {
  return urlRoot ? `${urlRoot}${url}` : url
}

const processedData = computed(() => {
  return data.value.map((item) => {
    const { teacherDueNums, studentDueNums, personNums, dtime, urlRoot, ossOrgPicUrl, ossAiPersonPic } = item

    const ossOriginalUrl = getCompleteUrl(urlRoot, ossOrgPicUrl)
    const ossAiOutputUrl = getCompleteUrl(urlRoot, ossAiPersonPic)

    return {
      /** 应到教师人数 */
      teacherDueNums,
      /** 应到学员人数 */
      studentDueNums,
      /** 实到人数 */
      personNums,

      /** 照片时间 */
      dtime,
      /** oss 原始图片地址列表 */
      ossOriginalUrl,
      /** oss Ai处理后 图片地址列表 */
      ossAiOutputUrl,
    }
  }).sort((a, b) => b.dtime.localeCompare(a.dtime))
})
// const ossOriginalUrls = computed(() => processedData.value.map(url => url.ossOriginalUrls))
const ossAiOutputUrls = computed(() => processedData.value.map(url => url.ossAiOutputUrl))

// 监听查询参数变化，查询新数据
watch(() => props.params, (newParams) => {
  if (drawer.value && newParams)
    fetchData(newParams)
}, { deep: true })
</script>

<template>
  <el-drawer v-model="drawer" :title :size>
    <el-skeleton :loading="loading" animated :count="4">
      <template #template>
        <div class="mb-12 h-[16px] flex items-center justify-between gap-20">
          <el-skeleton-item variant="text" />
          <el-skeleton-item class="w-[30%]!" variant="text" />
        </div>
        <el-skeleton-item class="mb-10 h-[240px]!" variant="image" />
      </template>
      <template #default>
        <div class="mb-[18px]">
          <div class="rounded-[5px] p-[5px]">
            <span class="font-bold">应到教师人数：</span>
            <span class="font-thin">{{ processedData[0]?.teacherDueNums ?? '' }}</span>
          </div>
          <div class="rounded-[5px] p-[5px]">
            <span class="font-bold">应到学员人数：</span>
            <span class="font-thin">{{ processedData[0]?.studentDueNums ?? '' }}</span>
          </div>
        </div>
        <el-timeline>
          <el-timeline-item v-for="({ dtime, ossOriginalUrl, personNums }, index) in processedData" :key="ossOriginalUrl" hide-timestamp placement="top">
            <div class="mb-8 flex items-center justify-between">
              <el-text tag="b">
                {{ dtime }}
              </el-text>
              <el-tag type="info" effect="dark">
                {{ personNums }}人
              </el-tag>
            </div>
            <el-image :src="ossOriginalUrl" lazy :preview-src-list="ossAiOutputUrls" :initial-index="index" />
          </el-timeline-item>
        </el-timeline>
      </template>
    </el-skeleton>
  </el-drawer>
</template>

<style lang="scss">
.el-drawer__body {
  padding-top: 0;
  padding-bottom: 0;
}
</style>
