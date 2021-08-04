<template>
  <page-layout :title="devName">
    <a-card :bordered="false">
      <detail-list title="环控点" >
        <detail-list-item :term="item.targetName" v-for="(item,index) in targets" :key="index">{{ item.value }}</detail-list-item>
<!--        <detail-list-item term="状态">已取货</detail-list-item>
        <detail-list-item term="销售单号">1234123421</detail-list-item>
        <detail-list-item term="子订单">3214321432</detail-list-item>  -->
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="环控点 2" >
        <detail-list-item :term="item.targetName" v-for="(item,index) in targets2" :key="index">{{ item.value }}</detail-list-item>
      </detail-list>
<!--      <detail-list title="用户信息">
        <detail-list-item term="用户姓名">付小小</detail-list-item>
        <detail-list-item term="联系电话">18100000000</detail-list-item>
        <detail-list-item term="常用快递">菜鸟仓储</detail-list-item>
        <detail-list-item term="取货地址">浙江省杭州市西湖区万塘路18号</detail-list-item>
        <detail-list-item term="备注">	无</detail-list-item>
      </detail-list>  -->
      <a-divider style="margin-bottom: 32px"/>


    </a-card>
  </page-layout>
</template>

<script>
  import PageLayout from '@/components/page/PageLayout'
  import STable from '@/components/table/'
  import DetailList from '@/components/tools/DetailList'
  import ABadge from "ant-design-vue/es/badge/Badge"
  import { getAction } from '@/api/manage'
//  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  const DetailListItem = DetailList.Item

  export default {
    name: 'devDetail',
    components: {
      PageLayout,
      ABadge,
      DetailList,
      DetailListItem,
      STable,
//      JeecgListMixin
    },
    data () {
      return {
        loading: false,
        devName: null,
        targets: [],
        targets2: [],
        url: {
          list: "/jst/qwertFc/list",
          devRedis: "/jst/qwertFc/devRedis",
          delete: "/jst/qwertFc/delete",
          deleteBatch: "/jst/qwertFc/deleteBatch",
          exportXlsUrl: "/jst/qwertFc/exportXls",
          importExcelUrl: "jst/qwertFc/importExcel",

        },

/*        registerTypeList:[{
          text:"业务受理"
        },{
          text:"业务管理"
        },{
          text:"文件管理"
        },{
          text:"信息查询"
        }],
*/
        goodsColumns: [
          {
            title: '商品编号',
            dataIndex: 'id',
            key: 'id'
          },
          {
            title: '商品名称',
            dataIndex: 'name',
            key: 'name'
          },
          {
            title: '商品条码',
            dataIndex: 'barcode',
            key: 'barcode'
          },
          {
            title: '单价',
            dataIndex: 'price',
            key: 'price',
            align: 'right'
          },
          {
            title: '数量（件）',
            dataIndex: 'num',
            key: 'num',
            align: 'right'
          },
          {
            title: '金额',
            dataIndex: 'amount',
            key: 'amount',
            align: 'right'
          }
        ],

        scheduleColumns: [
          {
            title: '时间',
            dataIndex: 'time',
            key: 'time'
          },
          {
            title: '当前进度',
            dataIndex: 'rate',
            key: 'rate'
          },
          {
            title: '状态',
            dataIndex: 'status',
            key: 'status',
            scopedSlots: { customRender: 'status' },
          },
          {
            title: '操作员ID',
            dataIndex: 'operator',
            key: 'operator'
          },
          {
            title: '耗时',
            dataIndex: 'cost',
            key: 'cost'
          }
        ],
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          'processing': '进行中',
          'success': '完成',
          'failed': '失败'
        }
        return statusMap[status]
      }
    },
    created() {
      var code=this.$route.params.code;
      this.loadData(code);
    },
    methods: {
      loadData(arg) {
        if(arg==null){
          return;
        }
        var code=this.$route.params.code;
        var code=arg;
        if(!this.url.devRedis){
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = {};//查询条件
        params.code=code;
        this.loading = true;
        getAction(this.url.devRedis, params).then((res) => {
          if (res.success) {
            this.devName=res.message;
             var tmpResult=res.result;
             let tr=JSON.parse(tmpResult);
            this.targets = tr.filter(item=> item.displayMode !==0)
            this.targets2 = tr.filter(item=> item.displayMode ===0)
   //          this.registerTypeList=res.result;
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },
      initDictConfig(){
      },
    },
    computed: {
//      key() {
//        this.loadData(this.$route.params.code);
//        return this.$route.name !== undefined? this.$route.name +new Date(): this.$route +new Date()
//      },
      title () {
        return this.$route.meta.title
      }
    },
    watch: { //通过watch来监听路由变化
      "$route": function() {
        this.loadData(this.$route.params.code);
      }
    },

  }
</script>

<style lang="less" scoped>
  .title {
    color: rgba(0,0,0,.85);
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 16px;
  }
</style>