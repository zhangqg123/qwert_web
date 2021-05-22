<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'devType', validatorRules.devType]" placeholder="请输入设备类型"></a-input>
        </a-form-item>
        <a-form-item label="指标编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetNo', validatorRules.targetNo]" placeholder="请输入指标编码"></a-input>
        </a-form-item>
        <a-form-item label="指标名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetName', validatorRules.targetName]" placeholder="请输入指标名称"></a-input>
        </a-form-item>
        <a-form-item label="单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'unit', validatorRules.unit]" placeholder="请输入单位"></a-input>
        </a-form-item>
        <a-form-item label="是否采集" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ifGet', validatorRules.ifGet]" placeholder="请输入是否采集"></a-input>
        </a-form-item>
        <a-form-item label="数据类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'dataType', validatorRules.dataType]" placeholder="请输入数据类型"></a-input>
        </a-form-item>
        <a-form-item label="协议" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'proType', validatorRules.proType]" placeholder="请输入协议"></a-input>
        </a-form-item>
        <a-form-item label="指令" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'instruct', validatorRules.instruct]" placeholder="请输入指令"></a-input>
        </a-form-item>
        <a-form-item label="采集时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'getTime', validatorRules.getTime]" placeholder="请输入采集时间"></a-input>
        </a-form-item>
        <a-form-item label="截取开始字节" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'offset', validatorRules.offset]" placeholder="请输入截取开始字节"></a-input>
        </a-form-item>
        <a-form-item label="截取字节数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'len', validatorRules.len]" placeholder="请输入截取字节数"></a-input>
        </a-form-item>
        <a-form-item label="截取位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'interceptBit', validatorRules.interceptBit]" placeholder="请输入截取位"></a-input>
        </a-form-item>
        <a-form-item label="表达式配置" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'espConfig', validatorRules.espConfig]" placeholder="请输入表达式配置"></a-input>
        </a-form-item>
        <a-form-item label="监控类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'monitorType', validatorRules.monitorType]" placeholder="请输入监控类型"></a-input>
        </a-form-item>
        <a-form-item label="信息点类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'infoType', validatorRules.infoType]" placeholder="请输入信息点类型"></a-input>
        </a-form-item>
        <a-form-item label="信息点数值类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'infoDatatype', validatorRules.infoDatatype]" placeholder="请输入信息点数值类型"></a-input>
        </a-form-item>
        <a-form-item label="信息点数值精度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'infoDataaccurate', validatorRules.infoDataaccurate]" placeholder="请输入信息点数值精度"></a-input>
        </a-form-item>
        <a-form-item label="控制上限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ctrlUp', validatorRules.ctrlUp]" placeholder="请输入控制上限"></a-input>
        </a-form-item>
        <a-form-item label="控制下限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'ctrlDown', validatorRules.ctrlDown]" placeholder="请输入控制下限"></a-input>
        </a-form-item>
        <a-form-item label="指令顺序" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetOrder', validatorRules.targetOrder]" placeholder="请输入指令顺序"></a-input>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  
  export default {
    name: "JstZcTargetModal",
    components: { 
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
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
          instruct: {rules: [
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
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'devType','targetNo','targetName','unit','ifGet','dataType','proType','instruct','getTime','offset','len','interceptBit','espConfig','monitorType','infoType','infoDatatype','infoDataaccurate','ctrlUp','ctrlDown','targetOrder'))
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
        this.form.setFieldsValue(pick(row,'devType','targetNo','targetName','unit','ifGet','dataType','proType','instruct','getTime','offset','len','interceptBit','espConfig','monitorType','infoType','infoDatatype','infoDataaccurate','ctrlUp','ctrlDown','targetOrder'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>