<script setup lang="ts">
import zhCn from 'element-plus/es/locale/lang/zh-cn'
import { routes } from 'vue-router/auto-routes'

import { router } from './router'

console.log('【APP ~ routes】\n', router.options.routes, '\n', routes)
</script>

<template>
  <ElConfigProvider
    :locale="zhCn"
    :button="{ autoInsertSpace: true }"
    :message="{ max: 3 }"
    :value-on-clear="() => undefined"
    :empty-values="[undefined, null]"
  >
    <RouterView />
  </ElConfigProvider>
</template>

<style lang="scss">
/** el_table 奇偶行不同的颜色 */
.el-table__body {
  --table_row_level-1: #c6e2ff;
  --table_row_level-2: #d1edc4;
}
.dark {
  .el-table__body {
    --table_row_level-1: #213d5b;
    --table_row_level-2: #2d481f;
  }
}

.el-link + .el-link {
  margin-left: 12px;
}
</style>
