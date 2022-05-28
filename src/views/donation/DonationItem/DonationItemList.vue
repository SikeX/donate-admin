<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="捐赠项目分类">
              <j-dict-select-tag
                placeholder="请选择捐赠项目分类"
                v-model="queryParam.donationClass"
                dictCode="donation_class,name,id"
              />
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="项目状态">
              <j-dict-select-tag placeholder="请选择项目状态" v-model="queryParam.status" dictCode="donation_status" />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="项目类别">
                <j-dict-select-tag
                  placeholder="请选择项目类别"
                  v-model="queryParam.category"
                  dictCode="donation_category"
                />
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left; overflow: hidden" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('捐赠项目')">导出</a-button> -->
      <!-- <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="j-table-force-nowrap"
        :scroll="{ x: true }"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px; font-style: italic">无图片</span>
          <img
            v-else
            :src="getImgView(text)"
            height="25px"
            alt=""
            style="max-width: 80px; font-size: 12px; font-style: italic"
          />
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px; font-style: italic">无文件</span>
          <a-button v-else :ghost="true" type="primary" icon="download" size="small" @click="downloadFile(text)">
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

    <donation-item-modal ref="modalForm" @ok="modalFormOk" />
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DonationItemModal from './modules/DonationItemModal'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import '@/assets/less/TableExpand.less'

export default {
  name: 'DonationItemList',
  mixins: [JeecgListMixin],
  components: {
    DonationItemModal,
  },
  data() {
    return {
      description: '捐赠项目管理页面',
      roleId: [],
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function (t, r, index) {
            return parseInt(index) + 1
          },
        },
        {
          title: '项目名称',
          align: 'center',
          dataIndex: 'name',
        },
        {
          title: '捐赠项目分类',
          align: 'center',
          dataIndex: 'donationClass_dictText',
        },
        {
          title: '项目状态',
          align: 'center',
          dataIndex: 'status_dictText',
        },
        {
          title: '项目类别',
          align: 'center',
          dataIndex: 'category_dictText',
        },
        {
          title: '创建人',
          align: 'center',
          dataIndex: 'createBy',
        },
        {
          title: '创建日期',
          align: 'center',
          dataIndex: 'createTime',
        },
        {
          title: '所属部门',
          align: 'center',
          dataIndex: 'sysOrgCode_dictText',
        },
        {
          title: '项目图片',
          align: 'center',
          dataIndex: 'picture',
          scopedSlots: { customRender: 'imgSlot' },
        },

        {
          title: '目标金额',
          align: 'center',
          dataIndex: 'targetMoney',
        },
        {
          title: '已筹金额',
          align: 'center',
          dataIndex: 'raisedMoney',
        },

        {
          title: '起赠金额',
          align: 'center',
          dataIndex: 'leastMoney',
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' },
        },
      ],
      url: {
        list: '/donationItem/donationItem/list',
        delete: '/donationItem/donationItem/delete',
        deleteBatch: '/donationItem/donationItem/deleteBatch',
        exportXlsUrl: '/donationItem/donationItem/exportXls',
        importExcelUrl: 'donationItem/donationItem/importExcel',
      },
      dictOptions: {},
      superFieldList: [],
    }
  },
  created() {
    this.getSuperFieldList()
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    },
  },
  methods: {
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'createBy', text: '创建人', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'createTime', text: '创建日期' })
      fieldList.push({
        type: 'string',
        value: 'sysOrgCode',
        text: '所属部门',
        dictCode: 'sys_depart,org_code,depart_name',
      })
      fieldList.push({ type: 'string', value: 'name', text: '项目名称', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'picture', text: '项目图片', dictCode: '' })
      fieldList.push({ type: 'string', value: 'itemDesc', text: '项目简介', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'detail', text: '项目详情', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'story', text: '捐赠故事', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'question', text: '常见问题', dictCode: '' })
      fieldList.push({
        type: 'string',
        value: 'donationClass',
        text: '捐赠项目分类',
        dictCode: 'donation_class,name,id',
      })
      fieldList.push({ type: 'int', value: 'status', text: '项目状态', dictCode: 'donation_status' })
      fieldList.push({ type: 'string', value: 'targetMoney', text: '目标金额', dictCode: '' })
      fieldList.push({ type: 'string', value: 'raisedMoney', text: '已筹金额', dictCode: '' })
      fieldList.push({ type: 'int', value: 'category', text: '项目类别', dictCode: 'donation_category' })
      fieldList.push({ type: 'string', value: 'leastMoney', text: '起赠金额', dictCode: '' })
      this.superFieldList = fieldList
    },
  },
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>