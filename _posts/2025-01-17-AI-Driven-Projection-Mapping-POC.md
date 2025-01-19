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

Now that we’ve downloaded and imported our model into Houdini, we will apply a series of transformations to align it with the viewport grid. We then split the mesh into Houdini groups, which we’ll later use to populate a custom attribute named “building_parts”. This attribute will prove useful shortly when we delve into the generative AI step, as it will be used to create masks I will use to merge all our generated AI layers using compositing.

For this proof of concept, we’ve limited ourselves to three zones of interest: the main door group, encompassing all areas within the arch; the sidewalls, consisting of the two side columns and the main wall; essentially everything else. At that point, our setup that handles the mesh looks like the one below.


![projection mapping houdini setup 1](https://miro.medium.com/v2/resize:fit:4202/format:webp/1*HmTh8uXhkvN265GA_D-TbQ.png)


Now, we return to Houdini root at the “/obj” level and create a camera and a point light, which we’ll utilize as a gobo light to project our image. 

--------------------------
TODO add picture from 1st article
--------
Additionally, we set up an environment light. Finally, as demonstrated in our previous article, we import our image-generative AI workflow. At that point, our “/obj” level looks like this.

![projection mapping houdini setup 3](https://miro.medium.com/v2/resize:fit:2800/format:webp/0*Tp4-KZp6GgBTRG-2)

We established the camera resolution at 512x512, which serves as the maximum resolution input image for the stable diffusion model.

As illustrated in the preceding screenshot, a critical procedure involved configuring the camera settings to attain consistent viewpoints between the camera and the gobo-equipped point light. This alignment is essential because we want the image projected from the gobo to match what has been captured by the camera.

Now, let’s explore the generative AI step further. Within our “/obj/sd” node, we’ll create three separate graphs, each tasked with generating an image for a specific part of the segmented mesh using the custom attribute “building_parts” we established earlier. This segmentation enables us to tailor the generative AI process to each distinct area of interest in the scene, ensuring precise and targeted image generation.

This means we use the same design to build three different prompt and model architectures for the doors or arch, the mainwall, and the sidewalls. Below is the design explained for the door area.

![projection mapping houdini setup 4](https://miro.medium.com/v2/resize:fit:2730/format:webp/1*zPZ9SnvJ2_t--jM8EuLvcw.png)

We connect all of those branches back using a merge node and connect this later one to a Python SOP node. This Python node will refresh the loading of the COP nodes, which will load the generated AI image residing within the “cop2net” node below. Once each branch has finished generating an image, this refresh triggers an update in the compositing process, consequently updating the projected image at the gobo level. This linkage is dynamic, as it’s directly associated with our “cop2net” node's final output node.

![projection mapping houdini setup 5](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xnIdqhz-r6J_tTSaGxJ-Zg.png)

![projection mapping houdini setup 6](https://miro.medium.com/v2/resize:fit:4160/format:webp/1*PqkIiF5wwTqI51lZmYrJUQ.png)

Above is the design for the cop2net node, showing the different stages of implementation of the stable diffusion images and the masks from the attributes into composite images. This leads to the final output image.

Next, we set the gobo light’s “Projection Map” parameter to dynamically retrieve the image directly from the cop2net1 node using the following expression “op:/obj/sd/cop2net1/OUT/”.

![projection mapping houdini setup 8](https://miro.medium.com/v2/resize:fit:198/format:webp/0*d2UsD3q3Z4WnbG1a)

We also enable the viewport’s high-quality lighting option (the bulb button on the left) and select “Show All Objects” from the top viewport button options. This ensures we can accurately preview the scene’s lighting and overall appearance in real-time feedback.

Here is an example of a new door style when applying the code and the different steps of scene view options:
![projection mapping houdini setup 9](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*zyQiB9l3c9mYTrC7mwP9vQ.png)


Of course, it is a proof of concept and the quality could be enhanced with better lighting, different testing with the parameters, we could use a new version of the stable diffusion model. As always, there is a lot of space for improvement in machine learning projects and that is the beauty of it!

Consider the excitement of a light festival and envision the endless possibilities if participants could directly engage with various elements of the lighting spectacle. The interactivity would provide so much excitement. The potential for innovation knows no bounds; all it takes is a single prompt to unleash boundless creativity.


----------------------------------------------------------------


![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()
![projection mapping houdini setup 4]()


![projection mapping houdini setup 4]()

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