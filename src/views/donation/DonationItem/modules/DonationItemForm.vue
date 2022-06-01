<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <!-- 主表单区域 -->
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="项目名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入项目名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目图片" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="picture">
              <j-image-upload v-model="model.picture"></j-image-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目简介" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="itemDesc">
              <a-input v-model="model.itemDesc" placeholder="请输入项目简介"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目详情" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detail">
              <j-editor v-if="formDisabled == false" v-model="model.detail" />
              <div v-if="formDisabled == true" v-html="model.detail"></div>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠故事" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="story">
              <j-editor v-if="formDisabled == false" v-model="model.story" />
              <div v-if="formDisabled == true" v-html="model.story"></div>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="常见问题" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="question">
              <j-editor v-if="formDisabled == false" v-model="model.question" />
              <div v-if="formDisabled == true" v-html="model.question"></div>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="捐赠项目分类" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="donationClass">
              <j-dict-select-tag
                type="list"
                v-model="model.donationClass"
                dictCode="donation_class,name,id"
                placeholder="请选择捐赠项目分类"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              v-if="roleId.indexOf('1473252071969705985') != -1"
              label="所属单位"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="donationClass"
            >
              <j-dict-select-tag
                type="list"
                v-model="model.sysOrgCode"
                dictCode="sys_depart,depart_name,org_code"
                placeholder="请选择项目所属部门"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="目标金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="targetMoney">
              <a-input v-model="model.targetMoney" placeholder="请输入目标金额"></a-input>
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24" >
            <a-form-model-item label="已筹金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="raisedMoney">
              <a-input v-model="model.raisedMoney" placeholder="请输入已筹金额" ></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <a-form-model-item label="起赠金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="leastMoney">
              <a-input v-model="model.leastMoney" placeholder="请输入起赠金额"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endTime">
              <j-date v-model="model.endTime" :showTime="true" dateFormat="YYYY-MM-DD HH:mm:ss" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="附件上传" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fileList">
              <j-upload v-model="model.file"></j-upload>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
    <!-- 子表单区域 -->
    <a-tabs v-model="activeKey" @change="handleChangeTabs">
      <a-tab-pane tab="捐赠选项" :key="refKeys[0]" :forceRender="true">
        <j-editable-table
          :ref="refKeys[0]"
          :loading="donationOptionTable.loading"
          :columns="donationOptionTable.columns"
          :dataSource="donationOptionTable.dataSource"
          :maxHeight="300"
          :disabled="formDisabled"
          :rowNumber="true"
          :rowSelection="true"
          :actionButton="true"
        />
      </a-tab-pane>
    </a-tabs>
  </a-spin>
</template>

<script>
import { getAction } from '@/api/manage'
import { FormTypes, getRefPromise, VALIDATE_NO_PASSED } from '@/utils/JEditableTableUtil'
import { JEditableTableModelMixin } from '@/mixins/JEditableTableModelMixin'
import { validateDuplicateValue } from '@/utils/util'
import { mapActions, mapGetters, mapState } from 'vuex'

export default {
  name: 'DonationItemForm',
  mixins: [JEditableTableModelMixin],
  components: {},
  data() {
    return {
      roleId: [],
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      labelCol2: {
        xs: { span: 24 },
        sm: { span: 3 },
      },
      wrapperCol2: {
        xs: { span: 24 },
        sm: { span: 20 },
      },
      model: {},
      // 新增时子表默认添加几行空数据
      addDefaultRowNum: 1,
      validatorRules: {
        name: [{ required: true, message: '请输入项目名称', trigger: 'blur' }],
        picture: [{ required: true, message: '请上传项目图片', trigger: 'blur' }],
        itemDesc: [{ required: true, message: '请输入项目描述', trigger: 'blur' }],
        detail: [{ required: true, message: '请输入项目详情', trigger: 'blur' }],
        story: [{ required: true, message: '请输入项目故事', trigger: 'blur' }],
        question: [{ required: true, message: '请输入项目问题', trigger: 'blur' }],
        donationClass: [{ required: true, message: '请选择捐赠项目分类', trigger: 'change' }],
        endTime: [{ required: true, message: '请选择结束时间', trigger: 'change' }],
        targetMoney: [
          { required: true },
          { pattern: /^(([1-9][0-9]*)|([0]\.\d{0,2}|[1-9][0-9]*\.\d{0,2}))$/, message: '请输入正确的金额!' },
        ],
        leastMoney: [
          { required: true },
          { pattern: /^(([1-9][0-9]*)|([0]\.\d{0,2}|[1-9][0-9]*\.\d{0,2}))$/, message: '请输入正确的金额!' },
        ],
      },
      refKeys: ['donationOption'],
      tableKeys: ['donationOption'],
      activeKey: 'donationOption',
      // 捐赠选项
      donationOptionTable: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '捐赠选项名字',
            key: 'name',
            type: FormTypes.input,
            width: '200px',
            placeholder: '请输入${title}',
            defaultValue: '',
          },
          {
            title: '选项金额',
            key: 'money',
            type: FormTypes.input,
            width: '200px',
            placeholder: '请输入${title}',
            defaultValue: '',
          },
        ],
      },
      url: {
        add: '/donationItem/donationItem/add',
        edit: '/donationItem/donationItem/edit',
        queryById: '/donationItem/donationItem/queryById',
        donationOption: {
          list: '/donationItem/donationItem/queryDonationOptionByMainId',
        },
      },
    }
  },
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false,
    },
  },
  computed: {
    formDisabled() {
      return this.disabled
    },
  },
  created() {
    console.log(this.userInfo())
    this.roleId = this.userInfo().roleId
  },
  methods: {
    ...mapGetters(['userInfo']),
    addBefore() {
      this.donationOptionTable.dataSource = []
    },
    getAllTable() {
      let values = this.tableKeys.map((key) => getRefPromise(this, key))
      return Promise.all(values)
    },
    /** 调用完edit()方法之后会自动调用此方法 */
    editAfter() {
      this.$nextTick(() => {})
      // 加载子表数据
      if (this.model.id) {
        let params = { id: this.model.id }
        this.requestSubTableData(this.url.donationOption.list, params, this.donationOptionTable)
        getAction(this.url.queryById, params).then((res) => {
          if (res.success) {
            this.model = res.result
            // console.log(model)
          }
        })
      }
    },
    //校验所有一对一子表表单
    validateSubForm(allValues) {
      return new Promise((resolve, reject) => {
        Promise.all([])
          .then(() => {
            resolve(allValues)
          })
          .catch((e) => {
            if (e.error === VALIDATE_NO_PASSED) {
              // 如果有未通过表单验证的子表，就自动跳转到它所在的tab
              this.activeKey = e.index == null ? this.activeKey : this.refKeys[e.index]
            } else {
              console.error(e)
            }
          })
      })
    },
    /** 整理成formData */
    classifyIntoFormData(allValues) {
      let main = Object.assign(this.model, allValues.formValue)
      return {
        ...main, // 展开
        donationOptionList: allValues.tablesValue[0].values,
      }
    },
    validateError(msg) {
      this.$message.error(msg)
    },
  },
}
</script>

<style scoped>
</style>