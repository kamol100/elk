<script setup lang="ts">
import type { UserLogin } from '~/types'

const emit = defineEmits<{
  (event: 'click'): void
}>()

const all = useUsers()

const sorted = computed(() => {
  return [
    currentUser.value!,
    ...all.value.filter(account => account.token !== currentUser.value?.token),
  ].filter(Boolean)
})

const router = useRouter()
const masto = useMasto()
const switchUser = (user: UserLogin) => {
  if (user.account.id === currentUser.value?.account.id)
    router.push(getAccountRoute(user.account))
  else
    masto.loginTo(user)
}
</script>

<template>
  <div sm:min-w-80 max-w-100vw mxa py2 flex="~ col" @click="emit('click')">
    <template v-for="user of sorted" :key="user.id">
      <button
        flex rounded px4 py3 text-left
        hover:bg-active cursor-pointer transition-100
        aria-label="Switch user"
        @click="switchUser(user)"
      >
        <AccountInfo :account="user.account" :hover-card="false" square />
        <div flex-auto />
        <div v-if="user.token === currentUser?.token" i-ri:check-line text-primary mya text-2xl />
      </button>
    </template>
    <div border="t base" pt2>
      <CommonDropdownItem
        :text="$t('user.add_existing')"
        icon="i-ri:user-add-line"
        @click="openSigninDialog"
      />
      <CommonDropdownItem
        v-if="isMastoInitialised && currentUser"
        :text="$t('user.sign_out_account', [getFullHandle(currentUser.account)])"
        icon="i-ri:logout-box-line rtl-flip"
        @click="signout"
      />
    </div>
  </div>
</template>
