<template>
  <div>
    <transition name="fade">
      <div v-if="show" class="modal is-active">
        <div class="modal-background" @click="$emit('close')" />
        <div class="modal-content fd-modal-card">
          <div class="card">
            <div class="card-content">
              <p class="title is-4" v-text="track.name" />
              <p class="subtitle" v-text="track.artists[0].name" />
              <div class="content is-small">
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.album')"
                  />
                  <a
                    class="title is-6 has-text-link"
                    @click="open_album"
                    v-text="album.name"
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.album-artist')"
                  />
                  <a
                    class="title is-6 has-text-link"
                    @click="open_artist"
                    v-text="album.artists[0].name"
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.release-date')"
                  />
                  <span
                    class="title is-6"
                    v-text="$filters.date(album.release_date)"
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.position')"
                  />
                  <span
                    class="title is-6"
                    v-text="[track.disc_number, track.track_number].join(' / ')"
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.duration')"
                  />
                  <span
                    class="title is-6"
                    v-text="$filters.durationInHours(track.duration_ms)"
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.track.path')"
                  />
                  <span class="title is-6" v-text="track.uri" />
                </p>
              </div>
            </div>
            <footer class="card-footer">
              <a class="card-footer-item has-text-dark" @click="queue_add">
                <span class="icon"
                  ><mdicon name="playlist-plus" size="16"
                /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.track.add')"
                />
              </a>
              <a class="card-footer-item has-text-dark" @click="queue_add_next">
                <span class="icon"
                  ><mdicon name="playlist-play" size="16"
                /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.track.add-next')"
                />
              </a>
              <a class="card-footer-item has-text-dark" @click="play">
                <span class="icon"><mdicon name="play" size="16" /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.track.play')"
                />
              </a>
            </footer>
          </div>
        </div>
        <button
          class="modal-close is-large"
          aria-label="close"
          @click="$emit('close')"
        />
      </div>
    </transition>
  </div>
</template>

<script>
import webapi from '@/webapi'

export default {
  name: 'SpotifyModalDialogTrack',
  props: ['show', 'track', 'album'],
  emits: ['close'],

  methods: {
    play: function () {
      this.$emit('close')
      webapi.player_play_uri(this.track.uri, false)
    },

    queue_add: function () {
      this.$emit('close')
      webapi.queue_add(this.track.uri)
    },

    queue_add_next: function () {
      this.$emit('close')
      webapi.queue_add_next(this.track.uri)
    },

    open_album: function () {
      this.$router.push({ path: '/music/spotify/albums/' + this.album.id })
    },

    open_artist: function () {
      this.$router.push({
        path: '/music/spotify/artists/' + this.album.artists[0].id
      })
    }
  }
}
</script>

<style></style>
