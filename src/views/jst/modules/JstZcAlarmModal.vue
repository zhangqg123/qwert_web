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

        <a-form-item label="设备编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'devNo', validatorRules.devNo]" placeholder="请输入设备编号"></a-input>
        </a-form-item>
        <a-form-item label="数据点编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'targetNo', validatorRules.targetNo]" placeholder="请输入数据点编号"></a-input>
        </a-form-item>
        <a-form-item label="报警值" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'alarmValue', validatorRules.alarmValue]" placeholder="请输入报警值"></a-input>
        </a-form-item>
        <a-form-item label="报警级别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'alarmLevel', validatorRules.alarmLevel]" placeholder="请输入报警级别"></a-input>
        </a-form-item>
        <a-form-item label="报警状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
  <!--        <a-input v-decorator="[ 'sendType', validatorRules.sendType]" placeholder="请输入报警状态"></a-input>  -->
          <j-dict-select-tag  v-decorator="['sendType', {}]" :triggerChange="true" placeholder="请输入报警状态"
                              dictCode="send_type"/>
        </a-form-item>
        <a-form-item label="报警时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择报警时间" v-decorator="[ 'sendTime', validatorRules.sendTime]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="处理状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'dealType', validatorRules.dealType]" placeholder="请输入处理状态"></a-input>
        </a-form-item>
        <a-form-item label="处理时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择处理时间" v-decorator="[ 'dealTime', validatorRules.dealTime]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="上报类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'reportType', validatorRules.reportType]" placeholder="请输入上报类型"></a-input>
        </a-form-item>
        <a-form-item label="上报时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择上报时间" v-decorator="[ 'reportTime', validatorRules.reportTime]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  

  export default {
    name: "JstZcAlarmModal",
    components: { 
      JDate,
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
          devNo: {rules: [
          ]},
          targetNo: {rules: [
          ]},
          alarmValue: {rules: [
          ]},
          alarmLevel: {rules: [
          ]},
          sendType: {rules: [
          ]},
          sendTime: {rules: [
          ]},
          dealType: {rules: [
          ]},
          dealTime: {rules: [
          ]},
          reportType: {rules: [
          ]},
          reportTime: {rules: [
          ]},
        },
        url: {
          add: "/jst/jstZcAlarm/add",
          edit: "/jst/jstZcAlarm/edit",
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
          this.form.setFieldsValue(pick(this.model,'devNo','targetNo','alarmValue','alarmLevel','sendType','sendTime','dealType','dealTime','reportType','reportTime'))
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
        this.form.setFieldsValue(pick(row,'devNo','targetNo','alarmValue','alarmLevel','sendType','sendTime','dealType','dealTime','reportType','reportTime'))
      },

      
    }
  }
</script>