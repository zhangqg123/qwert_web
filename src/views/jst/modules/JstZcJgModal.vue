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

        <a-form-item label="机柜模型编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'jgNo', validatorRules.jgNo]" placeholder="请输入机柜模型编号"></a-input>
        </a-form-item>
        <a-form-item label="列头柜出线编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouNo', validatorRules.lietouNo]" placeholder="请输入列头柜出线编号"></a-input>
        </a-form-item>
        <a-form-item label="微环境温度编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'whNo', validatorRules.whNo]" placeholder="请输入微环境温度编号"></a-input>
        </a-form-item>
        <a-form-item label="列头柜数据点编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouTarget', validatorRules.lietouTarget]" placeholder="请输入列头柜数据点编号"></a-input>
        </a-form-item>
        <a-form-item label="微环境数据点" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'whTarget', validatorRules.whTarget]" placeholder="请输入微环境数据点"></a-input>
        </a-form-item>
        <a-form-item label="有功电度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouYgdd', validatorRules.lietouYgdd]" placeholder="请输入有功电度"></a-input>
        </a-form-item>
        <a-form-item label="无功电度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouWgdd', validatorRules.lietouWgdd]" placeholder="请输入无功电度"></a-input>
        </a-form-item>
        <a-form-item label="有功功率" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouYggl', validatorRules.lietouYggl]" placeholder="请输入有功功率"></a-input>
        </a-form-item>
        <a-form-item label="无功功率" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouWggl', validatorRules.lietouWggl]" placeholder="请输入无功功率"></a-input>
        </a-form-item>
        <a-form-item label="功率因数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'lietouGlys', validatorRules.lietouGlys]" placeholder="请输入功率因数"></a-input>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: "JstZcJgModal",
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
          jgNo: {rules: [
          ]},
          lietouNo: {rules: [
          ]},
          whNo: {rules: [
          ]},
          lietouTarget: {rules: [
          ]},
          whTarget: {rules: [
          ]},
          lietouYgdd: {rules: [
          ]},
          lietouWgdd: {rules: [
          ]},
          lietouYggl: {rules: [
          ]},
          lietouWggl: {rules: [
          ]},
          lietouGlys: {rules: [
          ]},
        },
        url: {
          add: "/jst/jstZcJg/add",
          edit: "/jst/jstZcJg/edit",
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
          this.form.setFieldsValue(pick(this.model,'jgNo','lietouNo','whNo','lietouTarget','whTarget','lietouYgdd','lietouWgdd','lietouYggl','lietouWggl','lietouGlys'))
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
        this.form.setFieldsValue(pick(row,'jgNo','lietouNo','whNo','lietouTarget','whTarget','lietouYgdd','lietouWgdd','lietouYggl','lietouWggl','lietouGlys'))
      },

      
    }
  }
</script>