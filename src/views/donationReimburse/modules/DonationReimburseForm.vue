<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="报销类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reimburseCategory">
              <j-dict-select-tag
                type="list"
                v-model="model.reimburseCategory"
                dictCode="reimburse_category"
                placeholder="请选择报销类别"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报销详情" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detail">
              <a-textarea v-model="model.detail" rows="4" placeholder="请输入报销详情" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报销金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="money">
              <a-input v-model="model.money" placeholder="请输入报销金额"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="附件" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="file">
              <j-upload v-model="model.file"></j-upload>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'

export default {
  name: 'DonationOrderForm',
  components: {},
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false,
    },
  },
  data() {
    return {
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      confirmLoading: false,
      validatorRules: {
        reimburseCategory: [{ required: true, message: '请输入报销类别!' }],
        detail: [{ required: true, message: '请输入报销详情!' }],
        money: [
          { required: true, message: '请输入报销金额!' },
          { pattern: /^(([1-9][0-9]*)|([0]\.\d{0,2}|[1-9][0-9]*\.\d{0,2}))$/, message: '请输入正确的金额!' },
        ],
      },
      url: {
        add: '/donationItem/donationItem/addDonationReimburse',
        edit: '/donationItem/donationItem/editDonationReimburse',
        queryById: '/donationItem/donationItem/queryReimburseById',
      },
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    },
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
  },
  methods: {
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.editAfter(record)
    },
    editAfter() {
      this.$nextTick(() => {})
      // 加载子表数据
      if (this.model.id) {
        let params = { id: this.model.id }
        getAction(this.url.queryById, params).then((res) => {
          if (res.success) {
            this.model = res.result
            // console.log(model)
          }
        })
      }
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate((valid) => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          httpAction(httpurl, this.model, method)
            .then((res) => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
            })
        }
      })
    },
  },
}
</script>