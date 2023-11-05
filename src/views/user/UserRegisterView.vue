<template>
  <div id="userLoginView">
    <h2 style="margin: 0 auto 28px; color: #7348ff">用户登录</h2>
    <a-form
      :model="form"
      @submit="handleSubmit"
      label-align="left"
      :auto-label-width="true"
      class="form"
    >
      <a-form-item field="userAccount" label="账号">
        <a-input v-model="form.userAccount" placeholder="请输入账号" />
      </a-form-item>
      <a-form-item field="userPassword" tooltip="密码不少于8位" label="密码">
        <a-input-password
          v-model="form.userPassword"
          placeholder="请输入密码"
        />
      </a-form-item>
      <a-form-item
        field="userCheckPassword"
        tooltip="密码不少于8位"
        label="密码"
      >
        <a-input-password
          v-model="form.checkPassword"
          placeholder="请确认密码"
        />
      </a-form-item>
      <a-form-item>
        <a-space fill>
          <a-button
            type="primary"
            html-type="submit"
            style="background: #7348ff"
            >注册
          </a-button>
          <router-link to="/user/login"> 已有账号？去登录...</router-link>
        </a-space>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { reactive } from "vue";
import { UserControllerService, UserRegisterRequest } from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import { useStore } from "vuex";

const router = useRouter();
const store = useStore();

const form = reactive({
  userAccount: "",
  userPassword: "",
  checkPassword: "",
} as UserRegisterRequest);
/**
 * 提交表单
 * @param data
 */
const handleSubmit = async () => {
  let res = await UserControllerService.userRegisterUsingPost(form);
  if (res.code === 0) {
    await store.dispatch("user/getLoginUser");
    router.push({
      path: "/",
      replace: true,
    });
  } else {
    message.error(res.message);
  }
};
</script>

<style scoped>
.form {
  max-width: 480px;
  margin: 0 auto;
  padding: 30px 20px 0;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 20px;
}
</style>
