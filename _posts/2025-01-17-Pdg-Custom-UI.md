---
title: 'PDG Custom UI And Financial Package' 
tags: PDG Houdini Python Financial
---

In this article, I’ll share insights into the development of a personal Houdini package designed to simplify the creation of custom UIs on PDG nodes. This package streamlines the process, enabling rapid prototyping, efficient UI design, and seamless integration of external Python libraries to enhance workflows within Houdini's Procedural Dependency Graph (PDG).

I’ll also demonstrate how this workflow can serve as a foundation for developing more complex packages, such as a financial tool that queries and processes data from external sources, showcasing the versatility and scalability of this approach.
<!--more--> 

# Motivations

Houdini's PDG (Procedural Dependency Graph) offers powerful tools for managing complex tasks, enabling automation, scalability, and parallelism across workflows. It shines in areas like generating simulations, rendering, and processing data efficiently, making it invaluable for scaling up productions or iterative workflows.

However, the default workflow for creating custom/complex Houdini UIs can be time-consuming and often lacks scalability for rapid prototyping Iterating quickly becomes a bottleneck when user-friendly, flexible interfaces are needed for frequent adjustments.

Although UI development in Houdini may not be a priority for everyone, I invest a considerable amount of time in creating interfaces for both professional and personal projects. A streamlined approach to UI development allows me to allocate more energy toward prototyping and refining the core aspects of my ideas, ultimately enhancing both productivity and creativity. This focus on efficiency and innovation was the driving force behind the creation of my Houdini package, "Hou Databox."

At its core, "Hou Databox" was designed to bridge the gap between external Python libraries and Houdini's Procedural Dependency Graph (PDG), while also enabling the seamless transformation of Python snippets into TOP nodes. The package centers around the TOPs Python node as a fundamental component, addressing a key need: converting Python variables into PDG attributes. This capability ensures smooth data flow throughout the TOP graph, empowering users to efficiently reuse attributes in downstream nodes, further streamlining their workflows.

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

## Turning Snippets Into Nodes

"Hou Databox" accelerates prototyping by enabling quick iteration and seamless transformation of Python snippets into functional nodes. A key feature is the ability to save the node's state directly back into the PythonScripts.txt file of the designated package. This allows the node to be recreated later, ensuring a smooth and efficient workflow while preserving custom scripts for future use.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img19.png?raw=true" alt="pdg custom UIs">
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

By adopting the workflow provided by "Hou Databox," I was able to significantly scale up my node's UI capabilities, enabling the creation of nodes that would have otherwise been impractical within a reasonable timeframe. A prime example is a node I developed to query data from the financial website finviz.com. To put it into perspective, replicating the filter interface from the site required building a UI that accommodates approximately 70 menus or combo boxes, each containing between 10 and 20 options.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img16.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

Accomplishing this level of complexity manually with a standard HDA would have been highly time-consuming. However, with "Hou Databox," the streamlined process made it feasible to develop such intricate interfaces efficiently and effectively.

<div class="grid">
  <div class="cell cell--auto">
  <video width="100%" controls>
    <source src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/video1.mp4?raw=true" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  </div>
</div>

<br>

# Final Thoughts

In this article, I’ve shared how I leverage my "Hou Databox" package to efficiently design setups that integrate external Python libraries. I demonstrated how the package enables users to save nodes as Python snippets, which can later be recreated directly from the TAB menu, streamlining the workflow for both prototyping and reuse.

Additionally, I showcased how UI inheritance allows for the creation of topic-specific packages that benefit from the default UI and behavior of their parent packages. This approach fosters consistency while reducing the effort required to build specialized tools.

Finally, I highlighted how the package simplifies rapid iteration during UI design. By allowing on-the-fly tweaks to UI code and regenerating the interface without delving into the node’s parameter menu for each modification, "Hou Databox" significantly accelerates the process, making it a powerful tool for enhancing productivity and creativity.





<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img18.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img17.png?raw=true" alt="pdg custom UIs">
  </div>
</div>


<br>
----------------------



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





