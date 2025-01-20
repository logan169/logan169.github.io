---
title: 'Simulating World Generation Through Tectonic Plate Dynamics' 
tags: Procedural Houdini Python
---

In this article, I’ll share some results from my dev log for generating worlds using tectonic plate simulations, designed to create more realistic and immersive environments.<!--more--> 

![world generation picture 21](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world6.png?raw=true)


# Motivations

Back in 2020, the journey started as a simple idea of having fun with Houdini height fields to make environments. Over time, this concept evolved into a system inspired by real-world geology, designed to create dynamic and realistic terrains for procedurally generated landscapes. By simulating plate tectonics, fault lines, and geological deformations, this approach generates terrain formations that authentically reflect Earth's natural complexity, offering a powerful tool for realistic world-building in various applications.

# Implementation

## External Libraries

The base tectonic plate algorithm used in this project builds upon PlaTec, a 2D terrain generator leveraging a simplified plate tectonics model to create realistic heightmaps in real-time. PlaTec was developed as part of the Bachelor of Engineering [thesis](http://urn.fi/URN:NBN:fi:amk-201204023993) by Lauri Viitanen, released in 2012.

The simulation starts with a flat fractal map and splits it randomly into plates. Plates are moved around the map. If continents of two plates collide, they fold creating mountain belts. If oceanic crust of one plate collides with another plate, subduction occurs resulting in coastal mountain ranges or island chains.

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
</div>

## Integration In Houdini

Since Worldengine doesn’t accept an initial state as input, I modified its source code to enable this functionality. This adjustment allowed me to feed an updated version of the elevation map into the system and recompute the additional maps generated by Worldengine. 

The result is enhanced artistic control over the output, as I could iteratively refine the elevation map using Houdini height fields nodes to better suit the creative vision for the terrain and recompute the climate, biome, and other contextual data.

In the next phase, I integrated Worldengine into a Python COP node within Houdini, which allowed me to leverage Houdini’s powerful COP nodes to produce and edit terrain maps. This integration provided more control over the plate tectonic simulation process, as I could visualize the simulation's progress in real-time. 

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/platec.PNG?raw=true" alt="procedural world map">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/realtime_tectonic_plate.gif?raw=true" alt="procedural world map">
  </div>
  <div class="cell cell--2"></div>
</div>


With this setup, I could decide the optimal moment to halt the simulation and convert the resulting elevation maps into a Houdini height field, providing the flexibility to adjust the terrain as needed. I've created an HDA to center lands so I can eventually tile those in the future to create bigger world. 


<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/sattelite.PNG?raw=true" alt="procedural world map">
  </div>
  <div class="cell cell--2"></div>
</div>

## Sand Solver

One issue I encountered with the generated elevation maps was the spiky appearance of the highest points. Here are some screenshots showing up the looks of mountains out of the simulation

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma.png?raw=true" alt="procedural world map">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>


<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env_karma3.png?raw=true" alt="procedural world map">
  </div>
  <div class="cell cell--2"></div>
</div>

Houdini's height field nodes support erosion based on slope, but since mountains were thin, the results were less than ideal as lot of details were lost in the process. Additionally, the process required manually inputting varying elevation values each time. I needed a procedural approach to automate this process, which would eliminate the need for micromanaging erosion adjustments with each new generation. This would ensure consistent and reliable results without the need for manual fine-tuning after every map generation.

Part of the solution came from a [blog](https://pepefx.blogspot.com/2019/03/sand-without-grains-part-2-vex.html) that described how to implement a VEX sand solver in Houdini. After re-implementing this technique, I was able to achieve better erosion effects by simulating sand-like behavior. This approach helped automate the erosion process, removing the need for manual adjustments based on elevation values, while maintaining consistency and more natural results across different generations of the terrain. Here's how it looked with the regular implementation and applied to the outputed height field.

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_previz.gif?raw=true" alt="sand solver">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_algo-1.gif?raw=true" alt="sand solver">
  </div>
  <div class="cell cell--2"></div>
</div>

While the sand solver approach was an improvement, and really helped making dune like environments, I wasn't entirely satisfied with the results. So to enhance the effect, I modified the solver's source code to simulate rocks rolling down cliffs, rather than sand slowly sliding. This adjustment brought a more dynamic and rugged feel to the erosion, better representing the erosion patterns I'd envisioned for the terrain. Here's a look at the updated results after the modification.

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/env_topple_algo2.gif?raw=true" alt="sand solver">
  </div>
  <div class="cell cell--2"></div>
</div>


At that point, applying Houdini's erosion nodes to the modified version of the terrain generated more reasonable results. The combination of my changes to the solver with the built-in erosion functionality produced more natural and dynamic erosion effects, aligning closer to the desired outcome for the landscape as seen below.


<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/side_env.png?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/top_karma_render_env.PNG?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>

## Exporting To Unreal

Using Unreal Engine, I could then import the generated heightmaps, apply vegetation, and freely explore the terrain with the default character. This allowed me to experience the world in a more interactive way, seeing how the procedurally generated landscape translated into a playable environment.

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/unreal1.PNG?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world1.png?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world5.png?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>

<br>

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/procedural_world/world6.png?raw=true" alt="procedural world">
  </div>
  <div class="cell cell--2"></div>
</div>








