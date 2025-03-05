---
title: 'LLM Agents in Houdini' 
tags: AI Machine_Learning PDG Houdini Python
---

The integration of Large Language Models (LLMs) into professional creative software is rapidly transforming various industries. 
Recently, I explored agent-based LLM workflows and investigated how similar architectures could be applied within Houdini. 
This article introduces a proof of concept for setting up basic LLM agent setups in SideFX Houdini. 
Additionally, we will compare different LLM models, examine the challenges associated with such projects, 
and discuss strategies to address them.<!--more--> 

# Why Integrate LLM Agents In Houdini?

While Houdini offers unprecedented control and flexibility, its complexity can create steep 
learning curves and workflow bottlenecks. An intelligent agent that understands Houdini's architecture 
and can generate functional Python code opens exciting possibilities:

- Layout Automation: Rapidly prototype complex node networks based on natural language descriptions
- Technical Assistance: Generate specialized solvers and optimize existing networks
- Workflow Acceleration: Eliminate repetitive tasks through context-aware code generation

Over the past decade, generative large language models (LLMs) have emerged and gained prominence. 
Amid the hype surrounding these new tools, you’ve probably experienced the frustration of copying and pasting AI-generated code, only to find it doesn’t work or behave as expected. The Houdini LLM agent 
represents a more sophisticated solution that doesn't just generate code, 
but would also validates and self-corrects it within the Houdini environment. 

By embedding the LLM agent directly into your Houdini session, we're transforming a hit-or-miss code generation process into a robust, iterative workflow. This approach goes beyond simple code generation, creating an intelligent assistant that:

- Optionally, retrieve Houdini scene informations and use it as context to our query
- Generates Houdini hou module Python code
- Validates the code's functionality and, recursively debugs and fixes errors in real-time if needed

# Local vs. Hosted LLMs Agents

As AI-driven automation becomes more prevalent, businesses are increasingly dedicating resources to functional 
AI departments. The goal isn’t to replace workers but to delegate repetitive tasks to AI, allowing them to focus on more complex, value-driven work. When implementing LLM-powered automation, 
one of the key decisions is whether to rely on a hosted service or deploy models locally.

## Hosted LLM Services
Companies like OpenAI, Grok, and Anthropic offer powerful LLMs as SaaS solutions, making it easy to 
integrate advanced AI capabilities into existing workflows. These models provide cutting-edge performance, 
require no infrastructure management, and continuously improve with updates. However, they come with significant 
costs, especially for high-frequency inference tasks. Additionally, relying on external providers means entrusting 
them with potentially sensitive data, which may not be suitable for businesses handling proprietary intellectual 
property or operating under strict regulatory requirements.

## Local LLM Deployment
Running an LLM locally provides complete control over data security and long-term cost predictability. 
Studios and enterprises handling confidential data or intellectual property often prefer this approach 
to avoid external data exposure. However, the trade-off is performance models that can run efficiently on commercial-level GPUs tend to be smaller, leading to a decrease in generative quality and efficiency compared to their cloud-based counterparts, depending on the available hardware. Maintaining an internal 
AI infrastructure also requires dedicated technical resources to manage hardware, optimize inference, 
and update models as advancements emerge.

In this blog, we will deploy a local LLM server to develop our AI agent, experimenting with different LLM models and weight configurations to evaluate their impact on the agent’s generative performance.

# Implementation
To implement this setup, we will use the following technology stack:

- Ollama: Facilitates the retrieval of LLM models and serves them on our local server.
- LangChain & LangGraph: Enables the creation of our ReAct graph-based agent workflow.
- Houdini: Executes our code within PDG (Procedural Dependency Graph), leveraging the PDG Wedger to compare the generative capabilities of different models.

## Ollama Models

Here is a list of models we will be testing. 

```bash
(base) PS C:\Users\lschwartz> ollama list
NAME         ID       SIZE   MODIFIED
codestral:latest   0898a8b286d5  12 GB   16 hours ago
yi-coder:9b     39c63e7675d7  5.0 GB  17 hours ago
openthinker:32b   b3f4e577e166  19 GB   17 hours ago
deepscaler:latest  0031bcf7459f  3.6 GB  17 hours ago
llama3.3:latest   a6eb4748fd29  42 GB   4 days ago
deepseek-r1:70b   0c1615a8ca32  42 GB   4 days ago
llama3.2:latest   a80c4f17acd5  2.0 GB  4 days ago
codellama:34b    685be00e1532  19 GB   4 days ago
```
Since all the necessary components are already installed and available on your local network, we will leverage Houdini PDG Wedger to systematically test each LLM model using the same previously mentioned prompt query.
Ollama is designed to intelligently manage hardware resources by dynamically distributing model layers between the CPU and GPU. This means you can still run large models even if your GPU doesn’t have enough VRAM to accommodate them entirely. 

Another noteworthy feature of Ollama is its automatic multi-GPU support. If multiple GPUs are available, it will split model layers across them, optimizing inference speed. Thanks to this capability, I was able to run a 70B LLM model or multiple smaller models simultaneously on a system equipped with dual 3090 Ti GPUs.

## Runtime Environment Setup

We are utilizing the Houdini TOP Virtual Env Node to set up a dedicated virtual environment where all required Python dependencies are installed and made accessible to subsequent nodes as seen below. 

<div class="grid">
 <div class="cell cell--auto">
  <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-1.png?raw=true" alt="HOUDINI PDG AI LLM models">
 </div>
</div>

<br>

## Setuping A Wedger

Using Houdini PDG Wedger, we create a wedge for each of our installed Ollama LLM models. This approach enables us to compare the efficiency and speed of each model under identical conditions.

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
As you can see, the prompt provides a highly structured list of actions for the model to follow. This level of explicit instruction is necessary because we are employing a zero-shot approach asking the model to generate code without supplying any additional examples or contextual guidance.
This is especially important when using general-purpose LLMs, as there are currently no models fine-tuned specifically for Houdini at the time of writing. By testing models in a zero-shot setting, we can establish a baseline for their adaptability and determine how effectively they can produce meaningful outputs solely based on structured prompts. Finally, we will be using the PDG Wedger to run the same query across different models, we have updated the /obj/geo node’s name to include the current model name. This modification allows us to easily inspect and validate the output for each model.

<div class="grid">
 <div class="cell cell--auto">
  <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-3.png?raw=true" alt="HOUDINI PDG AI LLM models">
 </div>
</div>

<br>

## LangGraph Agent
Since covering LangChain and LangGraph components for agent creation is beyond the scope of this blog, I won’t dive into the specific technical details of the agent’s code implementation that topic deserves a dedicated post on its own. 

However, at a high level, the agent logic pseudo code key steps:

- Runs outside the PDG context using the previously created virtual environment.
- Calls Ollama with the current wedge LLM model name, stored in the PDG LLM_model attribute.
- Executes the generated code in a headless Houdini session:
  - If execution succeeds, the final code is returned and stored as a new code_generated PDG attribute for the current work item.
  - If execution fails, the agent re-queries the LLM, providing:
      - The initial prompt
      - The previously generated code
      - The exception raised during execution.   This allows the model to refine and update the code accordingly.
  - If the 25 iteration threshold is reached without finding a solution, abort the process. This protect us from infinite loops in which the LLM models might get trapped when being unable to generate runnable python code output. 

This approach enables a dynamic and iterative refinement process, for instance here is an example of the updated prompt after the model meets an execution error
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
  <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-6.png?raw=true" alt="HOUDINI PDG AI LLM models">
 </div>
</div>

<br>

## Executing Code

Once the code have been generated by models, we use the exec builtin python method to run the generated code string within the current houdini session using a python TOP node.

<div class="grid">
 <div class="cell cell--auto">
  <img src="https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/houdini_llm_agent/image-8.png?raw=true" alt="HOUDINI PDG AI LLM models">
 </div>
</div>

<br>

# Discussion

As seen in the previous screenshots, not all LLM models were able to successfully generate working Houdini hou Python code, each failing for different reasons. However, one key takeaway from this observation is that the effectiveness of the agent’s code generation is highly dependent on the specific model used. If you plan to undertake a similar project, I strongly recommend testing multiple models, as I have done here, to determine which best suits your base agent setup.

While not visible in the screenshots, some smaller models, such as CodeLlama and Codestral, occasionally managed to produce outputs before reaching the iteration threshold and aborting—aside from Llama 3.3. However, this came at the cost of a higher average fix iteration ratio, as the code generated by these smaller models was less reliable and required more corrections compared to the one generated by Llama 3.3.

Some models related to the DeepSeek family, while capable of generating Python code, struggled to adhere strictly to the system prompt's requirements specifically, outputting only runnable Python code. For clarity, the base behavior of each model is guided by a system prompt like the one I've used in my agent setup below, which explicitly instructs the model to generate only valid, executable Houdini (hou) package Python code, without explanations or markdown formatting:

```python
 prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an expert SideFx Houdini Python programmer specializing in the hou package library.
        Your task is to convert user requests into runnable hou package Python code .

        Rules:
        1. Only output valid, runnable hou package Python code without any explanations
        2. If data is needed, generate sample data that makes sense to answer the prompt
        3. Include appropriate imports, including the hou houdini python module
        4. Do NOT include markdown code blocks or any text other than pure Python code
        5. Ensure the code is properly indented

        Remember, your output will be directly executed, so ensure it is valid Python code."""),
        ("human", "{user_input}")
    ])
```
Despite these explicit instructions, some models deviated by including additional text, hallucinated code methods, "thoughts" or reasoning sections, or formatting python code elements, which disrupted automation workflows. 

In a later section, we will explore potential improvements to mitigate some challenges that comes with working with models that don't follow up system prompt or lack the relevant information to answer properly.

# Improving Our Agent
To enhance LLM performance in generating Houdini code, several strategies can be employed:

- Fine-Tuning LLMs: This involves adapting pre-trained models with Houdini-specific data, such as documentation and example scripts, to help the model better understand Houdini’s environment and produce more accurate code.

- Retrieval-Augmented Generation (RAG): RAG integrates LLMs with real-time retrieval of relevant Houdini documentation and code, improving context and accuracy. By storing Houdini data in a vector database, RAG agents can supply the LLM with up-to-date, task-specific information for more reliable code generation.

- Utilizing LangChain Tools: These tools standarize a task procedure by only delegating the method's input choices to the model. This helps reduce randomness in model outputs by providing structured inputs and outputs, thus guiding the model’s responses. However, not all models such as DeepSeek, support tools usage, so additional code logic may be required to accommodate these models as discussed [here](https://www.reddit.com/r/LangChain/comments/1j0wsz7/update_tool_calling_for_deepseekr1_with_langchain/).

- Optimizing Code Validation Logic: Testing code line by line, rather than as a complete block, reduces the amount of information the model needs to process when resolving errors, allowing it to generate more precise and refined code with each iteration.

Together, these strategies would improve the reliability and effectiveness of LLMs within Houdini workflows. This might be enough to mitigate the previously mentionned issues we've met.

# Final Thoughts

Integrating Large Language Models (LLMs) into Houdini workflows offers promising avenues for automating complex tasks and enhancing creative processes. 

The most reliable and capable model for generating executable code using a zero-shot approach was Llama 3.3 (70B) by far. While smaller LLMs like CodeLlama (34B) and Codestral (22B) occasionally produced working outputs, my experiments showed that, despite potential speed advantages, without advanced optimization, they consistently required more iterations than Llama 3.3 to generate code within a reasonable timeframe, making them less effective as a useful agent assistant.

In conclusion, selecting the ideal LLM for Houdini workflows requires balancing model size, performance, and available resources. Llama 3.3 (70B) stands out for its reliability and effectiveness in a zero-shot approach. When computational resources are limited, models such as CodeLlama and Codestral might offer viable alternatives when combined with optimizations discussed previously. 
