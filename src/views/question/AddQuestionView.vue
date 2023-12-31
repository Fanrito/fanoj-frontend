<template>
  <div id="addQuestionView">
    <a-form :model="form">
      <a-form-item field="title" label="标题">
        <a-input v-model="form.title" placeholder="请输入标题" />
      </a-form-item>
      <a-form-item field="tags" label="标签">
        <a-input-tag v-model="form.tags" placeholder="请选择标签" allow-clear />
      </a-form-item>
      <a-form-item field="content" label="题目内容">
        <MdEditor
          :value="form.content"
          :handle-change="onContentChange"
        ></MdEditor>
      </a-form-item>
      <a-form-item field="answer" label="答案">
        <MdEditor
          :value="form.answer"
          :handle-change="onAnswerChange"
        ></MdEditor>
      </a-form-item>

      <a-form-item label="判题配置" :content-flex="false" :merge-props="false">
        <a-space direction="vertical">
          <a-form-item
            style="min-width: 600px"
            label="时间限制(ms)"
            field="judgeConfig.timeLimit"
          >
            <a-input-number
              v-model="form.judgeConfig.timeLimit"
              placeholder="请输入限制时间"
              mode="button"
              size="large"
              class="input-demo"
              :min="0"
              :max="5000"
            />
          </a-form-item>
          <a-form-item
            style="min-width: 600px"
            label="内存限制(KB)"
            field="judgeConfig.memoryLimit"
          >
            <a-input-number
              v-model="form.judgeConfig.memoryLimit"
              placeholder="请输入内存限制"
              mode="button"
              size="large"
              class="input-demo"
              :min="0"
              :max="5000"
            />
          </a-form-item>
          <a-form-item
            style="min-width: 600px"
            label="堆栈限制(KB)"
            field="judgeConfig.stackLimit"
          >
            <a-input-number
              v-model="form.judgeConfig.stackLimit"
              placeholder="请输入堆栈限制"
              mode="button"
              size="large"
              class="input-demo"
              :min="0"
              :max="5000"
            />
          </a-form-item>
        </a-space>
      </a-form-item>

      <a-form-item label="判题配置" :content-flex="false" :merge-props="false">
        <a-space direction="vertical" style="min-width: 480px">
          <a-form-item
            v-for="(judgeCaseItem, index) of form.judgeCase"
            :label="`测试用例-${index}`"
            :key="index"
          >
            <a-space direction="vertical">
              <a-form-item
                :field="`form.judgeCase.input`"
                :label="`输入`"
                :key="index"
              >
                <a-textarea
                  v-model="judgeCaseItem.input"
                  placeholder="请输入用于测试的输入"
                  auto-size
                  style="min-width: 400px"
                />
              </a-form-item>
              <a-form-item
                :field="`form.judgeCase.input`"
                :label="`输出`"
                :key="index"
              >
                <a-textarea
                  v-model="judgeCaseItem.output"
                  placeholder="请输入用于测试的输出"
                  auto-size
                  style="min-width: 400px"
                />
              </a-form-item>
              <a-button
                @click="handleDelete(index)"
                :style="{ marginLeft: '10px' }"
                status="danger"
                >删除
              </a-button>
            </a-space>
          </a-form-item>
          <div>
            <a-button @click="handleAdd" type="outline" status="success"
              >新增测试用例
            </a-button>
          </div>
        </a-space>
      </a-form-item>

      <a-form-item>
        <a-button
          @click="doSubmit"
          type="primary"
          style="min-width: 200px; margin-top: 16px"
          >提交
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from "vue";
import MdEditor from "@/components/MdEditor.vue";
import {
  QuestionAddRequest,
  QuestionControllerService,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRoute, useRouter } from "vue-router";

const router = useRouter();
const route = useRoute();
// 如果页面地址包含update，视为更新页面
const updatePage = route.path.includes("update");

const form = reactive({
  title: "",
  tags: [],
  answer: "",
  content: "",
  judgeConfig: {
    memoryLimit: 1000,
    stackLimit: 1000,
    timeLimit: 1000,
  },
  judgeCase: [
    {
      input: "",
      output: "",
    },
  ],
} as QuestionAddRequest);

/**
 * 根据题目id获取老的数据
 */
const loadData = async () => {
  const id = route.query.id as string;
  if (!id) {
    return;
  }
  const res = await QuestionControllerService.getQuestionByIdUsingGet(
    id as never
  );
  console.log(res);
  if (res.code === 0) {
    Object.assign(form, res.data);
    if (!form.tags) {
      form.tags = [];
    } else {
      form.tags = JSON.parse(form.tags as never);
    }
    if (!form.judgeCase) {
      form.judgeCase = [
        {
          input: "",
          output: "",
        },
      ];
    } else {
      form.judgeCase = JSON.parse(form.judgeCase as never);
    }
    if (!form.judgeConfig) {
      form.judgeConfig = {
        memoryLimit: 1000,
        stackLimit: 1000,
        timeLimit: 1000,
      };
    } else {
      form.judgeConfig = JSON.parse(form.judgeConfig as never);
    }
  } else {
    message.error("加载失败" + res.message);
  }
};

onMounted(() => {
  loadData();
});

/**
 * 新增测试用例
 */
const handleAdd = () => {
  if (form.judgeCase) {
    form.judgeCase.push({
      input: "",
      output: "",
    });
  }
};
/**
 * 删除测试用例
 * @param index
 */
const handleDelete = (index: number) => {
  if (form.judgeCase) {
    form.judgeCase.splice(index, 1);
  }
};

const onContentChange = (value: string) => {
  form.content = value;
};
const onAnswerChange = (value: string) => {
  form.answer = value;
};

const doSubmit = async () => {
  console.log(form);
  if (updatePage) {
    const res = await QuestionControllerService.updateQuestionUsingPost(form);
    if (res.code === 0) {
      message.success("更新成功");
    } else {
      message.error("更新失败" + res.message);
    }
  } else {
    const res = await QuestionControllerService.addQuestionUsingPost(form);
    if (res.code === 0) {
      message.success("创建成功");
    } else {
      message.error("创建失败" + res.message);
    }
  }
};
</script>

<style scoped>
#addQuestionView {
}
</style>
