---
title: 'AI Driven Projection Mapping POC' 
tags: AI MLOPS Houdini Stable-Diffusion
---

Growing up in Lyon, France, I’ve been fortunate to witness the celebrated “Fête des Lumières” firsthand. This captivating annual occasion bathes Lyon in an exquisite showcase of light, metamorphosing its thoroughfares, iconic sites, and edifices into awe-inspiring masterpieces. Annually, on December 8th, millions of visitors from around the globe flock to witness this extraordinary spectacle, immersing themselves in the enchanting realm of light, color, and innovation that defines the Fête des Lumières.

{%- include extensions/youtube.html id='sGBTVngHvug' -%}

Today, I decided to embark on a personal challenge: using Houdini, a 3D software, to create and simulate a light art installation reminiscent of those seen at the “Fête des Lumières”. This idea has been brewing in my mind for quite some time; I’ve envisioned a unique approach. I aimed to place the visitor at the center of the experience by incorporating an onsite prompt system. This system would dynamically generate the projected image over the building based on prompts provided by visitors, utilizing generative AI technology. With that match plan in mind, let’s see how we can create a proof of concept that does this.

First things first, let’s found a 3D model worth of our inspiration. On my side, I’ve settled on using a 3D model of the door from the French medieval church of Sainte-Radegonde (Poitiers). The model is freely available for download [here](https://sketchfab.com/3d-models/eglise-sainte-radegonde-poitiers-86-66dab155a9fc44df8e9f10e830d536dd).

<div class="hero" style="background-color: #ccc;">
  <div class="hero__content">
    <img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/0*VBML6nzsFjJpf9Uo"></h3>
  </div>
</div>



----------------------------------------------------------------
This article shares some research and development done on procedural world generation based on plate tectonic simulations.


<div class="item">
  <div class="item__image">
    <img class="image image--lg" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/benoit_setup.gif?raw=true"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h4>test</h4>
    </div>
    <div class="item__description">
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    </div>
  </div>
</div>

![world generation picture 1](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/benoit_setup.gif?raw=true)

![world generation picture 2](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/benoit_setup3.gif.gif?raw=true)

![world generation picture 3](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/melting_mountains.gif?raw=true)

![world generation picture 4](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/top_env.PNG?raw=true)

![world generation picture 5](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/benoit_setup.gif?raw=true)