---
title: 'Building the 1st Houdini Finance Library' 
tags: PDG Houdini Python Finance
---

Leveraging the power of Hou Databox, I developed a specialized financial package that integrates seamlessly with Houdini's PDG. This library facilitates querying and processing financial data from external sources, such as stock or cryptocurrency APIs, and builds upon custom UIs to manage large datasets effectively. The streamlined workflow simplifies creating tools for financial analysis, such as nodes for data visualization, Monte Carlo simulations, or backtesting strategies. This demonstrates how the versatility of Hou Databox can scale to meet the demands of highly specialized domains like finance.
<!--more--> 

Note: "Hou Fin" is closely integrated with the "Hou Databox" and "Hou Pandas" Houdini plugins, which were described in detail in this [previous article](https://logan169.github.io/2025/01/17/simplifying-houdini-custom-ui-creation.html). If you’re interested in understanding the implementation, I recommend checking it out first for more context as I won't cover this here

# Implementation

Using my "Hou Pandas" Houdini package as a foundation, I developed a child package called "Hou Fin," which is specialized in financial analysis within PDG. All "Hou Fin" nodes inherit the default "Hou Pandas" UI, which includes functionality for importing and exporting dataframes, as mentioned previously. From there, the nodes build upon the UI with specific code tailored to their financial analysis features, enabling the rapid design of workflows that leverage external financial Python packages directly within Houdini. 

## Unlocking External Python Packages For Non-Coders

Here are several financial setups that rely on "Hou Pandas" and "Hou Fin" to query cryptocurrency and stock data, compute returns, find outliners and run Monte Carlo simulations on previous returns to approximate potential future ones. 

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img18.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

These setups were completed quickly because the UI creation process is now much faster, and you can easily copy and paste demo code from external Python documentation as a base for your node setup. The ability to iterate more quickly while working on such workflows offers several advantages. One key benefit is that non-coder end users can test the tools more frequently and provide valuable feedback, ultimately leading to more refined and effective tools in the end.

## Scalability

In financial analysis, the goal is often to identify a signal you can exploit amidst the noise. Scaling your analysis is crucial to increasing the chances of uncovering something relevant. This is where the workflow, when combined with wedger, becomes incredibly powerful. By leveraging wedger’s ability to manage and process a large number of stock tickers, the workflow enables you to efficiently scale your analysis and better pinpoint actionable insights.

Below is a series of screenshots showing the step-by-step setup, node by node, where I am backtesting a trading strategy (using backtrader) for each wedge stock tickers after retrieving the stock price (using yfinance) and outputting a comprehensive report at the end.


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

When running the above we do get the associated ticker price data baked into Excel files, and for each backtrading reports as well as you can see below.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img8.png?raw=true" alt="pdg custom UIs">
  </div>
</div>

<br>

## Building More Advanced UIs

As well, by adopting the workflow provided by "Hou Databox," I was able to significantly scale up my node's UI capabilities, enabling the creation of nodes that would have otherwise been impractical within a reasonable timeframe. A prime example is a node I developed to query data from the financial website finviz.com. To put it into perspective, replicating the filter interface from the site required building a UI that accommodates approximately 70 menus, each containing between 10 to 20 options.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/img16.png?raw=true" alt="pdg custom UIs">
  </div>
</div>


Accomplishing this level of complexity manually with a standard HDA would have been incredibly time-consuming, to the point where it would no longer be practical. However, with "Hou Databox," the streamlined process makes it possible to develop such intricate interfaces both efficiently and effectively.

<div class="grid">
  <div class="cell cell--auto">
  <video width="100%" controls>
    <source src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/pdg_ui/video1.mp4?raw=true" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  </div>
</div>

# Final Thoughts

In this article, I’ve shared how I leverage my "Hou Databox" package to efficiently design setups that integrate external Python libraries. I demonstrated how the package enables users to save nodes as Python snippets, which can later be recreated directly from the TAB menu, streamlining the workflow for both prototyping and reuse.

Additionally, I showcased how UI inheritance allows for the creation of topic-specific packages that benefit from the default UI and behavior of their parent packages. This approach fosters consistency while reducing the effort required to build specialized tools.

Finally, I highlighted how the package simplifies rapid iteration during UI design. By allowing on-the-fly tweaks to UI code and regenerating the interface without delving into the node’s parameter menu for each modification, "Hou Databox" significantly accelerates the process, making it a powerful tool for enhancing productivity and creativity.
