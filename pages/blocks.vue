<script setup lang="ts">
definePageMeta({
  middleware: 'auth',
})

const { t } = useI18n()

useHeadFixed({
  title: () => t('nav.blocked_users'),
})
</script>

<template>
  <MainContent back>
    <template #title>
      <span timeline-title-style>{{ $t('nav.blocked_users') }}</span>
    </template>

    <TimelineBlocks v-if="isMastoInitialised" />
  </MainContent>
</template>
