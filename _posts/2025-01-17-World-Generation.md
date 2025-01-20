---
title: 'Simulating World Generation Through Tectonic Plate Dynamics' 
tags: Procedural Houdini Python
---

In this article, I’ll share some results from my workflow for generating worlds using tectonic plate simulations, designed to create more realistic and immersive environments.<!--more--> 

![world generation picture 21](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world6.png?raw=true)


# Motivations

Back in 2020, the journey started as a simple idea of having fun with Houdini height fields to make environments. Over time, this concept evolved into a system inspired by real-world geology, designed to create dynamic and realistic terrains for procedurally generated landscapes. By simulating plate tectonics, fault lines, and geological deformations, this approach generates terrain formations that authentically reflect Earth's natural complexity, offering a powerful tool for realistic world-building in various applications.

# Implementation

The base tectonic plate algorithm used in this project builds upon PlaTec, a 2D terrain generator leveraging a simplified plate tectonics model to create realistic heightmaps in real-time. PlaTec was developed as part of the Bachelor of Engineering [thesis](http://urn.fi/URN:NBN:fi:amk-201204023993) by Lauri Viitanen, released in 2012, and is available for reference here. Below is a visual representation of Platec

{%- include extensions/youtube.html id='bi4b45tMEPE' -%}

PlaTec has been integrated into numerous software solutions since its release in 2012. For this project, I utilized  [worldengine](https://github.com/Mindwerks/worldengine), which builds on PlaTec's capabilities by running additional simulations to generate various maps alongside the elevation map produced by PlaTec. Worldengine outputs a comprehensive set of maps, including climate, precipitation, temperature, and biome distributions, providing rich contextual data for world-building that could then be reused within Houdini. 

Here’s a full list of maps you can expect from Worldengine.

<div class="grid-container">
  <div class="grid grid--p-3">
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_temperature.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Temperature</h4>
          </div>
        </div>
      </div>
    </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_satellite.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Satellite</h4>
          </div>
        </div>
      </div>
    </div>    
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_rivers.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>River</h4>
          </div>
        </div>
      </div>
    </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_precipitation.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Precipitation</h4>
          </div>
        </div>
      </div>
    </div>
  </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_ocean.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Ocean</h4>
          </div>
        </div>
      </div>
    </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_icecaps.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Ice Caps</h4>
          </div>
        </div>
      </div>
    </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_grayscale.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Elevation</h4>
          </div>
        </div>
      </div>
    </div>
    <div class="cell cell--6">
      <div class="item">
        <div class="item__image">
          <img class="image" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/seed_44579_biome.png?raw=true"/>
        </div>
        <div class="item__content">
          <div class="item__header">
            <h4>Biome</h4>
          </div>
        </div>
      </div>
    </div>
</div>

Since Worldengine doesn’t accept an initial state as input, I modified its source code to enable this functionality. This adjustment allowed me to feed an updated version of the elevation map into the system and recompute the additional maps generated by Worldengine. The result is enhanced artistic control over the output, as I could iteratively refine the elevation map using Houdini height fields nodes to better suit the creative vision for the terrain and recompute the climate, biome, and other contextual data.

In a second time, I've wrapped worldengine within a python COP node, so I can take advantage of all existing Houdini COPs nodes to produce/edits those maps. Doing this allowed me to gain more control over the the plate simulation process as I could see the live result and decide at which epoch I stop the simulation and turns the outputed elevation maps as a Houdini height field.


 steps like the temperature, precipitation

 
On my side I was passing throughExtra world data maps (precipitation, elevation, river,biomes) are created using

This workflow starts by generating some maps using some external library that I've integrated within Houdini through the use of HDA and Python COP. 

The base tectonic plate algorithm relies on PlaTec. PlaTec is a 2D terrain generator that uses a simplified model of plate tectonics to produce realistic heightmaps in real time. It was written as part of Bachelor of Engineering [thesis](http://urn.fi/URN:NBN:fi:amk-201204023993) of Viitanen, Lauri released in 2012. Here is a glimpse showing how it looks like


The simulator starts with a flat fractal map and splits it randomly into plates. Plates are moved around the map. If continents of two plates collide, they fold creating mountain belts. If oceanic crust of one plate collides with another plate, subduction occurs resulting in coastal mountain ranges or island chains.

This project forms part of my Bachelor of Engineering thesis at Metropolia University of Applied Sciences, Helsinki, Finland. The full thesis is available for free download from [insert source or link].
(http://urn.fi/URN:NBN:fi:amk-201204023993)

 which was 
{%- include extensions/youtube.html id='bi4b45tMEPE' -%}




# Challenges


# Final Thoughts


![world generation picture 1](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/benoit_setup3.gif?raw=true)

![world generation picture 2](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_algo-1.gif?raw=true)

![world generation picture 3](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_algo2.gif?raw=true)

![world generation picture 4](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_previz.gif?raw=true)

![world generation picture 5](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/input.png?raw=true)

![world generation picture 6](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/island.gif?raw=true)

![world generation picture 7](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/karm_render.PNG?raw=true)

![world generation picture 8](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/platec.PNG?raw=true)

![world generation picture 9](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/realtime_tectonic_plate.gif?raw=true)

![world generation picture 10](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/sattelite.PNG?raw=true)

![world generation picture 11](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma.png?raw=true)

![world generation picture 15](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma8.png?raw=true)


![world generation picture 12](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma3.png?raw=true)

![world generation picture 13](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma4.png?raw=true)

![world generation picture 15](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma6.png?raw=true)

![world generation picture 16](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env.png?raw=true)

![world generation picture 17](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/top_karma_render_env.PNG?raw=true)

![world generation picture 18](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/unreal1.PNG?raw=true)

![world generation picture 19](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world1.png?raw=true)

![world generation picture 20](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world5.png?raw=true)

![world generation picture 21](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world6.png?raw=true)

