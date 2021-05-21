<template>
  <v-app :class="{ 'small-screen': smallScreen }">
    <v-main>
      <v-layout id="app" column align-center justify-center>
        <div id="url-input">
          <v-text-field
            v-model="url"
            :rules="[urlRule]"
            :disabled="loading"
            :dense="smallScreen"
            type="url"
            placeholder="http(s)://"
            autocomplete="off"
            validate-on-blur
            filled
            rounded
            clearable
            @keypress.enter="shortenUrl"
          >
            <template #append-outer>
              <v-btn
                color="primary"
                :loading="loading"
                :disabled="canNotShorten"
                :x-large="!smallScreen"
                rounded
                depressed
                @click="shortenUrl"
                >Short it!</v-btn
              >
            </template>
          </v-text-field>
        </div>
        <div id="shorten-result" :class="{ 'no-result': !shortenedUrl }">
          <v-text-field
            :value="shortenedUrl"
            type="text"
            readonly
            append-outer-icon="mdi-content-copy"
            @click:append-outer="copyResult"
            @focus="onShortResultFocus"
          ></v-text-field>
          <v-tooltip activator="#shorten-result .v-input__icon--append-outer" right>
            <span>{{ showCopiedTooltip ? (copySuccess ? 'Copied' : 'Copy failed') : 'copy' }}</span>
          </v-tooltip>
        </div>
        <snackbar ref="snackbar" />
      </v-layout>
    </v-main>
  </v-app>
</template>

<script>
import Snackbar from '@/components/snackbar';
import copyText from '@/utils/clipboard';

const urlReg = /^https?:\/\/[\w_-]+(\.[\w_-]+)+[!-~]*$|^$/;

export default {
  name: 'App',
  components: { Snackbar },
  data: () => ({
    title: location.hostname,
    loading: false,
    url: '',
    lastUrl: '',
    shortenedUrl: '',
    showCopiedTooltip: false,
    copiedTooltipTimeout: null,
    copySuccess: true,
  }),
  computed: {
    trimedUrl() {
      return (this.url || '').trim();
    },
    isUrlValid() {
      return urlReg.test(this.trimedUrl);
    },
    canNotShorten() {
      return this.loading || !this.trimedUrl || !this.isUrlValid;
    },
    smallScreen() {
      return this.$vuetify.breakpoint.xs;
    },
  },
  methods: {
    urlRule() {
      return this.isUrlValid || 'Is not a valid URL';
    },
    shortenUrl() {
      if (this.canNotShorten || this.trimedUrl === this.lastUrl) return;
      this.shortenedUrl = '';
      this.loading = true;
      return fetch('/shorten', {
        method: 'POST',
        body: JSON.stringify({ url: this.trimedUrl }),
        headers: { 'content-type': 'application/json' },
      })
        .then(r => r.json())
        .then(({ code, msg, url }) => {
          if (code === 0) {
            this.shortenedUrl = url;
            this.lastUrl = this.trimedUrl;
          } else throw new Error(msg);
        })
        .catch(e => {
          this.$refs.snackbar.show(e.toString());
        })
        .finally(() => {
          this.loading = false;
        });
    },
    async copyResult() {
      this.copySuccess = await copyText(this.shortenedUrl);
      this.showCopiedTooltip = true;
      if (this.copiedTooltipTimeout) clearTimeout(this.copiedTooltipTimeout);
      this.copiedTooltipTimeout = setTimeout(() => {
        this.showCopiedTooltip = false;
        this.copiedTooltipTimeout = null;
      }, 3000);
    },
    onShortResultFocus() {
      const $input = document.querySelector('#shorten-result input');
      $input.selectionStart = 0;
      $input.selectionEnd = $input.value.length;
    },
  },
};
</script>

<style lang="scss">
html {
  overflow-y: auto !important;
}
#app {
  min-height: 80vh;
}
#nav {
  position: absolute;
  top: 8px;
  right: 8px;
}
#url-input {
  width: 80%;
  max-width: 1024px;
  .v-input__slot {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
  }
  .v-input__append-outer {
    margin: 0;
    .v-btn {
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
      height: 56px;
      transition: all 0.3s;
    }
  }
}
#shorten-result {
  &.no-result {
    opacity: 0;
    pointer-events: none;
  }
  .v-input {
    width: 300px;
    input {
      text-align: center;
    }
  }
}
.small-screen {
  #url-input {
    .v-input__append-outer {
      .v-btn {
        height: 40px;
      }
    }
  }
}
</style>
