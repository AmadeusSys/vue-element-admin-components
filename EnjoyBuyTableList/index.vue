<template>
  <div class="app-container">

    <div class="filter-container">
      <slot name="header">
        <el-button class="filter-item" style="margin-left: 10px;" @click="AddMaintainRecord" type="primary" icon="el-icon-plus" v-waves>{{$t('table.add')}}</el-button>
      </slot>
    </div>

    <el-table :data="tableListData" v-loading="listIsLoading" @row-click="rowClick" border fit highlight-current-row style="width: 100%" :row-key="rowKeyHandle">
      <slot></slot>
    </el-table>

    <div class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentListQueryPage" :page-sizes="[25,50]" :page-size="listQueryPageSize" layout="total, sizes, prev, pager, next, jumper" :page-count="totalPage">
      </el-pagination>
    </div>

  </div>
</template>

<script>
import waves from '@/directive/waves' // 水波纹指令
import EventName from '../eventName'
export default {
  name: 'enjoy-buy-table-list',
  directives: {
    waves
  },
  data() {
    return {
      listQueryPageSize: 25,
      listIsLoading: true,
      tableListData: [],
      totalPage: 0
    }
  },
  props: {
    /**
     * 表格名
     */
    tableName: {
      type: String,
      default: ''
    },
    rowKey: String
  },
  mounted() {
    this.loadingTableData()
    window.fromBus.$on(EventName.RE_LOADING_TABLE + this.tableName, (zero) => {
      if (zero) {
        this.currentListQueryPage = 1
      }
      this.loadingTableData()
    })
  },
  beforeDestroy() {
    window.fromBus.$off(EventName.RE_LOADING_TABLE)
  },
  methods: {
    rowKeyHandle(row) {
      if (this.rowKey != null) {
        return row[this.rowKey]
      }
    },
    rowClick(row, event, column) {
      this.$emit('eb-row-click', row, event, column)
    },
    AddMaintainRecord() {
      window.fromBus.$emit(EventName.SHOW_MAINTAIN_VIEW + this.tableName)
    },
    loadingTableData() {
      this.listIsLoading = true
      this.$emit(EventName.LOADING_TABLE_DATA, this)
    },
    loadTableData(response) {
      this.tableListData = response['data']
      this.totalPage = response['recordsTotalPage']
      this.listIsLoading = false
    },
    handleSizeChange(val) {
      this.currentListQueryPage = 1
      this.listQueryPageSize = val
      this.loadingTableData()
    },
    handleCurrentChange(val) {
      this.currentListQueryPage = val
      this.loadingTableData()
    }
  },
  computed: {
    currentListQueryPage: {
      cache: false,
      get() {
        const path = this.$route.path
        const searchHistory = sessionStorage.getItem(this.tableName + path + '_list_query_page')
        return parseInt(searchHistory) || 1
      },
      set(val) {
        const path = this.$route.path
        sessionStorage.setItem(this.tableName + path + '_list_query_page', val)
      }
    }
  }

}
</script>

<style scoped>

</style>
