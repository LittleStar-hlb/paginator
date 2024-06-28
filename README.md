# 分页器(vue3)

### 开始

```vue
<script lang="ts">
// 引入注册
import Pagination from '...'
export default {
  components: {
    Pagination,
  },
  setup() {
    ...

    return {}
  },
  methods: {
    currentPage(page: number) {
      ...
    },
  },
},
</script>

<template>
  <!-- page-count: 总页码(number) -->
  <!-- pager-count: 按钮数(number),默认为7-->
  <!-- current-page: 当前页码回调(function),点击页码触发 -->
  <Pagination
    :page-count="pageCount"
    :pager-count="4"
    @current-page="currentPage"
  >
  </Pagination>
</template>
```
