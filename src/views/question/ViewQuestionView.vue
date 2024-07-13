<template>
  <div id="viewQuestionView">
    <a-row :gutter="[24, 24]">
      <a-col :md="12" :xs="24">
        <a-tabs default-active-key="question">
          <a-tab-pane key="question" title="题目">
            <a-card :title="question.title" v-if="question">
              <a-descriptions
                title="判题条件"
                :column="{ xs: 1, md: 2, lg: 3 }"
              >
                <a-descriptions-item label="时间限制（ms）">
                  <a-tag>{{ question.judgeConfig.timeLimit ?? 0 }}</a-tag>
                </a-descriptions-item>
                <a-descriptions-item label="内存限制（KB）">
                  <a-tag>{{ question.judgeConfig.memoryLimit ?? 0 }}</a-tag>
                </a-descriptions-item>
                <a-descriptions-item label="堆栈限制（KB）">
                  <a-tag>{{ question.judgeConfig.stackLimit ?? 0 }}</a-tag>
                </a-descriptions-item>
              </a-descriptions>
              <template #extra>
                <a-space fill>
                  <a-tag
                    v-for="(tag, index) of question.tags"
                    :key="index"
                    color="green"
                    bordered
                    >{{ tag }}
                  </a-tag>
                </a-space>
              </template>
              <MdViewer :value="question.content || ''"></MdViewer>
            </a-card>
          </a-tab-pane>
          <a-tab-pane key="comment" title="评论"> 评论区正在开发中</a-tab-pane>
          <a-tab-pane key="answer" title="题解"> 暂时无法查看答案</a-tab-pane>
        </a-tabs>
      </a-col>
      <a-col :md="12" :xs="24">
        <a-form :model="form" layout="inline">
          <a-form-item field="title" label="编程语言" style="min-width: 240px">
            <a-select
              v-model="form.language"
              :style="{ width: '320px' }"
              placeholder="请选择编程语言"
            >
              <a-option>java</a-option>
              <a-option>cpp</a-option>
              <a-option>go</a-option>
            </a-select>
          </a-form-item>
        </a-form>
        <CodeEditor
          :handle-change="changeCode"
          :value="form.code"
          :language="form.language"
        ></CodeEditor>
        <a-divider size="0"></a-divider>
        <a-button @click="doSubmit" type="primary" style="min-width: 200px"
          >提交代码
        </a-button>
      </a-col>
    </a-row>
    <a-modal v-model:visible="visible" @ok="handleOk" @cancel="handleCancel">
      <template #title> {{ title }}</template>
      <div>
        <div>时间：{{ judgeInfo.time }} ms</div>
        <div>内存：{{ judgeInfo.memory }} kb</div>
      </div>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref, withDefaults, defineProps } from "vue";
import MdEditor from "@/components/MdEditor.vue";
import {
  JudgeInfo,
  QuestionAddRequest,
  QuestionControllerService,
  QuestionSubmitAddRequest,
  QuestionSubmitVO,
  QuestionVO,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRoute, useRouter } from "vue-router";
import CodeEditor from "@/components/CodeEditor.vue";
import MdViewer from "@/components/MdViewer.vue";

const router = useRouter();
const route = useRoute();

const visible = ref(false);
const title = ref<string>();
const judgeInfo = ref<JudgeInfo>({
  time: undefined,
  memory: undefined,
});

const handleOk = () => {
  visible.value = false;
};
const handleCancel = () => {
  visible.value = false;
};

interface Props {
  id: string;
}

const props = withDefaults(defineProps<Props>(), {
  id: () => "",
});

const question = ref<QuestionVO>();

/**
 * 根据题目id获取老的数据
 */
const loadData = async () => {
  const res = await QuestionControllerService.getQuestionVoByIdUsingGet(
    props.id as never
  );
  console.log(res);
  if (res.code === 0) {
    question.value = res.data;
  } else {
    message.error("加载失败" + res.message);
  }
};

const form = ref<QuestionSubmitAddRequest>({
  questionId: props.id as never,
  language: "java",
  code: "",
});

const changeCode = (value: string) => {
  form.value.code = value;
};

async function pollSubmitResult(submitId: number) {
  let result: QuestionSubmitVO | undefined;
  const pollInterval = 2000; // 轮询间隔，单位为毫秒。

  // 使用setTimeout和递归调用函数来实现轮询。
  const poll = async () => {
    try {
      const response =
        await QuestionControllerService.getQuestionSubmitByIdUsingGet(submitId);
      if (response.code === 0 && response.data?.status === 2) {
        // 如果code为0，表示评判完成，保存结果并清除定时器。
        result = response.data;
      } else {
        // 如果code不为0，使用setTimeout再次轮询。
        setTimeout(poll, pollInterval);
      }
    } catch (error) {
      console.error("Error polling submit result:", error);
      // 在出现错误时，你可能需要停止轮询或者尝试重新轮询。
      setTimeout(poll, pollInterval);
    }
  };

  // 启动轮询机制。
  poll();

  // 这个函数可以返回一个promise，如果需要的话，可以在结果准备好后resolve。
  return new Promise((resolve, reject) => {
    const checkResult = () => {
      if (result !== undefined) {
        resolve(result);
      } else {
        setTimeout(checkResult, pollInterval);
      }
    };

    checkResult();
  });
}

const doSubmit = async () => {
  if (form.value.code === "") {
    message.error("代码不能为空");
    return;
  }
  const res = await QuestionControllerService.doQuestionSubmitUsingPost(
    form.value
  );
  if (res.code === 0) {
    message.success("提交成功");
    const submitId = res.data;
    pollSubmitResult(submitId).then((result: any) => {
      console.log(result);
      visible.value = true;
      title.value = result.judgeInfo.message;
      judgeInfo.value = result.judgeInfo;
    });
  } else {
    message.error("提交失败, " + res.message);
  }
};

onMounted(() => {
  loadData();
});
</script>

<style scoped>
#viewQuestionView {
}
</style>
