<template>
  <div class="base-info">
    <a-spin :spinning="spinning">
      <a-form layout="horizontal" :label-col="formCol.labelCol"
              :wrapper-col="formCol.wrapperCol" :model="form">
        <a-form-item label="账号">
          {{ form.username }}
        </a-form-item>
        <a-form-item label="姓名">
          <a-input v-model:value="form.cname" placeholder="请输入姓名"/>
        </a-form-item>
        <a-form-item label="电话">
          <a-input v-model:value="form.tel" placeholder="请输入电话"/>
        </a-form-item>
        <a-form-item label="部门">
          <span style="padding: 0 10px">{{form.userpri ?? '未选择'}}</span>
          <dept-tree-modal @changedept-success="initData" v-bind="$attrs" />
        </a-form-item>
        <a-form-item label="角色">
          {{ rolename }} <br/>
          <a-checkbox v-model:checked="form.needKeylogin">
            Key验证登陆
          </a-checkbox>
          <a-checkbox :disabled="!form.needKeylogin" v-model:checked="form.notneedkey">
            非必须
          </a-checkbox>
          <a-checkbox v-model:checked="form.status">
            禁用用户
          </a-checkbox>
        </a-form-item>
        <a-form-item label="审批模式">
          <a-radio-group v-model:value="form.autoapprove">
            <a-radio value="0">
              自动审批
            </a-radio>
            <a-radio value="1">
              手动审批
            </a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="介绍">
          <a-textarea v-model:value="form.description" placeholder="用户简介"/>
        </a-form-item>
        <a-form-item :wrapper-col="{ span: 14, offset: 4 }">
          <a-button @click="saveChange" type="primary">
            保存修改
          </a-button>
        </a-form-item>
      </a-form>
    </a-spin>
  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, reactive, toRefs} from 'vue'
import { Spin, message} from 'ant-design-vue'
import {userBase, userSave} from "@/api/user";
import {DeptTreeModal} from './components'

export default defineComponent({
  name: "base-info",
  components: {DeptTreeModal, [Spin.name]: Spin},
  setup(props, {attrs}) {

    const formCol = {
      labelCol: {span: 4},
      wrapperCol: {span: 14},
    }

    const state = reactive({
      value: [],
      spinning: false, // 加载状态
      rolename: '',
      form: {
        "id": '',
        "username": '',
        "cname": '',
        "depid": '',
        autoapprove: 0,
        "needKeylogin": false as boolean,
        "notneedkey": false as boolean,
        "status": false as boolean,
        "tel": '',
        "userpri": '',
        "description": ''
      }
    })

    const initData = async () => {
      state.spinning = true
      const res = await userBase({}, attrs.uid).finally(() => state.spinning = false)
      const User = res.User
      state.rolename = res.rolename
      Object.keys(state.form).forEach(key => state.form[key] = User[key])
      // 将后端返回的数值转布尔值
      state.form.needKeylogin = res.User.needKeylogin == 1
      state.form.notneedkey = res.User.notneedkey == 1
      state.form.status = res.User.status == 0
    }

    onMounted(() => {
      initData()
    })

    const saveChange = async () => {
      const {needKeylogin, notneedkey, status} = state.form
      const params = {
        id: '',
        cname: '',
        tel: '',
        autoapprove: '0',
        needKeylogin: 0,
        notneedkey: 0,
        status: 0,
        description: '',
      }
      Object.keys(params).forEach(key => params[key] = state.form[key])

      params.needKeylogin = needKeylogin ? 1 : 0
      params.notneedkey = notneedkey ? 1 : 0
      params.status = status ? 0 : 1

      const res = await userSave(params, attrs.uid)
      if (res.Code == 1) {
        initData()
        message.success('修改成功')
      } else {
        message[res.type](res.message || '修改失败')
      }
    }

    return {
      ...toRefs(state),
      formCol,
      initData,
      saveChange
    }
  }
})
</script>

<style lang="scss" scoped>
.base-info {
  ::v-deep(.ant-descriptions-item-content) {
    white-space: nowrap;
    word-break: keep-all;

    button {
      margin-left: 10px;
    }
  }
}
</style>
