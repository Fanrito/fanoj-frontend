<template>
  <div id="globalHeader">
    <a-row class="grid-demo" align="center" :wrap="false">
      <a-col flex="auto">
        <div>
          <a-menu
            mode="horizontal"
            :selected-keys="selectedKeys"
            @menu-item-click="onMenuClick"
          >
            <a-menu-item
              key="0"
              :style="{ padding: 0, marginRight: '38px' }"
              disabled
            >
              <div class="title-bar">
                <img class="logo" src="../assets/logo.png" alt="logo" />
                <div class="title">Fan OJ</div>
              </div>
            </a-menu-item>
            <a-menu-item v-for="item in visibleRoutes" :key="item.path">
              {{ item.name }}
            </a-menu-item>
            <a-menu-item v-if="!isLogin">
              <router-link to="/user/login">登录</router-link>
            </a-menu-item>
            <a-menu-item v-if="isLogin">
              <router-link to="/user/update">更新个人信息</router-link>
            </a-menu-item>
            <a-menu-item v-if="isLogin">
              <router-link to="/user/logout">退出登录</router-link>
            </a-menu-item>
          </a-menu>
        </div>
      </a-col>
      <a-col flex="100px">
        <div>
          {{ store.state.user?.loginUser?.userName ?? "未登录" }}
        </div>
      </a-col>
    </a-row>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import { routes } from "@/router/routes";
import { useRoute, useRouter } from "vue-router";
import { useStore } from "vuex";
import checkAccess from "@/access/checkAccess";
import ACCESS_ENUM from "@/access/accessEnum";
import { UserControllerService } from "../../generated";

const router = useRouter();
const route = useRoute();
const store = useStore();

// 展示在菜单的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item, index) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // todo 根据权限过滤菜单
    if (
      !checkAccess(store.state.user.loginUser, item?.meta?.access as string)
    ) {
      return false;
    }
    return true;
  });
});

const selectedKeys = ref([route.path]);

let isLogin = ref();
// 远程请求获取登录信息
const getLoginState = async () => {
  const res = await UserControllerService.getLoginUserUsingGet();
  const loginUser = res.data;

  isLogin.value = loginUser;
};

onMounted(() => {
  getLoginState();
});

// 路由跳转后，更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

const onMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};

// onMounted(() => {});
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: #444;
  margin-left: 16px;
}

.logo {
  height: 40px;
  border: 1px solid #fff;
  border-radius: 90px;
}
</style>
