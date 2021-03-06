<template>
  <div id="dashboard">
    <SidebarMenu class="should-be-sidebar" @toggle="toggleSidebar" />
    <router-view id="routerview" :class="{'sidebar__active': !sidebarClosed }" />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import { cryptoWaitReady } from '@polkadot/util-crypto';
import keyring from '@vue-polkadot/vue-keyring';
import SidebarMenu from './components/SidebarMenu.vue';
import Connector from '@vue-polkadot/vue-api';

@Component({
  components: {
    SidebarMenu,
  },
})
export default class Dashboard extends Vue {
  private chainProperties: any;
  private ss58Format: any = 42;
  private sidebarClosed: boolean = true;

  private toggleSidebar(val: boolean) {
    this.sidebarClosed = val;
  }

  public async getChainProperties(): Promise<void> {
    this.ss58Format = Object.entries(this.$store.state.chainProperties)[0][1];
    console.log('ss58format - settings', this.ss58Format);
  }

  public async loadKeyring(): Promise<void> {
    this.getChainProperties();
    keyring.loadAll({
      ss58Format: this.ss58Format,
      type: 'sr25519',
      isDevelopment: false,
    });
  }

  public async mountWasmCrypto(): Promise<void> {
    await cryptoWaitReady();
    console.log('wasmCrypto loaded');
    this.loadKeyring();
    console.log('keyring init');
    this.$store.commit('keyringLoaded');
    console.log('keyring loaded');
  }

  public mounted(): void {
    this.mountWasmCrypto();
  }
}
</script>

<style lang="scss">
.should-be-sidebar {
  width: 5em;
  height: 100%;
  float: left;
  height: 100%;
  position: fixed;
}

#routerview {
  margin-left: 5em;
  padding: 0 0.6em;
}

#routerview.sidebar__active {
  margin-left: 16.5em;
}

// Import Bulma's core
@import "~bulma/sass/utilities/_all";
@import "./colors";

// Setup $colors to use as bulma classes (e.g. 'is-twitter')
$colors: (
    "white": ($white, $black),
    "black": ($black, $white),
    "light": ($light, $light-invert),
    "dark": ($dark, $dark-invert),
    "primary": ($primary, $primary-invert),
    "info": ($info, $info-invert),
    "success": ($success, $success-invert),
    "warning": ($warning, $warning-invert),
    "danger": ($danger, $danger-invert),
);

// Links
$link: $primary;
$link-invert: $primary-invert;
$link-focus-border: $primary;

// Import Bulma and Buefy styles
@import "~bulma";
@import "~buefy/src/scss/buefy";
</style>
