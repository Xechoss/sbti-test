<script setup lang="ts">
import { dimensionMeta, dimensionOrder, DIM_EXPLANATIONS, TYPE_IMAGES } from '../data/sbti';
import type { PersonalityType } from '../data/sbti';

const props = defineProps<{
  result: {
    finalType: PersonalityType;
    modeKicker: string;
    badge: string;
    sub: string;
    special: boolean;
    rawScores: Record<string, number>;
    levels: Record<string, string>;
  };
}>();

defineEmits(['restart', 'home']);

const getDimExplanation = (dim: string, level: string) => {
  return DIM_EXPLANATIONS[dim]?.[level] || '';
};

const getDimName = (dim: string) => {
  return dimensionMeta[dim]?.name || dim;
};

const imageSrc = props.result.finalType.code ? TYPE_IMAGES[props.result.finalType.code] : '';
</script>

<template>
  <section id="result">
    <div class="result-wrap card">
      <div class="result-layout">
        <div class="result-top">
          <div :class="['poster-box', { 'no-image': !imageSrc }]">
            <img v-if="imageSrc" :src="imageSrc" class="poster-image" alt="人格结果图" />
            <div class="poster-caption">{{ result.finalType.intro }}</div>
          </div>

          <div class="type-box">
            <div class="type-kicker">{{ result.modeKicker }}</div>
            <div class="type-name">{{ result.finalType.code }}（{{ result.finalType.cn }}）</div>
            <div class="match">{{ result.badge }}</div>
            <div class="type-subname">{{ result.sub }}</div>
          </div>
        </div>

        <div class="analysis-box">
          <h3>该人格的简单解读</h3>
          <p>{{ result.finalType.desc }}</p>
        </div>

        <div class="dim-box">
          <h3>十五维度评分</h3>
          <div class="dim-list">
            <div v-for="dim in dimensionOrder" :key="dim" class="dim-item">
              <div class="dim-item-top">
                <div class="dim-item-name">{{ getDimName(dim) }}</div>
                <div class="dim-item-score">{{ result.levels[dim] || '?' }} / {{ result.rawScores[dim] || 0 }}分</div>
              </div>
              <p>{{ getDimExplanation(dim, result.levels[dim] || '') }}</p>
            </div>
          </div>
        </div>

        <div class="note-box">
          <h3>友情提示</h3>
          <p>
            {{ result.special
              ? '本测试仅供娱乐。隐藏人格和傻乐兜底都属于作者故意埋的损招，请勿把它当成医学、心理学、相学、命理学或灵异学依据。'
              : '本测试仅供娱乐，别拿它当诊断、面试、相亲、分手、招魂、算命或人生判决书。你可以笑，但别太当真。'
            }}
          </p>
        </div>

        <details class="author-box">
          <summary>作者的话</summary>
          <div class="author-content">
            <p>本测试首发于b站up主蛆肉儿串儿（UID417038183），初衷是劝诫一位爱喝酒的朋友戒酒。</p>
            <p>由于作者的人格是SHIT愤世者，所以平等的攻击了各位，在此抱歉！！不过我是一个绝世大美女，你们一定会原谅我，有B站的朋友们也可以关注我。</p>
            <p>关于这个测试，我没法很好的平衡娱乐和专业性，因此对于一些人格的阐释较为模糊或完全不准（如屌丝可能并非真的屌丝），如有冒犯非常抱歉！！</p>
            <p>再鉴于时间精力有限，就随便搞了一个先这样玩玩，后续会慢慢完善修改的，总之好玩为主，还请不要用于盈利呀。</p>
          </div>
        </details>
      </div>

      <div class="result-actions">
        <div style="display:flex; gap:12px; flex-wrap:wrap;">
          <button @click="$emit('restart')" class="btn-secondary">重新测试</button>
          <button @click="$emit('home')" class="btn-primary">回到首页</button>
        </div>
      </div>
    </div>
  </section>
</template>
