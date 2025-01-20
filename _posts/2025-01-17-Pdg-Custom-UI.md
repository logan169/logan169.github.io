---
title: 'PDG Custom UI' 
tags: PDG Houdini Python
---

In this article, I’ll share some development on some personal Houdini package to easily create custom UI on pdg nodes.
<!--more--> 

# Motivations

Houdini's PDG (Procedural Dependency Graph) offers powerful tools for managing complex tasks, enabling automation, scalability, and parallelism across workflows. It shines in areas like generating simulations, rendering, and processing data efficiently, making it invaluable for scaling up productions or iterative workflows.

However, the default workflow for creating custom Houdini UIs can be time-consuming and often lacks scalability for rapid prototyping Iterating quickly becomes a bottleneck when user-friendly, flexible interfaces are needed for frequent adjustments.

While this might not be a concern for everyone, I spend a significant portion of my time building UIs in Houdini for both professional and personal projects. Having a streamlined solution to reduce the time spent on UI development allows me to focus more on prototyping and refining the core aspects of my ideas, enhancing productivity and creativity. This the reason why I created my Houdini package "Hou Databox".

# Implementation

Houdini provides several utility files like 123.py and externaldragdrop.py, enabling users to customize the software’s behavior for specific events. Similarly, it utilizes files like PythonScripts.txt to manage available Python node snippets. 

These files allow users to streamline their workflows by centralizing custom scripts and making them accessible when needed from the snippets button which the arrow button located at the top right of editor Houdini parm. In our case, we are taking advantage of this as well to store our "Hou Databox" python snippets as well. 

Using some custom code stored in our package we can override the way Houdini interpret specificly our "Hou Databox" snippets. When creating a custom "Hou Databox" node from the TAB menu, here is the steps that happens under the hood:
- We create a regular Houdini TOP Python node
- We override its default UI with our package Logic 
- We set the code to the associated PythonScript.txt file snippet.

"Hou Databox" will modify the UI in a way that create 2 new python editors. One to contains the code responsible to produce the UI, the other will contain the logic. In its simplier form, "Hou Databox" allows UI parms creation using  Houdini Vex syntax. The idea being that people familiar with vex could easily build UIs as well with Python nodes.


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





