<script setup lang="ts">
import type { mastodon } from 'masto'

const { account } = defineProps<{
  account: mastodon.v1.Account
  command?: boolean
}>()

const masto = useMasto()

const { t } = useI18n()

const createdAt = $(useFormattedDateTime(() => account.createdAt, {
  month: 'long',
  day: 'numeric',
  year: 'numeric',
}))

const relationship = $(useRelationship(account))

const namedFields = ref<mastodon.v1.AccountField[]>([])
const iconFields = ref<mastodon.v1.AccountField[]>([])

function getFieldIconTitle(fieldName: string) {
  return fieldName === 'Joined' ? t('account.joined') : fieldName
}

function previewHeader() {
  openMediaPreview([{
    id: `${account.acct}:header`,
    type: 'image',
    previewUrl: account.header,
    description: t('account.profile_description', [account.username]),
  }])
}

function previewAvatar() {
  openMediaPreview([{
    id: `${account.acct}:avatar`,
    type: 'image',
    previewUrl: account.avatar,
    description: t('account.avatar_description', [account.username]),
  }])
}

async function toggleNotify() {
  relationship!.notifying = !relationship!.notifying
  try {
    const newRel = await masto.v1.accounts.follow(account.id, { notify: relationship!.notifying })
    Object.assign(relationship!, newRel)
  }
  catch {
    // TODO error handling
    relationship!.notifying = !relationship!.notifying
  }
}

watchEffect(() => {
  const named: mastodon.v1.AccountField[] = []
  const icons: mastodon.v1.AccountField[] = []

  account.fields?.forEach((field) => {
    const icon = getAccountFieldIcon(field.name)
    if (icon)
      icons.push(field)
    else
      named.push(field)
  })
  icons.push({
    name: 'Joined',
    value: createdAt,
  })

  namedFields.value = named
  iconFields.value = icons
})

const isSelf = $computed(() => currentUser.value?.account.id === account.id)
</script>

<template>
  <div flex flex-col>
    <button border="b base" z-1>
      <img h-50 height="200" w-full object-cover :src="account.header" :alt="t('account.profile_description', [account.username])" @click="previewHeader">
    </button>
    <div p4 mt--18 flex flex-col gap-4>
      <div relative>
        <div flex="~ col gap-2 1">
          <button :class="{ 'rounded-full': !isSelf, 'squircle': isSelf }" w-30 h-30 p1 bg-base border-bg-base z-2 @click="previewAvatar">
            <AccountAvatar :square="isSelf" :account="account" hover:opacity-90 transition-opacity />
          </button>
          <div flex="~ col gap1">
            <div flex justify-between>
              <AccountDisplayName :account="account" font-bold sm:text-2xl text-xl />
              <AccountBotIndicator v-if="account.bot" show-label />
            </div>
            <AccountHandle :account="account" />
          </div>
        </div>
        <div absolute top-18 inset-ie-0 flex gap-2 items-center>
          <AccountMoreButton :account="account" :command="command" />

          <button v-if="!isSelf && relationship?.following" flex gap-1 items-center w-full rounded op75 hover="op100 text-pink" group @click="toggleNotify()">
            <div rounded-full p2 group-hover="bg-pink/10">
              <div v-if="relationship?.notifying" i-ri:bell-fill />
              <div v-else i-ri-bell-line />
            </div>
          </button>

          <AccountFollowButton :account="account" :command="command" />
          <!-- Edit profile -->
          <NuxtLink
            v-if="isSelf"
            to="/settings/profile/appearance"
            gap-1 items-center border="1" rounded-full flex="~ gap2 center" font-500 min-w-30 h-fit px3 py1
            hover="border-primary text-primary bg-active"
          >
            {{ $t('settings.profile.appearance.title') }}
          </NuxtLink>
        </div>
      </div>
      <div v-if="account.note" max-h-100 overflow-y-auto>
        <ContentRich text-4 text-base :content="account.note" :emojis="account.emojis" />
      </div>
      <div v-if="namedFields.length" flex="~ col wrap gap1">
        <div v-for="field in namedFields" :key="field.name" flex="~ gap-1" items-center>
          <div text-secondary uppercase text-xs font-bold>
            {{ field.name }} |
          </div>
          <ContentRich :content="field.value" :emojis="account.emojis" />
        </div>
      </div>
      <div v-if="iconFields.length" flex="~ wrap gap-4">
        <div v-for="field in iconFields" :key="field.name" flex="~ gap-1" items-center>
          <CommonTooltip :content="getFieldIconTitle(field.name)">
            <div text-secondary :class="getAccountFieldIcon(field.name)" :title="getFieldIconTitle(field.name)" />
          </CommonTooltip>
          <ContentRich text-sm filter-saturate-0 :content="field.value" :emojis="account.emojis" />
        </div>
      </div>
      <AccountPostsFollowers :account="account" />
    </div>
  </div>
</template>
