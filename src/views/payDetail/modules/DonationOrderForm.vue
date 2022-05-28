<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="订单编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="orderNo">
              <a-input v-model="model.orderNo" placeholder="请输入订单编号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="第三方支付订单"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="thirdPayOrder"
            >
              <a-input v-model="model.thirdPayOrder" placeholder="请输入第三方支付订单"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠项目" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="thirdPayOrder">
              <a-input v-model="model.itemName"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠选项" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="thirdPayOrder">
              <a-input v-model="model.optionId_dictText"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠份数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="piece">
              <a-input v-model="model.piece" placeholder="请输入捐赠份数"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="是否是自由捐" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="isFreeDonation">
              <a-input-number v-model="model.isFreeDonation" placeholder="请输入是否是自由捐" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="支付方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="payMethod">
              <a-input v-model="model.payMethod" placeholder="请输入支付方式"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="支付状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <a-input-number v-model="model.status" placeholder="请输入支付状态" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="支付留言" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="message">
              <a-input v-model="model.message" placeholder="请输入支付留言"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠总额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="money">
              <a-input-number v-model="model.money" placeholder="请输入捐赠总额" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="取消时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cancelTime">
              <j-date placeholder="请选择取消时间" v-model="model.cancelTime" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠人姓名" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入捐赠人姓名"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="邮件" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="email">
              <a-input v-model="model.email" placeholder="请输入邮件"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="department">
              <a-input v-model="model.department" placeholder="请输入部门"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="phone">
              <a-input v-model="model.phone" placeholder="请输入电话"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="是否校友" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="isSchoolmate">
              <a-input v-model="model.isSchoolmate" placeholder="请输入是否校友"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠留言" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="donationMsg">
              <a-input v-model="model.donationMsg" placeholder="请输入捐赠留言"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="删除状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="delFlag">
              <a-input-number v-model="model.delFlag" placeholder="请输入删除状态" style="width: 100%" />
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
      validatorRules: {},
      url: {
        add: '/donationOrder/donationOrder/add',
        edit: '/donationOrder/donationOrder/edit',
        queryById: '/donationOrder/donationOrder/queryById',
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