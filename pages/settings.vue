<script lang="ts" setup>
definePageMeta({
  wideLayout: true,
})

const { t } = useI18n()

useHeadFixed({
  title: () => t('nav.settings'),
})

const route = useRoute()

const isRootPath = computedEager(() => route.name === 'settings')
</script>

<template>
  <div>
    <div min-h-screen flex>
      <div border="e base" :class="isRootPath ? 'block lg:flex-none flex-1' : 'hidden lg:block'">
        <MainContent>
          <template #title>
            <div timeline-title-style flex items-center gap-2 @click="$scrollToTop">
              <div i-ri:settings-3-line />
              <span>{{ $t('nav.settings') }}</span>
            </div>
          </template>
          <div xl:w-97 lg:w-78 w-full>
            <SettingsItem
              v-if="isHydrated && currentUser "
              command
              icon="i-ri:user-line"
              :text="$t('settings.profile.label')"
              to="/settings/profile"
            />
            <SettingsItem
              command
              icon="i-ri-compasses-2-line"
              :text="$t('settings.interface.label')"
              to="/settings/interface"
            />
            <SettingsItem
              command
              icon="i-ri-leaf-line"
              :text="$t('settings.wellness.label')"
              to="/settings/wellness"
            />
            <SettingsItem
              v-if="isHydrated && currentUser"
              command
              icon="i-ri:notification-badge-line"
              :text="$t('settings.notifications_settings')"
              to="/settings/notifications"
            />
            <SettingsItem
              command
              icon="i-ri-globe-line"
              :text="$t('settings.language.label')"
              to="/settings/language"
            />
            <SettingsItem
              command
              icon="i-ri-equalizer-line"
              :text="$t('settings.preferences.label')"
              to="/settings/preferences"
            />
            <SettingsItem
              command
              icon="i-ri-group-line"
              :text="$t('settings.users.label')"
              to="/settings/users"
            />
            <SettingsItem
              command
              icon="i-ri:information-line"
              :text="$t('settings.about.label')"
              to="/settings/about"
            />
          </div>
        </MainContent>
      </div>
      <div flex-1 :class="isRootPath ? 'hidden lg:block' : 'block'">
        <NuxtPage />
      </div>
    </div>
  </div>
</template>
