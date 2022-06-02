<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <!-- 主表单区域 -->
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="协议捐赠用户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="userId">
              <j-dict-select-tag
                type="list"
                v-model="model.userId"
                dictCode="online_user,realname,id,role_id='1473253380001157121'"
                placeholder="请选择协议捐赠用户"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入项目名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目图片" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="picture">
              <j-image-upload isMultiple v-model="model.picture"></j-image-upload>
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
            <a-form-model-item label="支出情况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cost">
              <j-editor v-if="formDisabled == false" v-model="model.cost" />
              <div v-if="formDisabled == true" v-html="model.cost"></div>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="协议项目分类" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="protocolClass">
              <j-dict-select-tag
                type="list"
                v-model="model.protocolClass"
                dictCode="protocol_class,name,id"
                placeholder="请选择协议项目分类"
              />
            </a-form-model-item>
          </a-col>
          <!--          <a-col :span="24" >
            <a-form-model-item label="项目状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <j-switch v-model="model.status"  ></j-switch>
            </a-form-model-item>
          </a-col>-->
          <!--          <a-col :span="24" >
            <a-form-model-item label="项目类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="category">
              <j-dict-select-tag type="list" v-model="model.category" dictCode="donation_category" placeholder="请选择项目类别" />
            </a-form-model-item>
          </a-col>-->
          <a-col :span="24">
            <a-form-model-item
              v-if="roleId.indexOf('1465163864583323650') == -1"
              label="所属单位"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="protocolClass"
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
            <a-form-model-item label="上传附件" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="description">
              <j-upload v-model="model.description"></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目到款情况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getSituation">
              <j-dict-select-tag
                type="list"
                v-model="model.getSituation"
                dictCode="money_situation"
                placeholder="请选择项目到款情况"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目到账金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getMoney">
              <a-input-number v-model="model.getMoney" placeholder="请输入项目到账金额" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目到账时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getTime">
              <j-date
                placeholder="请选择项目到账时间"
                v-model="model.getTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="项目结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endTime">
              <j-date
                placeholder="请选择项目结束时间"
                v-model="model.endTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { FormTypes, getRefPromise, VALIDATE_NO_PASSED } from '@/utils/JEditableTableUtil'
import { JEditableTableModelMixin } from '@/mixins/JEditableTableModelMixin'
import { validateDuplicateValue } from '@/utils/util'
import { mapGetters } from 'vuex'

export default {
  name: 'ProtocolItemForm',
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
        userId: [{ required: true, message: '请选择协议项目捐赠人', trigger: 'blur' }],
        name: [{ required: true, message: '请输入协议项目名称', trigger: 'blur' }],
      },
      refKeys: ['protocolOption'],
      tableKeys: ['protocolOption'],
      activeKey: 'protocolOption',
      // 协议选项表
      protocolOptionTable: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '协议选项名字',
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
        add: '/protocolItem/protocolItem/add',
        edit: '/protocolItem/protocolItem/edit',
        queryById: '/protocolItem/protocolItem/queryById',
        protocolOption: {
          list: '/protocolItem/protocolItem/queryProtocolOptionByMainId',
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

<style scoped>
</style>