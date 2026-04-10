<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import SBTIIntro from './components/SBTIIntro.vue';
import SBTITest from './components/SBTITest.vue';
import SBTIResult from './components/SBTIResult.vue';
import {
  questions as allQuestions,
  specialQuestions,
  dimensionMeta,
  dimensionOrder,
  TYPE_LIBRARY,
  NORMAL_TYPES
} from './data/sbti';
import type { Question, PersonalityType, ResultData } from './data/sbti';
import './assets/main.css';

const currentScreen = ref<'intro' | 'test' | 'result'>('intro');
const shuffledQuestions = ref<Question[]>([]);
const answers = ref<Record<string, number>>({});
const previewMode = ref(false);

const visibleQuestions = computed(() => {
  const visible = [...shuffledQuestions.value];
  const gateIndex = visible.findIndex(q => q.id === 'drink_gate_q1');
  if (gateIndex !== -1 && answers.value['drink_gate_q1'] === 3) {
    visible.splice(gateIndex + 1, 0, specialQuestions[1]!);
  }
  return visible;
});

function shuffle<T>(array: T[]): T[] {
  const arr = [...array];
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    const temp = arr[i]!;
    arr[i] = arr[j]!;
    arr[j] = temp;
  }
  return arr;
}

function startTest(preview = false) {
  previewMode.value = preview;
  answers.value = {};
  const shuffledRegular = shuffle(allQuestions);
  const insertIndex = Math.floor(Math.random() * shuffledRegular.length) + 1;
  shuffledQuestions.value = [
    ...shuffledRegular.slice(0, insertIndex),
    specialQuestions[0]!,
    ...shuffledRegular.slice(insertIndex)
  ];
  currentScreen.value = 'test';
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

function sumToLevel(score: number) {
  if (score <= 3) return 'L';
  if (score === 4) return 'M';
  return 'H';
}

function levelNum(level: string) {
  return { L: 1, M: 2, H: 3 }[level as 'L' | 'M' | 'H'] || 0;
}

function parsePattern(pattern: string) {
  return pattern.replace(/-/g, '').split('');
}

const resultData = ref<ResultData | null>(null);

function computeResult() {
  const rawScores: Record<string, number> = {};
  const levels: Record<string, string> = {};
  Object.keys(dimensionMeta).forEach(dim => { rawScores[dim] = 0; });

  allQuestions.forEach(q => {
    if (q.dim) {
      rawScores[q.dim] = (rawScores[q.dim] || 0) + Number(answers.value[q.id] || 0);
    }
  });

  Object.entries(rawScores).forEach(([dim, score]) => {
    levels[dim] = sumToLevel(score);
  });

  const userVector = dimensionOrder.map(dim => levelNum(levels[dim]!));
  const ranked = NORMAL_TYPES.map(type => {
    const vector = parsePattern(type.pattern).map(levelNum);
    let distance = 0;
    let exact = 0;
    for (let i = 0; i < vector.length; i++) {
      const diff = Math.abs(userVector[i]! - vector[i]!);
      distance += diff;
      if (diff === 0) exact += 1;
    }
    const similarity = Math.max(0, Math.round((1 - distance / 30) * 100));
    return { ...type, ...TYPE_LIBRARY[type.code]!, distance, exact, similarity };
  }).sort((a, b) => {
    if (a.distance !== b.distance) return a.distance - b.distance;
    if (b.exact !== a.exact) return b.exact - a.exact;
    return b.similarity - a.similarity;
  });

  const bestNormal = ranked[0]!;
  const drunkTriggered = answers.value['drink_gate_q2'] === 2;

  let finalType: PersonalityType;
  let modeKicker = '你的主类型';
  let badge = `匹配度 ${bestNormal.similarity}% · 精准命中 ${bestNormal.exact}/15 维`;
  let sub = '维度命中度较高，当前结果可视为你的第一人格画像。';
  let special = false;

  if (drunkTriggered) {
    finalType = TYPE_LIBRARY['DRUNK']!;
    modeKicker = '隐藏人格已激活';
    badge = '匹配度 100% · 酒精异常因子已接管';
    sub = '乙醇亲和性过强，系统已直接跳过常规人格审判。';
    special = true;
  } else if (bestNormal.similarity < 60) {
    finalType = TYPE_LIBRARY['HHHH']!;
    modeKicker = '系统强制兜底';
    badge = `标准人格库最高匹配仅 ${bestNormal.similarity}%`;
    sub = '标准人格库对你的脑回路集体罢工了，于是系统把你强制分配给了 HHHH。';
    special = true;
  } else {
    finalType = bestNormal;
  }

  resultData.value = {
    rawScores,
    levels,
    finalType,
    modeKicker,
    badge,
    sub,
    special
  };
  currentScreen.value = 'result';
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

// Ensure special logic for drinking question
watch(() => answers.value['drink_gate_q1'], (newVal) => {
  if (newVal !== 3) {
    delete answers.value['drink_gate_q2'];
  }
});
</script>

<template>
  <div class="shell">
    <SBTIIntro
      v-if="currentScreen === 'intro'"
      @start="startTest(false)"
    />
    <SBTITest
      v-else-if="currentScreen === 'test'"
      :questions="visibleQuestions"
      v-model:answers="answers"
      :preview-mode="previewMode"
      @submit="computeResult"
      @back="currentScreen = 'intro'"
    />
    <SBTIResult
      v-else-if="currentScreen === 'result' && resultData"
      :result="resultData"
      @restart="startTest(false)"
      @home="currentScreen = 'intro'"
    />
  </div>
</template>
