<template>
  <nav v-if="navbarLinks.length" class="navbar-links">
    <div v-for="(item, index) in navbarLinks" :key="index" class="navbar-links__item">
      <template v-if="'children' in item && item.children">
        <DropdownLink :item="item" />
      </template>

      <template v-else>
        <Link :item="(item as MenuLink)" />
      </template>
    </div>
  </nav>
</template>

<script lang="ts" setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { isString, isLinkHttp } from 'vuepress/shared'
import { convertToPinyin } from '@vuepress-reco/shared'
import { useRoutePaths } from '@vuepress/helper/client'
import { useRouteLocale, useSiteData, useSiteLocaleData } from 'vuepress/client'
import { getNavLink, useThemeData, useThemeLocaleData } from '@composables/index.js'
import { useExtendPageData } from '@vuepress-reco/vuepress-plugin-page/composables'

import Link from './Link.vue'
import DropdownLink from './DropdownLink.vue'
import { resolveRepoType } from '@utils/index.js'

import type { ComputedRef } from 'vue'
import type {
  MenuLink,
  MenuGroup,
  MenuLinkGroup,
  AutoAddCategoryToNavbarOptions,
} from '../../types'

/**
 * Get navbar config of select language dropdown
 */
const useNavbarSelectLanguage = (): ComputedRef<MenuGroup<MenuLinkGroup>[]> => {
  const route = useRoute()
  const routePaths = useRoutePaths()
  const routeLocale = useRouteLocale()
  const site = useSiteData()
  const siteLocale = useSiteLocaleData()
  const theme = useThemeData()
  const themeLocal = useThemeLocaleData()

  return computed(() => {
    const localePaths = Object.keys(site.value.locales || {})
    // do not display language selection dropdown if there is only one language
    if (localePaths.length < 2) {
      return []
    }
    const currentPath = route?.path
    const currentFullPath = route?.fullPath

    const languageDropdown: MenuGroup<MenuLink> = {
      icon: 'Translate',
      text: themeLocal.value.selectLanguageText || 'Languages',
      children: localePaths.map((targetLocalePath) => {
        // target locale config of this langauge link
        const targetSiteLocale = site.value.locales?.[targetLocalePath] ?? {}
        const targetThemeLocale = theme.value.locales?.[targetLocalePath] ?? {}
        const targetLang = `${targetSiteLocale.lang}`

        const text = targetThemeLocale.selectLanguageName ?? targetLang
        let link

        if (targetLang === siteLocale.value.lang) {
          // if the target language is current language
          // stay at current link
          link = currentFullPath
        } else {
          // if the target language is not current language
          // try to link to the corresponding page of current page
          // or fallback to homepage
          const targetLocalePage = currentPath?.replace(
            routeLocale.value,
            targetLocalePath
          )
          if (
            routePaths.value.some((item) => item === targetLocalePage)
          ) {
            link = targetLocalePage
          } else {
            link = targetThemeLocale.home ?? targetLocalePath
          }
        }

        return {
          text,
          link,
          language: true
        }
      }),
    }

    return [languageDropdown]
  })
}

/**
 * Get navbar config of repository link
 */
const useNavbarRepo = (): ComputedRef<Array<MenuLink>> => {
  const themeLocal = useThemeLocaleData()

  const repo = computed(() => themeLocal.value.repo || themeLocal.value.docsRepo || '')
  const repoType = computed(() =>
    repo.value ? resolveRepoType(repo.value) : null
  )

  const repoLink = computed(() => {
    if (repoType.value === 'GitHub' && !isLinkHttp(repo.value)) {
      return `https://github.com/${repo.value}`
    }
    return repo.value
  })

  const repoLabel = computed(() => {
    if (!repoLink.value) return null
    if (themeLocal.value.repoLabel) return themeLocal.value.repoLabel
    if (repoType.value === null) return 'Source'
    return repoType.value
  })

  return computed(() => {
    if (!repoLink.value || !repoLabel.value) {
      return []
    }

    return [
      {
        text: repoLabel.value,
        link: repoLink.value,
      },
    ]
  })
}

function resolveNavbarItem(
  item: MenuLink | MenuLinkGroup | string,
): MenuLink | MenuLinkGroup {
  if (isString(item)) {
    return getNavLink(item)
  }

  if ('children' in item && Array.isArray(item.children)) {
    return {
      ...item,
      children: item.children.map(resolveNavbarItem),
    };
  }

  return item as MenuLink
}

const useNavbarConfig = (): ComputedRef<Array<MenuLinkGroup>> => {
  const themeLocal = useThemeLocaleData()
  const { categorySummary } = useExtendPageData()

  const parseCategories = computed(() => {
    return [
      {
        text: (themeLocal.value.autoAddCategoryToNavbar as AutoAddCategoryToNavbarOptions)?.categoryText || 'Categories',
        children: Object.values(categorySummary?.categories?.items || {}).map((c) => ({
          // @ts-ignore
          text: c.label,
          // @ts-ignore
          link: `/categories/${convertToPinyin(c.label)}/1.html`,
        }))
      },
      {
        text: (themeLocal.value.autoAddCategoryToNavbar as AutoAddCategoryToNavbarOptions)?.tagText || 'Tags',
        children: Object.values(categorySummary?.tags?.items || {}).map(t => ({
          // @ts-ignore
          text: t.label,
          // @ts-ignore
          link: `/tags/${convertToPinyin(t.label)}/1.html`,
        }))
      },
    ]
  })

  return computed(() => {
    let navItems = themeLocal.value.navbar || []

    if (themeLocal.value.autoAddCategoryToNavbar) {
      navItems.splice(
        (themeLocal.value.autoAddCategoryToNavbar as AutoAddCategoryToNavbarOptions)?.location || 0,
        0,
        ...parseCategories.value
      )
    }

    return navItems.map((item) => resolveNavbarItem(item as MenuLink | MenuLinkGroup))
  })
}

const navbarConfig = useNavbarConfig()
const navbarSelectLanguage = useNavbarSelectLanguage()
const navbarRepo = useNavbarRepo()

const navbarLinks: ComputedRef<Array<MenuLink | MenuGroup<MenuLinkGroup>>> = computed(() => [
  ...navbarConfig.value,
  ...navbarSelectLanguage.value,
  ...navbarRepo.value,
])
</script>
