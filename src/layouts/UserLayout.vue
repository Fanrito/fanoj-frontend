<template>
  <div id="userLayout">
    <a-layout style="min-height: 100vh">
      <a-layout-header class="header">
        <a-space>
          <img class="logo" src="../assets/logo.png" alt="logo" />
          <div class="title" style="color: #7348ff">Fan OJ</div>
        </a-space>
      </a-layout-header>
      <a-layout-content class="content">
        <router-view />
      </a-layout-content>
      <a-layout-footer class="footer">
        <a href="https://github.com/Fanrito" target="_blank">
          Developed by Fanrito
        </a>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<style scoped>
#userLayout {
  text-align: center;
  background: url("@/assets/loginBackground.png") no-repeat fixed center;
  background-size: cover;
}

#userLayout .header {
  margin: 16px 0;
}

#userLayout .content {
  margin-bottom: 16px;
  padding: 20px;
}

#userLayout .footer {
  padding: 16px;
  position: sticky;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
}

.logo {
  height: 40px;
  border: 1px solid #fff;
  border-radius: 90px;
}
</style>
<script setup lang="ts">
import GlobalHeader from "@/components/GlobalHeader.vue";
</script>
