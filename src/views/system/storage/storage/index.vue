<template>
  <cs-container>
    <page-header
      slot="header"
      :loading="loading"
      @submit="handleSubmit"
      ref="header"/>

    <page-main
      :loading="loading"
      :table-data="table"
      :navi-data="navi"
      :storage-id="storageId"
      @refresh="handleRefresh"
      @clearName="handleClearName"/>

    <page-footer
      slot="footer"
      :loading="loading"
      :current="page.current"
      :size="page.size"
      :total="page.total"
      @change="handlePaginationChange"/>
  </cs-container>
</template>

<script>
import { getStorageNavi, getStorageList } from '@/api/upload/storage'

export default {
  name: 'system-storage-storage',
  components: {
    'PageHeader': () => import('./components/PageHeader'),
    'PageMain': () => import('./components/PageMain'),
    'PageFooter': () => import('@/layout/header-aside/components/footer')
  },
  data() {
    return {
      loading: false,
      storageId: 0,
      table: [],
      navi: [],
      page: {
        current: 1,
        size: 0,
        total: 0
      }
    }
  },
  mounted() {
    this.$store.dispatch('careyshop/db/databasePage', { user: true })
      .then(res => {
        const size = res.get('size').value()
        this.page.size = size || 50
      })
      .finally(() => {
        this.handleSubmit()
      })
  },
  watch: {
    storageId: {
      handler(val) {
        getStorageNavi(val)
          .then(res => {
            this.navi = res.data || []
          })
      }
    }
  },
  methods: {
    // 刷新列表页面
    handleRefresh(storageId = null, isRestore = false) {
      this.$nextTick(() => {
        this.$refs.header.form['storage_id'] = storageId || 0
        this.$refs.header.handleFormSubmit(isRestore)
      })
    },
    // 分页变化改动
    handlePaginationChange(val) {
      this.page = val
      this.$nextTick(() => {
        this.$refs.header.handleFormSubmit()
      })
    },
    // 清除搜索名称
    handleClearName() {
      this.$refs.header.form.name = null
    },
    // 查询请求
    handleSubmit(form, isRestore = false) {
      if (isRestore) {
        this.page.current = 1
      }

      this.loading = true
      getStorageList({
        ...form,
        page_no: this.page.current,
        page_size: this.page.size
      })
        .then(res => {
          this.table = res.data.items || []
          this.page.total = res.data['total_result']
        })
        .finally(() => {
          this.loading = false
          this.storageId = form ? form.storage_id : 0
        })
    }
  }
}
</script>
