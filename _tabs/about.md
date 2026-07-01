---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

Hello Friend, I'm **KirolosMoheb**(_**d0natel00**_) This is My Blog To Publish any Thing Belongs To Security Like : Bugs I Found in Applications, CTF Writeups, Security Tips and Tricks, etc. That I Hope It Can Help All Pentesters !

### Personal Links :

**Hackerone**_> [hackerone.com/kirolosmoheb?type=user](https://hackerone.com/kirolosmoheb?type=user)

**Medium**_> [d0natel00.medium.com](https://d0natel00.medium.com/) 

**Contact**_> [contact.me.kiro@gmail.com](mailto:contact.me.kiro@gmail.com) 

<style>
.logo-slider {
  overflow: hidden;
  width: 100%;
  padding: 20px 0;
  position: relative;
}

.logo-slider::before,
.logo-slider::after {
  content: "";
  position: absolute;
  top: 0;
  width: 50px;
  height: 100%;
  z-index: 2;
}
.logo-slider::before {
  left: 0;
  background: linear-gradient(to right, var(--main-bg), transparent);
}
.logo-slider::after {
  right: 0;
  background: linear-gradient(to left, var(--main-bg), transparent);
}

.logo-track {
  display: flex;
  width: max-content;
  animation: scroll 8s linear infinite; 
}

.logo-track:hover {
  animation-play-state: paused;
}

.logo-slide {
  padding: 0 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo-slide img {
  height: 75px; 
  width: auto;
  max-width: 200px;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.logo-slide img:hover {
  transform: scale(1.1);
}

@keyframes scroll {
  0% { transform: translateX(0); }
  100% { transform: translateX(-50%); } 
}

/* The glowing animation class */
.glow-gold-white {
    text-decoration: none; 
    font-weight: bold;
    animation: pulse-glow2 4s infinite alternate ease-in-out;
}

/* The keyframes that dictate the color change */
@keyframes pulse-glow2 {
    0% {
      color: #FFD700; /* Gold */
      text-shadow: 0 0 8px rgba(255, 215, 0, 0.8), 0 0 15px rgba(255, 215, 0, 0.5);
    }
    100% {
      color: #ffffff; /* White */
      text-shadow: 0 0 8px rgba(255, 255, 255, 0.8), 0 0 15px rgba(255, 255, 255, 0.5);
    }
}
</style>

## Hacktivity :

- <a href="https://www.mars.com/" class="glow-gold-white">Mars</a>
- <a href="https://luminor.ee" class="glow-gold-white">Luminor Bank</a>

<!-- Company bar -->

<div class="logo-slider">
  <div class="logo-track">
    <!-- FIRST SET OF LOGOS -->
    <div class="logo-slide">
      <a href="https://www.mars.com/" target="_blank" rel="noopener noreferrer">
        <img src="/assets/img/mars.webp" alt="Mars">
      </a>
    </div>
    <div class="logo-slide">
      <a href="https://luminor.ee" target="_blank" rel="noopener noreferrer">
        <img src="/assets/img/luminor.webp" alt="Luminor Bank">
      </a>
    </div>
    <div class="logo-slide">
      <a href="https://www.mars.com/" target="_blank" rel="noopener noreferrer">
        <img src="/assets/img/mars.webp" alt="Mars">
      </a>
    </div>
    <div class="logo-slide">
      <a href="https://luminor.ee" target="_blank" rel="noopener noreferrer">
        <img src="/assets/img/luminor.webp" alt="Luminor Bank">
      </a>
    </div>
  </div>
</div>

<!-- Company bar -->

> Don't Hack Systems Without Explicit Authorization. Any Security Content in This Blog is Made For Education Only, So Use it Wisely !
{: .prompt-warning }