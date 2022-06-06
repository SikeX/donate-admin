<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('donation_item')">导出</a-button>
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
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio' }"
        :customRow="clickThenSelect"
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

    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="项目报销" key="1">
        <DonationReimburseList :mainId="selectedMainId" />
      </a-tab-pane>
    </a-tabs>

    <donationItem-modal ref="modalForm" @ok="modalFormOk"></donationItem-modal>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DonationItemModal from '../donation/DonationItem/modules/DonationItemModal'
import { getAction } from '@/api/manage'
import DonationReimburseList from './DonationReimburseList'
import '@/assets/less/TableExpand.less'

export default {
  name: 'DonationItemList',
  mixins: [JeecgListMixin],
  components: {
    DonationReimburseList,
    DonationItemModal,
  },
  data() {
    return {
      description: 'donation_item管理页面',
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
        list: '/donationItem/donationItem/listVerified',
        delete: '/donationItem/donationItem/delete',
        deleteBatch: '/donationItem/donationItem/deleteBatch',
        exportXlsUrl: '/donationItem/donationItem/exportXls',
        importExcelUrl: 'donationItem/donationItem/importExcel',
      },
      dictOptions: {},
      /* 分页参数 */
      ipagination: {
        current: 1,
        pageSize: 5,
        pageSizeOptions: ['5', '10', '50'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0,
      },
      selectedMainId: '',
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
    clickThenSelect(record) {
      return {
        on: {
          click: () => {
            this.onSelectChange(record.id.split(','), [record])
          },
        },
      }
    },
    onClearSelected() {
      this.selectedRowKeys = []
      this.selectionRows = []
      this.selectedMainId = ''
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedMainId = selectedRowKeys[0]
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    loadData(arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      this.onClearSelected()
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction(this.url.list, params).then((res) => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
        if (res.code === 510) {
          this.$message.warning(res.message)
        }
        this.loading = false
      })
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