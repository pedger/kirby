<template>
  <div v-if="user && view" class="k-topbar">
    <k-view>
      <div class="k-topbar-wrapper">
        <k-dropdown class="k-topbar-menu">
          <k-button
            :tooltip="$t('menu')"
            icon="bars"
            class="k-topbar-button k-topbar-menu-button"
            @click="$refs.menu.toggle()"
          >
            <k-icon type="angle-down" />
          </k-button>
          <k-dropdown-content ref="menu" class="k-topbar-menu">
            <ul>
              <template v-for="(entry, entryName) in views">
                <li
                  v-if="entry.menu"
                  :key="'menu-item-' + entryName"
                  :aria-current="$store.state.view === entryName"
                >
                  <k-dropdown-item
                    :disabled="$permissions.access[entryName] === false"
                    :icon="entry.icon"
                    :link="entry.link"
                  >
                    {{ menuTitle(entry, entryName) }}
                  </k-dropdown-item>
                </li>
              </template>
              <li><hr></li>
              <li :aria-current="$route.meta.view === 'account'">
                <k-dropdown-item icon="account" link="/account">
                  {{ $t("view.account") }}
                </k-dropdown-item>
              </li>
              <li><hr></li>
              <li>
                <k-dropdown-item icon="logout" link="/logout">
                  {{ $t("logout") }}
                </k-dropdown-item>
              </li>
            </ul>
          </k-dropdown-content>
        </k-dropdown>

        <k-link
          v-if="view"
          :to="view.link"
          class="k-topbar-button k-topbar-view-button"
        >
          <k-icon :type="view.icon" /> {{ breadcrumbTitle }}
        </k-link>

        <k-dropdown v-if="$store.state.breadcrumb.length > 1" class="k-topbar-breadcrumb-menu">
          <k-button class="k-topbar-button" @click="$refs.crumb.toggle()">
            …
            <k-icon type="angle-down" />
          </k-button>

          <k-dropdown-content ref="crumb">
            <k-dropdown-item :icon="view.icon" :link="view.link">
              {{ $t(`view.${$store.state.view}`, view.label) }}
            </k-dropdown-item>
            <k-dropdown-item
              v-for="(crumb, index) in $store.state.breadcrumb"
              :key="'crumb-' + index + '-dropdown'"
              :icon="view.icon"
              :link="crumb.link"
            >
              {{ crumb.label }}
            </k-dropdown-item>
          </k-dropdown-content>
        </k-dropdown>

        <nav class="k-topbar-crumbs">
          <k-link
            v-for="(crumb, index) in $store.state.breadcrumb"
            :key="'crumb-' + index"
            :to="crumb.link"
          >
            {{ crumb.label }}
          </k-link>
        </nav>

        <div class="k-topbar-signals">
          <!-- loader -->
          <span v-show="$store.state.isLoading" class="k-topbar-loader">
            <svg viewBox="0 0 16 18">
              <path fill="white" d="M8,0 L16,4.50265232 L16,13.5112142 L8,18.0138665 L0,13.5112142 L0,4.50265232 L8,0 Z M2.10648757,5.69852516 L2.10648757,12.3153414 L8,15.632396 L13.8935124,12.3153414 L13.8935124,5.69852516 L8,2.38147048 L2.10648757,5.69852516 Z" />
            </svg>
          </span>

          <!-- notifications -->
          <template v-if="notification">
            <k-button
              class="k-topbar-notification k-topbar-signals-button"
              theme="positive"
              @click="$store.dispatch('notification/close')"
            >
              {{ notification.message }}
            </k-button>
          </template>

          <!-- registration -->
          <template v-else-if="unregistered">
            <div class="k-registration">
              <p>{{ $t('license.unregistered') }}</p>
              <k-button
                :responsive="true"
                :tooltip="$t('license.unregistered')"
                class="k-topbar-signals-button"
                icon="key"
                @click="$emit('register')"
              >
                {{ $t('license.register') }}
              </k-button>
              <k-button
                :responsive="true"
                class="k-topbar-signals-button"
                link="https://getkirby.com/buy"
                target="_blank"
                icon="cart"
              >
                {{ $t('license.buy') }}
              </k-button>
            </div>
          </template>

          <!-- unsaved changes indicator -->
          <template>
            <k-form-indicator />
          </template>

          <!-- search -->
          <k-button
            :tooltip="$t('search')"
            class="k-topbar-signals-button"
            icon="search"
            @click="$emit('search')"
          />
        </div>
      </div>
    </k-view>
  </div>
</template>

<script>
import views from "@/config/views.js";

export default {
  computed: {
    breadcrumbTitle() {
      let title = this.$t(`view.${this.$store.state.view}`, this.view.label);

      if (this.$store.state.view === "site") {
        return this.$store.state.system.info.title || title;
      }

      return title;
    },
    view() {
      return views[this.$store.state.view];
    },
    views() {
      return views;
    },
    user() {
      return this.$store.state.user.current;
    },
    notification() {
      if (
        this.$store.state.notification.type &&
        this.$store.state.notification.type !== "error"
      ) {
        return this.$store.state.notification;
      } else {
        return null;
      }
    },
    unregistered() {
      return !this.$store.state.system.info.license ? true : false;
    }
  },
  methods: {
    menuTitle(view, viewName) {
      let title = this.$t("view." + viewName, view.label);

      if (viewName === "site") {
        return this.$store.state.system.info.site || title;
      }

      return title;
    }
  }
};
</script>

<style lang="scss">
.k-topbar {
  position: relative;
  color: $color-white;
  flex-shrink: 0;
  height: 2.5rem;
  line-height: 1;
  background: $color-gray-900;
}
.k-topbar-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  margin-left: -0.75rem;
  margin-right: -0.75rem;
}
.k-topbar-loader {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  height: 2.5rem;
  width: 2.5rem;
  padding: .75rem;
  background: $color-gray-900;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}
.k-topbar-loader svg {
  height: 18px;
  width: 18px;
  animation: Spin .9s linear infinite;
}

.k-topbar-menu {
  flex-shrink: 0;
}
.k-topbar-menu ul {
  padding: 0.5rem 0;
}
.k-topbar-menu-button {
  display: flex;
  align-items: center;
}
.k-topbar-menu-button .k-button-text {
  opacity: 1;
}
.k-topbar-signals-button,
.k-topbar-button {
  padding: 0.75rem;
  line-height: 1;
  font-size: $text-sm;
}
.k-topbar-signals .k-button .k-button-text {
  opacity: 1;
}
.k-topbar-button .k-button-text {
  display: flex;
  opacity: 1;
}
.k-topbar-view-button {
  flex-shrink: 0;
  display: flex;
  align-items: center;

  [dir="ltr"] & {
    padding-right: 0;
  }

  [dir="rtl"] & {
    padding-left: 0;
  }
}
.k-topbar-view-button .k-icon {
  [dir="ltr"] & {
    margin-right: 0.5rem;
  }

  [dir="rtl"] & {
    margin-left: 0.5rem;
  }
}
.k-topbar-crumbs {
  flex-grow: 1;
  display: flex;
  overflow-y: hidden;
}
.k-topbar-crumbs a {
  position: relative;
  font-size: $text-sm;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  display: none;
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
  line-height: 1;
  transition: opacity 0.3s;

  &::before {
    content: "/";
    padding: 0 0.5rem;
    opacity: 0.25;
  }
  &:focus,
  &:hover {
    opacity: 1;
  }

  @include highlight-tabbed;
}
.k-topbar-crumbs a:not(:last-child) {
  max-width: 15vw;
}
.k-topbar-breadcrumb-menu {
  flex-shrink: 0;
}
@media screen and (min-width: $breakpoint-sm) {
  .k-topbar-crumbs a {
    display: block;
  }
  .k-topbar-breadcrumb-menu {
    display: none;
  }
}
.k-topbar-signals {
  position: absolute;
  top: 0;
  background: $color-gray-900;
  height: 2.5rem;
  display: flex;
  align-items: center;

  [dir="ltr"] & {
    right: 0;
  }

  [dir="rtl"] & {
    left: 0;
  }
}
.k-topbar-signals::before {
  position: absolute;
  content: "";
  top: 0;
  bottom: 0;
  width: 0.5rem;

  [dir="ltr"] & {
    left: -0.5rem;
    background: -webkit-linear-gradient(
      left,
      rgba($color-gray-900, 0),
      rgba($color-gray-900, 1)
    );
  }

  [dir="rtl"] & {
    right: -0.5rem;
    background: -webkit-linear-gradient(
      right,
      rgba($color-gray-900, 0),
      rgba($color-gray-900, 1)
    );
  }
}
.k-topbar-signals .k-button {
  line-height: 1;
}

.k-topbar-notification {
  font-weight: $font-bold;
  line-height: 1;
  display: flex;
}
.k-topbar .k-button[data-theme="positive"] {
  color: $color-positive-on-dark;
}
.k-topbar .k-button[data-theme="negative"] {
  color: $color-negative-on-dark;
}
.k-topbar .k-button[data-theme="negative"] .k-button-text {
  display: none;

  @media screen and (min-width: $breakpoint-sm) {
    display: inline;
  }
}
.k-topbar .k-button[data-theme] .k-button-text {
  opacity: 1;
}
.k-topbar .k-dropdown-content {
  color: $color-gray-900;
  background: $color-white;
}
.k-topbar .k-dropdown-content hr:after {
  opacity: 0.1;
}
.k-topbar-menu [aria-current] .k-link {
  color: $color-focus;
  font-weight: 500;
}

.k-registration {
  display: inline-block;
  margin-right: 1rem;
  display: flex;
  align-items: center;
}
.k-registration p {
  color: $color-negative-on-dark;
  font-size: $text-sm;
  margin-right: 1rem;
  font-weight: 600;
  display: none;

  @media screen and (min-width: $breakpoint-lg) {
    display: block;
  }
}
.k-registration .k-button {
  color: $color-white;
}
</style>
