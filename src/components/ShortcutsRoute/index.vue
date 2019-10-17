<template>
  <div class="shortcuts-route">
    <div class="shortcuts-route__header">
      <div class="shortcuts-route__title">
        {{ title }}
      </div>
      <input
        class="shortcuts-route__search"
        placeholder="Search…"
        v-model="query"
        type="text"
        autofocus
        v-if="app"
      >
    </div>
    <div class="shortcuts-route__content">
      <template v-if="app">
        <template v-for="set in app.sets">
          <div class="shortcuts-route__set" :key="set.id" v-if="shortcutsBySet(set.id).length">
            <div class="shortcuts-route__set-title">
              {{ set.title }}
            </div>
            <div class="shortcuts-route__shortcut" v-for="shortcut in shortcutsBySet(set.id)" :key="shortcut.id">
              <div class="shortcuts-route__shortcut-title">
                {{ shortcut.title }}
              </div>
              <div class="shortcuts-route__shortcut-keys">
                <div class="shortcuts-route__shortcut-key" v-for="key in shortcut.resolvedKeys" :key="key">
                  {{ key | key | uppercase }}
                </div>
              </div>
            </div>
          </div>
        </template>
      </template>
      <div class="shortcuts-route__empty-state" v-else>
        <template v-if="title">
          Sorry but {{ title }} is not yet supported :-(
        </template>
        <template v-else>
          First open an app to see shortcuts.
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import Fuse from 'fuse.js'
import { ipcRenderer } from 'electron'

export default {
  data() {
    return {
      systemTitle: null,
      query: null,
    }
  },

  computed: {
    title() {
      return this.app ? this.app.title : this.systemTitle
    },

    app() {
      if (!this.systemTitle) {
        return null
      }

      return this.$db.apps.find(app => app.systemTitle === this.systemTitle)
    },
  },

  methods: {
    onCurrentApp(event, systemTitle) {
      console.log({ systemTitle })

      if (systemTitle !== 'Electron') {
        this.systemTitle = systemTitle
      }
    },

    shortcutsBySet(id) {
      const shortcuts = this.app.shortcutsBySet(id)

      if (!this.query) {
        return shortcuts
      }

      const fuse = new Fuse(shortcuts, {
        threshold: 0.2,
        keys: ['title'],
      })

      return fuse.search(this.query)
    },
  },

  mounted() {
    ipcRenderer.on('currentApp', this.onCurrentApp)
  },

  beforeDestroy() {
    ipcRenderer.removeListener('currentApp', this.onCurrentApp)
  },
}
</script>

<style lang="scss" src="./style.scss" scoped></style>