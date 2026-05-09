<template>
  <div class="home">
    <div class="hero-section">
      <h1>Discover New Music</h1>
      <p>Search and explore millions of songs and artists</p>
    </div>
    <Trending />
  </div>
</template>

<script>
import Trending from "@/components/Trending.vue";

export default {
  name: "Home",
  components: {
    Trending
  }
};
</script>

<style scoped>
.home {
  padding: 2rem 0;
}

.hero-section {
  text-align: center;
  padding: 4rem 2rem;
  background: linear-gradient(135deg, var(--secondary-color) 0%, #1a1a1a 100%);
  border-radius: var(--border-radius);
  margin-bottom: 2rem;
}

.hero-section h1 {
  font-size: 3rem;
  margin-bottom: 1rem;
  color: var(--text-color);
  font-weight: 800;
}

.hero-section p {
  font-size: 1.25rem;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 0;
}

@media (max-width: 768px) {
  .hero-section {
    padding: 3rem 1rem;
  }

  .hero-section h1 {
    font-size: 2.25rem;
  }

  .hero-section p {
    font-size: 1.1rem;
  }
}
</style>
