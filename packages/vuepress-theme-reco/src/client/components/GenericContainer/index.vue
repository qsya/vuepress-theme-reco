<template>
  <div v-if="!setedSitePassword" class="theme-container" :class="containerClass">
    <Navbar
      :class="{
        'not-open': !isOpenSeries,
        'navbar-container--has-series': isMobile && hasSeries
      }"
      @toggleMenus="toggleMobileMenus"
    />

    <SubNavbar
      v-if="hasSeries"
      :class="{
        'not-open': !isOpenSeries,
      }"
      @toggleSeries="toggleSeries"
    />

    <NavbarDropdownNemu />

    <Transition name="fade">
      <div v-if="isOpenSeries" class="series-mask" @click="toggleSeries(false)" />
    </Transition>

    <Series />

    <slot />

    <Catalog v-if="isShowCatalog" />

  </div>

  <div v-else class="theme-container" :class="containerClass">
    <Password v-if="siteLoaded && !sitePasswordPass" class="out" key="out" @pass="handlePass" />

    <div v-if="siteLoaded && sitePasswordPass">
      <Navbar
        :class="{
          'not-open': !isOpenSeries,
          'navbar-container__mobile': hasSeries
        }"
        @toggleMenus="toggleMobileMenus"
      />

      <SubNavbar
        v-if="hasSeries"
        :class="{
          'not-open': !isOpenSeries,
        }"
        @toggleSeries="toggleSeries"
      />

      <NavbarDropdownNemu />

      <Transition name="fade">
        <div v-if="isOpenSeries" class="series-mask" @click="toggleSeries(false)" />
      </Transition>

      <Series />

      <slot />

      <Catalog v-if="isShowCatalog" />
    </div>
  </div>
</template>

<script lang="ts" setup>
import Navbar from '../Navbar/index.vue'
import Series from '../Series/index.vue'
import Catalog from '../Catalog.vue'
import SubNavbar from '../SubNavbar.vue'
import Password from '../Password/index.vue'
import NavbarDropdownNemu from '../NavbarDropdownNemu.vue'

import { useSeriesItems } from '@composables/index.js'
const seriesItems = useSeriesItems()
const hasSeries = computed(() => seriesItems.value.length > 0)

import { useSeriesData } from '@composables/index.js'
const { isOpenSeries, toggleSeries } = useSeriesData()

import { usePageCatalog } from '@composables/index.js'
const { isShowCatalog } = usePageCatalog()

import { useMobileMenus } from '@composables/index.js'
const { toggleMobileMenus } = useMobileMenus()

import { useMobile } from '@composables/index.js'
const { isMobile } = useMobile()

// Handle password
import { usePassword } from './usePassword.js'
const { siteLoaded, sitePasswordPass, setedSitePassword, handlePass } = usePassword()

// Format className of container
import { useContainerClass } from './useContainerClass.js'
const { containerClass } = useContainerClass()

// Init code copy button
import { useInitCodeCopy } from './useInitCodeCopy.js'
const { initCodeCopy } = useInitCodeCopy()
initCodeCopy()

// Init series status
import { useSeries } from './useSeries.js'
import { computed } from 'vue'
const { initSeriesStatus } = useSeries()
initSeriesStatus(() => {
  toggleSeries(false)
  toggleMobileMenus(false)
})

</script>
