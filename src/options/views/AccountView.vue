<template>
  <div :class="$style['account-view']">
    <h2 :class="$style.title">API Configuration</h2>
    <div :class="$style['credit-wrap']">
      <CreditOpenAI />
    </div>
  </div>
</template>
<script setup lang="ts">
import CreditOpenAI from '../components/CreditOpenAI.vue'
</script>

<style lang="scss" module>
.credit-wrap {
  margin-bottom: 64px;
}

.credit-radio-group {
  margin-bottom: 24px;
}

.account-view {
  width: 465px;
  color: var(--color-baseline-text);
}

.title {
  margin: 0;
  margin-bottom: 24px;
  font-weight: 600;
  font-size: 24px;
}
</style>
