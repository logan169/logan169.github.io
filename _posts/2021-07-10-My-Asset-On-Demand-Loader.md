---
title: 'My "Asset On Demand" Loader'
tags: Coding CG
---

While working on side projects, I've realized that I was spending lot of time looking for assets on the web to work with instead of doing CG. The whole process of iteratively searching for an asset on one website, downloading, unziping and finally importing it was really slowing me. So I've decided to create an asset loader to help me out with this. 

Nothing creazy, but it got some particularities that makes it super usefull. The first one is that none of the assets are actually stored locally until I decide to import them. This doesn't looks like much but it does end up allowing me to scale up dramatically the amount of assets (roughly 60 000 assets) without having to be concerned about disk space issues. The whole process goes as follow, when I do import an asset from my asset loader interface, the code under the hood retrieves the associated asset from the website, download it and unzip it into the filepath destination of my choice.

![asset_loader](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/asset_loader.gif)


The other big particularity is that while it is a stand alone tool, it is able to communicate with any sofware I want to let it knows that the asset is ready to be imported at a specific path destination. 


![import_to_houdini](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/import_to_houdini.gif)

Given todays web technologies you can connect any softwares together with some web communication protocole knowledge to get advanced functionalities. To demonstrate my point, here is a prototype of how I'm interacting with houdini from some web page. Using the same code, I could easily use any production manager webapp (shotgun, ftrack) and turn them into an asset loader.

![browser_to_houdini2](https://raw.githubusercontent.com/logan169/logan169.github.io/master/assets/images/posts_images/asset_loader/browser_to_houdini2.gif)

I was able to achieve this thanks to my the couple of years I've worked as a web developer. This being said though my years in the Cinema industry, I've rarely came upon TDs having basics knownledges in web development and communication protocols. While it is not a mandatory things to know to do the job, I've always found that to be super limiting if you only got Qt in your competence palette to solve technical issues. Don't get me wrong here, I do believe that Qt has its place in a studio and is doing a fine jobs to solve some needs. But as the tech team, it is also our responsabilities to shake things up a bit and not be afraid to try to rethink how we are allowing artists to work and interact with their softwares in 2021.







If you think this is cool, wait a minute and think for one second. What if instead of connecting two artist sofwares we were connecting one software from two different artists? Given the setup is already there, it wouldn't make a difference from the code point as the only thing that we would need to change would be the destination ip. Doing so we could 








as Qt hasn't evolved that much compared to web technologies





  Most of the time they relied mostly on Qt applications to provide artists with a gently user interface (GUI) to interact with their software. 

What if we could propose an alternative way? 

   

