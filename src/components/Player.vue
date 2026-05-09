<template>
  <div class="player-container">
    <button class="back-button" @click="$router.go(-1)">
      <i class="fas fa-arrow-left"></i> Back
    </button>

    <div v-if="!isLoading" class="player-content">
      <!-- Album Art Section -->
      <div class="album-art">
        <div class="album-art-container">
          <img
            :src="
              getV2SongCoverArt() ||
                'https://via.placeholder.com/500x500?text=No+Image'
            "
            :alt="'Album Art'"
            class="album-image"
            :class="{ 'is-playing': isPlaying }"
          />
          <div class="album-overlay" @click="togglePlay">
            <button class="play-button" aria-label="Play/Pause">
              <i :class="isPlaying ? 'fas fa-pause' : 'fas fa-play'"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Song Info Section -->
      <div class="song-info">
        <h1 class="song-title">
          {{ songs.attributes?.name || "Unknown Title" }}
        </h1>
        <p class="song-artist">
          {{ songs.attributes?.artistName || "Unknown Artist" }}
        </p>

        <!-- Audio Player -->
        <div class="audio-player">
          <audio
            ref="audioPlayer"
            :src="audioSource"
            @timeupdate="updateProgress"
            @ended="onSongEnd"
          ></audio>

          <div class="progress-container">
            <span class="time current">{{ formatTime(currentTime) }}</span>
            <div class="progress-bar" @click="seek">
              <div class="progress" :style="{ width: progress + '%' }"></div>
            </div>
            <span class="time duration">{{ formatTime(duration) }}</span>
          </div>

          <div class="player-controls">
            <button
              class="control-btn"
              @click="toggleShuffle"
              :class="{ active: isShuffled }"
            >
              <i class="fas fa-random"></i>
            </button>
            <button class="control-btn" @click="previousTrack">
              <i class="fas fa-step-backward"></i>
            </button>
            <button class="control-btn play-btn" @click="togglePlay">
              <i :class="isPlaying ? 'fas fa-pause' : 'fas fa-play'"></i>
            </button>
            <button class="control-btn" @click="nextTrack">
              <i class="fas fa-step-forward"></i>
            </button>
            <button
              class="control-btn"
              @click="toggleRepeat"
              :class="{ active: isRepeated }"
            >
              <i class="fas fa-redo"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Song Details Section -->
      <div class="song-details">
        <h2 class="details-title">Song Details</h2>
        <div class="details-grid">
          <div class="detail-item">
            <span class="detail-label">Artist</span>
            <span class="detail-value">{{
              songs.attributes?.artistName || "N/A"
            }}</span>
          </div>
          <div class="detail-item">
            <span class="detail-label">Genre</span>
            <span class="detail-value">{{
              (songs.attributes?.genreNames &&
                songs.attributes?.genreNames[0]) ||
                "N/A"
            }}</span>
          </div>
          <div class="detail-item">
            <span class="detail-label">Type</span>
            <span class="detail-value">{{
              songs.type
                ? songs.type.charAt(0).toUpperCase() + songs.type.slice(1)
                : "N/A"
            }}</span>
          </div>
          <div class="detail-item">
            <span class="detail-label">Released</span>
            <span class="detail-value">{{
              songs.attributes?.releaseDate || "N/A"
            }}</span>
          </div>
        </div>

        <!-- Lyrics Section -->
        <div v-if="lyrics && lyrics.length > 0" class="lyrics-section">
          <h3>Lyrics</h3>
          <div class="lyrics-content">
            <p v-for="(line, index) in formattedLyrics" :key="index">
              {{ line }}
            </p>
          </div>
        </div>
        <div v-else-if="!isLoading" class="no-lyrics">
          <p>No lyrics available for this track.</p>
        </div>
      </div>
    </div>

    <!-- Background Overlay -->
    <div
      v-if="songs.images && songs.images.background"
      class="background-overlay"
      :style="{ backgroundImage: 'url(' + songs.images.background + ')' }"
    ></div>

    <!-- Loading State -->
    <div v-if="isLoading" class="loading-overlay">
      <div class="loading-spinner"></div>
      <p>Loading track details...</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Player",
  data() {
    return {
      id: this.$route.params.songid,
      songs: {},
      audioPlayer: null,
      isPlaying: false,
      currentTime: 0,
      duration: 0,
      progress: 0,
      volume: 0.7,
      isMuted: false,
      isShuffled: false,
      isRepeated: false,
      isLoading: true,
      lyrics: null,
      error: null
    };
  },
  computed: {
    audioSource: function() {
      if (
        this.songs.attributes &&
        this.songs.attributes.previews &&
        this.songs.attributes.previews[0] &&
        this.songs.attributes.previews[0].url
      ) {
        return this.songs.attributes.previews[0].url;
      }
      return "";
    },
    formattedLyrics: function() {
      if (!this.lyrics || typeof this.lyrics !== "string") return [];
      var lines = this.lyrics.split("\n");
      return lines.filter(function(line) {
        return line.trim() !== "";
      });
    }
  },
  watch: {
    audioSource(newVal) {
      if (newVal) {
        this.$nextTick(() => {
          this.setupAudioPlayer();
        });
      }
    },
    $route(to) {
      const isPlayerRoute = to.name === "Player";
      const songIdChanged = to.params.songid !== this.id;

      if (isPlayerRoute && songIdChanged) {
        this.id = to.params.songid;
        this.fetchV2SongDetails();
      }
    }
  },
  async mounted() {
    await this.fetchSongDetails();
    document.addEventListener("keydown", this.handleKeyPress);
  },
  beforeDestroy() {
    if (this.audioPlayer) {
      this.audioPlayer.pause();
      this.audioPlayer = null;
    }
    document.removeEventListener("keydown", this.handleKeyPress);
  },
  methods: {
    async fetchSongDetails() {
      this.isLoading = true;
      this.fetchV2SongDetails();
      this.isLoading = false;
    },
    async fetchV2SongDetails() {
      try {
        const response = await fetch(
          `https://shazam.p.rapidapi.com/songs/v2/get-details?id=${this.id}&l=en-US`,
          {
            method: "GET",
            headers: {
              "x-rapidapi-key":
                "d3c4ef33bfmshea48c175233c56dp1bf0b8jsnffa31f6c4ca3",
              "x-rapidapi-host": "shazam.p.rapidapi.com"
            }
          }
        );

        if (!response.ok) throw new Error("Failed to fetch song details");

        const songs = await response.json();
        this.songs = songs.data[0];
        this.isLoading = false;
      } catch (error) {
        this.isLoading = false;
        this.error = "Failed to load song details. Please try again later.";
      }
    },
    setupAudioPlayer() {
      if (this.audioPlayer) {
        this.audioPlayer.pause();
        this.audioPlayer = null;
      }

      this.audioPlayer = this.$refs.audioPlayer;

      if (this.audioPlayer) {
        this.audioPlayer.volume = this.volume;
        var self = this;
        this.audioPlayer.onloadedmetadata = function() {
          self.duration = self.audioPlayer.duration;
          self.play();
        };
      }
    },
    togglePlay() {
      if (!this.audioPlayer) return;

      if (this.audioPlayer.paused) {
        this.play();
      } else {
        this.pause();
      }
    },
    play() {
      if (!this.audioPlayer) return;

      this.audioPlayer
        .play()
        .then(() => {
          this.isPlaying = true;
        })
        .catch(error => {
          this.isPlaying = false;
        });
    },
    getV2SongCoverArt() {
      const height = 300;
      const weidth = 300;
      if (
        this.songs &&
        this.songs.attributes &&
        this.songs.attributes.artwork &&
        this.songs.attributes.artwork.url
      ) {
        return this.songs.attributes.artwork.url
          .replace("{w}", weidth)
          .replace("{h}", height);
      }
      return "";
    },
    pause() {
      if (!this.audioPlayer) return;
      this.audioPlayer.pause();
      this.isPlaying = false;
    },
    updateProgress() {
      if (!this.audioPlayer) return;
      this.currentTime = this.audioPlayer.currentTime;
      this.progress = (this.currentTime / this.duration) * 100;
    },
    seek(e) {
      if (!this.audioPlayer) return;
      const seekPosition = (e.offsetX / e.target.offsetWidth) * this.duration;
      this.audioPlayer.currentTime = seekPosition;
    },
    formatTime(seconds) {
      if (isNaN(seconds)) return "0:00";
      var minutes = Math.floor(seconds / 60);
      var remainingSeconds = Math.floor(seconds % 60);
      return `${minutes}:${
        remainingSeconds < 10 ? "0" : ""
      }${remainingSeconds}`;
    },
    toggleShuffle() {
      this.isShuffled = !this.isShuffled;
    },
    toggleRepeat() {
      this.isRepeated = !this.isRepeated;
    },
    nextTrack() {},
    previousTrack() {},
    onSongEnd() {
      this.isPlaying = false;
      if (this.isRepeated) {
        this.audioPlayer.currentTime = 0;
        this.play();
      } else {
        this.nextTrack();
      }
    },
    handleKeyPress(e) {
      if (e.code === "Space") {
        e.preventDefault();
        this.togglePlay();
      } else if (e.code === "ArrowRight") {
        this.seekForward();
      } else if (e.code === "ArrowLeft") {
        this.seekBackward();
      } else if (e.code === "KeyM") {
        this.toggleMute();
      }
    },
    seekForward() {
      if (!this.audioPlayer) return;
      this.audioPlayer.currentTime = Math.min(
        this.audioPlayer.duration,
        this.audioPlayer.currentTime + 5
      );
    },
    seekBackward() {
      if (!this.audioPlayer) return;
      this.audioPlayer.currentTime = Math.max(
        0,
        this.audioPlayer.currentTime - 5
      );
    },
    toggleMute() {
      if (!this.audioPlayer) return;
      this.isMuted = !this.isMuted;
      this.audioPlayer.muted = this.isMuted;
    }
  }
};
</script>

<style scoped>
/* Base Styles */
.player-container {
  position: relative;
  min-height: 100vh;
  color: #ffffff;
  padding: 2rem;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background: linear-gradient(to bottom, #1a1a1a, #2a2a2a);
}

.background-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  opacity: 0.2;
  z-index: 0;
  filter: blur(50px);
}

.player-content {
  position: relative;
  z-index: 1;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* Album Art Section */
.album-art {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.album-art-container {
  position: relative;
  width: 300px;
  height: 300px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  transition: transform 0.3s ease;
}

.album-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.album-image.is-playing {
  animation: rotateAlbumArt 15s linear infinite;
}

@keyframes rotateAlbumArt {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.album-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  cursor: pointer;
}

.album-art-container:hover .album-overlay {
  opacity: 1;
}

.play-button {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: #1db954;
  border: none;
  color: white;
  font-size: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.play-button:hover {
  transform: scale(1.1);
  background: #1ed760;
}

/* Song Info Section */
.song-info {
  text-align: center;
  margin-bottom: 3rem;
}

.song-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 0.5rem;
  color: #ffffff;
}

.song-artist {
  font-size: 1.25rem;
  color: #b3b3b3;
  margin: 0 0 2rem;
}

/* Audio Player */
.audio-player-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  color: #fff;
  position: relative;
}

.back-button {
  position: absolute;
  top: 1rem;
  left: 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 10;
}

.back-button:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
}

.back-button i {
  font-size: 0.9rem;
}

.audio-player {
  max-width: 600px;
  margin: 0 auto;
  padding: 1.5rem;
  background: rgba(40, 40, 40, 0.7);
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.progress-container {
  display: flex;
  align-items: center;
  margin: 1.5rem 0;
  gap: 1rem;
}

.time {
  font-size: 0.875rem;
  color: #b3b3b3;
  min-width: 40px;
  text-align: center;
}

.progress-bar {
  flex: 1;
  height: 6px;
  background: #404040;
  border-radius: 3px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: height 0.2s ease;
}

.progress-bar:hover {
  height: 8px;
}

.progress {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: #1db954;
  border-radius: 3px;
  transition: width 0.1s linear;
}

/* Player Controls */
.player-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.control-btn {
  background: none;
  border: none;
  color: #b3b3b3;
  font-size: 1.25rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.control-btn:hover {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}

.control-btn.active {
  color: #1db954;
}

.control-btn.play-btn {
  width: 50px;
  height: 50px;
  background: #1db954;
  color: white;
  font-size: 1.5rem;
}

.control-btn.play-btn:hover {
  transform: scale(1.1);
  background: #1ed760;
}

/* Song Details Section */
.song-details {
  background: rgba(30, 30, 30, 0.7);
  border-radius: 12px;
  padding: 2rem;
  margin-top: 2rem;
  backdrop-filter: blur(10px);
}

.details-title {
  font-size: 1.5rem;
  margin: 0 0 1.5rem;
  color: #ffffff;
  position: relative;
  padding-bottom: 0.75rem;
}

.details-title::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 60px;
  height: 3px;
  background: #1db954;
  border-radius: 3px;
}

.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.detail-item {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
}

.detail-label {
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #b3b3b3;
  margin-bottom: 0.5rem;
}

.detail-value {
  font-size: 1rem;
  color: #ffffff;
  font-weight: 500;
}

/* Lyrics Section */
.lyrics-section {
  margin-top: 2rem;
}

.lyrics-section h3 {
  font-size: 1.25rem;
  margin: 0 0 1rem;
  color: #ffffff;
}

.lyrics-content {
  max-height: 300px;
  overflow-y: auto;
  padding: 1rem;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  line-height: 1.8;
  color: #e0e0e0;
}

.lyrics-content p {
  margin: 0.5rem 0;
}

.no-lyrics {
  text-align: center;
  color: #b3b3b3;
  padding: 2rem 0;
  font-style: italic;
}

/* Loading State */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(255, 255, 255, 0.1);
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

/* Responsive Design */
@media (max-width: 768px) {
  .player-container {
    padding: 1rem;
  }

  .album-art-container {
    width: 250px;
    height: 250px;
  }

  .song-title {
    font-size: 2rem;
  }

  .song-artist {
    font-size: 1.1rem;
  }

  .details-grid {
    grid-template-columns: 1fr 1fr;
  }

  .audio-player {
    padding: 1rem;
  }

  .player-controls {
    gap: 1rem;
  }

  .control-btn {
    width: 36px;
    height: 36px;
    font-size: 1.1rem;
  }

  .control-btn.play-btn {
    width: 44px;
    height: 44px;
  }
}

@media (max-width: 480px) {
  .album-art-container {
    width: 200px;
    height: 200px;
  }

  .song-title {
    font-size: 1.75rem;
  }

  .song-artist {
    font-size: 1rem;
  }

  .details-grid {
    grid-template-columns: 1fr;
  }

  .progress-container {
    margin: 1rem 0;
  }

  .time {
    font-size: 0.75rem;
    min-width: 35px;
  }

  .song-details {
    padding: 1.25rem;
  }

  .details-title {
    font-size: 1.3rem;
  }
}
</style>
