---
title: 'AI Driven Projection Mapping POC' 
tags: AI MLOPS Houdini Stable-Diffusion
---

Growing up in Lyon, France, I’ve been fortunate to witness the celebrated “Fête des Lumières” firsthand. This annual celebration transforms the city into a mesmerizing canvas of light, showcasing its streets, landmarks, and buildings as breathtaking works of art. Each year, on December 8th, millions of visitors from around the world gather in Lyon to marvel at this extraordinary event, immersing themselves in a stunning fusion of light, color, and creativity that defines the essence of the Fête des Lumières.

{%- include extensions/youtube.html id='sGBTVngHvug' -%}

Today, I decided to take on a personal challenge: using Houdini, a 3D software, to create and simulate a light art installation inspired by the “Fête des Lumières.” This concept has been on my mind for some time, and I envisioned a unique approach. My goal was to place the visitor at the heart of the experience by integrating an onsite prompt system. This system would dynamically generate a projected image over the building based on prompts provided by the visitors, powered by generative AI. 

In this project, we will generate some image from prompts using [MLOPS Houdini package](https://www.sidefx.com/tutorials/introducing-mlops-machine-learning-operators/). The MLOPS Houdini library integrates machine learning capabilities within the Houdini environment, facilitating AI model deployment and integration into 3D workflows. It enables the seamless interaction between generative AI models and Houdini's procedural generation tools. 

Now let's explore how we can create a proof of concept for this whole idea. To begin, we need to find a fitting 3D model for our project. Personally, I decided to use a 3D model of the door from the French medieval church of Sainte-Radegonde in Poitiers. This model is freely available for download [here](https://sketchfab.com/3d-models/eglise-sainte-radegonde-poitiers-86-66dab155a9fc44df8e9f10e830d536dd), providing an excellent starting point for our light art installation. 

![projection mapping houdini setup 0](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*VBML6nzsFjJpf9Uo)

After importing the model into Houdini, the first step is to align it with the viewport grid using a series of transformations. The next step is to split the mesh into Houdini groups. These groups will be used to populate a custom attribute named "building_parts," which will be essential in the next steps. This attribute will help us create masks, which will be critical when we integrate generative AI-generated layers during the compositing phase of the project.


For this proof of concept, we've identified three zones of interest on the 3D model: the main door group (all areas within the arch), the sidewalls (including the two side columns and the main wall), and everything else. This segmentation enables us to focus on specific areas when applying transformations and generating AI-based projections. The mesh handling setup, which organizes these zones, serves as a foundation for the next steps in the process.At that point, our setup that handles the mesh looks like the one below.

![projection mapping houdini setup 1](https://miro.medium.com/v2/resize:fit:4202/format:webp/1*HmTh8uXhkvN265GA_D-TbQ.png)

Now, we return to Houdini root at the “/obj” level and create a camera and a point light, used as a gobo light to project our image over our mesh. We established the camera resolution at 512x512, which serves as the maximum resolution input image for the stable diffusion model we will be using. Once we are done setting up our setup, our “/obj” level looks like this.

![projection mapping houdini setup 3](https://miro.medium.com/v2/resize:fit:2800/format:webp/0*Tp4-KZp6GgBTRG-2)

As illustrated in the preceding screenshot, a critical procedure involved configuring the camera settings to attain consistent viewpoints between the camera and the gobo-equipped point light. This alignment is essential because we want the image projected from the gobo to match what has been captured by the camera.

Now, let’s explore the generative AI step further. Within our “/obj/sd” node containing our MLOPS setup, we’ll create three separate graphs, each tasked with generating an image for a specific part of the segmented mesh using the custom attribute “building_parts” we established earlier. This segmentation enables us to tailor the generative AI process to each distinct area of interest in the scene, ensuring precise and targeted image generation.

This means we use the same design to build three different prompt and model architectures for the doors or arch, the mainwall, and the sidewalls. Below is the design explained for the door area.

![projection mapping houdini setup 4](https://miro.medium.com/v2/resize:fit:2730/format:webp/1*zPZ9SnvJ2_t--jM8EuLvcw.png)

We connect all of those branches back using a merge node and connect this later one to a Python SOP node. This Python node will refresh the loading of the COP nodes, which will load the generated AI image residing within the “cop2net” node below. Once each branch has finished generating an image, this refresh triggers an update in the compositing process, consequently updating the projected image at the gobo level. This linkage is dynamic, as it’s directly associated with our “cop2net” node's final output node.

![projection mapping houdini setup 5](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xnIdqhz-r6J_tTSaGxJ-Zg.png)

Here is how our cop2net node looks like, showing the different stages of implementation of the stable diffusion images and the masks from the attributes into composite images which leads to the final output image.

![projection mapping houdini setup 6](https://miro.medium.com/v2/resize:fit:4160/format:webp/1*PqkIiF5wwTqI51lZmYrJUQ.png)


Next, we set the gobo light’s “Projection Map” parameter to dynamically retrieve the image directly from the cop2net1 node using the following expression “op:/obj/sd/cop2net1/OUT/”.

![projection mapping houdini setup 8](https://miro.medium.com/v2/resize:fit:198/format:webp/0*d2UsD3q3Z4WnbG1a)

We also enable the viewport’s high-quality lighting option (the bulb button on the left) and select “Show All Objects” from the top viewport button options. This ensures we can accurately preview the scene’s lighting and overall appearance in real-time feedback.

Here is an example of a new door style when applying the code and the different steps of scene view options:
![projection mapping houdini setup 9](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*zyQiB9l3c9mYTrC7mwP9vQ.png)


Of course, it is a woprkflow proof of concept and the quality could be enhanced with better lighting, different testing with the parameters, we could use a new version of the stable diffusion model. As always, there is a lot of space for improvement in machine learning projects and that is the beauty of it! At the time of doing it, this project have been using stable diffusion 1, while at the time of writting this blog stable diffusion 3 went out winter 2024.

Here are some viewpoer screenshot output example from the following prompts:


<div class="item">
  <div class="item__image">
    <img class="image image--lg" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/ai_projection_mapping_poc/roman_door_previz.gif?raw=true"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h5>"An intricately detailed Roman-style door, set in a classical stone archway flanked by columns adorned with Roman carvings. The surrounding architecture is inspired by ancient Roman temples, with ornate stonework and decorative reliefs."</h5>
    </div>
    <div class="item__description">
      <p>...</p>
    </div>
  </div>
</div>


<div class="item">
  <div class="item__image">
    <img class="image image--lg" src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/ai_projection_mapping_poc/roman_door_previz.gif?raw=true"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h5>“luxurious and vibrant tropical forest plants with vivid colored flowers, very detailed and lots of red, orange, greens”</h5>
    </div>
    <div class="item__description">
      <p>...</p>
    </div>
  </div>
</div>

![output ai projection mapping with romans doors ](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/ai_projection_mapping_poc/roman_door_previz.gif?raw=true)

- “luxurious and vibrant tropical forest plants with vivid colored flowers, very detailed and lots of red, orange, greens”

![output ai projection mapping with flowers ](https://miro.medium.com/v2/resize:fit:1024/format:webp/1*Mawsg9kMN-AAeG2OEZPeQw.gif)


Consider the excitement of a light festival and envision the endless possibilities if participants could directly engage with various elements of the lighting spectacle. The interactivity would provide so much excitement. The potential for innovation knows no bounds; all it takes is a single prompt to unleash boundless creativity.

