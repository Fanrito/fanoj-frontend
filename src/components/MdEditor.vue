<template>
  <Editor
    :value="value"
    :mode="mode"
    :plugins="plugins"
    @change="handleChange"
  />
</template>

<script setup lang="ts">
import gfm from "@bytemd/plugin-gfm";
import highlight from "@bytemd/plugin-highlight";
import "highlight.js/styles/default.css";
import { Editor, Viewer } from "@bytemd/vue-next";
import { ref, withDefaults, defineProps } from "vue";

/**
 * 定义组件属性的类型
 */
interface Props {
  value: string;
  handleChange: (v: string) => void;
  mode?: string;
}

const plugins = [gfm(), highlight()];

/**
 * 指定初始值
 */
const props = withDefaults(defineProps<Props>(), {
  value: () => "",
  mode: () => "split",
  handleChange: (v: string) => {
    console.log(v);
  },
});
</script>

<style>
/* 隐藏最后一个图标 */
.bytemd-toolbar-icon:last-child {
  display: none;
}
</style>
