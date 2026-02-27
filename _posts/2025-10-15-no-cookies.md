---
layout: post
title:  "no cookies"
date:   2025-10-15 09:29:20 +0700
categories: jekyll update
usemathjax: true
---

<style>
  #nocookie-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.4);
    backdrop-filter: blur(3px);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9998;
  }

  #nocookie-popup {
    background: #222;
    color: white;
    padding: 20px 25px;
    border: 2px solid #555;
    border-radius: 6px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.4);
    font-family: sans-serif;
    z-index: 9999;
    max-width: 300px;
    text-align: center;
  }

  #nocookie-popup button {
    background: #444;
    color: white;
    border: none;
    padding: 8px 14px;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 12px;
    font-size: 0.9em;
  }

  #nocookie-popup button:hover {
    background: #666;
  }

  .blurred {
    filter: blur(6px);
    transition: filter 0.3s ease;
  }
</style>

<div id="site-content">
  <p>Hi, this site doesn't use cookies. Don't worry about it.</p>
  <p>Inspired by <a href="https://www.reddit.com/r/ProgrammerHumor/comments/l5gg3t/this_website_doesnt_use_cookies/">this comic </a></p>
</div>

<div id="nocookie-overlay">
  <div id="nocookie-popup">
    <p>This site doesn’t use cookies.</p>
    <button id="dismiss">Don’t show again</button>
  </div>
</div>

<script>
  const siteContent = document.getElementById('site-content');
  siteContent.classList.add('blurred');

  document.getElementById('dismiss').addEventListener('click', () => {
    document.getElementById('nocookie-overlay').remove();
    siteContent.classList.remove('blurred');
  });
</script>
