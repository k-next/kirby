<template>
  <div
    v-if="!$store.state.system.info.isBroken"
    :data-dragging="$store.state.drag"
    :data-loading="$store.state.isLoading"
    :data-route="$route.name"
    :data-topbar="inside"
    :data-translation="translation"
    :data-translation-default="defaultTranslation"
    class="k-panel"
    tabindex="0"
  >
    <!-- Icons -->
    <keep-alive>
      <k-icons />
    </keep-alive>

    <!-- Header -->
    <header v-if="inside" class="k-panel-header">
      <k-topbar
        @register="$refs.registration.open()"
        @search="$refs.search.open();"
      />
    </header>

    <!-- Main view -->
    <main class="k-panel-view">
      <router-view />
    </main>

    <!-- Form buttons -->
    <k-form-buttons v-if="inside" />

    <!-- Search dialog -->
    <k-search
      v-if="inside"
      ref="search"
      :type="searchType"
      :types="searchTypes"
    />

    <!-- Error dialog -->
    <k-error-dialog />

    <!-- Fatal iframe -->
    <template v-if="fatal">
      <div class="k-fatal">
        <div class="k-fatal-box">
          <k-headline>The JSON response of the API could not be parsed:</k-headline>
          <iframe ref="fatal" class="k-fatal-iframe" />
        </div>
      </div>
    </template>

    <!-- Offline warning -->
    <div
      v-if="offline"
      class="k-offline-warning"
    >
      <p>The Panel is currently offline</p>
    </div>

    <!-- Registration dialog -->
    <k-registration v-if="inside" ref="registration" />
  </div>
  <div v-else class="k-panel">
    <main class="k-panel-view">
      <k-error-view>
        <p v-if="debug">
          {{ $store.state.system.info.error }}
        </p>
        <p v-else>
          The Panel cannot connect to the API
        </p>
      </k-error-view>
    </main>
  </div>
</template>

<script>
import Icons from "@/components/Misc/Icons.vue";
import Registration from "@/components/Dialogs/RegistrationDialog.vue";
import config from "@/config/config.js";
import search from "@/config/search.js"

export default {
  name: "App",
  components: {
    "k-icons": Icons,
    "k-registration": Registration,
  },
  data() {
    return {
      offline: false,
      dragging: false,
      debug: config.debug
    };
  },
  computed: {
    inside() {
      return !this.$route.meta.outside && this.$store.state.user.current
        ? true
        : false;
    },
    defaultTranslation() {
      return this.$store.state.languages.current ? this.$store.state.languages.current === this.$store.state.languages.default : false;
    },
    fatal() {
      return this.$store.state.fatal;
    },
    searchType() {
      return this.$store.state.view === 'users' ? 'users' : 'pages';
    },
    searchTypes() {
      return search(this);
    },
    translation() {
      return this.$store.state.languages.current ? this.$store.state.languages.current.code : false;
    }
  },
  watch: {
    fatal(html) {
      if (html !== null) {
        this.$nextTick(() => {
          try {
            let doc = this.$refs.fatal.contentWindow.document;
            doc.open();
            doc.write(html);
            doc.close();
          } catch (e) {
            console.error(e);
          }
        })
      }
    }
  },
  created() {
    this.$events.$on("offline", this.isOffline);
    this.$events.$on("online", this.isOnline);
    this.$events.$on("drop", this.drop);
  },
  destroyed() {
    this.$events.$off("offline", this.isOffline);
    this.$events.$off("online", this.isOnline);
    this.$events.$off("drop", this.drop);
  },
  methods: {
    drop() {
      // remove any drop data from the store
      this.$store.dispatch("drag", null);
    },
    isOnline() {
      this.offline = false;
    },
    isOffline() {
      if (this.$store.state.system.info.isLocal === false) {
        this.offline = true;
      }
    }
  }
};
</script>

<style lang="scss">
*,
*::before,
*::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  /** Colors **/
  --color-backdrop: #{$color-backdrop};
  --color-background: #{$color-background};
  --color-border: #{$color-border};
  --color-focus: #{$color-focus};
  --color-focus-light: #{$color-focus-on-dark};
  --color-focus-outline: #{$color-focus-outline};
  --color-negative: #{$color-negative};
  --color-negative-light: #{$color-negative-on-dark};
  --color-negative-outline: #{$color-negative-outline};
  --color-notice: #{$color-notice};
  --color-notice-light: #{$color-notice-on-dark};
  --color-positive: #{$color-positive};
  --color-positive-light: #{$color-positive-on-dark};
  --color-positive-outline: #{$color-positive-outline};
  --color-text: #{$color-gray-900};
  --color-text-light: #{$color-gray-600};

  /** Font families **/
  --font-family-mono: #{$font-mono};
  --font-family-sans: #{$font-sans};

  /** Font sizes **/
  --font-size-tiny: #{$text-xs};
  --font-size-small: #{$text-sm};
  --font-size-medium: #{$text-base};
  --font-size-large: #{$text-xl};
  --font-size-huge: #{$text-2xl};
  --font-size-monster: #{$text-3xl};

  /** Shadows **/
  --box-shadow-dropdown: #{$shadow-lg};
  --box-shadow-item: #{$shadow};
  --box-shadow-focus: #{$shadow-xl};
}

noscript {
  padding: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  text-align: center;
}

html {
  font-family: $font-sans;
  background: $color-background;
}

html,
body {
  color: $color-gray-900;
  overflow: hidden;
  height: 100%;
}

a {
  color: inherit;
  text-decoration: none;
}

li {
  list-style: none;
}

strong,
b {
  font-weight: $font-bold;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.k-panel {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: $color-background;
}
.k-panel:focus {
  outline: 0;
}
.k-panel[data-loading] {
  animation: LoadingCursor 0.5s;
}
.k-panel-header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  z-index: z-index(navigation);
}
.k-panel .k-form-buttons {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: z-index(navigation);
}
.k-panel-view {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  padding-bottom: 6rem;
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
  transform: translate3d(0, 0, 0);
}
.k-panel[data-topbar] .k-panel-view {
  top: 2.5rem;
}
.k-panel[data-loading]::after,
.k-panel[data-dragging] {
  user-select: none;
}
.k-offline-warning {
  position: fixed;
  content: " ";
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: z-index(loader);
}
.k-offline-warning {
  background: rgba($color-gray-900, 0.7);
  content: "offline";
  display: flex;
  align-items: center;
  justify-content: center;
  color: $color-white;
}

@keyframes LoadingCursor {
  100% {
    cursor: progress;
  }
}

@keyframes Spin {
  100% {
    transform: rotate(360deg);
  }
}

.k-offscreen {
  clip-path: inset(100%);
  clip: rect(1px, 1px, 1px, 1px);
  height: 1px;
  overflow: hidden;
  position: absolute;
  white-space: nowrap;
  width: 1px;
}

.k-icons {
  position: absolute;
  width: 0;
  height: 0;
}

.k-fatal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: $color-backdrop;
  display: flex;
  z-index: z-index(dialog);
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
}
.k-fatal-box {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: #fff;
  padding: .75rem 1.5rem 1.5rem;
  box-shadow: $shadow-xl;
  border-radius: $rounded;
}
.k-fatal-box .k-headline {
  margin-bottom: .75rem;
}
.k-fatal-iframe {
  border: 0;
  width: 100%;
  flex-grow: 1;
  border: 2px solid $color-border;
}
</style>
