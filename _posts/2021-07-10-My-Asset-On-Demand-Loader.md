---
title: 'My "Asset On Demand" Loader'
tags: Coding CG
---

While working on side projects, I've realized that I was spending a lot of time looking for assets on the web to work with instead of doing CG. The whole process of iteratively searching for an asset on one website, downloading, unzipping and finally importing it was really slowing me down so I've decided to create an asset loader to help me out with this. 
<!--more--> 


### An Asset Loader To Rule Them All

The asset loader I've developed has some particularities that make it super useful. The first one is that none of the assets are actually stored locally until I decide to import them. This doesn't look like much but it does end up allowing me to scale up dramatically the amount of assets (roughly 60 000 assets) without having to be concerned about disk space issues. When I import an asset from my asset loader interface, the code under the hood retrieves the associated asset from the website, downloads it and unzips it into the file path destination of my choice.

![asset_loader](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/asset_loader.gif)

The other big particularity is that while it is a stand alone tool, it is able to communicate with any software I want to let it know that the asset is ready to be imported at a specific path destination.

![import_to_houdini](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/import_to_houdini.gif)

### Adding A Pinch Of Web Magic

Given todays' technologies you can connect any softwares together to get advanced functionalities using some web communication protocol knowledge. To demonstrate my point, here is a prototype on how I'm interacting with houdini from some web page. In order to do it, I've just remade the asset loader communication code parts using javascript.

![browser_to_houdini2](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/browser_to_houdini2.gif)

Using the same code, I could easily use any production manager webapp (shotgun, ftrack) and turn them into an asset loader or link a webhook to auto-reload an artist's asset version if a new one just got published.

This is just another example of many use case scenarios. But the road doesn't end up there, sky and probably your imagination is the limit. 
As a last concluding example, using a similar setup I was able to connect multiple houdini sessions together to create an interconnected scene where artists could collaborate together in real time but this might be for another post.


### Power-Knowledge  

After working for the last 5 years in the Cinema industry, 95% of TDs I've met didn't have basic web-dev knowledge. While it is not a mandatory thing to do the job, I feel that the value of this knowledge has been (and still) underrated in our profession. I think modern pipelines wouldn't look the same at all if everyone knew this and I hope that this post and all possibilities you might think of coming from inter-connecting softwares at a studio level will convince you to approach the problems we are facing on a daily basis with a different angle.
