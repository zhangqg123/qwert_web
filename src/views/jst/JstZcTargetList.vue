<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
   <!--       <a-col :md="6" :sm="8">
            <a-form-item label="指标名称">
              <j-input placeholder="请输入指标名称" v-model="queryParam.targetName"></j-input>
            </a-form-item>
          </a-col> -->
          <a-col :md="4" :sm="6">
            <a-form-item label="用户">
              <j-dict-select-tag @change="handleChangeOrgUser" dictCode="jst_zc_orguser,orguser_name,orguser_id" v-model="queryParam.orgUser" :triggerChange="true" placeholder="请输入单位" />
            </a-form-item>
          </a-col>
          <a-col :md="4" :sm="8">
            <a-form-item label="报警点">
              <!--            <a-input placeholder="请输入模型编号" v-model="queryParam.devicemodel"></a-input>  -->
              <j-dict-select-tag v-model="queryParam.alarmPoint" placeholder="请选择报警点" dictCode="alarm_point"/>
            </a-form-item>
          </a-col>
          <a-col :md="5" :sm="8">
            <a-form-item label="类型">
              <j-dict-select-tag @change="handleChangeType" :triggerChange="true" v-model="queryParam.devType" placeholder="请选择分类" :dictCode="dictCode2.val" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备">
              <j-dict-select-tag v-model="queryParam.devNo" placeholder="请选择名称"  :dictCode="dictCode3.val"/>
            </a-form-item>
          </a-col>

          <a-col :md="4" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
         <!--     <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>-->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('jst_zc_target')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
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
        <div slot="filterDropdown">
          <a-card>
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row style="width: 400px">
                <template v-for="(item,index) in defColumns">
                  <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                    <a-col :span="12"><a-checkbox :value="item.dataIndex"><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type='setting' :style="{ fontSize:'16px',color:  '#108ee9' }" />

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

    <jstZcTarget-modal ref="modalForm" @ok="modalFormOk"></jstZcTarget-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JstZcTargetModal from './modules/JstZcTargetModal'
  import JInput from '@/components/jeecg/JInput.vue';
  import { deleteAction, getAction,downFile,getFileAccessHttpUrl } from '@/api/manage'
  import Vue from 'vue'

  export default {
    name: "JstZcTargetList",
    mixins:[JeecgListMixin],
    components: {
      JstZcTargetModal,JInput
    },
    data () {
      return {
        model: {},
        description: 'jst_zc_target管理页面',
        dictCode2: {
          val:"jst_zc_cat,zc_catname,origin_id ,has_child ='0'&&id>'002' order by pid "
        },        // 表头
        dictCode3: {
          val:"jst_zc_dev,dev_name,dev_no,org_user='guangfa'"
        },        // 表头
      //表头
        columns:[],
        //列设置
        settingColumns:[],
        // 表头
        defColumns: [
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
            dataIndex: 'devType_dictText'
          },
/*          {
            title:'指标编码',
            align:"center",
            dataIndex: 'targetNo'
          }, */
          {
            title:'指标名称',
            align:"center",
            dataIndex: 'targetName'
          },
          {
            title:'显示模式',
            align:"center",
            dataIndex: 'displayMode_dictText'
          },
/*
          {
            title:'协议',
            align:"center",
            dataIndex: 'proType'
          },
*/
          {
            title:'报警点',
            align:"center",
            dataIndex: 'alarmPoint_dictText'
          },
          {
            title:'指令',
            align:"center",
            dataIndex: 'instruct'
          },
          {
            title:'寄存器地址',
            align:"center",
            dataIndex: 'address'
          },
          {
            title:'点类型',
            align:"center",
            dataIndex: 'infoType'
          },
          {
            title:'报警值',
            align:"center",
            dataIndex: 'ctrlUp'
          },
          {
            title:'报警表达式',
            align:"center",
            dataIndex: 'ctrlDown'
          },
          {
            title:'计算因子',
            align:"center",
            dataIndex: 'yinzi'
          },

          /*         {
            title:'数据类型',
            align:"center",
            dataIndex: 'dataType_dictText'
          },  */
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: {
              filterDropdown: 'filterDropdown',
              filterIcon: 'filterIcon',
              customRender: 'action'}
          }
        ],
        url: {
          list: "/jst/jstZcTarget/list?column=dev_type&order=asc",
          orgUserList: "/jst/jstZcTarget/orgUserList?column=dev_type&order=asc",
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
        var httpurl=null;
        if(params.orgUser!=null&&params.devType==null){
          httpurl=this.url.orgUserList;
        }else{
          httpurl=this.url.list;
        }
        this.loading = true;
        getAction(httpurl, params).then((res) => {
          if (res.success) {
            //update-begin---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
            this.dataSource = res.result.records||res.result;
            if(res.result.total)
            {
              this.ipagination.total = res.result.total;
            }else{
              this.ipagination.total = 0;
            }
            //update-end---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },
      handleChangeOrgUser(value){
        var tmp=null;
        var tmp2=null;
        if(value!=null){
          tmp = "jst_zc_cat,zc_catname,origin_id,org_user= '"+value+"'&&has_child ='0'&&id>'002' order by pid ";
          tmp2 = "jst_zc_dev,dev_name,dev_no,org_user= '"+value+"'";
        }else{
          tmp = "jst_zc_cat,zc_catname,origin_id,has_child ='0'&&id>'002' order by pid ";
          tmp2 = "jst_zc_dev,dev_name,dev_no,org_user= 'guangfa'";
        }
        this.queryParam.orgUser=value;
//        this.queryParam.devType=null;
//        this.queryParam.devNo=null;
        this.$set(this.dictCode2, "val", tmp);
        this.$set(this.dictCode3, "val", tmp2);
      },
      handleChangeType(value){
        var tmp2=null;
        var orgUser=this.queryParam.orgUser;
        if(orgUser!=null && value!=null){
          tmp2 = "jst_zc_dev,dev_name,dev_no,org_user= '"+orgUser+"'&&dev_cat='"+value+"'";
        }else{
          tmp2 = "jst_zc_dev,dev_name,dev_no,org_user= 'guangfa'";
        }
//        this.queryParam.orgUser=value;
        this.queryParam.devType=value;
        this.$set(this.dictCode3, "val", tmp2);
      },
      //列设置更改事件
      onColSettingsChange (checkedValues) {
        var key = this.$route.name+":colsettings";
        Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
        this.settingColumns = checkedValues;
        const cols = this.defColumns.filter(item => {
          if(item.key =='rowIndex'|| item.dataIndex=='action'){
            return true
          }
          if (this.settingColumns.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.columns =  cols;
      },
      initColumns(){
        //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
        //this.defColumns = colAuthFilter(this.defColumns,'testdemo:');

        var key = this.$route.name+":colsettings";
        let colSettings= Vue.ls.get(key);
        if(colSettings==null||colSettings==undefined){
          let allSettingColumns = [];
          this.defColumns.forEach(function (item,i,array ) {
            allSettingColumns.push(item.dataIndex);
          })
          this.settingColumns = allSettingColumns;
          this.columns = this.defColumns;
        }else{
          this.settingColumns = colSettings;
          const cols = this.defColumns.filter(item => {
            if(item.key =='rowIndex'|| item.dataIndex=='action'){
              return true;
            }
            if (colSettings.includes(item.dataIndex)) {
              return true;
            }
            return false;
          })
          this.columns =  cols;
        }
      },
      initDictConfig(){
      }
    },
    created() {
      this.initColumns();
    },

  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>