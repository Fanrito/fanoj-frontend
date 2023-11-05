<template>
  <div id="userLoginView">
    <h2 style="margin: 0 auto 28px; color: #7348ff">更新个人信息</h2>
    <a-form
      :model="form"
      @submit="handleSubmit"
      label-align="left"
      :auto-label-width="true"
      class="form"
    >
      <a-form-item field="userAvatar" label="头像">
        <a-avatar v-model="form.userAvatar" placeholder="默认头像" />
      </a-form-item>
      <a-form-item field="userName" label="用户名">
        <a-input v-model="form.userName" placeholder="请输入用户名" />
      </a-form-item>
      <a-form-item field="userProfile" label="个人简介">
        <a-input v-model="form.userProfile" placeholder="简单介绍一下自己吧~" />
      </a-form-item>
      <a-form-item>
        <a-space fill>
          <a-button
            type="primary"
            html-type="submit"
            style="background: #7348ff"
            >提交
          </a-button>
        </a-space>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive } from "vue";
import {
  UserControllerService,
  UserRegisterRequest,
  UserUpdateMyRequest,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import { useStore } from "vuex";

const router = useRouter();
const store = useStore();

const form = reactive({
  userAvatar: "",
  userName: "",
  userProfile: "",
} as UserUpdateMyRequest);

/**
 * 获取当前信息
 */
const getUserInfo = async () => {
  let res = await UserControllerService.getLoginUserUsingGet();
  if (res.code === 0) {
    Object.assign(form, res.data);
  } else {
    message.error(res.message as never);
  }
};

onMounted(() => {
  getUserInfo();
});

/**
 * 提交表单
 * @param data
 */
const handleSubmit = async () => {
  let res = await UserControllerService.updateMyUserUsingPost(form);
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
