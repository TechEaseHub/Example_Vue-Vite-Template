<script setup lang="ts">
import type { CrudProps } from '@/components/CRUD/hooks/useConfig.types'
import { definitions, fields, type UserInfo } from './crud-default'

definePage({
  path: '/example/default',
  meta: {
    title: 'Crud 默认',
    order: -2,
  },
})

const aribitaion: CrudProps<UserInfo> = {
  debug: true,
  url: '/user',
  rowKey: 'userId',
  apiEndpoints: {
    deleteFlag: 'delete',
  },
  source: { definitions, fields },
}
</script>

<template>
  <CRUD v-bind="aribitaion" />
</template>

<style scoped lang="scss">

</style>
