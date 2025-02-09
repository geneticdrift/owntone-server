<template>
  <div>
    <content-with-heading>
      <template #options>
        <index-button-list :index="tracks.indexList" />
        <div class="columns">
          <div class="column">
            <p
              class="heading"
              style="margin-bottom: 24px"
              v-text="$t('page.artist.sort-by.title')"
            />
            <dropdown-menu
              v-model="selected_groupby_option_id"
              :options="groupby_options"
            />
          </div>
        </div>
      </template>
      <template #heading-left>
        <p class="title is-4" v-text="artist.name" />
      </template>
      <template #heading-right>
        <div class="buttons is-centered">
          <a
            class="button is-small is-light is-rounded"
            @click="show_artist_details_modal = true"
          >
            <span class="icon"
              ><mdicon name="dots-horizontal" size="16"
            /></span>
          </a>
          <a class="button is-small is-dark is-rounded" @click="play">
            <span class="icon"><mdicon name="shuffle" size="16" /></span>
            <span v-text="$t('page.artist.shuffle')" />
          </a>
        </div>
      </template>
      <template #content>
        <p class="heading has-text-centered-mobile">
          <a
            class="has-text-link"
            @click="open_artist"
            v-text="
              $t('page.artist.album-count', { count: artist.album_count })
            "
          />
          <span>&nbsp;|&nbsp;</span>
          <span
            v-text="
              $t('page.artist.track-count', { count: artist.track_count })
            "
          />
        </p>
        <list-tracks :tracks="tracks" :uris="track_uris" />
        <modal-dialog-artist
          :show="show_artist_details_modal"
          :artist="artist"
          @close="show_artist_details_modal = false"
        />
      </template>
    </content-with-heading>
  </div>
</template>

<script>
import ContentWithHeading from '@/templates/ContentWithHeading.vue'
import DropdownMenu from '@/components/DropdownMenu.vue'
import IndexButtonList from '@/components/IndexButtonList.vue'
import ListTracks from '@/components/ListTracks.vue'
import ModalDialogArtist from '@/components/ModalDialogArtist.vue'
import webapi from '@/webapi'
import * as types from '@/store/mutation_types'
import { byName, byRating, GroupByList } from '@/lib/GroupByList'

const dataObject = {
  load: function (to) {
    return Promise.all([
      webapi.library_artist(to.params.artist_id),
      webapi.library_artist_tracks(to.params.artist_id)
    ])
  },

  set: function (vm, response) {
    vm.artist = response[0].data
    vm.tracks_list = new GroupByList(response[1].data.tracks)
  }
}

export default {
  name: 'PageArtistTracks',
  components: {
    ContentWithHeading,
    DropdownMenu,
    IndexButtonList,
    ListTracks,
    ModalDialogArtist
  },

  beforeRouteEnter(to, from, next) {
    dataObject.load(to).then((response) => {
      next((vm) => dataObject.set(vm, response))
    })
  },
  beforeRouteUpdate(to, from, next) {
    const vm = this
    dataObject.load(to).then((response) => {
      dataObject.set(vm, response)
      next()
    })
  },

  data() {
    return {
      artist: {},
      groupby_options: [
        {
          id: 1,
          name: this.$t('page.artist.sort-by.name'),
          options: byName('title_sort')
        },
        {
          id: 2,
          name: this.$t('page.artist.sort-by.rating'),
          options: byRating('rating', {
            direction: 'desc'
          })
        }
      ],
      show_artist_details_modal: false,
      tracks_list: new GroupByList()
    }
  },

  computed: {
    selected_groupby_option_id: {
      get() {
        return this.$store.state.artist_tracks_sort
      },
      set(value) {
        this.$store.commit(types.ARTIST_TRACKS_SORT, value)
      }
    },
    tracks() {
      const groupBy = this.groupby_options.find(
        (o) => o.id === this.selected_groupby_option_id
      )
      this.tracks_list.group(groupBy.options)
      return this.tracks_list
    },
    track_uris() {
      return this.tracks_list.items.map((a) => a.uri).join(',')
    }
  },

  methods: {
    open_artist: function () {
      this.show_details_modal = false
      this.$router.push({ path: '/music/artists/' + this.artist.id })
    },

    play: function () {
      webapi.player_play_uri(this.tracks_list.map((a) => a.uri).join(','), true)
    }
  }
}
</script>

<style></style>
