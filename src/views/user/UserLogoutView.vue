<template>
  <div id="userLogoutView"></div>
</template>

<script setup lang="ts">
import { UserControllerService } from "../../../generated";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import message from "@arco-design/web-vue/es/message";
import { onMounted } from "vue";

const router = useRouter();
const store = useStore();
const logout = async () => {
  let res = await UserControllerService.userLogoutUsingPost();
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
onMounted(() => {
  logout();
});
</script>

<style scoped>
#userLogoutView {
}
</style>
