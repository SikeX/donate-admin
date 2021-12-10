<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <!-- 主表单区域 -->
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24" >
            <a-form-model-item label="项目名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入项目名称" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目图片" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="picture">
              <j-image-upload isMultiple  v-model="model.picture" ></j-image-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目详情" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detail">
              <j-editor v-model="model.detail" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="捐赠故事" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="story">
              <j-editor v-model="model.story" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="常见问题" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="question">
              <j-editor v-model="model.question" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="协议项目分类" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="protocolClass">
              <j-dict-select-tag type="list" v-model="model.protocolClass" dictCode="protocol_class,name,id" placeholder="请选择协议项目分类" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <j-switch v-model="model.status"  ></j-switch>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="category">
              <j-dict-select-tag type="list" v-model="model.category" dictCode="donation_category" placeholder="请选择项目类别" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
      <!-- 子表单区域 -->
    <a-tabs v-model="activeKey" @change="handleChangeTabs">
      <a-tab-pane tab="协议选项表" :key="refKeys[0]" :forceRender="true">
        <j-editable-table
          :ref="refKeys[0]"
          :loading="protocolOptionTable.loading"
          :columns="protocolOptionTable.columns"
          :dataSource="protocolOptionTable.dataSource"
          :maxHeight="300"
          :disabled="formDisabled"
          :rowNumber="true"
          :rowSelection="true"
          :actionButton="true"/>
      </a-tab-pane>
    </a-tabs>
  </a-spin>
</template>

<script>

  import { getAction } from '@/api/manage'
  import { FormTypes,getRefPromise,VALIDATE_NO_PASSED } from '@/utils/JEditableTableUtil'
  import { JEditableTableModelMixin } from '@/mixins/JEditableTableModelMixin'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'ProtocolItemForm',
    mixins: [JEditableTableModelMixin],
    components: {
    },
    data() {
      return {
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
        model:{
        },
        // 新增时子表默认添加几行空数据
        addDefaultRowNum: 1,
        validatorRules: {
        },
        refKeys: ['protocolOption', ],
        tableKeys:['protocolOption', ],
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
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue:'',
            },
            {
              title: '选项金额',
              key: 'money',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue:'',
            },
          ]
        },
        url: {
          add: "/protocolItem/protocolItem/add",
          edit: "/protocolItem/protocolItem/edit",
          queryById: "/protocolItem/protocolItem/queryById",
          protocolOption: {
            list: '/protocolItem/protocolItem/queryProtocolOptionByMainId'
          },
        }
      }
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
    },
    methods: {
      addBefore(){
        this.protocolOptionTable.dataSource=[]
      },
      getAllTable() {
        let values = this.tableKeys.map(key => getRefPromise(this, key))
        return Promise.all(values)
      },
      /** 调用完edit()方法之后会自动调用此方法 */
      editAfter() {
        this.$nextTick(() => {
        })
        // 加载子表数据
        if (this.model.id) {
          let params = { id: this.model.id }
          this.requestSubTableData(this.url.protocolOption.list, params, this.protocolOptionTable)
        }
      },
      //校验所有一对一子表表单
      validateSubForm(allValues){
          return new Promise((resolve,reject)=>{
            Promise.all([
            ]).then(() => {
              resolve(allValues)
            }).catch(e => {
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
          protocolOptionList: allValues.tablesValue[0].values,
        }
      },
      validateError(msg){
        this.$message.error(msg)
      },

    }
  }
</script>

<style scoped>
</style>