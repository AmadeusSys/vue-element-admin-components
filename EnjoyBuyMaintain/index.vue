<template>
  <el-dialog  :title="tableTitle" :visible.sync="currentValue" :show-close="false" :before-close="handleClose" append-to-body center width="75%">
    <el-form ref="baseFormRef" :model="formData" :rules="formRules"  class="form-container" label-width="120px" v-loading="dataIsLoading" label-position="left">
      <slot></slot>
    </el-form>
    <slot name="ebFooter">
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitButtonOnClick" :disabled="dataIsLoading" v-waves>确 定</el-button>
        <el-button @click="currentValue = false" :disabled="dataIsLoading" v-waves>取 消</el-button>
      </div>
    </slot>
  </el-dialog>
</template>

<script>
import waves from '@/directive/waves' // 水波纹指令
import EventName from '../eventName'
import common from '@/utils/common'
export default {
  name: 'enjoy-buy-maintain',
  directives: {
    waves
  },
  data() {
    return {
      tableTitle: '',
      currentValue: false,
      formData: {},
      queryData: {},
      extendData: {},
      dataIsLoading: false
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
    /**
     * 表单主键
     */
    formPrimaryKey: {
      type: String,
      required: true
    },
    /**
     * 表单校验规则
     */
    formRules: {
      type: Object,
      required: true
    },
    /**
     * 表单名称
     * */
    formTitleText: {
      type: String,
      default: ''
    }
  },
  mounted() {
    window.fromBus.$on(EventName.SHOW_MAINTAIN_VIEW + this.tableName, (data, extendData) => {
      this.currentValue = true
      this.queryData = data
      this.extendData = extendData
      this.formData = {}
      this.$nextTick(() => {
        this.initMaintain()
        this.$refs['baseFormRef'].resetFields()
      })
    })
    window.fromBus.$on(EventName.HIDDEN_MAINTAIN_VIEW + this.tableName, () => {
      this.currentValue = false
    })
    window.fromBus.$on(EventName.LOADING_END + this.tableName, () => {
      this.dataIsLoading = false
    })
    window.fromBus.$on(EventName.LOADING_START + this.tableName, () => {
      this.dataIsLoading = true
    })
  },
  beforeDestroy() {
    window.fromBus.$off(EventName.SHOW_MAINTAIN_VIEW + this.tableName)
    window.fromBus.$off(EventName.HIDDEN_MAINTAIN_VIEW + this.tableName)
    window.fromBus.$off(EventName.LOADING_END + this.tableName)
    window.fromBus.$off(EventName.LOADING_START + this.tableName)
  },
  methods: {
    /**
     * 初始化维护信息
     * */
    initMaintain() {
      this.dataIsLoading = true
      if (this.queryData != null && common.isNotEmpty(this.queryData[this.formPrimaryKey])) {
        this.tableTitle = `${this.$t('form.edit')}${this.formTitleText}`
        this.$emit(EventName.FORM_INIT, this.queryData, this.slotInitComplete, this.extendData)
      } else {
        this.tableTitle = `${this.$t('form.add')}${this.formTitleText}`
        this.$emit(EventName.FORM_INIT, this.queryData, this.slotInitComplete, this.extendData)
      }
    },
    handleClose(done) {
      return false
    },
    slotInitComplete(formData = {}, isSuccess = true) {
      if (isSuccess) {
        this.formData = formData
        this.dataIsLoading = false
      } else {
        this.currentValue = false
      }
    },
    /**
     * 表单提交前置处理方法
     */
    beforeSubmitFrom() {
      this.dataIsLoading = true
      const submitData = {}
      /**
       * 获取验证数组中的值
       */
      for (const name in this.formRules) {
        submitData[name] = this.formData[name]
      }
      /**
       * 判断是否需要添加组件
       */
      if (common.isNotEmpty(this.formData[this.formPrimaryKey])) {
        submitData[this.formPrimaryKey] = this.formData[this.formPrimaryKey]
      }
      this.$emit(EventName.FORM_SUBMIT, submitData, this.afterSubmitFrom)
    },
    afterSubmitFrom(isSuccess = true) {
      this.dataIsLoading = false
      if (isSuccess) {
        this.currentValue = false
        window.fromBus.$emit(EventName.RE_LOADING_TABLE + this.tableName)
      }
    },
    /**
     * 提交按钮处理方法
     */
    submitButtonOnClick() {
      this.$refs['baseFormRef'].validate((valid) => {
        if (valid) {
          this.beforeSubmitFrom()
        } else {
          console.log('error submit!!', valid)
          return false
        }
      })
    }
  }
}
</script>
