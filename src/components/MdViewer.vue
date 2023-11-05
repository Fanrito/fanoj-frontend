<template>
  <Viewer :value="value" :plugins="plugins" />
</template>

<script setup lang="ts">
import gfm from "@bytemd/plugin-gfm";
import highlight from "@bytemd/plugin-highlight";
import "highlight.js/styles/default.css";
import { Viewer } from "@bytemd/vue-next";
import { ref, withDefaults, defineProps } from "vue";

/**
 * 定义组件属性的类型
 */
interface Props {
  value: string;
}

const plugins = [gfm(), highlight()];

/**
 * 指定初始值
 */
const props = withDefaults(defineProps<Props>(), {
  value: () => "",
});
</script>

<style>
/* 隐藏最后一个图标 */
.bytemd-toolbar-icon:last-child {
  display: none;
}
</style>
