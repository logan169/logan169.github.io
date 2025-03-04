---
title: 'LLM Agents in Houdini: Rethinking VFX Workflows' 
tags: AI Machine_Learning PDG Houdini Python
---

The integration of Large Language Models (LLMs) into professional creative software has become a transformative trend across industries. Following the success of AI-driven projection mapping systems, I've been exploring how to put in place similar agent-based architectures within Houdini workflows. This article explores how to do a proof of concept implementing LLM agents within SideFX Houdini .
<!--more--> 

# Why Integrate LLM Agents In Houdini?
Houdini's node-based paradigm and Python API make it an ideal candidate for LLM integration. While Houdini offers unprecedented control and flexibility, its complexity can create steep learning curves and workflow bottlenecks. An intelligent agent that understands Houdini's architecture and can generate functional Python code opens exciting possibilities:

- Layout Automation: Rapidly prototype complex node networks based on natural language descriptions
- Technical Assistance: Generate specialized solvers and optimize existing networks
- Knowledge Transfer: Bridge knowledge gaps for artists transitioning between specializations
- Workflow Acceleration: Eliminate repetitive tasks through context-aware code generation

Over the past decade, generative large language models (LLMs) have emerged and gained prominence. Aside the hype surrounding those new tools, you've probably already experienced the frustration of copy-pasting AI-generated code, only to find it doesn't work/behave as expected. 

The Houdini LLM agent represents a more sophisticated solution – an intelligent system that doesn't just generate code, but would also validates and self-corrects it within the Houdini environment. 

By embedding the LLM agent directly into your Houdini session, we're transforming a hit-or-miss code generation process into a robust, iterative workflow. This approach goes beyond simple code generation, creating an intelligent assistant that:

- Retrieve Houdini scene informations and use it as context to answer your requests or resolve your issue
- Generates Houdini Python code
- Validates the code's functionality in real-time
- Recursively debugs and fixes errors

The agent concept opens up powerful new possibilities. Instead of hoping for working code, you now have a tool that actively works to understand and meet your specific requirements. It's not just about generating code – it's about creating a collaborative AI assistant that speaks the language of Houdini, understands your workflow, and iteratively improves its output.  

# Using Local LLM Agents

While companies like Grok, OpenAI, and Anthropic have transformed LLM inference into a SaaS business, deploying AI models locally remains a crucial consideration for many organizations—especially those handling sensitive intellectual property or operating in highly regulated industries.

As data privacy becomes an increasing priority, businesses are seeking solutions that keep sensitive information within their own controlled environments. By leveraging local LLM servers, studios and enterprises can harness AI-driven code generation and workflow automation while ensuring strict control over data security and movement.

In this blog, we will set up a local LLM server to build our agent, exploring various LLM models and weight configurations to assess their impact on the agent’s generative capabilities.

# Implementation
To implement this setup, we will rely on the following technology stack:

- Ollama: Facilitates the retrieval of LLM models and serves them on our local server.
- LangChain & LangGraph: Enables the creation of our ReAct graph-based agent workflow.
- Houdini: Executes our code within PDG (Procedural Dependency Graph), leveraging the PDG Wedger to compare the generative capabilities of different models.

## Ollama Models

Here is a list of models we will be testing. 

```bash
(base) PS C:\Users\lschwartz> ollama list
NAME                 ID              SIZE      MODIFIED
codestral:latest     0898a8b286d5    12 GB     16 hours ago
yi-coder:9b          39c63e7675d7    5.0 GB    17 hours ago
openthinker:32b      b3f4e577e166    19 GB     17 hours ago
deepscaler:latest    0031bcf7459f    3.6 GB    17 hours ago
llama3.3:latest      a6eb4748fd29    42 GB     4 days ago
deepseek-r1:70b      0c1615a8ca32    42 GB     4 days ago
llama3.2:latest      a80c4f17acd5    2.0 GB    4 days ago
codellama:34b        685be00e1532    19 GB     4 days ago
```
Since all the necessary components are already installed and available on your local network, we will leverage Houdini PDG Wedger to systematically test each LLM model using the same previously mentioned prompt query.

Ollama is designed to intelligently manage hardware resources by dynamically distributing model layers between the CPU and GPU. This means you can still run large models even if your GPU doesn’t have enough VRAM to accommodate them entirely. 

Another noteworthy feature of Ollama is its automatic multi-GPU support. If multiple GPUs are available, it will split model layers across them, optimizing inference speed. Thanks to this capability, I was able to run a 70B LLM model or multiple smaller models simultaneously on a system equipped with dual 3090 Ti GPUs.

## Runtime Environment Setup

We are utilizing the Houdini TOP Virtual Env Node to set up a dedicated virtual environment where all required Python dependencies are installed and made accessible to subsequent nodes. 

Below is a glimpse of how this setup is structured. 

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-1.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>

## Setuping A Wedger

Using Houdini PDG Wedger, we create a wedge for each of our installed Ollama LLM models. This approach enables us to compare the efficiency and speed of each model, allowing for a clear assessment of their performance under identical conditions.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-2.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>

## Prompt Engineering

In our current proof of concept, we require a standardized prompt that will be used across all models. Below is the prompt we will use for all our testing:

```python
user_prompt = f"""
run the following commands:
    - Create a geo node in the /obj context named geo_{work_item.stringAttribValue("llm_model")
}
    - Within the /obj/geo node, create a sphere node named sphere10 
    - Set the sphere10 node parm named scale to 2
    - Create a color node named set_color, connect its input to the sphere10 node output
    - Set the set_color node parmTuple named color to (1,0,0)
    - on the set_color node object, call the setDisplayFlag method with True as parameter 
"""
```
As you can see, the prompt provides a highly structured list of actions for the model to follow. This level of explicit instruction is necessary because we are employing a zero-shot approach—asking the model to generate code without supplying any additional examples or contextual guidance.

This is especially important when using general-purpose LLMs, as there are currently no models fine-tuned specifically for Houdini at the time of writing.

In order to improve the model’s responses in the future, we could supplement the prompt with additional Houdini documentation or node graph code representations. This would provide relevant context, allowing the LLM to generate more accurate and structured outputs. This approach is known as Retrieval-Augmented Generation (RAG), where external reference materials are incorporated into the prompt to enhance the model’s ability to generate domain-specific responses.

To conclude on this topic, since we will be using the PDG Wedger to run the same query across different models, we have updated the /obj/geo node’s name to include the current model name. This modification allows us to easily inspect and validate the output for each model, ensuring a more structured comparison of their generative capabilities.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-3.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>

## Coding Our Agent
Since covering LangChain and LangGraph for agent creation is beyond the scope of this blog, I won’t dive into the technical details of the agent’s code implementation—that topic deserves a dedicated post on its own. 

However, at a high level, the agent logic pseudo code key steps:

- Runs outside the PDG context using the previously created virtual environment.
- Calls Ollama with the current wedge LLM model name, stored in the PDG LLM_model attribute.
- Executes the generated code in a headless Houdini session:
    - If the 25 iteration threshold is reached without finding a solution, abort the process. This protect us from infinite loops in which the LLM models might get trapped if it isn't able to generate working python code as requested in our prompt. 
    - If execution fails, the agent re-queries the LLM, providing:
      - The initial prompt
      - The previously generated code
      - The exception raised during execution.     This allows the model to refine and update the code accordingly.
    - If execution succeeds, the final code is returned and stored as a new code_generated PDG attribute for the current work item.

This approach enables a dynamic and iterative refinement process, improving the quality of generated code while leveraging Houdini’s procedural capabilities.

Here is an example of the updated prompt after the model meets an execution error
```
Based on this user initial request:

run the following commands:
    - Create a geo node in the /obj context named geo_openthinker:32b
    - Within the /obj/geo node, create a sphere node named sphere10 
    - Set the sphere10 node parm named scale to 2
    - Create a color node named set_color, connect its input to the sphere10 node output
    - Set the set_color node parmTuple named color to (1,0,0)
    - on the set_color node object, call the setDisplayFlag method with True as parameter 


Fix the following code to address this error:
Error executing code: 'NoneType' object has no attribute 'set'
import hou

def setup_system():
    obj_node = hou.node('/obj')
    geo_node = obj_node.createNode('geo', 'geo_openthinker')
    sphere = geo_node.createNode('sphere', 'sphere10')
    sphere.parm('rad').set(2)
    color_node = geo_node.createNode('attribcreate', 'set_color')
    color_node.setNextInput(sphere)
    color_node.parm('name').set('Cd')
    value_parm = color_node.parmTuple('value')
    value_parm.set((1, 0, 0))
    geo_node.layoutChildren()

setup_system()
###################
```

All the code and logic are encapsulated within the Python TOP node `run_llm_and_generate_valid_code`. Once execution is complete, we can observe a table displaying successful work items and their corresponding details, as shown below.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-4.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>

We observe that not all LLM models successfully generate working Houdini code, with some failing after reaching the iteration threshold.

By analyzing the work item logs, we can see that certain models struggled significantly more than others to adhere to the prompt requirements, often failing to provide an output that consisted exclusively of runnable Python code. Instead, their responses included extraneous elements such as:

- The generated output included comments within the code block.
- Some models added "thoughts" or reasoning sections, explaining their process before or after the code.
- Certain models structured their response in a conversational or markdown format, rather than providing a raw executable script.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-8.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>




## Executing Code In Current Session

Once the code have been generated by models, we use the exec builtin python method to run the generated code string within the current houdini session using a python TOP node.

<div class="grid">
  <div class="cell cell--auto">
    <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-7.png?raw=true" alt="HOUDINI PDG AI LLM models">
  </div>
</div>

<br>

# Final Thoughts

In this article, I’ve shared how I leverage my "Hou Databox" package to efficiently design setups that integrate external Python libraries. I demonstrated how the package enables users to save nodes as Python snippets, which can later be recreated directly from the TAB menu, streamlining the workflow for both prototyping and reuse.

Additionally, I showcased how UI inheritance allows for the creation of topic-specific packages that benefit from the default UI and behavior of their parent packages. This approach fosters consistency while reducing the effort required to build specialized tools.

Finally, I highlighted how the package simplifies rapid iteration during UI design. By allowing on-the-fly tweaks to UI code and regenerating the interface without delving into the node’s parameter menu for each modification, "Hou Databox" significantly accelerates the process, making it a powerful tool for enhancing productivity and creativity.
