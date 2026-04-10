<script setup lang="ts">
import { computed } from 'vue';
import type { Question } from '../data/sbti';

const props = defineProps<{
  questions: Question[];
  answers: Record<string, number>;
  previewMode: boolean;
}>();

const emit = defineEmits(['update:answers', 'submit', 'back']);

const dimensionMeta = {
  S1: 'S1 自尊自信',
  S2: 'S2 自我清晰度',
  S3: 'S3 核心价值',
  E1: 'E1 依恋安全感',
  E2: 'E2 情感投入度',
  E3: 'E3 边界与依赖',
  A1: 'A1 世界观倾向',
  A2: 'A2 规则与灵活度',
  A3: 'A3 人生意义感',
  Ac1: 'Ac1 动机导向',
  Ac2: 'Ac2 决策风格',
  Ac3: 'Ac3 执行模式',
  So1: 'So1 社交主动性',
  So2: 'So2 人际边界感',
  So3: 'So3 表达与真实度'
};

const doneCount = computed(() => {
  return props.questions.filter(q => props.answers[q.id] !== undefined).length;
});

const progressPercent = computed(() => {
  return props.questions.length ? (doneCount.value / props.questions.length) * 100 : 0;
});

const isComplete = computed(() => {
  return doneCount.value === props.questions.length && props.questions.length > 0;
});

const getQuestionMetaLabel = (q: Question) => {
  if (q.special) return '补充题';
  return props.previewMode && q.dim ? (dimensionMeta[q.dim as keyof typeof dimensionMeta] || '维度') : '维度已隐藏';
};

const updateAnswer = (questionId: string, value: number) => {
  const newAnswers = { ...props.answers, [questionId]: value };
  emit('update:answers', newAnswers);
};

const getOptionCode = (index: number) => {
  return ['A', 'B', 'C', 'D'][index] || String(index + 1);
};
</script>

<template>
  <section id="test">
    <div class="test-wrap card">
      <div class="topbar">
        <div class="progress">
          <span :style="{ width: progressPercent + '%' }"></span>
        </div>
        <div class="progress-text">{{ doneCount }} / {{ questions.length }}</div>
      </div>

      <div class="question-list">
        <article v-for="(q, index) in questions" :key="q.id" class="question">
          <div class="question-meta">
            <div class="badge">第 {{ index + 1 }} 题</div>
            <div>{{ getQuestionMetaLabel(q) }}</div>
          </div>
          <div class="question-title">{{ q.text }}</div>
          <div class="options">
            <label
              v-for="(opt, optIdx) in q.options"
              :key="optIdx"
              class="option"
            >
              <input
                type="radio"
                :name="q.id"
                :value="opt.value"
                :checked="answers[q.id] === opt.value"
                @change="updateAnswer(q.id, opt.value)"
              />
              <div class="option-code">{{ getOptionCode(optIdx) }}</div>
              <div>{{ opt.label }}</div>
            </label>
          </div>
        </article>
      </div>

      <div class="actions-bottom">
        <div class="hint">
          {{ isComplete ? '都做完了。现在可以把你的电子魂魄交给结果页审判。' : '全选完才会放行。世界已经够乱了，起码把题做完整。' }}
        </div>
        <div style="display:flex; gap:12px; flex-wrap:wrap;">
          <button @click="$emit('back')" class="btn-secondary">返回首页</button>
          <button @click="$emit('submit')" class="btn-primary" :disabled="!isComplete">提交并查看结果</button>
        </div>
      </div>
    </div>
  </section>
</template>
