---
title: 'PDG Custom UI' 
tags: PDG Houdini Python
---

In this article, I’ll share some development on some personal Houdini package to easily create custom UI on pdg nodes.
<!--more--> 

# Motivations

Houdini's PDG (Procedural Dependency Graph) offers powerful tools for managing complex tasks, enabling automation, scalability, and parallelism across workflows. It shines in areas like generating simulations, rendering, and processing data efficiently, making it invaluable for scaling up productions or iterative workflows.

However, the default workflow for creating custom Houdini UIs can be time-consuming and often lacks scalability for rapid prototyping Iterating quickly becomes a bottleneck when user-friendly, flexible interfaces are needed for frequent adjustments.

While this might not be a concern for everyone, I spend a significant portion of my time building UIs in Houdini for both professional and personal projects. Having a streamlined solution to reduce the time spent on UI development allows me to focus more on prototyping and refining the core aspects of my ideas, enhancing productivity and creativity. This the reason why I created my Houdini package "Hou Databox". Since the orignal idea was to bridge external python library with Houdini PDG to scale them up and create variation based on Houdini TOPs Wedge node, I've focus my dev on TOPs Python node.  

# Implementation

Houdini provides several utility files like 123.py and externaldragdrop.py, enabling users to customize the software’s behavior for specific events. Similarly, it utilizes files like PythonScripts.txt to manage available Python node snippets. 

These files enable users to streamline their workflows by centralizing custom scripts, making them easily accessible via the Snippets button, located at the top-right corner of the Houdini parameter editor (indicated by the arrow icon). Similarly, we leverage this functionality to store our "Hou Databox" Python snippets.

With custom code stored in our package, we can override how Houdini interprets our specific "Hou Databox" snippets. When a custom "Hou Databox" node is created from the TAB menu, the following steps occur behind the scenes:

- A standard Houdini TOP Python node is created.
- The node's default UI is overridden using the logic from our package to create 2 new python editors. One contains the code responsible to produce the UI, the other the logic.
- The PythonScript.txt Hou Databox snippets containing both UI and logic is splitted, so we can populate each editor with the associated code.

As seen below, in its simplier form, "Hou Databox" allows UI parms creation using Houdini Vex syntax. The idea being that people familiar with vex could easily build UIs as well with Python nodes.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img5.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

Aside from the base UI parms you can create with the simplier VEX syntax, the package supports any Houdini Parm and there is a node that show how to build any of those with the more advanced python approach.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img6.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img11.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img12.png?raw=true" alt="pdg custom UIs">
  </div>
</div>


To organize and manage Python snippets efficiently within Houdini, we utilize the Pythonscripts.txt file. This file serves as a centralized location for storing reusable scripts, making it easy to access and reference them during development. By maintaining our snippets in this format, we streamline workflows, enhance project organization, and ensure consistency across tasks. This approach is particularly useful for maintaining clarity and scalability when dealing with complex setups or frequent script iterations.


<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img1.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img2.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img3.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img4.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img5.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img6.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img7.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img8.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img1.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img9.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img10.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

## Package Inheritance

## Using It To Make A Houdini Finance Package

## Integration In Houdini

## Sand Solver

## Exporting To Unreal

# Final Thoughts

In this article, I have shown my workflow for generating custom pdg ui. I've also shared some information about how you can worlds using tectonic plate simulations. This project is still ongoing, and I plan to use the data maps (precipitation, temperature, biome) from WorldEngine to instance Megascans assets for vegetation and cliff coverage. 

Additionally, I’m considering incorporating a [Graph-Based Wave Function Collapse (GBWFC)](https://logan169.github.io/2025/01/17/Graph-Based-WFC.html) to automatically place Megascans assets in a way that ensures a more realistic and cohesive look, but this will be for another article.





