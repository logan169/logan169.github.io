---
title: 'PDG Custom UI And Financial Package' 
tags: PDG Houdini Python Financial
---

In this article, I’ll share insights into the development of a personal Houdini package designed to simplify the creation of custom UIs on PDG nodes. This package streamlines the process, enabling rapid prototyping, efficient UI design, and seamless integration of external Python libraries to enhance workflows within Houdini's Procedural Dependency Graph (PDG). I’ll also demonstrate how this workflow can serve as a foundation for developing more complex packages, such as a financial tool that queries and processes data from external sources, showcasing the versatility and scalability of this approach.
<!--more--> 

# Motivations

Houdini's PDG offers powerful tools for managing complex tasks, enabling automation, scalability, and parallelism across workflows. It excels in areas like generating simulations, rendering, and processing data efficiently, making it invaluable for scaling up productions or iterative workflows. Due to these strengths, PDG is an ideal place to bridge Houdini with external Python libraries, providing an easy way to integrate the functionality of external Python packages into Houdini’s environment.

Creating custom and complex Houdini UIs can be a time-consuming process, and the default workflow often lacks the scalability needed for rapid prototyping. This becomes a bottleneck when user-friendly and flexible interfaces are required for frequent adjustments, slowing down iteration and development. While UI development in Houdini may not be a priority for everyone, I dedicate significant time designing interfaces for both professional and personal projects. A more streamlined approach to UI development allows me to focus on quickly prototyping and refining the core aspects of my ideas, ultimately boosting both productivity and creativity. 

This drive for efficiency and innovation led to the creation of my Houdini package, "Hou Databox." At its core, "Hou Databox" was designed to bridge the gap between external Python libraries and Houdini's PDG, while also enabling the seamless transformation of Python snippets into TOP nodes. The package centers around the TOPs Python node as a fundamental component, addressing a key need: converting Python variables into PDG attributes. This capability ensures smooth data flow throughout the TOP graph, empowering users to efficiently reuse attributes in downstream nodes, further streamlining their workflows.

A final requirement was the ability to build child packages that could construct their UIs on top of the parent's package base UI. This functionality would enable UI inheritance, eliminating the need to duplicate UI code across multiple packages using "Hou Databox."

# Implementation

Houdini provides several utility files like 123.py and externaldragdrop.py, enabling users to customize the software’s behavior for specific events. Similarly, it utilizes files like PythonScripts.txt to manage available Python node snippets. 

These files enable users to streamline their workflows by centralizing custom scripts, making them easily accessible via the Snippets button, located at the top-right corner of the Houdini parameter editor (indicated by the arrow icon). Similarly, we leverage this functionality to store our "Hou Databox" Python snippets.

With custom code stored in our package, we can override how Houdini interprets specific "Hou Databox" snippets. When Houdini starts, it converts these snippets into Houdini tools, making them easily accessible from the TAB menu. At that point, creating a "Hou Databox" node triggers the associated tool and executes the previously described chain of operations behind the scenes. 

When a custom "Hou Databox" node is created, the following steps occur behind the scenes:

- A standard Houdini TOP Python node is generated.
- The node's default UI is replaced using the logic from our package, creating two new Python editors: one for the code that generates the UI, and the other for the logic.
- The PythonScript.txt "Hou Databox" snippets, containing both UI and logic, are split so that each editor is populated with the corresponding code.
- The UI editor is executed, and the node's custom UI is built within the node interface.

## UI Creation

As shown below, in its simpler form, "Hou Databox" enables the creation of UI parameters using Houdini's VEX syntax. The goal is to allow users already familiar with VEX to seamlessly build UIs within Python nodes as well.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img5.png?raw=true" alt="pdg custom UIs">
  </div>
</div>


In addition to the basic UI parameters that can be created using the simpler VEX syntax, the "Hou Databox" package supports all existing Houdini parameters. It even includes an example node that demonstrates how to construct any of these parameters using a more advanced Python-based approach. Due to the large number of parameters, I’ve included a couple of screenshots to provide a clearer view, as it was not possible to capture all of them in a single image.

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

"Hou Databox" accelerates prototyping by enabling quick iteration and the seamless transformation of Python snippets into functional nodes. Additionally, it allows the reverse operation once the user is satisfied with the custom node snippets they've created. At this point, the user can save the node's state directly back into the PythonScripts.txt file of the designated package via the menu. This functionality ensures that the node can be easily recreated later, preserving custom scripts and allowing for a smooth, efficient workflow that supports future use and modifications.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img19.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

## Simplifying Environment Setup 

To facilitate quick prototyping, "Hou Databox" also includes the ability to populate a create environment node with the selected Houdini package's requirements.txt file. This feature makes the necessary dependencies readily accessible from Houdini's Python nodes, streamlining the integration of external libraries and enhancing the overall workflow.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img13.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

## UI Inheritance

A final feature was implemented to allow child packages to construct their UIs on top of the parent’s base UI. This functionality enables UI inheritance, eliminating the need to duplicate UI code across multiple packages using "Hou Databox". When a custom node from a child package relying on "Hou Databox" is created from the TAB menu, the following steps occur behind the scenes:

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

Notice the "Package parms" folder in the top right UI, compared to the previous screenshot provided at the package level by the "Hou Pandas" package. This update allows me to specify if a node accepts dataframe inputs and which variable name should be assigned to them. Additionally, I can apply the same functionality at the export level to turn any variable containing a dataframe into a PDG attribute. For optimization purposes, this package also supports the functionality of loading or saving a dataframe from/to an Excel file.


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

<br>

# Making The 1st Financial Houdini Package

Using "Hou Pandas" as a foundation, I developed a child package called "Hou Fin," which is specialized in financial analysis within PDG. All "Hou Fin" nodes inherit the default "Hou Pandas" UI, which includes functionality for importing and exporting dataframes, as mentioned previously. From there, the nodes build upon the UI with specific code tailored to their financial analysis features, enabling the rapid design of workflows that leverage external financial Python packages directly within Houdini. 

Now, I’ll use "Hou Fin" as a test case to demonstrate the advantages of using "Hou Databox" to design and streamline this type of project, showcasing how it facilitates the development of custom tools and workflows in financial analysis.

## Unlocking External Python Packages For Non-Coders

Here are several financial setups that rely on "Hou Pandas" and "Hou Fin" to query cryptocurrency and stock data, compute returns, and run Monte Carlo simulations on previous returns to approximate potential future ones. 

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img18.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

These setups were completed quickly because the UI creation process is now much faster, and you can easily copy and paste demo code from external Python documentation as a base for your node setup. The ability to iterate more quickly while working on such workflows offers several advantages. One key benefit is that non-coder end users can test the tools more frequently and provide valuable feedback, ultimately leading to more refined and effective tools in the end.

## Scalability

In financial analysis, the goal is often to identify a signal you can exploit amidst the noise. Scaling your analysis is crucial to increasing the chances of uncovering something relevant. 

This is where the workflow, when combined with Wedger, becomes incredibly powerful. By leveraging Wedger’s ability to manage and process a large number of stock tickers, the workflow enables you to efficiently scale your analysis and better pinpoint actionable insights.

Below is a series of screenshots showing the step-by-step setup, node by node, where I am backtesting a trading strategy for each wedge stock ticker and outputting a comprehensive report at the end.


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
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img1.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

Finally when running the above we do get the associated ticker price data baked into Excel files, and for each backtrading reports as well as you can see below.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img8.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

## Building More Advanced UIs

As well, by adopting the workflow provided by "Hou Databox," I was able to significantly scale up my node's UI capabilities, enabling the creation of nodes that would have otherwise been impractical within a reasonable timeframe. A prime example is a node I developed to query data from the financial website finviz.com. To put it into perspective, replicating the filter interface from the site required building a UI that accommodates approximately 70 menus or combo boxes, each containing between 10 and 20 options.

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


