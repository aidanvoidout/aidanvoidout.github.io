---
layout: post
title:  "noun verbed"
date:   2025-08-29 09:29:20 +0700
categories: jekyll update
usemathjax: true
---

<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@700&display=swap" rel="stylesheet">
<style>
.boss-bar-container {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  pointer-events: none;
  z-index: 99999;
  font-family: 'Cinzel', serif;
  overflow: hidden;
}

.boss-bar-container .boss-bar {
  position: absolute;
  width: 100%;
  height: 8.5rem;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: linear-gradient(
    to bottom,
    rgba(0,0,0,0),
    rgba(0,0,0,0.4) 19%,
    rgba(0,0,0,0.8) 50%,
    rgba(0,0,0,0.7) 80%,
    rgba(0,0,0,0)
  );
  z-index: 0;
  opacity: 0;
  filter: blur(4px);
}

.boss-bar-container .boss-defeat {
  position: relative;
  font-size: 5rem;
  letter-spacing: 0.25rem;
  text-align: center;
  text-transform: uppercase;
  z-index: 1;
  background: linear-gradient(to bottom, #f7e7b2, #d9b650, #8c6a2f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow:
    0 0 10px rgba(255,215,100,0.6),
    0 0 30px rgba(255,180,50,0.4),
    0 0 60px rgba(150,100,20,0.2);
  opacity: 0;
}

.boss-bar-container .boss-defeat::before {
  content: attr(data-text);
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  font-size: inherit;
  letter-spacing: 0.25rem;
  opacity: 0.25;
  transform: scaleX(1);
  filter: blur(2px);
  background: linear-gradient(to bottom, #c9a74d, #7d5c28);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.boss-bar-container.active .boss-bar {
  animation: boss-bar-fade 6s ease-in-out forwards;
}

.boss-bar-container.active .boss-defeat {
  animation: boss-front-fade 6s ease-in-out forwards;
}

.boss-bar-container.active .boss-defeat::before {
  animation: boss-back-stretch-fade 6s ease-in-out forwards;
}

@keyframes boss-front-fade {
  0%   { opacity: 0; transform: scale(1.15); }
  15%  { opacity: 1; transform: scale(1); }
  75%  { opacity: 1; transform: scale(1); }
  90%  { opacity: 0; transform: scale(0.98); }
  100% { opacity: 0; transform: scale(0.98); }
}

@keyframes boss-back-stretch-fade {
  0%   { opacity: 0.25; transform: scaleX(1); }
  10%  { opacity: 0.25; transform: scaleX(1); }
  90%  { opacity: 0; transform: scaleX(1.3); }
  100% { opacity: 0; transform: scaleX(1.3); }
}

@keyframes boss-bar-fade {
  0%   { opacity: 0; }
  15%  { opacity: 1; }
  75%  { opacity: 1; }
  90%  { opacity: 0; }
  100% { opacity: 0; }
}

@media (max-width: 480px) {
  .boss-bar-container .boss-defeat,
  .boss-bar-container .boss-defeat::before {
    font-size: 3rem;
  }
}
</style>

<div class="boss-bar-container">
  <div class="boss-bar"></div>
  <div class="boss-defeat" data-text="NOUN VERBED">NOUN VERBED</div>
</div>

I tried to learn css animation and failed horribly, so have this partially ChatGPTed elden ring boss text generator.

<div>
  <input type="text" id="boss-text" placeholder="NOUN VERBED">
  <button id="boss-trigger">noun your verb</button>
</div>

<script>
const container = document.querySelector('.boss-bar-container');
const defeatText = document.querySelector('.boss-defeat');
const input = document.getElementById('boss-text');
const button = document.getElementById('boss-trigger');

button.addEventListener('click', () => {
  const text = input.value.trim() || 'NOUN VERBED';
  defeatText.textContent = text;
  defeatText.setAttribute('data-text', text);

  container.classList.remove('active');
  void container.offsetWidth;
  container.classList.add('active');
});
</script>
