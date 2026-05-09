<template>
  <div class="search-results">
    <!-- Back Navigation -->
    <button @click="goBack" class="back-button">
      <i class="fas fa-arrow-left"></i> Back to Home
    </button>

    <!-- Loading State -->
    <div v-if="loading" class="loading-state">
      <div class="skeleton-section">
        <div class="skeleton-title"></div>
        <div class="tracks-grid">
          <div v-for="i in 6" :key="'skeleton-' + i" class="track-card">
            <div class="skeleton-card">
              <div class="skeleton-image"></div>
              <div class="skeleton-details">
                <div class="skeleton-line title"></div>
                <div class="skeleton-line artist"></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Loading state for Hit Tracks -->
      <div class="skeleton-section">
        <div class="skeleton-title"></div>
        <div class="tracks-grid">
          <div v-for="i in 10" :key="'skeleton-' + i" class="track-card">
            <div class="skeleton-card">
              <div class="skeleton-image"></div>
              <div class="skeleton-details">
                <div class="skeleton-line title"></div>
                <div class="skeleton-line artist"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Search Results Section -->
    <div v-else-if="show && tracks.length > 0">
      <h2 class="section-title">
        Search Results for "{{ $route.params.searched }}"
      </h2>
      <div class="tracks-grid">
        <div
          v-for="(data, index) in tracks"
          :key="'search-' + index"
          class="track-card"
        >
          <div class="card-inner">
            <router-link
              :to="{
                name: 'Player',
                params: { songid: data.id },
                query: { v2: true }
              }"
              class="card-link"
            >
              <div class="card-image">
                <img
                  :src="
                    getV2SongCoverArt(data) ||
                      'https://via.placeholder.com/500x500?text=No+Image'
                  "
                  :alt="data.attributes.name || 'Track cover'"
                  loading="lazy"
                />
                <div class="card-overlay">
                  <button class="play-button" aria-label="Play">
                    <i class="fas fa-play"></i>
                  </button>
                </div>
                <div class="card-gradient"></div>
              </div>
              <div class="card-details">
                <h3
                  class="track-title"
                  :title="data.attributes.name || 'Unknown Title'"
                >
                  {{ data.attributes.name || "Unknown Title" }}
                </h3>
                <p
                  class="track-artist"
                  :title="data.attributes.artistName || 'Unknown Artist'"
                >
                  {{ data.attributes.artistName || "Unknown Artist" }}
                </p>
              </div>
            </router-link>
          </div>
        </div>
      </div>
    </div>

    <!-- Artist's Top Hits Section -->
    <div v-if="show && hits.length > 0" class="artist-hits">
      <h2 class="section-title">Artist's Top Hits</h2>
      <div class="tracks-grid">
        <div
          v-for="(data, index) in hits"
          :key="'hits-' + index"
          class="track-card"
        >
          <div class="card-inner">
            <router-link
              :to="{
                name: 'Player',
                params: { songid: data.id },
                query: { v2: true }
              }"
              class="card-link"
            >
              <div class="card-image">
                <img
                  :src="
                    getV2SongCoverArt(data) ||
                      'https://via.placeholder.com/500x500?text=No+Image'
                  "
                  :alt="data.attributes.name || 'Track cover'"
                  loading="lazy"
                />
                <div class="card-overlay">
                  <button class="play-button" aria-label="Play">
                    <i class="fas fa-play"></i>
                  </button>
                </div>
                <div class="card-gradient"></div>
              </div>
              <div class="card-details">
                <h3
                  class="track-title"
                  :title="data.attributes.name || 'Unknown Title'"
                >
                  {{ data.attributes.name || "Unknown Title" }}
                </h3>
                <p
                  class="track-artist"
                  :title="data.attributes.artistName || 'Unknown Artist'"
                >
                  {{ data.attributes.artistName || "Unknown Artist" }}
                </p>
              </div>
            </router-link>
          </div>
        </div>
      </div>
    </div>

    <!-- No Results Message -->
    <div
      v-if="show && tracks.length === 0 && hits.length === 0"
      class="no-results"
    >
      <i class="fas fa-search no-results-icon"></i>
      <h2>No results found for "{{ $route.params.searched }}"</h2>
      <p>Try different keywords or check the spelling</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "SearchResults",
  data() {
    return {
      show: false,
      loading: true,
      error: null,
      songs: {}, // This object gets incoming object from the api
      tracks: [], // This stores the arrays of the searched song
      hits: [], // This stores the hits songs returned by api
      artistid: "850571371" // This extracts the id from the api
    };
  },
  async mounted() {
    try {
      this.loading = true;
      let url = process.env.VUE_APP_SEARCH_SONGS
            .replace("{0}", encodeURIComponent(this.$route.params.searched));
      const response = await fetch(url,
        {
          method: "GET",
          headers: {
            "x-rapidapi-key": process.env.VUE_APP_RAPID_API_KEY,
            "x-rapidapi-host": process.env.VUE_APP_RAPID_API_HOST
          }
        }
      );

      const data = await response.json();
      this.songs = data.results;
      this.tracks = this.songs?.songs?.data || [];
      this.show = true;

      // If we have tracks, get artist's top hits
      if (this.tracks.length > 0) {
        this.artistid = this.songs?.artists?.data[0]?.id ?? this.artistid;
        if (this.artistid) {
          await this.gethitsongs();
        }
      }
    } catch (error) {
      this.error = "Failed to load search results. Please try again.";
    } finally {
      this.loading = false;
    }
  },
  methods: {
    // Go back to previous page or home
    goBack() {
      if (window.history.length > 1) {
        this.$router.go(-1);
      } else {
        this.$router.push("/");
      }
    },
    getV2SongCoverArt(song) {
      const height = 300;
      const weidth = 300;
      if (
        song &&
        song.attributes &&
        song.attributes.artwork &&
        song.attributes.artwork.url
      ) {
        return song.attributes.artwork.url
          .replace("{w}", weidth)
          .replace("{h}", height);
      }
      return "";
    },
    // This function gets the hit songs of the artist
    async gethitsongs() {
      if (!this.artistid) return;

      try {
        let url = process.env.VUE_APP_GET_TOP_SONGS
                .replace("{0}",this.artistid);
        const response = await fetch(url,
          {
            method: "GET",
            headers: {
              "x-rapidapi-key": process.env.VUE_APP_RAPID_API_KEY,
              "x-rapidapi-host": process.env.VUE_APP_RAPID_API_HOST
            }
          }
        );

        if (!response.ok) throw new Error("Failed to fetch artist top songs");

        const data = await response.json();
        this.hits = data.data || [];
      } catch (error) {
        this.error = "Failed to load artist top hits.";
      }
    }
  },
  // This watcher calls when the artist ID changes
  watch: {
    artistid(id) {
      if (id) {
        this.gethitsongs();
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* Back Button Styles */
.back-button {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: transparent;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 500;
  margin-bottom: 1.5rem;
  transition: all 0.2s ease;
}

.back-button:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: translateX(-4px);
}

.back-button i {
  font-size: 0.9em;
}

/* Skeleton Loader Styles */
.loading-state {
  width: 100%;
}

.skeleton-section {
  margin-bottom: 3rem;
}

.skeleton-title {
  width: 200px;
  height: 28px;
  background: #2a2a2a;
  margin-bottom: 1.5rem;
  border-radius: 4px;
  position: relative;
  overflow: hidden;
}

.skeleton-card {
  background: #2a2a2a;
  border-radius: 12px;
  overflow: hidden;
  height: 100%;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.skeleton-image {
  width: 100%;
  padding-bottom: 100%;
  background: #3a3a3a;
  position: relative;
  overflow: hidden;
}

.skeleton-details {
  padding: 1rem;
}

.skeleton-line {
  height: 16px;
  background: #3a3a3a;
  border-radius: 4px;
  margin-bottom: 0.75rem;
  position: relative;
  overflow: hidden;
}

.skeleton-line.title {
  width: 90%;
  height: 20px;
  margin-bottom: 0.5rem;
}

.skeleton-line.artist {
  width: 70%;
  height: 14px;
  margin-bottom: 0;
}

/* Skeleton Animation */
@keyframes shimmer {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

.skeleton-line::after,
.skeleton-image::after,
.skeleton-title::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.05),
    transparent
  );
  animation: shimmer 1.5s infinite;
}

/* Error State */
.error-message {
  text-align: center;
  padding: 2rem;
  color: #ff6b6b;
  background: rgba(255, 107, 107, 0.1);
  border-radius: 8px;
  margin: 2rem 0;
}

.error-message i {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  display: block;
}

/* Existing Styles */
.search-results {
  padding: 24px 32px;
  max-width: 1600px;
  margin: 0 auto;
}

.section-title {
  font-size: 1.75rem;
  color: #ffffff;
  margin: 2rem 0 1.5rem;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.tracks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 24px;
  margin-bottom: 40px;
}

.track-card {
  position: relative;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  background: #181818;
  padding: 16px;
  height: 100%;
}

.track-card:hover {
  background: #282828;
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5);
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.card-link {
  text-decoration: none;
  color: inherit;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.card-image {
  position: relative;
  width: 100%;
  padding-bottom: 100%;
  margin-bottom: 16px;
  background: #282828;
  border-radius: 4px;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
}

.card-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.card-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 2;
}

.card-gradient {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 50%;
  background: linear-gradient(to bottom, transparent, rgba(0, 0, 0, 0.7));
  z-index: 1;
}

.track-card:hover .card-overlay {
  opacity: 1;
}

.track-card:hover .card-image img {
  transform: scale(1.05);
}

.play-button {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #1db954;
  border: none;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  opacity: 0;
  transform: translateY(8px);
  transition: all 0.3s ease;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.track-card:hover .play-button {
  opacity: 1;
  transform: translateY(0);
}

.play-button:hover {
  transform: scale(1.1) !important;
  background: #1ed760;
}

.card-details {
  padding: 8px 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.track-title {
  font-size: 1rem;
  font-weight: 600;
  color: #ffffff;
  margin: 0 0 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.track-artist {
  font-size: 0.875rem;
  color: #b3b3b3;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.no-results {
  text-align: center;
  padding: 80px 20px;
  color: #b3b3b3;
}

.no-results-icon {
  font-size: 64px;
  margin-bottom: 24px;
  color: #535353;
}

.no-results h2 {
  color: #ffffff;
  margin-bottom: 12px;
  font-size: 1.5rem;
}

.artist-hits {
  margin-top: 40px;
}

/* Responsive Design */
@media (max-width: 1200px) {
  .tracks-grid {
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  }
}

@media (max-width: 992px) {
  .tracks-grid {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 20px;
  }

  .section-title {
    font-size: 1.5rem;
  }
}

@media (max-width: 768px) {
  .search-results {
    padding: 16px;
  }

  .tracks-grid {
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
    gap: 16px;
  }

  .section-title {
    font-size: 1.4rem;
    margin: 1.5rem 0 1.25rem;
  }

  .track-title {
    font-size: 0.9375rem;
  }

  .track-artist {
    font-size: 0.8125rem;
  }
}

@media (max-width: 480px) {
  .tracks-grid {
    grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
    gap: 12px;
  }

  .track-card {
    padding: 12px;
  }

  .section-title {
    font-size: 1.3rem;
    margin: 1.25rem 0 1rem;
  }
}
</style>
