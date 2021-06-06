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
  <!--    <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('qwert_fc')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
 <!--     <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown> -->
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        row-key=“id”
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
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
<!--
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
-->
      </a-table>
    </div>
    <router-view :key="key"></router-view>

    <qwert-fc-modal ref="modalForm" @ok="modalFormOk"></qwert-fc-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { getAction } from '@/api/manage'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import QwertFcModal from './modules/QwertFcModal'

  export default {
    name: 'QwertFcList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      QwertFcModal
    },
    data () {
      return {
        loading: false,
        description: 'qwert_fc管理页面',
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
            title:'设备编号',
            align:"center",
            dataIndex: 'devNo'
          },
          {
            title:'数据点名称',
            align:"center",
            dataIndex: 'targetName'
          },
          {
            title:'数据点编号',
            align:"center",
            dataIndex: 'tagName'
          },
          {
            title:'点值',
            align:"center",
            dataIndex: 'pv'
          },
          {
            title:'时间',
            align:"center",
            dataIndex: 'time',
            customRender:function (text) {
              return !text?"":text
            }
          },

          /*          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          } */
        ],
        url: {
          list: "/jst/qwertFc/list",
          delete: "/jst/qwertFc/delete",
          deleteBatch: "/jst/qwertFc/deleteBatch",
          exportXlsUrl: "/jst/qwertFc/exportXls",
          importExcelUrl: "jst/qwertFc/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
      this.getSuperFieldList();
    },

//    mounted() {
//      this.loadData(this.$route.params.code);
//    },

    computed: {
      key() {
        this.loadData(this.$route.params.code);
        return this.$route.name !== undefined? this.$route.name +new Date(): this.$route +new Date()
      },
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      loadData(arg) {
        if(arg==null){
          return;
        }
        var code=this.$route.params.code;
        if(!this.url.list){
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        params.code=code;
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records;
//          if(!this._isMobile()){
            this.ipagination.total = res.result.total;
//          }
//            this.runflag=res.flag;
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },

//      watch: { //通过watch来监听路由变化
//        "$route": function() {
//          this.loadData(this.$route.params.code);
//        }
//      },
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'devNo',text:'设备编号',dictCode:''})
        fieldList.push({type:'string',value:'targetName',text:'数据点名称',dictCode:''})
        fieldList.push({type:'date',value:'time',text:'时间'})
        fieldList.push({type:'string',value:'tagName',text:'数据点编号',dictCode:''})
        fieldList.push({type:'double',value:'pv',text:'点值',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>