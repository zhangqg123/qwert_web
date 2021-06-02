<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="单位用户">
              <j-dict-select-tag v-model="queryParam.orgUser" @change="handleChangeOrgUser" :triggerChange="true" placeholder="请选择单位" dictCode="jst_zc_orguser,orguser_name,orguser_id"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备位置">
              <j-dict-select-tag v-model="queryParam.devPos" placeholder="请选择位置" :dictCode="dictCode4.val"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备类型">
              <j-dict-select-tag v-model="queryParam.devCat" placeholder="请选择分类" :dictCode="dictCode2.val" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
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
      <a-button type="primary" icon="download" @click="handleExportXls('jst_zc_dev')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-button @click="handleRead" :loading="loading" type="default" icon="plus">读取</a-button>
      <a-button @click="handleClose" type="default" icon="plus">结束</a-button>
<!--      <a-button @click="handleAmq" type="default" icon="plus">队列</a-button> -->
<!--      <p style="text-align: center" v-if="!stepRunning">监测停止</p>
      <p style="text-align: center" v-else>正在监测...</p>  -->
      <a-tooltip v-if="runflag" :title="运行" style="margin-left:8px">
        <a-icon type="loading"/>
      </a-tooltip>
      <a-tooltip v-else title="中断" style="margin-left:8px">
        <a-icon type="exclamation-circle" style="color:red;"/>
      </a-tooltip>

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

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a @click="handleTest(record)">调试</a>
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

    <jstZcDev-modal ref="modalForm" @ok="modalFormOk"></jstZcDev-modal>
    <select-cat-target-list-modal ref="CatTargetListModal" @choseUser="choseUser"></select-cat-target-list-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getAction } from '@/api/manage'
  import JstZcDevModal from './modules/JstZcDevModal'
  import { multipleRead, readClose,readAmq } from '@/api/api';
  import pick from 'lodash.pick'
  import SelectCatTargetListModal from './modules/SelectCatTargetListModal'

  export default {
    name: "JstZcDevList",
    mixins:[JeecgListMixin],
    components: {
      JstZcDevModal,
      SelectCatTargetListModal
    },
    data () {
      return {
        loading: false,
        runflag: false,
        description: 'jst_zc_dev管理页面',
        dictCode2: {
          val:"jst_zc_cat,zc_catname,origin_id ,has_child ='0'&&id>'002' order by pid "
        },        // 表头
        dictCode4: {
          val:"jst_zc_position,pos_name,pos_id "
        },        // 表头
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
            title:'设备名称',
            align:"center",
            dataIndex: 'devName'
          },
          {
            title:'设备分类',
            align:"center",
            dataIndex: 'devCat_dictText'
          },
          {
            title:'设备编号',
            align:"center",
            dataIndex: 'devNo'
          },
          {
            title:'位置',
            align:"center",
            dataIndex: 'devPos_dictText'
          },
          {
            title:'详细位置',
            align:"center",
            dataIndex: 'position'
          },
          {
            title:'状态',
            align:"center",
            dataIndex: 'status_dictText'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/jst/jstZcDev/list?column=dev_no&order=asc",
          delete: "/jst/jstZcDev/delete",
          deleteBatch: "/jst/jstZcDev/deleteBatch",
          exportXlsUrl: "/jst/jstZcDev/exportXls",
          importExcelUrl: "jst/jstZcDev/importExcel",
          testUrl:"/jst/jstZcDev/conntest",
  //        readUrl:"/jst/jstZcDev/handleRead"
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
            this.runflag=res.flag;
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },
      initDictConfig(){
      },
      handleTest (record) {
        this.$refs.CatTargetListModal.add(record);
      },
      // 子modal回调
      choseUser:function(){
      },
      handleRead(){
        this.loading = true;
        const that = this;
 //       let httpurl = '';
        let params=[];
          params.devCat=this.queryParam.devCat;
        if(params.devCat==null){
          this.$message.error("请选择设备分类!");
          this.loading = false;
          return;
        }
        that.$message.success("开始读取数据");
        multipleRead(params).then((res)=>{
          if(res.success){
  //          that.$message.success("开始读取数据");
            that.runflag=res.flag;
          }
        }).finally(() => {
          that.loading = false
        })
      },
      handleClose(){
 //       this.loading = true;
        const that = this;
        let params=[];
        readClose(params).then((res)=>{
          if(res.success){
            that.$message.error("中断连接");
            that.runflag=false;
          }
        }).finally(() => {
 //         that.loading = false
        })

      },
      handleAmq(){
        const that = this;
        let params=[];
        readAmq(params).then((res)=>{
          if(res.success){
            that.$message.success("运行队列");
          }
        }).finally(() => {
          //         that.loading = false
        })
      },
      handleChangeOrgUser(value){
        var tmp=null;
        var tmp2=null;
        if(value!=null){
          tmp = "jst_zc_cat,zc_catname,origin_id,org_user= '"+value+"'&&has_child ='0'&&id>'002' order by pid ";
          tmp2 = "jst_zc_position,pos_name,pos_id,org_user= '"+value+"'";
        }else{
          tmp = "jst_zc_cat,zc_catname,origin_id,has_child ='0'&&id>'002' order by pid ";
          tmp2 = "jst_zc_position,pos_name,pos_id ";
        }
        this.queryParam.orgUser=value;
//        this.queryParam.devCat='';
        this.$set(this.dictCode2, "val", tmp);
        this.$set(this.dictCode4, "val", tmp2);
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>