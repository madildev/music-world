<template>
  <div class="trending-container">
    <div class="search-section">
      <h2>Discover Music</h2>
      <div class="search-container">
        <input
          type="text"
          v-model="searched"
          placeholder="Search for songs, artists, or albums..."
          class="search-input"
        />
        <router-link
          :to="{ name: 'Search', params: { searched: searched } }"
          class="search-button"
        >
          <i class="fas fa-search"></i>
        </router-link>
      </div>
    </div>

    <h3 class="section-title">Trending Now</h3>
    <!-- Skeleton Loaders -->
    <div v-if="loading" class="loading-state">
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

    <!-- Actual Results -->
    <div v-if="!loading && hits.length > 0" class="trending-grid">
      <div v-for="(track, index) in hits" :key="index" class="track-card">
        <div class="card-inner">
          <router-link
            :to="{ name: 'Player', params: { songid: track.id } }"
            class="card-link"
          >
            <div class="card-image">
              <img
                  :src="
                    getV2SongCoverArt(track) ||
                      'https://via.placeholder.com/500x500?text=No+Image'
                  "
                  :alt="track.attributes.name || 'Track cover'"
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
              <h3 class="track-title" :title="track.attributes.name || 'Unknown Title'">
                {{ track.attributes.name || "Unknown Title" }}
              </h3>
              <p
                class="track-artist"
                :title="track.attributes.artistName || 'Unknown Artist'"
              >
                {{ track.attributes.artistName || "Unknown Artist" }}
              </p>
            </div>
          </router-link>
        </div>
      </div>
    </div>

    <!-- No Results Message -->
    <div
      v-if="!loading && hits?.length === 0 && !isSearching"
      class="no-results"
    >
      <i class="fas fa-music"></i>
      <p>The trending tracks are playing hard to get.</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Trending",
  data() {
    return {
      hits: [],
      show: false, // This controls the visibility of the elements
      searched: "",
      loading: true,
      isSearching: false,
      artistid: "320569549"
    };
  },
  // This is mounted hook that runs when the component is mounted
  async mounted() {
    this.loading = true;
    this.isSearching = true;
    try {
       await this.fetchTrendingTracks();
    } catch (error) {
      this.hits = [];
    } finally {
      this.loading = false;
      this.isSearching = false;
    }
  },
  methods: {
    async fetchTrendingTracks() 
    {
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
        if(this.hits?.length > 0)
          this.show = true;
      } catch (error) {
        this.error = "Failed to load artist top hits.";
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
    }
  },
  watch: {
    // Reset searching state when component is mounted or when search is cleared
    $route() {
      this.isSearching = false;
    }
  }
};
</script>

<style scoped>
/* Skeleton Loader Styles */
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

.tracks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 24px;
  margin-bottom: 40px;
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
.skeleton-image::after {
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

/* No Results Styling */
.no-results {
  text-align: center;
  padding: 4rem 2rem;
  color: #b3b3b3;
  width: 100%;
  grid-column: 1 / -1;
}

.no-results i {
  font-size: 3rem;
  margin-bottom: 1rem;
  opacity: 0.5;
}

.no-results p {
  font-size: 1.1rem;
  margin: 0;
}

/* Existing Styles */
.trending-container {
  padding: 0 1.5rem 2rem;
  max-width: 1400px;
  margin: 0 auto;
  color: #e0e0e0;
}

.section-title {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 2rem 0 1.5rem;
  color: #ffffff;
  position: relative;
  padding-bottom: 0.75rem;
}

.section-title::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 60px;
  height: 4px;
  background: linear-gradient(90deg, #1db954, #1ed760);
  border-radius: 2px;
}

.search-section {
  margin-bottom: 2rem;
  text-align: center;
  position: relative;
  padding: 2rem 0;
  background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 100%);
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  margin-top: 1rem;
}

.search-section h2 {
  font-size: 2rem;
  margin-bottom: 1.5rem;
  color: var(--text-color);
}

.search-container {
  display: flex;
  max-width: 600px;
  margin: 0 auto;
  position: relative;
  padding: 0 1rem;
  z-index: 1;
}

.search-input {
  flex: 1;
  padding: 1rem 1.5rem;
  border: none;
  border-radius: 30px;
  font-size: 1.05rem;
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.search-input:focus {
  outline: none;
  background: rgba(255, 255, 255, 0.12);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

.search-button {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(135deg, #1db954, #1ed760);
  border: none;
  border-radius: 50%;
  width: 46px;
  height: 46px;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(29, 185, 84, 0.3);
  text-decoration: none;
}

.search-button:hover {
  background: #1ed760;
  transform: translateY(-50%) scale(1.05);
}

/* Search Loading State */
.search-loading {
  margin-top: 2rem;
  text-align: center;
  padding: 1.5rem;
  color: rgba(255, 255, 255, 0.8);
}

.search-loader {
  display: inline-block;
  width: 50px;
  height: 50px;
  border: 4px solid rgba(29, 185, 84, 0.2);
  border-radius: 50%;
  border-top-color: #1db954;
  animation: spin 1s ease-in-out infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Grid Layout */
.trending-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1.75rem;
  padding: 1rem 0;
}

.track-card {
  perspective: 1000px;
  height: 100%;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  transform-style: preserve-3d;
  border-radius: 12px;
  overflow: hidden;
  background: #282828;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}

.track-card:hover .card-inner {
  transform: translateY(-8px);
}

.card-link {
  display: block;
  height: 100%;
  text-decoration: none;
  color: inherit;
}

.card-image {
  position: relative;
  width: 100%;
  padding-bottom: 100%;
  overflow: hidden;
  background: #333;
}

.card-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: all 0.5s ease;
  z-index: 1;
}

.card-gradient {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.1) 0%,
    rgba(0, 0, 0, 0.7) 100%
  );
  z-index: 2;
  opacity: 0.8;
  transition: opacity 0.3s ease;
}

.track-card:hover .card-gradient {
  opacity: 0.6;
}

.card-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 3;
}

.track-card:hover .card-overlay {
  opacity: 1;
}

.play-button {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: #1db954;
  border: none;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: scale(0.8);
  transition: all 0.3s ease;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.track-card:hover .play-button {
  transform: scale(1);
}

.play-button:hover {
  transform: scale(1.1);
  background: #1ed760;
}

.card-details {
  padding: 1.25rem 1rem;
  position: relative;
  z-index: 2;
  background: #181818;
  transition: background 0.3s ease;
}

.track-card:hover .card-details {
  background: #282828;
}

.track-title {
  margin: 0 0 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  color: #ffffff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: color 0.3s ease;
}

.track-artist {
  margin: 0;
  font-size: 0.875rem;
  color: #b3b3b3;
  font-weight: 400;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: color 0.3s ease;
}

.track-card:hover .track-title,
.track-card:hover .track-artist {
  color: #1db954;
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

@keyframes loading {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

@media (max-width: 1200px) {
  .trending-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  }
}

@media (max-width: 992px) {
  .trending-grid {
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 1.5rem;
  }
}

@media (max-width: 768px) {
  .trending-grid {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 1.25rem;
  }

  .section-title {
    font-size: 1.5rem;
    margin: 1.5rem 0 1.25rem;
  }

  .search-section h2 {
    font-size: 1.5rem;
    margin-bottom: 1.25rem;
  }

  .search-container {
    margin-bottom: 1.5rem;
  }
}

@media (max-width: 576px) {
  .trending-grid {
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
    gap: 1rem;
  }

  .section-title {
    font-size: 1.4rem;
    margin: 1.25rem 0 1rem;
  }

  .card-details {
    padding: 1rem 0.75rem;
  }

  .track-title {
    font-size: 0.9375rem;
  }

  .track-artist {
    font-size: 0.8125rem;
  }
}

@media (max-width: 400px) {
  .trending-grid {
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 0.75rem;
  }

  .card-details {
    min-height: 60px;
  }

  .card-details h3 {
    font-size: 0.8125rem;
  }

  .card-details h4 {
    font-size: 0.6875rem;
  }
}
</style>
