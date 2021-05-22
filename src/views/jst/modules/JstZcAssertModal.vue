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
          <a-input v-decorator="[ 'assertCat', validatorRules.assertCat]" placeholder="请输入设备类型"></a-input>
        </a-form-item>
        <a-form-item label="数字点编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
      <!--    <a-input v-decorator="[ 'assertTarget', validatorRules.assertTarget]" placeholder="请输入数字点编号"></a-input> -->
          <j-dict-select-tag :triggerChange="true"   v-decorator="['assertTarget', validatorRules.assertTarget]" placeholder="请输入地址类型"  :dictCode="dictCode2.val"/>
        </a-form-item>
        <a-form-item label="报警点名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assertName', validatorRules.assertName]" placeholder="请输入报警点名称"></a-input>
        </a-form-item>
        <a-form-item label="寄存器地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'address', validatorRules.address]" placeholder="请输入寄存器地址"></a-input>
        </a-form-item>
        <a-form-item label="指令" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'instruct', validatorRules.instruct]" placeholder="请输入指令"></a-input>
        </a-form-item>
        <a-form-item label="报警设置数值" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'alarmValue', validatorRules.alarmValue]" placeholder="请输入报警设置数值"></a-input>
        </a-form-item>
        <a-form-item label="开关状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assertStatus', validatorRules.assertStatus]" placeholder="请输入开关状态数值"></a-input>
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
    name: "JstZcAssertModal",
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
          assertCat: {rules: [
          ]},
          assertTarget: {rules: [
          ]},
          assertName: {rules: [
          ]},
          address: {rules: [
          ]},
          instruct: {rules: [
          ]},
          alarmValue: {rules: [
          ]},
        },
        dictCode2: {
  //        val:"jst_zc_target,target_name,target_no,dev_type='DLY_SPM323'"
        },

        url: {
          add: "/jst/jstZcAssert/add",
          edit: "/jst/jstZcAssert/edit",
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
        this.assertCat=record.assertCat;
   //     this.dictCode="jst_zc_target,target_name,target_no,dev_type='DLY_SPM323'";
        var tmp="jst_zc_target,target_name,target_no,dev_type='"+this.assertCat+"'";
        this.$set(this.dictCode2, "val", tmp);
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'assertCat','assertTarget','assertName','address','instruct','alarmValue','assertStatus'))
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
        this.form.setFieldsValue(pick(row,'assertCat','assertTarget','assertName','address','instruct','alarmValue'))
      },

      
    }
  }
</script>