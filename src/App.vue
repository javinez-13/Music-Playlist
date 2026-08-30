<template>
  <div class="app-shell">
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
      <section class="workspace">
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
                  <button
                    class="favorite-button"
                    :class="{ active: song.isFavorite }"
                    @click="toggleFavorite(song)"
                  >
                    {{ song.isFavorite ? '♥' : '♡' }}
                  </button>

                </div>
                <div class="song-meta">

                  <span>
                    {{ song.genre }}
                  </span>

                  <span>•</span>

                  <span>
                    {{ song.duration }}
                  </span>

                </div>
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
          <div class="playlist-status">

            <h3>
              Playlist Status
            </h3>
            <p v-if="playlistCount === 0">
              Your playlist is empty.
            </p>
            <p v-else-if="playlistCount < 3">
              Add more songs to build your playlist!
            </p>
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
    <div
      v-if="notification"
      class="notification"
    >
      {{ notification }}
    </div>

  </div>
</template>


```vue
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

  data() {
    const songs: Song[] = [
      {
        id: 1,
        title: 'Tadhana',
        artist: 'Up Dharma Down',
        genre: 'OPM',
        duration: '4:01',
        cover: 'https://i1.sndcdn.com/artworks-klRPLROGlkbVZu6X-hac0FQ-t500x500.jpg',
        isFavorite: true,
        inPlaylist: true
      },

      {
        id: 2,
        title: 'Mundo',
        artist: 'IV of Spades',
        genre: 'OPM',
        duration: '4:16',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSLF2fCYmgPfNhgwCtNpY4epq0NCm8I_hux6JDRuTq0oA&s=10',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 3,
        title: 'Buwan',
        artist: 'Juan Karlos',
        genre: 'OPM',
        duration: '4:27',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRJ0WDIh6Maz2F7yCEd7RK7lMhMb9fHrvMH98GurUdEqw&s=10',
        isFavorite: true,
        inPlaylist: true
      },

      {
        id: 4,
        title: 'Raining in Manila',
        artist: 'Lola Amour',
        genre: 'OPM',
        duration: '4:17',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRFwIGlC0cH6qQU0zebG5DIxn2l68w8o5Rq5Tf7sz9iOg&s=10',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 5,
        title: 'Kathang Isip',
        artist: 'Ben&Ben',
        genre: 'OPM',
        duration: '4:17',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTap_FoINujsMRKNoOcPZYBi9TPgxEiGoyB3coGk_4pew&s=10',
        isFavorite: true,
        inPlaylist: false
      },

      {
        id: 6,
        title: 'Leaves',
        artist: 'Ben&Ben',
        genre: 'OPM',
        duration: '4:00',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS6h_FP8bYWW6_XNyEGq1KJ06b1HAwPqA6qCKaL_yB4RA&s=10',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 7,
        title: 'With a Smile',
        artist: 'Eraserheads',
        genre: 'OPM',
        duration: '4:37',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRF7JcDs5SvHD2RO11cdM7f4eNsjrghjqeFtWmOuRLC0w&s=10',
        isFavorite: false,
        inPlaylist: true
      },

      {
        id: 8,
        title: '214',
        artist: 'Rivermaya',
        genre: 'OPM',
        duration: '4:35',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSy_8hnMqA4RHqTAKfvX4Nzt0KPjI_evaOSludBKZPcAQ&s=10',
        isFavorite: false,
        inPlaylist: false
      },

      {
        id: 9,
        title: 'Migraine',
        artist: 'Moonstar88',
        genre: 'OPM',
        duration: '4:03',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQbakfOpwvwjzwzzJPFoGdBCuNKOJxP8c7aZInscXZ5dw&s=10',
        isFavorite: true,
        inPlaylist: false
      },

      {
        id: 10,
        title: 'Nobela',
        artist: 'Join The Club',
        genre: 'OPM',
        duration: '4:14',
        cover: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRdojML_O-EJDWnki4BdHKSP25hcrHirQYUSzvjMyOY4A&s=10',
        isFavorite: false,
        inPlaylist: false
      }
    ]

    return {
      appName: 'My OPM Playlist',

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
        'OPM'
      ]
    }
  },

  computed: {
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

    playlistCount(): number {
      return this.songs.filter(
        song => song.inPlaylist
      ).length
    },

    favoriteCount(): number {
      return this.songs.filter(
        song => song.isFavorite
      ).length
    }
  },

  methods: {
    addSong(song: Song) {
      song.inPlaylist = true

      this.showNotification(
        `"${song.title}" added to your playlist.`
      )
    },

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

    toggleFavorite(song: Song) {
      song.isFavorite =
        !song.isFavorite

      this.showNotification(
        song.isFavorite
          ? `"${song.title}" added to favorites.`
          : `"${song.title}" removed from favorites.`
      )
    },

    selectSong(song: Song) {
      this.currentSong = song

      this.showNotification(
        `Now viewing "${song.title}".`
      )
    },

    searchSongs() {
      // Search is handled automatically
      // through the filteredSongs computed property.
    },

    filterSongs() {
      // Filtering is handled automatically
      // through the filteredSongs computed property.
    },

    togglePlaylistOnly() {
      this.showPlaylistOnly =
        !this.showPlaylistOnly
    },

    resetFilters() {
      this.searchQuery = ''
      this.selectedFilter = 'All'
      this.showPlaylistOnly = false
    },

    toggleDarkMode() {
      this.darkMode =
        !this.darkMode

      document.body.classList.toggle(
        'dark-mode',
        this.darkMode
      )
    },

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
```
