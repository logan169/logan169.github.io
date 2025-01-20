---
title: 'PDG Custom UI' 
tags: PDG Houdini Python
---

In this article, I’ll share some development on some personal Houdini package to easily create custom UI on pdg nodes.
<!--more--> 

# Motivations

Houdini's PDG (Procedural Dependency Graph) offers powerful tools for managing complex tasks, enabling automation, scalability, and parallelism across workflows. It shines in areas like generating simulations, rendering, and processing data efficiently, making it invaluable for scaling up productions or iterative workflows.

However, the default workflow for creating custom Houdini UIs can be time-consuming and often lacks scalability for rapid prototyping Iterating quickly becomes a bottleneck when user-friendly, flexible interfaces are needed for frequent adjustments.

Although this may not be a priority for everyone, I dedicate a significant amount of my time to building UIs in Houdini for both professional and personal projects. A streamlined solution that reduces the time spent on UI development enables me to focus more on prototyping and refining the core elements of my ideas, ultimately boosting productivity and creativity. This is the driving force behind the creation of my Houdini package, "Hou Databox."


The motivation behind creating "Hou Databox" stemmed from the need to bridge external Python libraries with Houdini PDG, enabling the scaling of processes and the creation of variations based on the Houdini TOPs Wedge node. This led to a focus on the TOPs Python node as a core component. Additionally, there was a need to turn Python variables into PDG attributes, allowing them to flow seamlessly through the TOP graph and be reused in downstream nodes.

A final requirement was the ability to build child packages that could construct their UIs on top of the parent's package base UI. This functionality would enable UI inheritance, eliminating the need to duplicate UI code across multiple packages using "Hou Databox."

# Implementation

Houdini provides several utility files like 123.py and externaldragdrop.py, enabling users to customize the software’s behavior for specific events. Similarly, it utilizes files like PythonScripts.txt to manage available Python node snippets. 

These files enable users to streamline their workflows by centralizing custom scripts, making them easily accessible via the Snippets button, located at the top-right corner of the Houdini parameter editor (indicated by the arrow icon). Similarly, we leverage this functionality to store our "Hou Databox" Python snippets.

With custom code stored in our package, we can override how Houdini interprets specific "Hou Databox" snippets. When Houdini starts, it converts these snippets into Houdini tools, making them easily accessible from the TAB menu. At that point, creating a "Hou Databox" node triggers the associated tool and executes the previously described chain of operations behind the scenes. 

When a custom "Hou Databox" node is created, the following steps occur behind the scenes:

- A standard Houdini TOP Python node is generated.
- The node's default UI is replaced using the logic from our package, creating two new Python editors: one for the code that generates the UI, and the other for the underlying logic.
- (If this is being done from a child Hou Databox package), it checks whether there is a parent package's default UI associated with the current package and, if so, builds the default UI.
- (If this is being done from a child Hou Databox package), the current package's UI modifications are applied on top of the existing UI.
- The PythonScript.txt "Hou Databox" snippets, containing both UI and logic, are split so that each editor is populated with the corresponding code.
- The UI editor is executed, and the node's custom UI is built within the node interface.
  

## UI Creation

As shown below, in its simpler form, "Hou Databox" enables the creation of UI parameters using Houdini's VEX syntax. The goal is to allow users already familiar with VEX to seamlessly build UIs within Python nodes as well.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img5.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

In addition to the basic UI parameters you can create using the simpler VEX syntax, the package supports all existing Houdini parameters. It even includes an example node that demonstrates how to construct any of these parameters using a more advanced Python-based approach.

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


## Virtual Env Creation

To facilitate quick prototyping, "Hou Databox" also includes support for creating virtual environments with any package specified in a requirements.txt file, making them readily accessible from Houdini Python nodes.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img13.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

## UI Inheritance

A final feature was implemented to allow child packages to construct their UIs on top of the parent’s base UI. This functionality enables UI inheritance, eliminating the need to duplicate UI code across multiple packages using "Hou Databox."


When a custom node from a child package relying on "Hou Databox" is created from the TAB menu, the following steps occur behind the scenes:

- A standard Houdini TOP Python node is generated.
- The node's default UI is replaced using the logic from our package, creating two new Python editors: one for the code that generates the UI, and the other for the underlying logic.
- Checks whether there is a parent package's default UI associated with the current package and, if so, builds the default UI.
- The current package's UI modifications are applied on top of the existing UI.
- The PythonScript.txt "Hou Databox" snippets, containing both UI and logic, are split so that each editor is populated with the corresponding code.
- The UI editor is executed, and the node's custom UI is built within the node interface.

To demonstrate the UI inheritance functionality behind "Hou Databox," I created a child Houdini package called "Hou Pandas" to handle dataframes, a standard format for working with tabular data in Python. This package inherits all the functionality and default UIs provided by "Hou Databox."

In addition, "Hou Pandas" introduces support for importing and remapping PDG attributes from upstream nodes, allowing them to be interacted with from the node's logic editor. In its simplest form, a vanilla node looks like this:

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img14.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

Notice the update in the top right UI, compared to the previous screenshot provided at the package level by the "Hou Pandas" package. This update allows me to specify if a node accepts dataframe inputs and which variable name should be assigned to them. Additionally, I can apply the same functionality at the export level to turn any variable containing a dataframe into a PDG attribute. For optimization purposes, this package also supports the functionality of loading or saving a dataframe from/to an Excel file.


<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img10.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

This package also enables the creation of custom nodes specialized in dataframe manipulation. This is extremely useful, as any child package from "Hou Pandas" can leverage these nodes to modify their dataframes, without the need to recreate similar nodes in each child packages.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img15.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

Finally, using "Hou Pandas" as a base, I created another child package called "Hou Fin," which is specialized in financial analysis within PDG. Any "Hou Fin" nodes inherit the default "Hou Pandas" UI, which allows for importing and exporting dataframes. From there, they build upon the node's UI with specific code tailored to their financial analysis functionality.


<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img16.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>
----------------------


To follow up on my inheritance example, Using this workflow I created another child package named "Hou Fin" inheriting from "Hou Pandas" to do financial analysis for me in PDG.



One of the 
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


<br>

## Package Inheritance

## Using It To Make A Houdini Finance Package

## Integration In Houdini

## Sand Solver

## Exporting To Unreal

# Final Thoughts

In this article, I have shown my workflow for generating custom pdg ui. I've also shared some information about how you can worlds using tectonic plate simulations. This project is still ongoing, and I plan to use the data maps (precipitation, temperature, biome) from WorldEngine to instance Megascans assets for vegetation and cliff coverage. 

Additionally, I’m considering incorporating a [Graph-Based Wave Function Collapse (GBWFC)](https://logan169.github.io/2025/01/17/Graph-Based-WFC.html) to automatically place Megascans assets in a way that ensures a more realistic and cohesive look, but this will be for another article.





