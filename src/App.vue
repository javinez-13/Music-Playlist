<template>
  <div class="app-shell">

    <!-- HEADER -->
    <header class="hero">
      <div class="brand">
        <div class="logo">♫</div>

        <div>
          <p class="eyebrow">PLAYLIST MANAGER</p>
          <h1>{{ appName }}</h1>
          <p class="subtitle">
            Organize your favorite songs in one place.
          </p>
        </div>
      </div>

      <button
        class="theme-button"
        @click="toggleDarkMode"
      >
        {{ darkMode ? '☀️ Light' : '🌙 Dark' }}
      </button>
    </header>

    <main>

      <!-- STATISTICS -->
      <section class="stats-grid">

        <article class="stat-card">
          <span class="stat-icon">🎵</span>

          <div>
            <span class="stat-label">Total Songs</span>
            <strong>{{ songs.length }}</strong>
          </div>
        </article>

        <article class="stat-card">
          <span class="stat-icon">📀</span>

          <div>
            <span class="stat-label">Playlist</span>
            <strong>{{ playlistCount }}</strong>
          </div>
        </article>

        <article class="stat-card">
          <span class="stat-icon">♥</span>

          <div>
            <span class="stat-label">Favorites</span>
            <strong>{{ favoriteCount }}</strong>
          </div>
        </article>

      </section>


      <!-- MAIN WORKSPACE -->
      <section class="workspace">

        <!-- SONG CATALOG -->
        <div class="catalog-panel">

          <div class="section-heading">

            <div>
              <h2>Discover Songs</h2>

              <p>
                Search, filter, and build your personal playlist.
              </p>
            </div>

            <span class="result-count">
              {{ filteredSongs.length }} results
            </span>

          </div>


          <!-- SEARCH AND FILTER -->
          <div class="filters">

            <label class="search-box">

              <span>⌕</span>

              <input
                v-model="searchQuery"
                @input="searchSongs"
                type="search"
                placeholder="Search songs or artists..."
              />

            </label>


            <label class="select-box">

              <span class="filter-label">
                Genre
              </span>

              <select
                v-model="selectedFilter"
                @change="filterSongs"
              >

                <option
                  v-for="genre in genres"
                  :key="genre"
                  :value="genre"
                >
                  {{ genre }}
                </option>

              </select>

            </label>


            <button
              class="playlist-filter"
              :class="{ active: showPlaylistOnly }"
              @click="togglePlaylistOnly"
            >
              {{ showPlaylistOnly ? '✓ My Playlist' : 'My Playlist' }}
            </button>

          </div>


          <!-- SONG LIST -->

          <div
            v-if="filteredSongs.length > 0"
            class="song-grid"
          >

            <article
              v-for="song in filteredSongs"
              :key="song.id"
              class="song-card"
              :class="{ 'in-playlist': song.inPlaylist }"
            >

              <!-- ALBUM COVER -->
              <div class="cover-wrap">

                <img
                  :src="song.cover"
                  :alt="`${song.title} cover`"
                  class="cover"
                />

                <button
                  class="play-button"
                  @click="selectSong(song)"
                  :aria-label="`Select ${song.title}`"
                >
                  ▶
                </button>

                <span
                  v-if="song.inPlaylist"
                  class="added-badge"
                >
                  Added
                </span>

              </div>


              <!-- SONG INFORMATION -->
              <div class="song-info">

                <div class="song-topline">

                  <div>

                    <h3>
                      {{ song.title }}
                    </h3>

                    <p>
                      {{ song.artist }}
                    </p>

                  </div>


                  <!-- FAVORITE -->
                  <button
                    class="favorite-button"
                    :class="{ active: song.isFavorite }"
                    @click="toggleFavorite(song)"
                  >
                    {{ song.isFavorite ? '♥' : '♡' }}
                  </button>

                </div>


                <!-- SONG DETAILS -->
                <div class="song-meta">

                  <span>
                    {{ song.genre }}
                  </span>

                  <span>•</span>

                  <span>
                    {{ song.duration }}
                  </span>

                </div>


                <!-- ADD / REMOVE -->
                <button
                  v-if="song.inPlaylist"
                  class="action-button remove"
                  @click="removeSong(song.id)"
                >
                  Remove from Playlist
                </button>

                <button
                  v-else-if="playlistCount < 50"
                  class="action-button add"
                  @click="addSong(song)"
                >
                  + Add to Playlist
                </button>

                <button
                  v-else
                  class="action-button disabled"
                  disabled
                >
                  Playlist is full
                </button>

              </div>

            </article>

          </div>


          <!-- EMPTY SEARCH RESULT -->
          <div
            v-else
            class="empty-state"
          >

            <div class="empty-icon">
              ♪
            </div>

            <h3>
              No songs found
            </h3>

            <p>
              Try another search or filter.
            </p>

            <button
              class="reset-button"
              @click="resetFilters"
            >
              Reset Filters
            </button>

          </div>

        </div>


        <!-- CURRENT SONG -->
        <aside class="now-playing">

          <div class="now-heading">

            <div>

              <span class="eyebrow">
                NOW SELECTED
              </span>

              <h2>
                Current Song
              </h2>

            </div>

            <span class="pulse-dot"></span>

          </div>


          <!-- SONG SELECTED -->
          <div
            v-if="currentSong"
            class="current-content"
          >

            <img
              :src="currentSong.cover"
              :alt="currentSong.title"
              class="current-cover"
            />

            <h3>
              {{ currentSong.title }}
            </h3>

            <p class="current-artist">
              {{ currentSong.artist }}
            </p>

            <div class="current-meta">

              <span>
                {{ currentSong.genre }}
              </span>

              <span>
                {{ currentSong.duration }}
              </span>

            </div>


            <button
              v-if="!currentSong.inPlaylist"
              class="wide-button"
              @click="addSong(currentSong)"
            >
              + Add to Playlist
            </button>

            <button
              v-else
              class="wide-button secondary"
              @click="removeSong(currentSong.id)"
            >
              Remove from Playlist
            </button>

          </div>


          <!-- NO SONG SELECTED -->
          <div
            v-else
            class="no-selection"
          >

            <div class="headphones">
              🎧
            </div>

            <h3>
              No song selected
            </h3>

            <p>
              Choose a song from the list to see
              its details here.
            </p>

          </div>


          <!-- PLAYLIST STATUS -->
          <div class="playlist-status">

            <h3>
              Playlist Status
            </h3>


            <!-- v-if -->
            <p v-if="playlistCount === 0">
              Your playlist is empty.
            </p>


            <!-- v-else-if -->
            <p v-else-if="playlistCount < 3">
              Add more songs to build your playlist!
            </p>


            <!-- v-else -->
            <p v-else>
              Your playlist is ready to go! 🎶
            </p>


            <div class="progress-track">

              <div
                class="progress-bar"
                :style="{
                  width: `${Math.min(playlistCount * 10, 100)}%`
                }"
              ></div>

            </div>

          </div>

        </aside>

      </section>

    </main>


    <!-- NOTIFICATION -->
    <div
      v-if="notification"
      class="notification"
    >
      {{ notification }}
    </div>

  </div>
</template>


<script lang="ts">

import { defineComponent } from 'vue'


interface Song {

  id: number

  title: string

  artist: string

  genre: string

  duration: string

  cover: string

  isFavorite: boolean

  inPlaylist: boolean

}


export default defineComponent({

  name: 'App',


  /*
   * data()
   * Stores the application's state.
   */
  data() {

    const songs: Song[] = [

      {
        id: 1,
        title: 'Golden Hour',
        artist: 'Nova Lane',
        genre: 'Pop',
        duration: '3:42',
        cover: 'https://picsum.photos/seed/goldenhour/600/600',
        isFavorite: true,
        inPlaylist: true
      },

      {
        id: 2,
        title: 'Neon Skies',
        artist: 'The Daylights',
        genre: 'Rock',
        duration: '4:08',
        cover: 'https://picsum.photos/seed/neonskies/600/600',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 3,
        title: 'Midnight Drive',
        artist: 'Kai Rivers',
        genre: 'Hip-Hop',
        duration: '3:31',
        cover: 'https://picsum.photos/seed/midnightdrive/600/600',
        isFavorite: true,
        inPlaylist: true
      },

      {
        id: 4,
        title: 'Island Lights',
        artist: 'Maya Cruz',
        genre: 'OPM',
        duration: '4:15',
        cover: 'https://picsum.photos/seed/islandlights/600/600',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 5,
        title: 'After the Rain',
        artist: 'Luna Park',
        genre: 'R&B',
        duration: '3:56',
        cover: 'https://picsum.photos/seed/aftertherain/600/600',
        isFavorite: true,
        inPlaylist: false
      },

      {
        id: 6,
        title: 'Electric Heart',
        artist: 'Echo Avenue',
        genre: 'Pop',
        duration: '3:24',
        cover: 'https://picsum.photos/seed/electricheart/600/600',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 7,
        title: 'Paper Planes',
        artist: 'Sunday Club',
        genre: 'OPM',
        duration: '4:02',
        cover: 'https://picsum.photos/seed/paperplanes/600/600',
        isFavorite: false,
        inPlaylist: true
      },

      {
        id: 8,
        title: 'City Noise',
        artist: 'Northbound',
        genre: 'Rock',
        duration: '3:47',
        cover: 'https://picsum.photos/seed/citynoise/600/600',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 9,
        title: 'Slow Motion',
        artist: 'Velvet Room',
        genre: 'R&B',
        duration: '4:21',
        cover: 'https://picsum.photos/seed/slowmotion/600/600',
        isFavorite: true,
        inPlaylist: false
      },

      {
        id: 10,
        title: 'Weekend Energy',
        artist: 'Juno Beats',
        genre: 'Hip-Hop',
        duration: '3:18',
        cover: 'https://picsum.photos/seed/weekendenergy/600/600',
        isFavorite: false,
        inPlaylist: false
      }

    ]


    return {

      appName: 'My Music Playlist',

      songs,

      searchQuery: '',

      selectedFilter: 'All',

      currentSong: null as Song | null,

      showPlaylistOnly: false,

      darkMode: false,

      notification: '',

      notificationTimer:
        null as ReturnType<typeof setTimeout> | null,

      genres: [
        'All',
        'Pop',
        'Rock',
        'Hip-Hop',
        'R&B',
        'OPM'
      ]

    }

  },


  /*
   * computed properties
   */

  computed: {

    /*
     * computed property:
     * Filters songs based on search and genre.
     */
    filteredSongs(): Song[] {

      const query =
        this.searchQuery.trim().toLowerCase()


      return this.songs.filter((song) => {

        const matchesSearch =

          !query ||

          song.title
            .toLowerCase()
            .includes(query) ||

          song.artist
            .toLowerCase()
            .includes(query)


        const matchesGenre =

          this.selectedFilter === 'All' ||

          song.genre === this.selectedFilter


        const matchesPlaylist =

          !this.showPlaylistOnly ||

          song.inPlaylist


        return (

          matchesSearch &&

          matchesGenre &&

          matchesPlaylist

        )

      })

    },


    /*
     * computed property:
     * Counts songs in the playlist.
     */
    playlistCount(): number {

      return this.songs.filter(
        song => song.inPlaylist
      ).length

    },


    /*
     * computed property:
     * Counts favorite songs.
     */
    favoriteCount(): number {

      return this.songs.filter(
        song => song.isFavorite
      ).length

    }

  },


  /*
   * METHODS
   */
  methods: {

    /*
     * method:
     * Adds a song to the playlist.
     */
    addSong(song: Song) {

      song.inPlaylist = true

      this.showNotification(
        `"${song.title}" added to your playlist.`
      )

    },


    /*
     * method:
     * Removes a song from the playlist.
     */
    removeSong(id: number) {

      const song =
        this.songs.find(
          item => item.id === id
        )


      if (song) {

        song.inPlaylist = false

        this.showNotification(
          `"${song.title}" removed from your playlist.`
        )

      }

    },


    /*
     * method:
     * Favorites / unfavorites a song.
     */
    toggleFavorite(song: Song) {

      song.isFavorite =
        !song.isFavorite


      this.showNotification(

        song.isFavorite

          ? `"${song.title}" added to favorites.`

          : `"${song.title}" removed from favorites.`

      )

    },


    /*
     * method:
     * Selects the current song.
     */
    selectSong(song: Song) {

      this.currentSong = song

      this.showNotification(
        `Now viewing "${song.title}".`
      )

    },


    /*
     * Vue event method for search.
     */
    searchSongs() {

      // Search updates automatically using v-model.

    },


    /*
     * Vue event method for filtering.
     */
    filterSongs() {

      // Filtering updates automatically using v-model.

    },


    /*
     * Shows only playlist songs.
     */
    togglePlaylistOnly() {

      this.showPlaylistOnly =
        !this.showPlaylistOnly

    },


    /*
     * Resets all filters.
     */
    resetFilters() {

      this.searchQuery = ''

      this.selectedFilter = 'All'

      this.showPlaylistOnly = false

    },


    /*
     * Changes between light and dark mode.
     */
    toggleDarkMode() {

      this.darkMode =
        !this.darkMode


      document.body.classList.toggle(
        'dark-mode',
        this.darkMode
      )

    },


    /*
     * Displays a temporary notification.
     */
    showNotification(message: string) {

      this.notification = message


      if (this.notificationTimer) {

        clearTimeout(
          this.notificationTimer
        )

      }


      this.notificationTimer =
        setTimeout(() => {

          this.notification = ''

        }, 2200)

    }

  }

})

</script>