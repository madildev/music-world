<template>
  <div id="app">
    <div class="app-container">
      <nav class="main-nav">
        <div class="logo">
          <router-link to="/" class="logo-link">
            <i class="fas fa-music"></i>
            <h1>MusicWorld</h1>
          </router-link>
        </div>
      </nav>

      <main class="main-content">
        <router-view></router-view>
      </main>

      <footer class="footer">
        <div class="footer-content">
          <p class="copyright">&copy; 2025 MusicWorld. All rights reserved.</p>
          <div class="social-links">
            <a
              href="https://github.com/madildev"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="GitHub"
            >
              <i class="fab fa-github"></i>
            </a>
            <a
              href="https://linkedin.com/in/madildev"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="LinkedIn"
            >
              <i class="fab fa-linkedin-in"></i>
            </a>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  name: "App"
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;800&display=swap");

:root {
  --primary-color: #1db954;
  --secondary-color: #191414;
  --text-color: #ffffff;
  --border-radius: 12px;
  --transition-speed: 0.3s;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Oxygen, Ubuntu, Cantarell, sans-serif;
  background-color: var(--secondary-color);
  color: var(--text-color);
}

#app {
  min-height: 100vh;
}

.app-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
}

.main-nav {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 1.5rem 2rem;
  margin-bottom: 1rem;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 0 0 12px 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.logo-link {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  text-decoration: none;
  transition: transform 0.3s ease;
}

.logo-link:hover {
  transform: translateX(5px);
}

.logo i {
  font-size: 1.8rem;
  color: var(--primary-color);
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

.logo h1 {
  margin: 0;
  background: linear-gradient(45deg, var(--primary-color), #4caf50);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-size: 1.8rem;
  font-weight: 800;
  letter-spacing: -0.5px;
  font-family: "Poppins", "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
  text-transform: uppercase;
  position: relative;
}

.logo h1::after {
  content: "";
  position: absolute;
  bottom: -5px;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, var(--primary-color), transparent);
  border-radius: 2px;
}

.main-content {
  flex: 1;
}

.footer {
  background-color: #1a1a1a;
  color: #fff;
  padding: 2rem 0;
  margin-top: 3rem;
  border-top: 1px solid #333;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  text-align: center;
}

.copyright {
  color: #b3b3b3;
  margin: 0;
  font-size: 0.95rem;
}

.social-links {
  display: flex;
  gap: 1.5rem;
}

.social-links a {
  color: #b3b3b3;
  font-size: 1.5rem;
  transition: color 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #2a2a2a;
  transition: all 0.3s ease;
}

.social-links a:hover {
  color: #fff;
  background-color: #1db954;
  transform: translateY(-2px);
}

.footer-bottom {
  text-align: center;
  padding-top: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.footer-bottom p {
  color: rgba(255, 255, 255, 0.5);
}

@media screen and (max-width: 768px) {
  .nav-links {
    display: none;
  }

  .footer-content {
    grid-template-columns: 1fr;
    text-align: center;
  }

  .social-links {
    justify-content: center;
  }
}

@media screen and (max-width: 480px) {
  .logo h1 {
    font-size: 1.25rem;
  }

  .footer-section h3 {
    font-size: 1rem;
  }
}
</style>
