<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="捐赠类型">
              <j-search-select-tag
                placeholder="请输入捐赠类别查询"
                v-model="queryParam.classId"
                dict="donation_class, name, id"
              >
              </j-search-select-tag>
              <!-- <j-input placeholder="请输入捐赠项目查询" v-model="queryParam.itemName"></j-input> -->
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="捐赠项目">
              <j-search-select-tag
                placeholder="请输入捐赠项目查询"
                v-model="queryParam.itemId"
                dict="donation_item, name, id"
              >
              </j-search-select-tag>
              <!-- <j-input placeholder="请输入捐赠项目查询" v-model="queryParam.itemName"></j-input> -->
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="捐赠时间">
              <j-date placeholder="请输入查询时间" v-model="queryParam.createTime" dateFormat="YYYY-MM-DD" />
            </a-form-item>
          </a-col>

          <span style="float: left; overflow: hidden" class="table-page-search-submitButtons">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </a-col>
          </span>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button type="primary" icon="download" @click="handleExportXls('捐赠订单')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <!-- <a-button type="primary" icon="import">导入</a-button> -->
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
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
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
          <a @click="handleDetail(record)">详情</a>
        </span>
      </a-table>
    </div>

    <donation-order-modal ref="modalForm" @ok="modalFormOk"></donation-order-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DonationOrderModal from './modules/DonationOrderModal'
import dayjs from 'dayjs'

export default {
  name: 'DonationOrderList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    DonationOrderModal,
  },
  data() {
    return {
      description: '捐赠订单管理页面',
      queryParam: {
        itemName: '',
        createTime: dayjs().format('YYYY-MM-DD'),
      },
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
          title: '订单编号',
          align: 'center',
          dataIndex: 'orderNo',
        },
        // {
        //   title: '第三方支付订单',
        //   align: 'center',
        //   dataIndex: 'thirdPayOrder',
        // },
        {
          title: '捐赠项目',
          align: 'center',
          dataIndex: 'itemName',
        },
        {
          title: '捐赠选项',
          align: 'center',
          dataIndex: 'optionId_dictText',
          customRender: function (text, record, index) {
            return text === '1' ? '自由捐' : text
          },
        },
        {
          title: '捐赠人姓名',
          align: 'center',
          dataIndex: 'name',
        },
        {
          title: '捐赠份数',
          align: 'center',
          dataIndex: 'piece',
        },
        {
          title: '捐赠总额',
          align: 'center',
          dataIndex: 'money',
        },
        {
          title: '电话',
          align: 'center',
          dataIndex: 'phone',
        },
        {
          title: '邮件',
          align: 'center',
          dataIndex: 'email',
        },
        {
          title: '捐赠时间',
          align: 'center',
          dataIndex: 'createTime',
        },
        {
          title: '部门',
          align: 'center',
          dataIndex: 'department_dictText',
        },
        {
          title: '支付方式',
          align: 'center',
          dataIndex: 'payMethod',
          customRender: function (text, record, index) {
            return text === '1' ? '微信' : '支付宝'
          },
        },
        // {
        //   title: '支付状态',
        //   align: 'center',
        //   dataIndex: 'status',
        // },
        {
          title: '是否校友',
          align: 'center',
          dataIndex: 'isSchoolmate',
          customRender: function (text, record, index) {
            return text === '1' ? '是' : '否'
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
        list: '/donationOrder/donationOrder/list',
        delete: '/donationOrder/donationOrder/delete',
        deleteBatch: '/donationOrder/donationOrder/deleteBatch',
        exportXlsUrl: '/donationOrder/donationOrder/exportXls',
        importExcelUrl: 'donationOrder/donationOrder/importExcel',
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
      fieldList.push({ type: 'string', value: 'orderNo', text: '订单编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'thirdPayOrder', text: '第三方支付订单', dictCode: '' })
      fieldList.push({ type: 'string', value: 'piece', text: '捐赠份数', dictCode: '' })
      fieldList.push({ type: 'int', value: 'isFreeDonation', text: '是否是自由捐', dictCode: '' })
      fieldList.push({ type: 'string', value: 'payMethod', text: '支付方式', dictCode: '' })
      fieldList.push({ type: 'int', value: 'status', text: '支付状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'message', text: '支付留言', dictCode: '' })
      fieldList.push({ type: 'int', value: 'money', text: '捐赠总额', dictCode: '' })
      fieldList.push({ type: 'date', value: 'cancelTime', text: '取消时间' })
      fieldList.push({ type: 'string', value: 'name', text: '捐赠人姓名', dictCode: '' })
      fieldList.push({ type: 'string', value: 'email', text: '邮件', dictCode: '' })
      fieldList.push({ type: 'string', value: 'department', text: '部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'phone', text: '电话', dictCode: '' })
      fieldList.push({ type: 'string', value: 'isSchoolmate', text: '是否校友', dictCode: '' })
      fieldList.push({ type: 'string', value: 'donationMsg', text: '捐赠留言', dictCode: '' })
      fieldList.push({ type: 'int', value: 'delFlag', text: '删除状态', dictCode: '' })
      this.superFieldList = fieldList
    },
  },
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>