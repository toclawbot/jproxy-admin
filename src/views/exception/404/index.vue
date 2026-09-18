<template>
  <div class="content">
    <a-result
      class="result"
      status="404"
      :subtitle="$t('exception.result.404.description')"
    >
    </a-result>
    <div class="operation-row">
      <a-button key="again" style="margin-right: 16px" @click="again">
        {{ $t('exception.result.404.retry') }}
      </a-button>
      <a-button key="back" type="primary" @click="back">
        {{ $t('exception.result.404.back') }}
      </a-button>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { useRouter } from 'vue-router';
  import { DEFAULT_ROUTE_NAME } from '@/router/constants';

  const router = useRouter();
  const again = () => {
    router.go(0);
  };
  const back = () => {
    router.push({ name: DEFAULT_ROUTE_NAME });
  };
</script>

<style scoped lang="less">
  .content {
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -95px;
    margin-top: -200px;
    text-align: center;
  }

  /* 移动端响应式优化 */
  @media (max-width: 768px) {
    .content {
      position: static;
      margin: 0 auto;
      padding: 48px 24px;
      transform: none;
      
      :deep(.arco-result) {
        padding: 24px 16px;
      }
      
      :deep(.arco-result-title) {
        font-size: 64px;
      }
      
      :deep(.arco-result-subtitle) {
        font-size: 16px;
        padding: 0 16px;
      }
    }
    
    .operation-row {
      :deep(.arco-btn) {
        width: 100%;
        margin: 8px 0 !important;
      }
    }
  }
  
  @media (max-width: 480px) {
    .content {
      padding: 32px 16px;
      
      :deep(.arco-result-title) {
        font-size: 48px;
      }
      
      :deep(.arco-result-subtitle) {
        font-size: 14px;
      }
    }
  }
</style>
