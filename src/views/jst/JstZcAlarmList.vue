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
      <a-button type="primary" icon="download" @click="handleExportXls('jst_zc_alarm')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
      <template style="width: 30px">
        <a-tooltip v-if="runflag" :title="运行" style="margin-left:8px">
          <span>{{readCat}}</span><a-icon type="loading" style="margin-left:8px"/>
        </a-tooltip>
        <a-tooltip v-else title="中断" style="margin-left:8px">
          <span>{{readCat}}</span><a-icon type="exclamation-circle" style="margin-left:8px;color:red;"/>
        </a-tooltip>
      </template>
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
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"

        @change="handleTableChange">
        <!-- 字符串超长截取省略号显示-->
        <span slot="alarmValueRender" slot-scope="text">
          <j-ellipsis :value="text" :length="20" />
        </span>

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

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
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

    <jstZcAlarm-modal ref="modalForm" @ok="modalFormOk"></jstZcAlarm-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JstZcAlarmModal from './modules/JstZcAlarmModal'
  import { getAction } from '@/api/manage'

  export default {
    name: "JstZcAlarmList",
    mixins:[JeecgListMixin],
    components: {
      JstZcAlarmModal
    },
    data () {
      return {
        description: 'jst_zc_alarm管理页面',
        runflag: false,
        readCat: null,
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
            title:'设备名称',
            align:"center",
            dataIndex: 'devName'
          },
  /*        {
            title:'数据点编号',
            align:"center",
            dataIndex: 'targetNo'
          },*/
          {
            title:'报警值',
            align:"center",
            width: 250,
            dataIndex: 'alarmValue',
            scopedSlots: {customRender: 'alarmValueRender'}
          },
          {
            title:'报警级别',
            align:"center",
            dataIndex: 'alarmLevel'
          },
          {
            title:'报警状态',
            align:"center",
            dataIndex: 'sendType_dictText'
          },
          {
            title:'报警时间',
            align:"center",
            dataIndex: 'sendTime',
  //          sorter: (a, b) => {
  //            return a.sendTime.localeCompare(b.sendTime)
  //          },
          },
/*          {
            title:'处理状态',
            align:"center",
            dataIndex: 'dealType'
          },
          {
            title:'上报类型',
            align:"center",
            dataIndex: 'reportType'
          },  */
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/jst/jstZcAlarm/list?column=send_time&order=desc",
          delete: "/jst/jstZcAlarm/delete",
          deleteBatch: "/jst/jstZcAlarm/deleteBatch",
          exportXlsUrl: "/jst/jstZcAlarm/exportXls",
          importExcelUrl: "jst/jstZcAlarm/importExcel",
        },
        dictOptions:{},
      }
    },
    created() {
//      if(!this.disableMixinCreated){
        console.log(' -- mixin created -- ')
//        this.loadData();
        //初始化字典配置 在自己页面定义
        this.initDictConfig();
//      }
    },
    mounted(){

      const timer = setInterval(() =>{
        this.loadData();
      }, 30000);
// 通过$once来监听定时器
// 在beforeDestroy钩子触发时清除定时器
      this.$once('hook:beforeDestroy', () => {
        clearInterval(timer);
      })
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      loadData(arg) {
        if(!this.url.list){
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records;
//          if(!this._isMobile()){
            this.ipagination.total = res.result.total;
//          }
            var tmpres=res.extMessage.split(",");
            if(tmpres[1]=="true"){
              this.runflag=true;
              this.readCat=tmpres[0];
            }else{
              this.runflag=false;
              this.readCat="停止";
            }
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },
      initDictConfig(){
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>