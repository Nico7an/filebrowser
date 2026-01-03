<template>
  <header>
    <!-- Logo and Menu -->
    <div style="display: flex; align-items: center; gap: 16px">
      <Action
        v-if="showMenu"
        class="menu-button"
        icon="menu"
        :label="t('buttons.toggleSidebar')"
        @action="layoutStore.showHover('sidebar')"
      />
      <div v-if="showLogo" style="display: flex; align-items: center; gap: 8px">
        <img :src="logoURL" style="height: 40px" />
        <span style="font-size: 22px; color: var(--textPrimary); font-weight: 400">Drive</span>
      </div>
    </div>

    <!-- Search Bar - Google Drive Style -->
    <div class="gdrive-search" style="flex: 1; max-width: 720px">
      <i class="material-icons">search</i>
      <slot />
    </div>

    <!-- Right side actions -->
    <div style="display: flex; align-items: center; gap: 4px">
      <slot name="actions" />
    </div>

    <!-- Dropdown for more actions -->
    <div
      id="dropdown"
      :class="{ active: layoutStore.currentPromptName === 'more' }"
    >
    </div>

    <Action
      v-if="ifActionsSlot"
      id="more"
      icon="more_vert"
      :label="t('buttons.more')"
      @action="layoutStore.showHover('more')"
    />

    <div
      class="overlay"
      v-show="layoutStore.currentPromptName == 'more'"
      @click="layoutStore.closeHovers"
    />
  </header>
</template>

<script setup lang="ts">
import { useLayoutStore } from "@/stores/layout";

import { logoURL } from "@/utils/constants";

import Action from "@/components/header/Action.vue";
import { computed, useSlots } from "vue";
import { useI18n } from "vue-i18n";

defineProps<{
  showLogo?: boolean;
  showMenu?: boolean;
}>();

const layoutStore = useLayoutStore();
const slots = useSlots();

const { t } = useI18n();

const ifActionsSlot = computed(() => (slots.actions ? true : false));
</script>

<style></style>
