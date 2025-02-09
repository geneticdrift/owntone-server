<template>
  <div class="field">
    <label class="switch">
      <input
        ref="setting"
        type="checkbox"
        :checked="value"
        style="margin-right: 5px"
        @change="set_update_timer"
      />
      <slot name="label" />
      <i
        class="is-size-7"
        :class="{ 'has-text-info': is_success, 'has-text-danger': is_error }"
        v-text="info"
      />
    </label>
    <p v-if="$slots['info']" class="help">
      <slot name="info" />
    </p>
  </div>
</template>

<script>
import webapi from '@/webapi'
import * as types from '@/store/mutation_types'

export default {
  name: 'SettingsCheckbox',
  props: ['category_name', 'option_name'],

  data() {
    return {
      timerDelay: 2000,
      timerId: -1,
      statusUpdate: ''
    }
  },

  computed: {
    category() {
      return this.$store.state.settings.categories.find(
        (elem) => elem.name === this.category_name
      )
    },

    option() {
      if (!this.category) {
        return {}
      }
      return this.category.options.find(
        (elem) => elem.name === this.option_name
      )
    },

    value() {
      return this.option.value
    },

    info() {
      if (this.statusUpdate === 'success') {
        return this.$t('setting.saved')
      } else if (this.statusUpdate === 'error') {
        return this.$t('setting.not-saved')
      }
      return ''
    },

    is_success() {
      return this.statusUpdate === 'success'
    },

    is_error() {
      return this.statusUpdate === 'error'
    }
  },

  methods: {
    set_update_timer() {
      if (this.timerId > 0) {
        window.clearTimeout(this.timerId)
        this.timerId = -1
      }

      this.statusUpdate = ''
      const newValue = this.$refs.setting.checked
      if (newValue !== this.value) {
        this.timerId = window.setTimeout(this.update_setting, this.timerDelay)
      }
    },

    update_setting() {
      this.timerId = -1

      const newValue = this.$refs.setting.checked
      console.log(this.$refs.setting)
      if (newValue === this.value) {
        this.statusUpdate = ''
        return
      }

      const option = {
        category: this.category.name,
        name: this.option_name,
        value: newValue
      }
      webapi
        .settings_update(this.category.name, option)
        .then(() => {
          this.$store.commit(types.UPDATE_SETTINGS_OPTION, option)
          this.statusUpdate = 'success'
        })
        .catch(() => {
          this.statusUpdate = 'error'
          this.$refs.setting.checked = this.value
        })
        .finally(() => {
          this.timerId = window.setTimeout(this.clear_status, this.timerDelay)
        })
    },

    clear_status: function () {
      this.statusUpdate = ''
    }
  }
}
</script>

<style></style>
