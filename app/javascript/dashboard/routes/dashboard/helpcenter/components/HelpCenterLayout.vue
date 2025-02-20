<template>
  <div class="row app-wrapper">
    <sidebar
      :route="currentRoute"
      @open-notification-panel="openNotificationPanel"
      @open-key-shortcut-modal="toggleKeyShortcutModal"
      @close-key-shortcut-modal="closeKeyShortcutModal"
    />
    <div v-if="portals.length">
      <help-center-sidebar
        :class="sidebarClassName"
        :header-title="headerTitle"
        :sub-title="localeName(selectedPortalLocale)"
        :accessible-menu-items="accessibleMenuItems"
        :additional-secondary-menu-items="additionalSecondaryMenuItems"
        @open-popover="openPortalPopover"
        @open-modal="onClickOpenAddCatogoryModal"
      />
    </div>
    <section class="app-content columns" :class="contentClassName">
      <router-view />
      <command-bar />
      <woot-key-shortcut-modal
        v-if="showShortcutModal"
        @close="closeKeyShortcutModal"
        @clickaway="closeKeyShortcutModal"
      />
      <notification-panel
        v-if="showNotificationPanel"
        @close="closeNotificationPanel"
      />
      <portal-popover
        v-if="showPortalPopover"
        :portals="portals"
        :active-portal-slug="selectedPortalSlug"
        @close-popover="closePortalPopover"
      />
      <add-category
        v-if="showAddCategoryModal"
        :show.sync="showAddCategoryModal"
        :portal-name="selectedPortalName"
        :locale="selectedPortalLocale"
        @cancel="onClickCloseAddCategoryModal"
      />
    </section>
  </div>
</template>
<script>
import { mapGetters } from 'vuex';

import { frontendURL } from '../../../../helper/URLHelper';
import Sidebar from 'dashboard/components/layout/Sidebar';
import { BUS_EVENTS } from 'shared/constants/busEvents';
import PortalPopover from '../components/PortalPopover.vue';
import HelpCenterSidebar from '../components/Sidebar/Sidebar.vue';
import CommandBar from 'dashboard/routes/dashboard/commands/commandbar.vue';
import WootKeyShortcutModal from 'dashboard/components/widgets/modal/WootKeyShortcutModal';
import NotificationPanel from 'dashboard/routes/dashboard/notifications/components/NotificationPanel';
import portalMixin from '../mixins/portalMixin';
import AddCategory from '../pages/categories/AddCategory';

export default {
  components: {
    Sidebar,
    HelpCenterSidebar,
    CommandBar,
    WootKeyShortcutModal,
    NotificationPanel,
    PortalPopover,
    AddCategory,
  },
  mixins: [portalMixin],
  data() {
    return {
      isSidebarOpen: false,
      isOnDesktop: true,
      showShortcutModal: false,
      showNotificationPanel: false,
      showPortalPopover: false,
      showAddCategoryModal: false,
      lastActivePortalSlug: '',
    };
  },

  computed: {
    ...mapGetters({
      accountId: 'getCurrentAccountId',
      portals: 'portals/allPortals',
      categories: 'categories/allCategories',
      meta: 'portals/getMeta',
      isFetching: 'portals/isFetchingPortals',
    }),
    selectedPortal() {
      const slug = this.$route.params.portalSlug || this.lastActivePortalSlug;
      if (slug) return this.$store.getters['portals/portalBySlug'](slug);

      return this.$store.getters['portals/allPortals'][0];
    },
    sidebarClassName() {
      if (this.isOnDesktop) {
        return '';
      }
      if (this.isSidebarOpen) {
        return 'off-canvas is-open';
      }
      return 'off-canvas is-transition-push is-closed';
    },
    contentClassName() {
      if (this.isOnDesktop) {
        return '';
      }
      if (this.isSidebarOpen) {
        return 'off-canvas-content is-open-left has-transition-push';
      }
      return 'off-canvas-content has-transition-push';
    },
    selectedPortalName() {
      return this.selectedPortal ? this.selectedPortal.name : '';
    },
    selectedPortalSlug() {
      return this.selectedPortal ? this.selectedPortal?.slug : '';
    },
    selectedPortalLocale() {
      return this.selectedPortal
        ? this.selectedPortal?.meta?.default_locale
        : '';
    },
    accessibleMenuItems() {
      if (!this.selectedPortal) return [];
      const {
        meta: {
          all_articles_count: allArticlesCount,
          mine_articles_count: mineArticlesCount,
          draft_articles_count: draftArticlesCount,
          archived_articles_count: archivedArticlesCount,
        } = {},
      } = this.selectedPortal;
      return [
        {
          icon: 'book',
          label: 'HELP_CENTER.ALL_ARTICLES',
          key: 'list_all_locale_articles',
          count: allArticlesCount,
          toState: frontendURL(
            `accounts/${this.accountId}/portals/${this.selectedPortalSlug}/${this.selectedPortalLocale}/articles`
          ),
          toolTip: 'All Articles',
          toStateName: 'list_all_locale_articles',
        },
        {
          icon: 'pen',
          label: 'HELP_CENTER.MY_ARTICLES',
          key: 'list_mine_articles',
          count: mineArticlesCount,
          toState: frontendURL(
            `accounts/${this.accountId}/portals/${this.selectedPortalSlug}/${this.selectedPortalLocale}/articles/mine`
          ),
          toolTip: 'My articles',
          toStateName: 'list_mine_articles',
        },
        {
          icon: 'draft',
          label: 'HELP_CENTER.DRAFT',
          key: 'list_draft_articles',
          count: draftArticlesCount,
          toState: frontendURL(
            `accounts/${this.accountId}/portals/${this.selectedPortalSlug}/${this.selectedPortalLocale}/articles/draft`
          ),
          toolTip: 'Draft',
          toStateName: 'list_draft_articles',
        },
        {
          icon: 'archive',
          label: 'HELP_CENTER.ARCHIVED',
          key: 'list_archived_articles',
          count: archivedArticlesCount,
          toState: frontendURL(
            `accounts/${this.accountId}/portals/${this.selectedPortalSlug}/${this.selectedPortalLocale}/articles/archived`
          ),
          toolTip: 'Archived',
          toStateName: 'list_archived_articles',
        },
      ];
    },
    additionalSecondaryMenuItems() {
      if (!this.selectedPortal) return [];
      return [
        {
          icon: 'folder',
          label: 'HELP_CENTER.CATEGORY',
          hasSubMenu: true,
          key: 'category',
          children: this.categories.map(category => ({
            id: category.id,
            label: category.name,
            count: category.meta.articles_count,
            truncateLabel: true,
            toState: frontendURL(
              `accounts/${this.accountId}/portals/${this.selectedPortalSlug}/${category.locale}/categories/${category.slug}`
            ),
          })),
        },
      ];
    },
    currentRoute() {
      return '  ';
    },
    headerTitle() {
      return this.selectedPortal ? this.selectedPortal.name : '';
    },
  },
  mounted() {
    window.addEventListener('resize', this.handleResize);
    this.handleResize();
    bus.$on(BUS_EVENTS.TOGGLE_SIDEMENU, this.toggleSidebar);

    const slug = this.$route.params.portalSlug;
    if (slug) this.lastActivePortalSlug = slug;

    this.fetchPortalsAndItsCategories();
  },
  beforeDestroy() {
    bus.$off(BUS_EVENTS.TOGGLE_SIDEMENU, this.toggleSidebar);
    window.removeEventListener('resize', this.handleResize);
  },
  updated() {
    const slug = this.$route.params.portalSlug;
    if (slug) this.lastActivePortalSlug = slug;
  },
  methods: {
    handleResize() {
      if (window.innerWidth > 1200) {
        this.isOnDesktop = true;
      } else {
        this.isOnDesktop = false;
      }
    },
    toggleSidebar() {
      this.isSidebarOpen = !this.isSidebarOpen;
    },
    fetchPortalsAndItsCategories() {
      this.$store.dispatch('portals/index').then(() => {
        this.$store.dispatch('categories/index', {
          portalSlug: this.selectedPortalSlug,
        });
      });
      this.$store.dispatch('agents/get');
    },
    toggleKeyShortcutModal() {
      this.showShortcutModal = true;
    },
    closeKeyShortcutModal() {
      this.showShortcutModal = false;
    },
    openNotificationPanel() {
      this.showNotificationPanel = true;
    },
    closeNotificationPanel() {
      this.showNotificationPanel = false;
    },
    openPortalPopover() {
      this.showPortalPopover = !this.showPortalPopover;
    },
    closePortalPopover() {
      this.showPortalPopover = false;
    },
    onClickOpenAddCatogoryModal() {
      this.showAddCategoryModal = true;
    },
    onClickCloseAddCategoryModal() {
      this.showAddCategoryModal = false;
    },
  },
};
</script>
<style lang="scss" scoped>
.off-canvas-content.is-open-left.has-transition-push {
  transform: translateX(var(--space-giga));
}
</style>
