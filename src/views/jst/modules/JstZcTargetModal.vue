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

        <a-form-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
        <!--  <a-input v-decorator="[ 'devType', validatorRules.devType]" placeholder="请输入设备类型"></a-input> -->
          <j-popup
            v-decorator="['devType', validatorRules.devType]"
            :trigger-change="true"
            org-fields="origin_id"
            dest-fields="devType"
            code="jstzccat"
            @callback="popupCallback"/>
        </a-form-item>
        <a-form-item label="指标编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetNo', validatorRules.targetNo]" placeholder="请输入指标编码"></a-input>
        </a-form-item>
        <a-form-item label="指标名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetName', validatorRules.targetName]" placeholder="请输入指标名称"></a-input>
        </a-form-item>
        <a-form-item label="信息点类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag  @change="handleChangeInfoType" v-decorator="['infoType', {}]" :triggerChange="true" placeholder="请输入信息点类型"
                              dictCode="info_type"/>
        </a-form-item>
        <a-form-item label="指令" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'instruct', validatorRules.instruct]" placeholder="请输入指令"></a-input>
        </a-form-item>
        <a-form-item label="寄存器地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'address', validatorRules.address]" placeholder="请输入寄存器地址"></a-input>
        </a-form-item>
        <a-form-item label="数据类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag :triggerChange="true"   v-decorator="['dataType', validatorRules.dataType]" placeholder="请输入地址类型"  dictCode="data_type"/>
        </a-form-item>
        <a-form-item v-if="infoType==='digital'" label="事件0到1"  :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'evt01', validatorRules.evt01]" placeholder="请输入"></a-input>
        </a-form-item>
        <a-form-item v-if="infoType==='digital'" label="事件1到0" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'evt10', validatorRules.evt10]" placeholder="请输入"></a-input>
        </a-form-item>
        <a-form-item v-if="infoType==='analog'" label="报警低限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'valMin', validatorRules.valMin]" placeholder="请输入报警低限"></a-input>
        </a-form-item>
        <a-form-item v-if="infoType==='analog'" label="恢复低限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'restoreMin', validatorRules.restoreMin]" placeholder="请输入恢复低限"></a-input>
        </a-form-item>
        <a-form-item v-if="infoType==='analog'" label="恢复高限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'restoreMax', validatorRules.restoreMax]" placeholder="请输入恢复高限"></a-input>
        </a-form-item>
        <a-form-item v-if="infoType==='analog'" label="报警高限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'valMax', validatorRules.valMax]" placeholder="请输入报警高限"></a-input>
        </a-form-item>
        <a-form-item label="报警点" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <!--      <a-input v-decorator="[ 'alarmPoint', validatorRules.alarmPoint]" placeholder="请输入控制上限"></a-input>
                <j-dict-select-tag type="radio" v-decorator="['sex', {}]" :trigger-change="true" dictCode="sex"/> -->
          <j-dict-select-tag  v-decorator="['alarmPoint', {}]" :triggerChange="true" placeholder="请输入是否报警点"
                              dictCode="alarm_point"/>
        </a-form-item>
        <a-form-item label="报警值" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ctrlUp', validatorRules.ctrlUp]" placeholder="请输入控制上限"></a-input>
        </a-form-item>
        <a-form-item label="报警表达式" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ctrlDown', validatorRules.ctrlDown]" placeholder="请输入控制下限"></a-input>
        </a-form-item>
        <a-form-item label="计算因子" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'yinzi', validatorRules.yinzi]" placeholder="请输入计算因子"></a-input>
        </a-form-item>
        <a-form-item label="截取位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'interceptBit', validatorRules.interceptBit]" placeholder="请输入截取位"></a-input>
        </a-form-item>
        <a-form-item label="显示模式" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag  v-decorator="['displayMode', {}]" :triggerChange="true" placeholder="请输入显示模式"
                              dictCode="display_mode"/>
        </a-form-item>
<!--
        <a-form-item label="采集时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'getTime', validatorRules.getTime]" placeholder="请输入采集时间"></a-input>
        </a-form-item>
        <a-form-item label="位移" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'offset', validatorRules.offset]" placeholder="请输入位移"></a-input>
        </a-form-item>
        <a-form-item label="位移长度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'len', validatorRules.len]" placeholder="请输入位移长度"></a-input>
        </a-form-item>
        <a-form-item label="表达式配置" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'espConfig', validatorRules.espConfig]" placeholder="请输入表达式配置"></a-input>
        </a-form-item>
        <a-form-item label="监控类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'monitorType', validatorRules.monitorType]" placeholder="请输入监控类型"></a-input>
        </a-form-item>
        <a-form-item label="信息点数值类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'infoDatatype', validatorRules.infoDatatype]" placeholder="请输入信息点数值类型"></a-input>
        </a-form-item>
        <a-form-item label="信息点数值精度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'infoDataaccurate', validatorRules.infoDataaccurate]" placeholder="请输入信息点数值精度"></a-input>
        </a-form-item>
        <a-form-item label="指令顺序" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetOrder', validatorRules.targetOrder]" placeholder="请输入指令顺序"></a-input>
        </a-form-item>
        <a-form-item label="单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'unit', validatorRules.unit]" placeholder="请输入单位"></a-input>
        </a-form-item>
        <a-form-item label="是否采集" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ifGet', validatorRules.ifGet]" placeholder="请输入是否采集"></a-input>
        </a-form-item>
        <a-form-item label="协议" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'proType', validatorRules.proType]" placeholder="请输入协议"></a-input>
        </a-form-item>  -->
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDictSelectTag from '@/components/dict/JDictSelectTag.vue'

  export default {
    name: "JstZcTargetModal",
    components: {
      JDictSelectTag
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        infoType: null,
        model: {},
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
          devType: {rules: [
          ]},
          targetNo: {rules: [
          ]},
          targetName: {rules: [
          ]},
          unit: {rules: [
          ]},
          ifGet: {rules: [
          ]},
          dataType: {rules: [
          ]},
          proType: {rules: [
          ]},
          address: {rules: [
            ]},
          addressType: {rules: [
            ]},
          instruct: {rules: [
          ]},
          evt01: {rules: [
            ]},
          evt10: {rules: [
            ]},
          valMin: {rules: [
            ]},
          restoreMin: {rules: [
            ]},
          restoreMax: {rules: [
            ]},
          valMax: {rules: [
            ]},
          getTime: {rules: [
          ]},
          offset: {rules: [
          ]},
          len: {rules: [
          ]},
          interceptBit: {rules: [
          ]},
          espConfig: {rules: [
          ]},
          monitorType: {rules: [
          ]},
          infoType: {rules: [
          ]},
          infoDatatype: {rules: [
          ]},
          infoDataaccurate: {rules: [
          ]},
          ctrlUp: {rules: [
          ]},
          ctrlDown: {rules: [
          ]},
          targetOrder: {rules: [
          ]},
          displayMode: {rules: [
            ]},
        },
        url: {
          add: "/jst/jstZcTarget/add",
          edit: "/jst/jstZcTarget/edit",
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
        this.visible = true;
        this.infoType=this.model.infoType;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'devType','targetNo','targetName','unit','ifGet','dataType','proType','address','addressType','instruct','evt01','evt10','valMin','restoreMin','restoreMax','valMax','getTime','offset','len','interceptBit','espConfig','monitorType','infoType','infoDatatype','infoDataaccurate','alarmPoint','ctrlUp','ctrlDown','yinzi','targetOrder','displayMode'))
        })
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
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
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
        this.form.setFieldsValue(pick(row,'devType','targetNo','targetName','unit','ifGet','dataType','proType','address','addressType','instruct','evt01','evt10','valMin','restoreMin','restoreMax','valMax','getTime','offset','len','interceptBit','espConfig','monitorType','infoType','infoDatatype','infoDataaccurate','alarmPoint','ctrlUp','ctrlDown','targetOrder','displayMode'))
      },
      handleChangeInfoType(value){
        this.infoType = value;
      }
    }
  }
</script>