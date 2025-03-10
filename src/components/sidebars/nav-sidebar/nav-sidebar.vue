<template>
  <div class="nav-sidebar">
    <v-blocker v-show="active" class="blocker" :z-index="2" @click="disableNav" />
    <transition name="nav">
      <aside :class="{ active }">
        <button class="a11y-close" @click="disableNav">Close nav</button>

        <module-bar />

        <section class="main-bar">
          <project-switcher />

          <template v-for="section in navStructure">
            <nav-bookmarks
              v-if="section.include && section.include === 'bookmarks' && bookmarks.length > 0"
              :key="section.id"
              :title="section.title"
              class="menu-section"
              :bookmarks="bookmarks"
            />
            <nav-menu
              v-else-if="section.include && section.include === 'collections'"
              :key="section.id"
              :title="section.title"
              class="menu-section"
              :links="linksCollections"
            />
            <nav-menu
              v-else
              :key="section.id"
              class="menu-section"
              :title="section.title"
              :links="section.links ? section.links : []"
            />
          </template>
        </section>
      </aside>
    </transition>
  </div>
</template>
<script>
import ProjectSwitcher from "./project-switcher.vue";
import NavMenu from "./nav-menu.vue";
import NavBookmarks from "./nav-bookmarks.vue";
import VBlocker from "../../blocker.vue";
import { TOGGLE_NAV } from "@/store/mutation-types";
import { mapState } from "vuex";
import ModuleBar from "./module-bar";

export default {
  name: "NavSidebar",
  components: {
    ProjectSwitcher,
    NavMenu,
    NavBookmarks,
    VBlocker,
    ModuleBar
  },
  computed: {
    ...mapState(["currentProjectKey"]),
    permissions() {
      return this.$store.state.permissions;
    },
    collections() {
      const collections = this.$store.state.collections;

      if (collections == null) return [];

      return Object.values(collections)
        .filter(
          collection =>
            collection.hidden == false &&
            collection.managed == true &&
            collection.collection.startsWith("directus_") === false
        )
        .filter(collection => {
          if (collection.status_mapping && this.permissions[collection.collection].statuses) {
            return _.some(
              this.permissions[collection.collection].statuses,
              permission => permission.read !== "none"
            );
          }

          return this.permissions[collection.collection].read !== "none";
        });
    },
    projectName() {
      return this.$store.getters.currentProject.project_name;
    },
    active() {
      return this.$store.state.sidebars.nav;
    },
    bookmarks() {
      return this.$store.state.bookmarks;
    },

    // This is the default structure of the navigation pane
    // By default it will list collections, bookmarks
    // This is the thing that will be overridden by the nav_override field
    // in directus_roles
    defaultNavStructure() {
      return [
        {
          title: "$t:collections",
          include: "collections"
        },
        {
          title: "$t:bookmarks",
          include: "bookmarks"
        }
      ];
    },

    // The structure of the navigation. Will return the stored value for the role
    // nav override or the default structure above if it isn't set
    // It will also replace the `includes` with links for the actual sections
    navStructure() {
      const userRole = this.$store.state.currentUser.role;
      const navOverride = userRole.nav_override;

      return navOverride || this.defaultNavStructure;
    },

    linksCollections() {
      return this.collections.map(({ collection, icon }) => ({
        path: `/${this.currentProjectKey}/collections/${collection}`,
        name: this.$helpers.formatCollection(collection),
        icon
      }));
    }
  },
  methods: {
    logout() {
      this.$store.dispatch("logout");
    },
    deleteBookmark(id) {
      this.$store.dispatch("deleteBookmark", id);
    },
    toBookmark(bookmark) {
      /* eslint-disable camelcase */
      const { collection, search_query, filters, view_options, view_type, view_query } = bookmark;

      this.$store
        .dispatch("setListingPreferences", {
          collection,
          updates: {
            search_query,
            filters,
            view_options,
            view_type,
            view_query
          }
        })
        .then(() => {
          this.$router.push(`/${this.currentProjectKey}/collections/${collection}`);
        });
    },
    disableNav() {
      this.$store.commit(TOGGLE_NAV, false);
    }
  }
};
</script>

<style lang="scss" scoped>
aside {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  z-index: 30;
  width: var(--nav-sidebar-width);
  background-color: var(--sidebar-background-color);
  color: var(--sidebar-text-color);
  display: flex;

  transform: translateX(-100%);
  visibility: hidden;
  transition: transform var(--slow) var(--transition-out),
    visibility 0ms var(--transition-out) var(--slow);

  &.active {
    transform: translateX(0);
    transition: transform var(--slow) var(--transition-in);
    visibility: visible;
  }

  @media (min-width: 800px) {
    transform: translateX(0);
    transition: none;
    visibility: visible;
    width: var(--nav-sidebar-width);
  }

  > div {
    height: 100%;
  }

  & .a11y-close {
    position: absolute;
    z-index: 15;
    left: -999px;
    background-color: yellow;
    padding: 5px;

    .user-is-tabbing &:focus {
      top: 13px;
      left: 13px;
    }

    @media (min-width: 800px) {
      display: none;
    }
  }
}

.main-bar {
  position: relative;
  padding: 12px;
  padding-top: 0;
  height: 100%;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
  flex-basis: 220px;
  flex-shrink: 0;
}

.menu-section + .menu-section {
  border-top: 2px solid var(--sidebar-background-color-alt);
  padding-top: 16px;
}
</style>

<style>
@media (min-width: 800px) {
  body {
    padding-left: var(--nav-sidebar-width);
  }
}
</style>
