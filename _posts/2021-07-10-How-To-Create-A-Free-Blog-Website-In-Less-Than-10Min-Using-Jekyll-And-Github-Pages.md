---
title: 'How To Create A Free Blog Website In Less Than 10Min Using Jekyll And Github Pages'
tags: Github Jekyll
---

### Introduction:

Jekyll is a static website generator. The simplicity of Jekyll’s theming layer and writing workflow associated with all the open-source themes available out there makes Jekyll a though worth choice when you don't have either the time or competences to make one from scratch. Depending if you are planning to host a blog, documentation, a company webpage or simply your resume you might wanna start from an available template on [GitHub](https://github.com/search?q=jekyll+theme) or [awesomeopensource.com](https://awesomeopensource.com/projects/jekyll-themes). 


For our blog, we will use [TeXt-theme](https://tianqi.name/jekyll-TeXt-theme/). As I'm planning to make a technical blog out of it, I've decided to go for this template as it comes with advanced functionalities like advanced pages formatting, charts, diagrams, chat plugin and more. Technically, you can pick any other Jekyll template you'd like as most repositories storing them ship with the correct travis.yml file to automatically turn the repository to a Github page every time it got updated.

So the match plan is to use this Jekyll website template and host it on Github using Github Pages. If you have never heard of Github before, [GitHub](https://github.com) is a web Git repository hosting service with an extra layer of its own features like Github pages for instance. Basically, it is being used to easily store and share code with potential collaborators but could also be used to host a static website that belongs to one of your repositories. 

It is important to know that while Github allows users to have their repositories being either private or public, they will only allow you to host pages from a public repo without a paid plan. If you go for a paid plan different that entrepise you'd also need to know that the hosted website will always be publicly accessible to anyone knowing the url. If this is an issue for you, as you might be planning to write some API key/passwords in the config file then I'd suggest to use Gitlab that allows you to host static website for free with a private repository or even some cloud provider like Digital Ocean if you are planning to end up scaling your game with a dedicated server in the future. 

Ok, so at this point you are expected to have decided which Jekyll template you'd like to start from if you picked one different than mine and have created a Github account if you have none and you are logged in. 

### Lets Do It!

Now from the Jekyll template's Github page, click on the fork button.

![testt](screenshots/jekyll_blog_source_repo.png)


This should fork the repo to your Github account. Once you get there, you'd need to rename the repository name by clicking on Settings/cog icon and rename the repository name to either one of those options: 

If you are planning to only use one website, I'd suggest renaming to this format {GitHub_username}.github.io. This will load your website under the url looking like https://{GitHub_username}.github.io . 

If you are planning to get one website per repo then I'd use the project name instead then the url would be https://{GitHub_username}.github.io/{GitHub_project_name}




Now visit the url following the previously mentioned format and you should see your website accessible on the world wide web! If you don't, have a look at the settings > pages section and double check that the displayed website url is the right one you've typed.



Technically, if you are only planning to do a single page app website like for a resume I wouldn't even try setting up a local coding environment and use the Github IDE instead. so you could potentially complete the whole process without ever having to install anything locally which is a gain of time that is most of the time under evaluated in other blogs when it comes to web development. 

Last hint for the road, if you end up doing it locally, remember to add the -l -o to the command `bundle exec jekyll serve -l -o` . Those tags will auto-reload your website on any saved changes and open the browser for you.
