<template>
  <BasicModal v-bind="$attrs" @register="registerModal" title="修改密码" @ok="handleSubmit" destroyOnClose :width="400">
    <a-form class="antd-modal-form" ref="formRef" :model="formState" :rules="validatorRules">
      <a-form-item name="oldPassword">
        <div class="black font-size-13">旧密码</div>
        <div class="pass-padding">
          <a-input-password placeholder="请输入旧密码" v-model:value="formState.oldPassword" />
        </div>
      </a-form-item>
      <a-form-item name="password">
        <span class="black font-size-13">新密码</span>
        <div class="pass-padding">
          <a-input-password v-model:value="formState.password" placeholder="新密码" autocomplete="new-password" />
        </div>
        <span class="gray-9e font-size-13">8-20位，需包含字母和数字</span>
      </a-form-item>
    </a-form>
  </BasicModal>
</template>
<script lang="ts" name="user-pass-word-modal" setup>
  import { ref, unref, reactive } from 'vue';
  import { BasicModal, useModalInner } from '/@/components/Modal';
  import { Rule } from '@/components/Form';
  import { updatePasswordNotBindPhone } from '../UserSetting.api';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useUserStore } from '/@/store/modules/user';

  const { createMessage } = useMessage();
  //用户名
  const username = ref<string>('');
  const formRef = ref();
  const formState = reactive({
    oldPassword: '',
    password: '',
  });
  // 声明Emits
  const emit = defineEmits(['success', 'register']);
  //表单赋值
  const [registerModal, { setModalProps, closeModal }] = useModalInner(async (data) => {
    setModalProps({ confirmLoading: false });
    username.value = data.record.username;
    Object.assign(formState, { password: '', oldPassword: '' });
  });
  const userStore = useUserStore();
  const validatorRules: Record<string, Rule[]> = {
    oldPassword: [{ required: true, message: '请输入旧密码' }],
    password: [
      { required: true, validator: checkPassword },
      { pattern: /^(?=.*[0-9])(?=.*[a-zA-Z])(.{8,20})$/, message: '8-20位，需包含字母和数字' },
    ],
  };

  //表单提交事件
  async function handleSubmit() {
    try {
      let values = await formRef.value.validateFields();
      setModalProps({ confirmLoading: true });
      //提交表单
      values.username = unref(username);
      await updatePasswordNotBindPhone(values).then((res) => {
        if (res.success) {
          createMessage.info({
            content: '密码修改成功，请重新登录！3s后自动退出登录',
            duration: 3,
          });
          //3s后返回登录页面
          setTimeout(() => {
            userStore.logout(true);
          }, 3000);
          //关闭弹窗
          closeModal();
        } else {
          createMessage.warn(res.message);
        }
      });
    } finally {
      setModalProps({ confirmLoading: false });
    }
  }

  /**
   * 验证新密码是否为空
   */
  function checkPassword(_rule: Rule, value: string) {
    if (value === '') {
      return Promise.reject('请输入新密码');
    }
    return Promise.resolve();
  }
</script>
<style lang="less" scoped>
  .black {
    color: @text-color;
  }
  .font-size-13 {
    font-size: 13px;
    line-height: 15px;
  }
  .gray-9e {
    color: #9e9e9e;
  }
  .float-left {
    float: left;
  }
  .pass-padding {
    padding-top: 10px;
    padding-bottom: 10px;
  }
  .antd-modal-form {
    padding: 10px 24px 10px 24px;
  }
  :deep(.ant-form-item) {
    margin-bottom: 10px;
  }
</style>
