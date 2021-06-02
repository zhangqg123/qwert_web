<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="单位用户" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag @change="handleChangeOrgUser" dictCode="jst_zc_orguser,orguser_name,orguser_id" v-decorator="['orgUser', {}]" :triggerChange="true" placeholder="请输入单位" />
        </a-form-item>
        <a-form-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'devName', validatorRules.devName]" placeholder="请输入设备名称"></a-input>
        </a-form-item>
        <a-form-item label="设备分类" :labelCol="labelCol" :wrapperCol="wrapperCol">
      <!--    <a-input v-decorator="[ 'devCat', validatorRules.devCat]" placeholder="请输入设备分类"></a-input>
          <j-popup
            v-decorator="['devCat', validatorRules.devCat]"
            :trigger-change="true"
            org-fields="origin_id"
            dest-fields="devCat"
            code="jstzccat"
            @callback="popupCallback"/> -->
          <j-dict-select-tag v-decorator="[ 'devCat', validatorRules.devCat]" placeholder="请选择分类" :triggerChange="true" :dictCode="dictCode2.val" />
        </a-form-item>
        <a-form-item label="设备编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'devNo', validatorRules.devNo]" placeholder="请输入设备编号"></a-input>
        </a-form-item>
        <a-form-item label="模型编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'modNo', validatorRules.modNo]" placeholder="请输入模型编号"></a-input>
        </a-form-item>
        <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag v-decorator="['status', {}]" :triggerChange="true" placeholder="请输入设备状态"
                              dictCode="ac_status"/>
        </a-form-item>
        <a-form-item label="位置" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'position', validatorRules.position]" placeholder="请输入位置"></a-input>
        </a-form-item>
        <a-form-item label="连接类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <!--      <a-input v-decorator="[ 'type', {}]" placeholder="请输入连接类型"></a-input> -->
          <j-dict-select-tag @change="handleChangeProtocolType" v-decorator="['type', {}]" :triggerChange="true" placeholder="请输入连接类型"
                             dictCode="conn_type"/>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSTCP'||contype==='SOCKET'||contype==='SNMP'" label="ip地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ipAddress', {}]" placeholder="请输入ip地址"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSTCP'||contype==='SOCKET'||contype==='SNMP'" label="端口" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'port', {}]" placeholder="请输入端口号"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="COM口" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'com', {}]" placeholder="请输入com口"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSTCP'||contype==='SOCKET'||contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="从机编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'slave', {}]" placeholder="请输入从机编号"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="波特率" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'baudRate', {}]" placeholder="请输入波特率"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="数据位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'dataBits', {}]" placeholder="请输入数据位"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="停止位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'stopBits', {}]" placeholder="请输入停止位"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="奇偶校验" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'parity', {}]" placeholder="请输入奇偶校验"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSTCP'||contype==='SOCKET'||contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="分包位数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'packageBit', {}]" placeholder="分包位数"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='MODBUSTCP'||contype==='SOCKET'||contype==='MODBUSRTU'||contype==='MODBUSASCII'"  label="位数间隔" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'bitNumber', {}]" placeholder="位数间隔"></a-input>
        </a-form-item>
        <a-form-item v-if="contype==='SNMP'" label="版本号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'version', {}]" placeholder="请输入版本号"></a-input>
        </a-form-item>
        <a-form-item label="协议类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag v-decorator="['proType', {}]" :triggerChange="true"  placeholder="请输入协议类型"
                             dictCode="pro_type"/>
        </a-form-item>
        <a-form-item label="超时" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'timeOut', validatorRules.timeOut]" placeholder="请输入超时"></a-input>
        </a-form-item>
        <a-form-item label="重试" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'retry', {}]" placeholder="请输入重试次数"></a-input>
        </a-form-item>
        <a-form-item label="休眠时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'sleeptime', {}]" placeholder="请输入休眠时间"></a-input>
        </a-form-item>
        <a-form-item label="连接信息" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'conInfo', validatorRules.conInfo]" placeholder="请输入连接信息"></a-input>
        </a-form-item>
        <a-form-item label="属性信息" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'proInfo', validatorRules.proInfo]" placeholder="请输入属性信息"></a-input>
        </a-form-item>
        <a-form-item label="扩展信息" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'extInfo', validatorRules.extInfo]" placeholder="请输入扩展信息"></a-input>
        </a-form-item>
        <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'remark', validatorRules.remark]" placeholder="请输入备注"></a-input>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JTreeSelect from '@/components/jeecg/JTreeSelect'

  export default {
    name: "JstZcDevModal",
    components: {
      JTreeSelect
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        contype: null,
        protocol:null,
        model: {},
        dictCode2: {
          val:"jst_zc_cat,zc_catname,origin_id ,has_child ='0'&&id>'002' order by pid "
        },        // 表头
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          devName: {rules: [
          ]},
          devCat: {rules: [
          ]},
          devNo: {rules: [
          ]},
          modNo: {rules: [
            ]},
          position: {rules: [
          ]},
          extInfo: {rules: [
          ]},
          conInfo: {rules: [
          ]},
          proInfo: {rules: [
          ]},
          remark: {rules: [
          ]},
        },
        url: {
          add: "/jst/jstZcDev/add",
          edit: "/jst/jstZcDev/edit",
        }
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        if(!record.id){
          this.visible = true;
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model,'orgUser','devName','devCat','devNo','modNo','status','position','extInfo','conInfo','proInfo','remark'))
          })
        }else{
          this.handleChangeOrgUser(this.model.orgUser);
          var tempInfo=JSON.parse(this.model.conInfo);
          if(tempInfo==null) {
            this.visible = true;
            this.$nextTick(() => {
              this.form.setFieldsValue(pick(this.model,'orgUser','devName','devCat','devNo','modNo','status','position','extInfo','conInfo','proInfo','remark'))
            })

          }else{
            this.model.ipAddress = tempInfo.ipAddress;
            this.model.port = tempInfo.port;
            this.model.type = tempInfo.type;
            this.model.proType = tempInfo.proType;
            this.model.retry = tempInfo.retry;
            this.model.sleeptime = tempInfo.sleeptime;
            this.model.timeOut = tempInfo.timeOut;
            this.contype = this.model.type;

            if (this.model.type == "MODBUSTCP" || this.model.type == "SOCKET" || this.model.type == "MODBUSRTU" || this.model.type == "MODBUSASCII") {
              this.model.slave = tempInfo.slave;
              this.model.packageBit = tempInfo.packageBit;
              this.model.bitNumber = tempInfo.bitNumber;
            }
            if (this.model.type == "MODBUSRTU" || this.model.type == "SOCKET" || this.model.type == "MODBUSASCII") {
              this.model.com = tempInfo.com;
              this.model.baudRate = tempInfo.baudRate;
              this.model.dataBits = tempInfo.dataBits;
              this.model.stopBits = tempInfo.stopBits;
              this.model.parity = tempInfo.parity;
            }
            if (this.model.type == "SNMP") {
              this.model.version = tempInfo.version;
            }
            this.visible = true;
            if (this.model.type == "MODBUSTCP" || this.model.type == "SOCKET") {
              this.$nextTick(() => {
                this.form.setFieldsValue(pick(this.model,'orgUser', 'devName', 'devCat', 'devNo', 'modNo', 'status', 'position', 'extInfo', 'conInfo', 'ipAddress', 'port', 'type', 'slave', 'packageBit', 'bitNumber','proType', 'retry', 'sleeptime', 'timeOut', 'proInfo', 'remark'))
              })
            }
            if (this.model.type == "MODBUSRTU" || this.model.type == "MODBUSASCII") {
              this.$nextTick(() => {
                this.form.setFieldsValue(pick(this.model,'orgUser', 'devName', 'devCat', 'devNo', 'modNo', 'status', 'position', 'extInfo', 'conInfo', 'type', 'com', 'baudRate', 'dataBits', 'stopBits', 'parity', 'slave', 'packageBit', 'bitNumber','proType', 'retry', 'sleeptime', 'timeOut', 'proInfo', 'remark'))
              })
            }
            if (this.model.type == "SNMP") {
              this.$nextTick(() => {
                this.form.setFieldsValue(pick(this.model, 'orgUser','devName', 'devCat', 'devNo', 'modNo', 'status', 'position', 'extInfo', 'conInfo', 'ipAddress', 'port', 'type','proType', 'retry', 'sleeptime', 'timeOut', 'proInfo', 'remark', 'version'))
              })
            }
          }
        }
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            var tip=values.ipAddress;
            var tport=values.port;
            let tempConInfo={};
//            if(values.conInfo){
//              tempConInfo=JSON.parse(values.conInfo);
//            }
            tempConInfo.type=values.type;
            tempConInfo.proType=values.proType;

            tempConInfo.ipAddress=tip;
            tempConInfo.port=tport;
            tempConInfo.retry=values.retry;
            tempConInfo.sleeptime=values.sleeptime;

            if(values.type=="MODBUSTCP"||values.type=="SOCKET"||values.type=="MODBUSRTU"||values.type=="MODBUSASCII"){
              tempConInfo.slave=values.slave;
              tempConInfo.packageBit=values.packageBit;
              tempConInfo.bitNumber=values.bitNumber;
            }
            if(values.type=="MODBUSRTU"||values.type=="MODBUSASCII"){
              tempConInfo.com=values.com;
              tempConInfo.baudRate=values.baudRate;
              tempConInfo.dataBits=values.dataBits;
              tempConInfo.stopBits=values.stopBits;
              tempConInfo.parity=values.parity;
            }
            if(values.type=="SNMP"){
              tempConInfo.version=values.version;
            }
            tempConInfo.timeOut=values.timeOut;

            let tempInfo=JSON.stringify(tempConInfo);
            values.conInfo=tempInfo;
            let formData = Object.assign(this.model, values);

            console.log("表单提交数据:",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }

        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'orgUser','devName','devCat','devNo','modNo','position','extInfo','conInfo','proInfo','remark'))
      },
      handleChangeProtocolType(value){
        //如果是邮件类型那么则改变模板内容是富文本编辑器
        this.contype = value;
      },
      handleChangeOrgUser(value){
        var tmp=null;
        if(value!=null){
          tmp = "jst_zc_cat,zc_catname,origin_id,org_user= '"+value+"'&&has_child ='0'&&id>'002' order by pid ";
        }else{
          tmp = "jst_zc_cat,zc_catname,origin_id,has_child ='0'&&id>'002' order by pid ";
        }
        this.$set(this.dictCode2, "val", tmp);
      }
    }
  }
</script>