<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('协议项目')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="j-table-force-nowrap"
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <protocol-item-modal ref="modalForm" @ok="modalFormOk"/>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ProtocolItemModal from './modules/ProtocolItemModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'

  export default {
    name: "ProtocolItemList",
    mixins:[JeecgListMixin],
    components: {
      ProtocolItemModal
    },
    data () {
      return {
        description: '协议项目管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'项目名称',
            align:"center",
            dataIndex: 'name'
          },
          {
            title:'项目图片',
            align:"center",
            dataIndex: 'picture',
            scopedSlots: {customRender: 'imgSlot'}
          },
          {
            title:'协议项目分类',
            align:"center",
            dataIndex: 'protocolClass_dictText'
          },
          {
            title:'项目状态',
            align:"center",
            dataIndex: 'status',
            customRender: (text) => (!text ? "" : (text == "Y" ? "是" : "否"))
          },
          {
            title:'项目类别',
            align:"center",
            dataIndex: 'category_dictText'
          },
          {
            title:'上传附件',
            align:"center",
            dataIndex: 'description',
            scopedSlots: {customRender: 'fileSlot'}
          },
          {
            title:'项目到款情况',
            align:"center",
            dataIndex: 'getSituation_dictText'
          },
          {
            title:'项目到账金额',
            align:"center",
            dataIndex: 'getMoney'
          },
          {
            title:'项目到账时间',
            align:"center",
            dataIndex: 'getTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/protocolItem/protocolItem/list",
          delete: "/protocolItem/protocolItem/delete",
          deleteBatch: "/protocolItem/protocolItem/deleteBatch",
          exportXlsUrl: "/protocolItem/protocolItem/exportXls",
          importExcelUrl: "protocolItem/protocolItem/importExcel",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
      this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
         fieldList.push({type:'string',value:'name',text:'项目名称',dictCode:''})
         fieldList.push({type:'Text',value:'picture',text:'项目图片',dictCode:''})
         fieldList.push({type:'Text',value:'detail',text:'项目详情',dictCode:''})
         fieldList.push({type:'Text',value:'story',text:'捐赠故事',dictCode:''})
         fieldList.push({type:'Text',value:'question',text:'常见问题',dictCode:''})
         fieldList.push({type:'string',value:'protocolClass',text:'协议项目分类',dictCode:'protocol_class,name,id'})
         fieldList.push({type:'switch',value:'status',text:'项目状态'})
         fieldList.push({type:'int',value:'category',text:'项目类别',dictCode:'donation_category'})
         fieldList.push({type:'string',value:'description',text:'上传附件',dictCode:''})
         fieldList.push({type:'string',value:'getSituation',text:'项目到款情况',dictCode:'money_situation'})
         fieldList.push({type:'int',value:'getMoney',text:'项目到账金额',dictCode:''})
         fieldList.push({type:'datetime',value:'getTime',text:'项目到账时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>