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
            <a-form-model-item label="支出情况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cost">
              <j-editor v-model="model.cost" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="协议项目分类" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="protocolClass">
              <j-dict-select-tag type="list" v-model="model.protocolClass" dictCode="protocol_class,name,id" placeholder="请选择协议项目分类" />
            </a-form-model-item>
          </a-col>
<!--          <a-col :span="24">
            <a-form-model-item label="项目状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <a-input-number v-model="model.status" placeholder="请输入项目状态" style="width: 100%" />
            </a-form-model-item>
          </a-col>-->
          <a-col :span="24">
            <a-form-model-item label="所属部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sysOrgCode">
              <a-input v-model="model.sysOrgCode" placeholder="请输入所属部门"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="协议项目描述" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="protocolItemDesc">
              <a-input v-model="model.protocolItemDesc" placeholder="请输入协议项目描述"  ></a-input>
            </a-form-model-item>
          </a-col>
<!--          <a-col :span="24" >
            <a-form-model-item label="项目类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="category">
              <j-dict-select-tag type="list" v-model="model.category" dictCode="donation_category" placeholder="请选择项目类别" />
            </a-form-model-item>
          </a-col>-->
          <a-col :span="24" >
            <a-form-model-item label="上传附件" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="description">
              <j-upload v-model="model.description"  ></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目到款情况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getSituation">
              <j-dict-select-tag type="list" v-model="model.getSituation" dictCode="money_situation" placeholder="请选择项目到款情况" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目到账金额" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getMoney">
              <a-input-number v-model="model.getMoney" placeholder="请输入项目到账金额" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" >
            <a-form-model-item label="项目到账时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="getTime">
              <j-date placeholder="请选择项目到账时间" v-model="model.getTime" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
<!--      &lt;!&ndash; 子表单区域 &ndash;&gt;
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
    </a-tabs>-->
  </a-spin>
</template>

<script>



  import { httpAction } from "../../../../api/manage";

  export default {
    name: 'ProtocolItemForm',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data() {
      return {
        model:{
        },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 6 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/item/protocolItem/add",
          edit: "/item/protocolItem/edit",
          queryById: "/item/protocolItem/queryById",
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
      //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }

        })
      },

    }
  }
</script>

<style scoped>
</style>