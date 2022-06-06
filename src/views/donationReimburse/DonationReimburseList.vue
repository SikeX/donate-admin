<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">
    <!-- 操作按钮区域 -->
    <div class="table-operator" v-if="mainId">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('项目报销')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
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
        :scroll="{ x: true }"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
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
          <a @click="handleDetail(record)">查看</a>
        </span>
      </a-table>
    </div>

    <donationReimburse-modal ref="modalForm" @ok="modalFormOk" :mainId="mainId"></donationReimburse-modal>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DonationReimburseModal from './modules/DonationReimburseModal'

export default {
  name: 'DonationReimburseList',
  mixins: [JeecgListMixin],
  components: { DonationReimburseModal },
  props: {
    mainId: {
      type: String,
      default: '',
      required: false,
    },
  },
  watch: {
    mainId: {
      immediate: true,
      handler(val) {
        if (!this.mainId) {
          this.clearList()
        } else {
          this.queryParam['itemId'] = val
          this.loadData(1)
        }
      },
    },
  },
  data() {
    return {
      description: 'donation_item管理页面',
      disableMixinCreated: true,
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
          title: '报销类别',
          align: 'center',
          dataIndex: 'reimburseCategory_dictText',
        },
        {
          title: '报销详情',
          align: 'center',
          dataIndex: 'detail',
        },
        {
          title: '报销金额',
          align: 'center',
          dataIndex: 'money',
        },
        {
          title: '审核状态',
          align: 'center',
          dataIndex: 'status',
          customRender: function (text) {
            if (text == 1) {
              return '通过'
            } else if (text == 2) {
              return '待审核'
            } else if (text == 0) {
              return '驳回'
            }
          },
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
        list: '/donationItem/donationItem/listDonationReimburseByMainId',
        delete: '/donationItem/donationItem/deleteDonationReimburse',
        deleteBatch: '/donationItem/donationItem/deleteBatchDonationReimburse',
        exportXlsUrl: '/donationItem/donationItem/exportDonationReimburse',
        importUrl: '/donationItem/donationItem/importDonationReimburse',
      },
      dictOptions: {},
      superFieldList: [],
    }
  },
  created() {
    this.getSuperFieldList()
  },
  computed: {
    importExcelUrl() {
      return `${window._CONFIG['domianURL']}/${this.url.importUrl}/${this.mainId}`
    },
  },
  methods: {
    clearList() {
      this.dataSource = []
      this.selectedRowKeys = []
      this.ipagination.current = 1
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'name', text: '项目名称', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'picture', text: '项目图片', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'itemDesc', text: '项目简介', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'detail', text: '项目详情', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'story', text: '捐赠故事', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'question', text: '常见问题', dictCode: '' })
      fieldList.push({ type: 'string', value: 'donationClass', text: '捐赠项目分类', dictCode: '' })
      fieldList.push({ type: 'int', value: 'delFlag', text: '删除状态', dictCode: '' })
      fieldList.push({ type: 'int', value: 'status', text: '项目状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'targetMoney', text: '目标金额', dictCode: '' })
      fieldList.push({ type: 'string', value: 'raisedMoney', text: '已筹金额', dictCode: '' })
      fieldList.push({ type: 'int', value: 'category', text: '项目类别', dictCode: '' })
      fieldList.push({ type: 'string', value: 'leastMoney', text: '起赠金额', dictCode: '' })
      fieldList.push({ type: 'Text', value: 'file', text: '项目文件', dictCode: '' })
      fieldList.push({ type: 'date', value: 'endTime', text: '结束时间' })
      this.superFieldList = fieldList
    },
  },
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
