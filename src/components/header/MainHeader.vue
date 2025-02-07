<template>
  <header class='desktop1'>
    <img :src='lightLogo' class='attachment-large size-large logo' @click='onLogoClick'>
    <div class='nav'>
      <ul>
        <li><a class='nav-link' href='#'>{{ $t('MSG_HOME') }}</a></li>
        <li><a class='nav-link' href='https://procyon-vip.medium.com'>{{ $t('MSG_BLOG') }}</a></li>
        <li><a class='nav-link' href='#/faqs'>{{ $t('MSG_SUPPORT_AND_FAQ') }}</a></li>
        <li><a class='nav-link' href='#/contact'>{{ $t('MSG_CONTACT') }}</a></li>
        <LangSwitcher />
        <SignHelper v-if='!logined.getLogined' />
        <q-btn
          v-else
          size='1.1rem'
          flat dense round rounded
          :icon='"img:" + userAvatar'
          class='user-icon'
        >
          <q-menu
            self='top right'
            anchor='bottom right'
            transition-show='jump-down'
            transition-hide='jump-up'
          >
            <q-list class='dropdown'>
              <ExpandList
                :menu='menu'
                :show-icon='true'
                :show-icon-right='true'
                :show-label='false'
                :handle-router='false'
                :margin='true'
                @switch-menu='onSwitchMenu'
              />
            </q-list>
          </q-menu>
        </q-btn>
      </ul>
    </div>
  </header>

  <header class='mobile'>
    <img :src='logo' class='attachment-large size-large logo'>

    <div class='header-inner'>
      <LangSwitcher />
      <SignHelper v-if='!logined.getLogined' />
      <q-btn
        v-else
        size='1.1rem'
        flat dense round rounded
        :icon='"img:" + userAvatar'
        class='user-icon'
      >
        <q-menu
          self='top right'
          anchor='bottom right'
          transition-show='jump-down'
          transition-hide='jump-up'
        >
          <q-list class='dropdown'>
            <ExpandList
              :menu='menu'
              :show-icon='true'
              :show-icon-right='true'
              :show-label='false'
              :handle-router='false'
              :margin='true'
              @switch-menu='onSwitchMenu'
            />
          </q-list>
        </q-menu>
      </q-btn>
    </div>
    <div class='hr' />
    <div class='nav'>
      <ul>
        <li><a class='nav-link' href='#'>{{ $t('MSG_HOME') }}</a></li>
        <li><a class='nav-link' href='https://procyon-vip.medium.com'>{{ $t('MSG_BLOG') }}</a></li>
        <li><a class='nav-link' href='#/faqs'>{{ $t('MSG_SUPPORT_AND_FAQ') }}</a></li>
        <li><a class='nav-link' href='#/contact'>{{ $t('MSG_CONTACT') }}</a></li>
      </ul>
    </div>
  </header>
</template>

<script setup lang='ts'>
import { useInspireStore, useLoginedUserStore, NotificationType, useUserStore } from 'npool-cli-v2'
import { defineAsyncComponent, computed, watch, onMounted } from 'vue'
import { HeaderAvatarMenu, MenuItem } from 'src/menus/menus'
import { useRouter } from 'vue-router'

import lightLogo from '../../assets/procyon-light.svg'
import logo from '../../assets/procyon-logo.svg'
import { useI18n } from 'vue-i18n'

import userAvatar from '../../assets/icon-user.svg'

const LangSwitcher = defineAsyncComponent(() => import('src/components/lang/LangSwitcher.vue'))
const SignHelper = defineAsyncComponent(() => import('src/components/header/SignHelper.vue'))
const ExpandList = defineAsyncComponent(() => import('src/components/list/ExpandList.vue'))

// eslint-disable-next-line @typescript-eslint/unbound-method
const { t } = useI18n({ useScope: 'global' })

const logined = useLoginedUserStore()
const inspire = useInspireStore()
const user = useUserStore()

const router = useRouter()

const onSwitchMenu = (item: MenuItem) => {
  if (item.label === 'MSG_LOGOUT') {
    user.logout({
      Message: {
        Error: {
          Title: t('MSG_LOGOUT_FAIL'),
          Popup: true,
          Type: NotificationType.Error
        }
      }
    })
    void router.push({ path: '/' })
    return
  }

  void router.push({
    path: item.target,
    query: {
      menuId: item.menuId
    }
  })
}

const menu = computed(() => {
  const myMenu = HeaderAvatarMenu()
  myMenu.children = myMenu.children.filter((m) => m.label !== 'MSG_REFERRAL' || inspire.InvitationCode?.InvitationCode?.length)
  return myMenu
})

const onLogoClick = () => {
  void router.push({ path: '/' })
}

const userLogined = computed(() => logined.getLogined)

watch(userLogined, () => {
  if (!userLogined.value) {
    return
  }
  setTimeout(() => {
    initialize()
  }, 2000)
})

const initialize = () => {
  inspire.getInvitationCode({
    Message: {}
  }, () => {
    if (inspire.InvitationCode?.InvitationCode?.length) {
      inspire.getReferrals({
        Message: {}
      }, () => {
        // TODO
      })
      inspire.getPurchaseAmountSettings({
        Message: {}
      }, () => {
        // TODO
      })
    }
  })
  user.getLoginHistories({
    Message: {}
  }, () => {
    // TODO
  })
}

onMounted(() => {
  if (userLogined.value) {
    setTimeout(() => {
      initialize()
    }, 2000)
  }
})

</script>

<style lang='sass' scoped>
.dropdown
  min-width: 200px
</style>
