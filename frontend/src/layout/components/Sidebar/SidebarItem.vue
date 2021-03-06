<template>
  <div
    v-if="(!item.meta || !item.meta.hidden) && scopeAllowed"
    :class="[
      'menu-wrapper',
      isCollapse ? 'simple-mode' : 'full-mode',
      {'first-level': isFirstLevel}
    ]"
  >
    <v-img src="@/assets/logo-dark.png" height="200px"></v-img>
    <template v-if="theOnlyOneChild && !theOnlyOneChild.children">
      <sidebar-item-link
        v-if="theOnlyOneChild.meta"
        :to="resolvePath(theOnlyOneChild.path)"
      >
        <el-menu-item
          :index="resolvePath(theOnlyOneChild.path)"
          :class="{'submenu-title-noDropdown': isFirstLevel}"
        >
          <i
            v-if="theOnlyOneChild.meta.icon"
            :class="theOnlyOneChild.meta.icon"
            :style="isCollapse ? {'width': '100%'} : null"
          />
          <span
            v-if="theOnlyOneChild.meta.title"
            slot="title"
          >
            {{ theOnlyOneChild.meta.title }}
          </span>
        </el-menu-item>
      </sidebar-item-link>
    </template>
    <el-submenu
      v-else
      :index="resolvePath(item.path)"
      popper-append-to-body
    >
      <template slot="title">
        <i
          v-if="item.meta && item.meta.icon"
          :class="item.meta.icon"
        />
        <span
          v-if="item.meta && item.meta.title"
          slot="title"
        >
          {{ item.meta.title }}
        </span>
      </template>
      <template v-if="item.children">
        <sidebar-item
          v-for="child in item.children"
          :key="child.path"
          :item="child"
          :is-collapse="isCollapse"
          :is-first-level="false"
          :base-path="resolvePath(child.path)"
          class="nest-menu"
        />
      </template>
    </el-submenu>
  </div>
</template>

<script lang="ts">
import path from 'path';
import { Component, Prop, Vue } from 'vue-property-decorator';
import { RouteConfig } from 'vue-router';
import SidebarItemLink from './SidebarItemLink.vue';

import { getModule } from 'vuex-module-decorators';
import AppState from '@/store/modules/app';

@Component({
  // Set 'name' here to prevent uglifyjs from causing recursive component not work
  // See https://medium.com/haiiro-io/element-component-name-with-vue-class-component-f3b435656561 for detail
  name: 'SidebarItem',
  components: {
    SidebarItemLink,
  },
})
export default class extends Vue {
  @Prop({ required: true }) private item!: RouteConfig

  @Prop({ default: false }) private isCollapse!: boolean

  @Prop({ default: true }) private isFirstLevel!: boolean

  @Prop({ default: '' }) private basePath!: string

  private appModule = getModule(AppState, this.$store)

  get showingChildNumber() {
    if (this.item.children) {
      const showingChildren = this.item.children.filter((item) => {
        if (item.meta && item.meta.hidden) {
          return false;
        }
        return true;
      });
      return showingChildren.length;
    }
    return 0;
  }

  get theOnlyOneChild() {
    if (this.showingChildNumber > 1) {
      return null;
    }
    if (this.item.children) {
      for (const child of this.item.children) {
        if (!child.meta || !child.meta.hidden) {
          return child;
        }
      }
    }
    // If there is no children, return itself with path removed,
    // because this.basePath already conatins item's path information
    return { ...this.item, path: '' };
  }

  get scopeAllowed() {
    if (this.item.meta) {
      for (const scope of this.appModule.userAllowedScopes) {
        if (scope.includes(this.item.meta.scope)) {
          return true;
        }
      }
    }
    return false;
  }

  private resolvePath(routePath: string) {
    const isExternal = (externalPath: string) => /^(https?:|mailto:|tel:)/.test(externalPath);

    if (isExternal(routePath)) {
      return routePath;
    }
    if (isExternal(this.basePath)) {
      return this.basePath;
    }
    return path.resolve(this.basePath, routePath);
  }
}
</script>

<style lang="scss">
@import "~@/styles/_variables.scss";

a {
  text-decoration: none;
}

.el-submenu.is-active > .el-submenu__title {
  color: $subMenuActiveText !important;
}

.full-mode {
  min-width: $sideBarWidth !important;

  // root level menu items
  .el-menu-item, .el-submenu__title{
    background-color: $menuBg !important;

    &:hover {
      background-color: $subMenuHover !important;
    }
  }

  // submenu-items
  .nest-menu .el-submenu>.el-submenu__title,
  .el-submenu .el-menu-item {
    background-color: $subMenuBg !important;

    &:hover {
      background-color: $subMenuHover !important;
    }
  }
}

.simple-mode {
  &.first-level {
    .submenu-title-noDropdown {
      padding: 0 !important;
      position: relative;

      .el-tooltip {
        padding: 0 !important;
      }
    }

    .el-submenu {
      overflow: hidden;

      &>.el-submenu__title {
        padding: 0px !important;

        .el-submenu__icon-arrow {
          display: none;
        }

        &>span {
          visibility: hidden;
        }
      }
    }
  }
}
</style>
