<template>
  <div v-on-clickaway="closePortalPopover" class="portal-popover__container">
    <header>
      <div class="actions">
        <h2 class="block-title">
          {{ $t('HELP_CENTER.PORTAL.POPOVER.TITLE') }}
        </h2>
        <div>
          <woot-button
            variant="smooth"
            color-scheme="secondary"
            icon="settings"
            size="small"
            @click="openPortalPage"
          >
            {{ $t('HELP_CENTER.PORTAL.POPOVER.PORTAL_SETTINGS') }}
          </woot-button>
          <woot-button
            variant="clear"
            color-scheme="secondary"
            icon="dismiss"
            size="small"
            @click="closePortalPopover"
          />
        </div>
      </div>
      <p class="subtitle">
        {{ $t('HELP_CENTER.PORTAL.POPOVER.SUBTITLE') }}
      </p>
    </header>
    <div>
      <portal-switch
        v-for="portal in portals"
        :key="portal.id"
        :portal="portal"
        :active-portal-slug="activePortalSlug"
        :active="portal.slug === activePortalSlug"
        @open-portal-page="onPortalSelect"
      />
    </div>
    <footer>
      <woot-button variant="link" @click="closePortalPopover">
        {{ $t('HELP_CENTER.PORTAL.POPOVER.CANCEL_BUTTON_LABEL') }}
      </woot-button>
      <woot-button @click="() => {}">
        {{ $t('HELP_CENTER.PORTAL.POPOVER.CHOOSE_LOCALE_BUTTON') }}
      </woot-button>
    </footer>
  </div>
</template>

<script>
import { mixin as clickaway } from 'vue-clickaway';
import PortalSwitch from './PortalSwitch.vue';
export default {
  components: {
    PortalSwitch,
  },
  mixins: [clickaway],
  props: {
    portals: {
      type: Array,
      default: () => [],
    },
    activePortalSlug: {
      type: String,
      default: '',
    },
  },

  methods: {
    closePortalPopover() {
      this.$emit('close-popover');
    },
    onPortalSelect() {
      this.$emit('close-popover');
    },
    openPortalPage() {
      this.closePortalPopover();
      this.$router.push({
        name: 'list_all_portals',
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.portal-popover__container {
  position: absolute;
  overflow-y: scroll;
  max-height: 96vh;
  padding: var(--space-normal);
  background-color: var(--white);
  border-radius: var(--border-radius-normal);
  box-shadow: var(--shadow-large);
  max-width: 48rem;
  z-index: var(--z-index-high);

  header {
    .actions {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: var(--space-normal);

      .new-popover-link {
        display: flex;
        align-items: center;
        padding: var(--space-half) var(--space-one);
        background-color: var(--s-25);
        font-size: var(--font-size-mini);
        color: var(--s-500);
        margin-left: var(--space-small);
        border-radius: var(--border-radius-normal);
        span {
          margin-left: var(--space-small);
        }
      }

      .subtitle {
        font-size: var(--font-size-mini);
        color: var(--s-600);
        margin-top: var(--space-small);
      }
    }
  }

  footer {
    display: flex;
    justify-content: end;
    align-items: center;
    gap: var(--space-small);
  }
}
</style>
