<template>
  <a-card :bordered="false">
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
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"

        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>
      </a-table>
    </div>

  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
//  import JstZcTargetModal from './modules/JstZcTargetModal'

  export default {
    name: "JstZcCatTargetModel",
    mixins:[JeecgListMixin],
    components: {
//      JstZcTargetModal
    },
    data () {
      return {
        model: {},
        description: 'jst_zc_cat_target查询页面',
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
            title:'设备类型',
            align:"center",
            dataIndex: 'devType'
          },
          {
            title:'指标编码',
            align:"center",
            dataIndex: 'targetNo'
          },
          {
            title:'指标名称',
            align:"center",
            dataIndex: 'targetName'
          },
/*          {
            title:'单位',
            align:"center",
            dataIndex: 'unit'
          },
*/
          {
            title:'协议',
            align:"center",
            dataIndex: 'proType'
          },
          {
            title:'寄存器地址',
            align:"center",
            dataIndex: 'address'
          },
          {
            title:'地址类型',
            align:"center",
            dataIndex: 'addressType_dictText'
          },
/*          {
            title:'指令',
            align:"center",
            dataIndex: 'instruct'
          }, */
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/jst/jstZcTarget/list?column=dev_type&order=asc",
          delete: "/jst/jstZcTarget/delete",
          deleteBatch: "/jst/jstZcTarget/deleteBatch",
          exportXlsUrl: "/jst/jstZcTarget/exportXls",
          importExcelUrl: "jst/jstZcTarget/importExcel",
        },
        dictOptions:{},
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
      },
      modalFormOk2() {
        // 新增/修改 成功时，重载列表
        this.loadData();
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>