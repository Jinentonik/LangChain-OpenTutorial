<style>
.custom {
    background-color: #008d8d;
    color: white;
    padding: 0.25em 0.5em 0.25em 0.5em;
    white-space: pre-wrap;       /* css-3 */
    white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
    white-space: -pre-wrap;      /* Opera 4-6 */
    white-space: -o-pre-wrap;    /* Opera 7 */
    word-wrap: break-word;
}

pre {
    background-color: #027c7c;
    padding-left: 0.5em;
}

</style>

# Generate synthetic test dataset (with RAGAS)

- Author: [Yoonji](https://github.com/samdaseuss)
- Peer Review: [MinJi Kang](https://www.linkedin.com/in/minji-kang-995b32230/), [Youngjun cho](https://github.com/choincnp)
- This is a part of [LangChain Open Tutorial](https://github.com/LangChain-OpenTutorial/LangChain-OpenTutorial)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/LangChain-OpenTutorial/LangChain-OpenTutorial/blob/main/99-TEMPLATE/00-BASE-TEMPLATE-EXAMPLE.ipynb) [![Open in GitHub](https://img.shields.io/badge/Open%20in%20GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/LangChain-OpenTutorial/LangChain-OpenTutorial/blob/main/99-TEMPLATE/00-BASE-TEMPLATE-EXAMPLE.ipynb)

## Overview

### Welcome Back!

Hi everyone! Welcome to our first lecture in the evaluation section.  
We're going to try something special today!  
While we've been building RAG systems, we haven't really talked about how to test if they're working well.  
To properly evaluate a RAG system, we need good test data—and that's exactly what we'll be creating in this tutorial!  
We'll learn how to build datasets that will help us measure our RAG pipeline's performance.  


### Today, what we are going to learn...
In this session, we'll focus on using RAGAS to create evaluation datasets for RAG systems. Our main tasks will include:
- Preprocessing documents for evaluation.
- Defining evaluation objects.
- Defining Knowledge Graphs, creating Nodes, and establishing relationships between nodes
- Concepts of Extractor 
- Configuring data distributions to generate various types of test questions.

We'll explore these concepts through hands-on practice, giving you a practical foundation for building evaluation datasets.

### Why this matters...
The goal is to craft datasets that objectively assess the performance of your RAG system. A well-designed test can highlight how your system handles diverse questions and scenarios, revealing both strengths and areas needing improvement.

By the end of this tutorial, you'll have the skills to build robust datasets for comprehensive evaluation. 

Without further ado, let's get started!

### Table of Contents
- [Overview](#overview)
- [Environment Setup](#environment-setup)
- [Looking Back at What We've Learned](#looking-back-at-what-weve-learned)
- [Installation](#installation)
- [What is RAGAS?](#what-is-ragas)
- [RAGAS in Python](#ragas-in-python)
- [Document Preprocessing](#document-preprocessing)
- [Dataset Generation](#dataset-generation)
- [Distribution of Question Types](#distribution-of-question-types)
- [Summary: Moving Forward with Generated and Prepared Datasets](#summary-moving-forward-with-generated-and-prepared-datasets)
- [Bonus: Refactoring Section](#bonus-refactoring-section)

### References

- [Testset Generation for RAG](https://docs.ragas.io/en/stable/getstarted/rag_testset_generation/)
- [Testset Generation for RAG : Core Concepts > Test Data Generation > RAG](https://docs.ragas.io/en/stable/concepts/test_data_generation/rag/)

----

## Environment Setup

Set up the environment. You may refer to [Environment Setup](https://wikidocs.net/257836) for more details.

**[Note]**
- `langchain-opentutorial` is a package that provides a set of easy-to-use environment setup, useful functions and utilities for tutorials. 
- You can checkout the [`langchain-opentutorial`](https://github.com/LangChain-OpenTutorial/langchain-opentutorial-pypi) for more details.

```python
%%capture --no-stderr
%pip install langchain-opentutorial
```

```python
# Install required packages
from langchain_opentutorial import package

package.install(
    [
        "langchain",
        "langchain_core",
        "langchain_community",
        "langchain_text_splitters",
        "langchain_openai",
    ],
    verbose=False,
    upgrade=False,
)
```

```python
# Set environment variables
from langchain_opentutorial import set_env

set_env(
    {
        "OPENAI_API_KEY": "",
        "LANGCHAIN_API_KEY": "",
        "LANGCHAIN_TRACING_V2": "true",
        "LANGCHAIN_ENDPOINT": "https://api.smith.langchain.com",
        "LANGCHAIN_PROJECT": "Generate synthetic test dataset (with RAGAS)",
    }
)
```

<pre class="custom">Environment variables have been set successfully.
</pre>

You can alternatively set API keys such as `OPENAI_API_KEY` in a `.env` file and load them.

[Note] This is not necessary if you've already set the required API keys in previous steps.

```python
# Load API keys from .env file
from dotenv import load_dotenv

load_dotenv(override=True)
```




<pre class="custom">True</pre>



## Looking Back at What We've Learned
In this session, let's review what we've learned so far.

### We Have Learned About RAG

LLM is a powerful technology, but it has limitations in reflecting real-time information due to the constraints of its training data.

For example, let's say NASA discovered a new planet yesterday, making the total number of planets in the solar system nine. What would happen if we asked an LLM about the number of planets in the solar system? Because LLM responds based on its trained data, it would say there are eight planets. We call this phenomenon **hallucination** , and to resolve this, we need to wait for a model **version up** .

RAG emerged to overcome these limitations. Instead of immediately responding to user questions, the RAG pipeline first searches for the latest information from external knowledge repositories and then generates responses based on this information. This enables the system to provide answers that reflect the most **up-to-date** information.

### Is Our RAG Design Effective?

You have learned various techniques for implementing RAG. Some of you may have already built your own RAG systems and applied them to your work.

However, we need to ask an important question: Is our RAG system truly a 'good' RAG? How can we judge the quality of RAG?

Simply saying "this RAG doesn't perform well" is not enough. We need to be able to measure and verify RAG's performance through objective evaluation metrics.

### Why Use Synthetic Test Dataset?

Evaluating the performance of RAG systems is a crucial process. However, manually creating hundreds of question-answer pairs requires enormous time and effort.

Moreover, manually written questions often remain at a simple and superficial level, making it difficult to thoroughly evaluate the performance of RAG systems.

By utilizing synthetic data to solve these problems, we can reduce developer time spent on building test datasets by up to 90%. Additionally, it enables more thorough performance evaluation by automatically generating test cases of various difficulty levels and types.

## Installation

To proceed with this tutorial, you need to install the `RAGAS` and `pdfplumber` package. Through the command below, we'll install the `RAGAS`and `pdfplumber` package, and immediately after, we'll explore **the concept of `RAGAS`** and learn about Python's **`RAGAS` package** in detail.

```python
%pip install -qU ragas pdfplumber
```

<pre class="custom">Note: you may need to restart the kernel to use updated packages.
</pre>

## What is `RAGAS`?
`RAGAS` (Retrieval Augmented Generation Assessment Suite) is a comprehensive evaluation framework designed to assess the performance of RAG systems. It helps developers and researchers measure how well their RAG implementations are working through various metrics and evaluation methods.

Let's revisit the example we saw earlier.

Let's say NASA discovered a new planet yesterday, making the total number of planets in our solar system nine. To evaluate the performance of a RAG system, let's ask the test question "How many planets are in our solar system?" `RAGAS` evaluates the system's response using these key metrics:

1. **Answer Relevancy**: Checks if the answer directly addresses the question about the number of planets
2. **Context Relevancy**: Checks if the system retrieved the recent NASA announcement instead of old astronomy textbooks
3. **Faithfulness**: Checks if the answer about nine planets is based on the NASA announcement and not on outdated data
4. **Context Precision**: Checks if the system used the NASA announcement efficiently without including unnecessary space information

For example, if the RAG system responds with **outdated information** saying there are eight planets, `RAGAS` will give it a low context relevancy score. Or if it makes claims about the new planet that aren't in the NASA announcement, it will receive a low faithfulness score.

## `RAGAS` in Python
You can easily use `RAGAS` with Python libraries.

`Ragas` is a library that provides tools to supercharge the evaluation of Large Language Model (LLM) applications. It is designed to help you evaluate your LLM applications with ease and confidence.

## Document Processing
Let's prepare our documents through preprocessing before building the dataset!

### Document
While the official `RAGAS` package website demonstrates tutorials using markdown, in this tutorial, we'll be working with **pdf files** . Please use the files located in the **data folder** .

```python
file_path = 'data/Newwhitepaper_Agents2.pdf'
```

### Document Preprocessing
We will use PDFPlumberLoader to load PDF files and process document pages starting from index 3 through the final index.

```python
from langchain_community.document_loaders import PDFPlumberLoader

# Create document loader
loader = PDFPlumberLoader(file_path)

# Load documents
docs = loader.load()

# Exclude table of contents and last page
docs = docs[3:-1]

# Get the number of document pages
len(docs)
```




<pre class="custom">38</pre>



The output documents from `PDFPlumberLoader` include detailed metadata about the PDF and its pages, returning one document per page.

Each document object includes a `metadata` dictionary that can be used to store additional information about the document, which can be accessed through `metadata`.

Please check if the `metadata` dictionary contains a key called `filename` .

This key will be used in the **Test datasets generation process** . The `filename` attribute in `metadata` is used to identify chunks belonging to the same document.

```python
# Set metadata ('filename' must exist)
for doc in docs:
    doc.metadata["filename"] = doc.metadata["source"]
```

```python
docs
```




<pre class="custom">[Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 3, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content="Agents\nThis combination of reasoning,\nlogic, and access to external\ninformation that are all connected\nto a Generative AI model invokes\nthe concept of an agent.\nIntroduction\nHumans are fantastic at messy pattern recognition tasks. However, they often rely on tools\n- like books, Google Search, or a calculator - to supplement their prior knowledge before\narriving at a conclusion. Just like humans, Generative AI models can be trained to use tools\nto access real-time information or suggest a real-world action. For example, a model can\nleverage a database retrieval tool to access specific information, like a customer's purchase\nhistory, so it can generate tailored shopping recommendations. Alternatively, based on a\nuser's query, a model can make various API calls to send an email response to a colleague\nor complete a financial transaction on your behalf. To do so, the model must not only have\naccess to a set of external tools, it needs the ability to plan and execute any task in a self-\ndirected fashion. This combination of reasoning, logic, and access to external information\nthat are all connected to a Generative AI model invokes the concept of an agent, or a\nprogram that extends beyond the standalone capabilities of a Generative AI model. This\nwhitepaper dives into all these and associated aspects in more detail.\nSeptember 2024 4\n"),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 4, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nWhat is an agent?\nIn its most fundamental form, a Generative AI agent can be defined as an application that\nattempts to achieve a goal by observing the world and acting upon it using the tools that it\nhas at its disposal. Agents are autonomous and can act independently of human intervention,\nespecially when provided with proper goals or objectives they are meant to achieve. Agents\ncan also be proactive in their approach to reaching their goals. Even in the absence of\nexplicit instruction sets from a human, an agent can reason about what it should do next to\nachieve its ultimate goal. While the notion of agents in AI is quite general and powerful, this\nwhitepaper focuses on the specific types of agents that Generative AI models are capable of\nbuilding at the time of publication.\nIn order to understand the inner workings of an agent, let’s first introduce the foundational\ncomponents that drive the agent’s behavior, actions, and decision making. The combination\nof these components can be described as a cognitive architecture, and there are many\nsuch architectures that can be achieved by the mixing and matching of these components.\nFocusing on the core functionalities, there are three essential components in an agent’s\ncognitive architecture as shown in Figure 1.\nSeptember 2024 5\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 5, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nFigure 1. General agent architecture and components\nThe model\nIn the scope of an agent, a model refers to the language model (LM) that will be utilized as\nthe centralized decision maker for agent processes. The model used by an agent can be one\nor multiple LM’s of any size (small / large) that are capable of following instruction based\nreasoning and logic frameworks, like ReAct, Chain-of-Thought, or Tree-of-Thoughts. Models\ncan be general purpose, multimodal or fine-tuned based on the needs of your specific agent\narchitecture. For best production results, you should leverage a model that best fits your\ndesired end application and, ideally, has been trained on data signatures associated with the\ntools that you plan to use in the cognitive architecture. It’s important to note that the model is\ntypically not trained with the specific configuration settings (i.e. tool choices, orchestration/\nreasoning setup) of the agent. However, it’s possible to further refine the model for the\nagent’s tasks by providing it with examples that showcase the agent’s capabilities, including\ninstances of the agent using specific tools or reasoning steps in various contexts.\nSeptember 2024 6\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 6, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nThe tools\nFoundational models, despite their impressive text and image generation, remain constrained\nby their inability to interact with the outside world. Tools bridge this gap, empowering agents\nto interact with external data and services while unlocking a wider range of actions beyond\nthat of the underlying model alone. Tools can take a variety of forms and have varying\ndepths of complexity, but typically align with common web API methods like GET, POST,\nPATCH, and DELETE. For example, a tool could update customer information in a database\nor fetch weather data to influence a travel recommendation that the agent is providing to\nthe user. With tools, agents can access and process real-world information. This empowers\nthem to support more specialized systems like retrieval augmented generation (RAG),\nwhich significantly extends an agent’s capabilities beyond what the foundational model can\nachieve on its own. We’ll discuss tools in more detail below, but the most important thing\nto understand is that tools bridge the gap between the agent’s internal capabilities and the\nexternal world, unlocking a broader range of possibilities.\nThe orchestration layer\nThe orchestration layer describes a cyclical process that governs how the agent takes in\ninformation, performs some internal reasoning, and uses that reasoning to inform its next\naction or decision. In general, this loop will continue until an agent has reached its goal or a\nstopping point. The complexity of the orchestration layer can vary greatly depending on the\nagent and task it’s performing. Some loops can be simple calculations with decision rules,\nwhile others may contain chained logic, involve additional machine learning algorithms, or\nimplement other probabilistic reasoning techniques. We’ll discuss more about the detailed\nimplementation of the agent orchestration layers in the cognitive architecture section.\nSeptember 2024 7\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 7, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nAgents vs. models\nTo gain a clearer understanding of the distinction between agents and models, consider the\nfollowing chart:\nModels Agents\nKnowledge is limited to what is available in their Knowledge is extended through the connection\ntraining data. with external systems via tools\nSingle inference / prediction based on the Managed session history (i.e. chat history) to\nuser query. Unless explicitly implemented for allow for multi turn inference / prediction based\nthe model, there is no management of session on user queries and decisions made in the\nhistory or continuous context. (i.e. chat history) orchestration layer. In this context, a ‘turn’ is\ndefined as an interaction between the interacting\nsystem and the agent. (i.e. 1 incoming event/\nquery and 1 agent response)\nNo native tool implementation. Tools are natively implemented in agent\narchitecture.\nNo native logic layer implemented. Users can Native cognitive architecture that uses reasoning\nform prompts as simple questions or use frameworks like CoT, ReAct, or other pre-built\nreasoning frameworks (CoT, ReAct, etc.) to agent frameworks like LangChain.\nform complex prompts to guide the model in\nprediction.\nCognitive architectures: How agents operate\nImagine a chef in a busy kitchen. Their goal is to create delicious dishes for restaurant\npatrons which involves some cycle of planning, execution, and adjustment.\nSeptember 2024 8\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 8, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\n• They gather information, like the patron’s order and what ingredients are in the pantry\nand refrigerator.\n• They perform some internal reasoning about what dishes and flavor profiles they can\ncreate based on the information they have just gathered.\n• They take action to create the dish: chopping vegetables, blending spices, searing meat.\nAt each stage in the process the chef makes adjustments as needed, refining their plan as\ningredients are depleted or customer feedback is received, and uses the set of previous\noutcomes to determine the next plan of action. This cycle of information intake, planning,\nexecuting, and adjusting describes a unique cognitive architecture that the chef employs to\nreach their goal.\nJust like the chef, agents can use cognitive architectures to reach their end goals by\niteratively processing information, making informed decisions, and refining next actions\nbased on previous outputs. At the core of agent cognitive architectures lies the orchestration\nlayer, responsible for maintaining memory, state, reasoning and planning. It uses the rapidly\nevolving field of prompt engineering and associated frameworks to guide reasoning and\nplanning, enabling the agent to interact more effectively with its environment and complete\ntasks. Research in the area of prompt engineering frameworks and task planning for\nlanguage models is rapidly evolving, yielding a variety of promising approaches. While not an\nexhaustive list, these are a few of the most popular frameworks and reasoning techniques\navailable at the time of this publication:\n• ReAct, a prompt engineering framework that provides a thought process strategy for\nlanguage models to Reason and take action on a user query, with or without in-context\nexamples. ReAct prompting has shown to outperform several SOTA baselines and improve\nhuman interoperability and trustworthiness of LLMs.\nSeptember 2024 9\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 9, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\n• Chain-of-Thought (CoT), a prompt engineering framework that enables reasoning\ncapabilities through intermediate steps. There are various sub-techniques of CoT including\nself-consistency, active-prompt, and multimodal CoT that each have strengths and\nweaknesses depending on the specific application.\n• Tree-of-thoughts (ToT),, a prompt engineering framework that is well suited for\nexploration or strategic lookahead tasks. It generalizes over chain-of-thought prompting\nand allows the model to explore various thought chains that serve as intermediate steps\nfor general problem solving with language models.\nAgents can utilize one of the above reasoning techniques, or many other techniques, to\nchoose the next best action for the given user request. For example, let’s consider an agent\nthat is programmed to use the ReAct framework to choose the correct actions and tools for\nthe user query. The sequence of events might go something like this:\n1. User sends query to the agent\n2. Agent begins the ReAct sequence\n3. The agent provides a prompt to the model, asking it to generate one of the next ReAct\nsteps and its corresponding output:\na. Question: The input question from the user query, provided with the prompt\nb. Thought: The model’s thoughts about what it should do next\nc. Action: The model’s decision on what action to take next\ni. This is where tool choice can occur\nii. For example, an action could be one of [Flights, Search, Code, None], where the first\n3 represent a known tool that the model can choose, and the last represents “no\ntool choice”\nSeptember 2024 10\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 10, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nd. Action input: The model’s decision on what inputs to provide to the tool (if any)\ne. Observation: The result of the action / action input sequence\ni. This thought / action / action input / observation could repeat N-times as needed\nf. Final answer: The model’s final answer to provide to the original user query\n4. The ReAct loop concludes and a final answer is provided back to the user\nFigure 2. Example agent with ReAct reasoning in the orchestration layer\nAs shown in Figure 2, the model, tools, and agent configuration work together to provide\na grounded, concise response back to the user based on the user’s original query. While\nthe model could have guessed at an answer (hallucinated) based on its prior knowledge,\nit instead used a tool (Flights) to search for real-time external information. This additional\ninformation was provided to the model, allowing it to make a more informed decision based\non real factual data and to summarize this information back to the user.\nSeptember 2024 11\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 11, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nIn summary, the quality of agent responses can be tied directly to the model’s ability to\nreason and act about these various tasks, including the ability to select the right tools, and\nhow well that tools has been defined. Like a chef crafting a dish with fresh ingredients and\nattentive to customer feedback, agents rely on sound reasoning and reliable information to\ndeliver optimal results. In the next section, we’ll dive into the various ways agents connect\nwith fresh data.\nTools: Our keys to the outside world\nWhile language models excel at processing information, they lack the ability to directly\nperceive and influence the real world. This limits their usefulness in situations requiring\ninteraction with external systems or data. This means that, in a sense, a language model\nis only as good as what it has learned from its training data. But regardless of how much\ndata we throw at a model, they still lack the fundamental ability to interact with the outside\nworld. So how can we empower our models to have real-time, context-aware interaction with\nexternal systems? Functions, Extensions, Data Stores and Plugins are all ways to provide this\ncritical capability to the model.\nWhile they go by many names, tools are what create a link between our foundational models\nand the outside world. This link to external systems and data allows our agent to perform a\nwider variety of tasks and do so with more accuracy and reliability. For instance, tools can\nenable agents to adjust smart home settings, update calendars, fetch user information from\na database, or send emails based on a specific set of instructions.\nAs of the date of this publication, there are three primary tool types that Google models are\nable to interact with: Extensions, Functions, and Data Stores. By equipping agents with tools,\nwe unlock a vast potential for them to not only understand the world but also act upon it,\nopening doors to a myriad of new applications and possibilities.\nSeptember 2024 12\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 12, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nExtensions\nThe easiest way to understand Extensions is to think of them as bridging the gap between\nan API and an agent in a standardized way, allowing agents to seamlessly execute APIs\nregardless of their underlying implementation. Let’s say that you’ve built an agent with a goal\nof helping users book flights. You know that you want to use the Google Flights API to retrieve\nflight information, but you’re not sure how you’re going to get your agent to make calls to this\nAPI endpoint.\nFigure 3. How do Agents interact with External APIs?\nOne approach could be to implement custom code that would take the incoming user query,\nparse the query for relevant information, then make the API call. For example, in a flight\nbooking use case a user might state “I want to book a flight from Austin to Zurich.” In this\nscenario, our custom code solution would need to extract “Austin” and “Zurich” as relevant\nentities from the user query before attempting to make the API call. But what happens if the\nuser says “I want to book a flight to Zurich” and never provides a departure city? The API call\nwould fail without the required data and more code would need to be implemented in order\nto catch edge and corner cases like this. This approach is not scalable and could easily break\nin any scenario that falls outside of the implemented custom code.\nSeptember 2024 13\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 13, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nA more resilient approach would be to use an Extension. An Extension bridges the gap\nbetween an agent and an API by:\n1. Teaching the agent how to use the API endpoint using examples.\n2. Teaching the agent what arguments or parameters are needed to successfully call the\nAPI endpoint.\nFigure 4. Extensions connect Agents to External APIs\nExtensions can be crafted independently of the agent, but should be provided as part of the\nagent’s configuration. The agent uses the model and examples at run time to decide which\nExtension, if any, would be suitable for solving the user’s query. This highlights a key strength\nof Extensions, their built-in example types, that allow the agent to dynamically select the\nmost appropriate Extension for the task.\nFigure 5. 1-to-many relationship between Agents, Extensions and APIs\nSeptember 2024 14\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 14, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nThink of this the same way that a software developer decides which API endpoints to use\nwhile solving and solutioning for a user’s problem. If the user wants to book a flight, the\ndeveloper might use the Google Flights API. If the user wants to know where the nearest\ncoffee shop is relative to their location, the developer might use the Google Maps API. In\nthis same way, the agent / model stack uses a set of known Extensions to decide which one\nwill be the best fit for the user’s query. If you’d like to see Extensions in action, you can try\nthem out on the Gemini application by going to Settings > Extensions and then enabling any\nyou would like to test. For example, you could enable the Google Flights extension then ask\nGemini “Show me flights from Austin to Zurich leaving next Friday.”\nSample Extensions\nTo simplify the usage of Extensions, Google provides some out of the box extensions that\ncan be quickly imported into your project and used with minimal configurations. For example,\nthe Code Interpreter extension in Snippet 1 allows you to generate and run Python code from\na natural language description.\nSeptember 2024 15\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 15, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nPython\nimport vertexai\nimport pprint\nPROJECT_ID = "YOUR_PROJECT_ID"\nREGION = "us-central1"\nvertexai.init(project=PROJECT_ID, location=REGION)\nfrom vertexai.preview.extensions import Extension\nextension_code_interpreter = Extension.from_hub("code_interpreter")\nCODE_QUERY = """Write a python method to invert a binary tree in O(n) time."""\nresponse = extension_code_interpreter.execute(\noperation_id = "generate_and_execute",\noperation_params = {"query": CODE_QUERY}\n)\nprint("Generated Code:")\npprint.pprint({response[\'generated_code\']})\n# The above snippet will generate the following code.\n```\nGenerated Code:\nclass TreeNode:\ndef __init__(self, val=0, left=None, right=None):\nself.val = val\nself.left = left\nself.right = right\nContinues next page...\nSeptember 2024 16\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 16, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nPython\ndef invert_binary_tree(root):\n"""\nInverts a binary tree.\nArgs:\nroot: The root of the binary tree.\nReturns:\nThe root of the inverted binary tree.\n"""\nif not root:\nreturn None\n# Swap the left and right children recursively\nroot.left, root.right =\ninvert_binary_tree(root.right), invert_binary_tree(root.left)\nreturn root\n# Example usage:\n# Construct a sample binary tree\nroot = TreeNode(4)\nroot.left = TreeNode(2)\nroot.right = TreeNode(7)\nroot.left.left = TreeNode(1)\nroot.left.right = TreeNode(3)\nroot.right.left = TreeNode(6)\nroot.right.right = TreeNode(9)\n# Invert the binary tree\ninverted_root = invert_binary_tree(root)\n```\nSnippet 1. Code Interpreter Extension can generate and run Python code\nSeptember 2024 17\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 17, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nTo summarize, Extensions provide a way for agents to perceive, interact, and influence the\noutside world in a myriad of ways. The selection and invocation of these Extensions is guided\nby the use of Examples, all of which are defined as part of the Extension configuration.\nFunctions\nIn the world of software engineering, functions are defined as self-contained modules\nof code that accomplish a specific task and can be reused as needed. When a software\ndeveloper is writing a program, they will often create many functions to do various tasks.\nThey will also define the logic for when to call function_a versus function_b, as well as the\nexpected inputs and outputs.\nFunctions work very similarly in the world of agents, but we can replace the software\ndeveloper with a model. A model can take a set of known functions and decide when to use\neach Function and what arguments the Function needs based on its specification. Functions\ndiffer from Extensions in a few ways, most notably:\n1. A model outputs a Function and its arguments, but doesn’t make a live API call.\n2. Functions are executed on the client-side, while Extensions are executed on\nthe agent-side.\nUsing our Google Flights example again, a simple setup for functions might look like the\nexample in Figure 7.\nSeptember 2024 18\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 18, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nFigure 7. How do functions interact with external APIs?\nNote that the main difference here is that neither the Function nor the agent interact directly\nwith the Google Flights API. So how does the API call actually happen?\nWith functions, the logic and execution of calling the actual API endpoint is offloaded away\nfrom the agent and back to the client-side application as seen in Figure 8 and Figure 9 below.\nThis offers the developer more granular control over the flow of data in the application. There\nare many reasons why a Developer might choose to use functions over Extensions, but a few\ncommon use cases are:\n• API calls need to be made at another layer of the application stack, outside of the direct\nagent architecture flow (e.g. a middleware system, a front end framework, etc.)\n• Security or Authentication restrictions that prevent the agent from calling an API directly\n(e.g API is not exposed to the internet, or non-accessible by agent infrastructure)\n• Timing or order-of-operations constraints that prevent the agent from making API calls in\nreal-time. (i.e. batch operations, human-in-the-loop review, etc.)\nSeptember 2024 19\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 19, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\n• Additional data transformation logic needs to be applied to the API Response that the\nagent cannot perform. For example, consider an API endpoint that doesn’t provide a\nfiltering mechanism for limiting the number of results returned. Using Functions on the\nclient-side provides the developer additional opportunities to make these transformations.\n• The developer wants to iterate on agent development without deploying additional\ninfrastructure for the API endpoints (i.e. Function Calling can act like “stubbing” of APIs)\nWhile the difference in internal architecture between the two approaches is subtle as seen in\nFigure 8, the additional control and decoupled dependency on external infrastructure makes\nFunction Calling an appealing option for the Developer.\nFigure 8. Delineating client vs. agent side control for extensions and function calling\nSeptember 2024 20\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 20, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nUse cases\nA model can be used to invoke functions in order to handle complex, client-side execution\nflows for the end user, where the agent Developer might not want the language model to\nmanage the API execution (as is the case with Extensions). Let’s consider the following\nexample where an agent is being trained as a travel concierge to interact with users that want\nto book vacation trips. The goal is to get the agent to produce a list of cities that we can use\nin our middleware application to download images, data, etc. for the user’s trip planning. A\nuser might say something like:\nI’d like to take a ski trip with my family but I’m not sure where to go.\nIn a typical prompt to the model, the output might look like the following:\nSure, here’s a list of cities that you can consider for family ski trips:\n• Crested Butte, Colorado, USA\n• Whistler, BC, Canada\n• Zermatt, Switzerland\nWhile the above output contains the data that we need (city names), the format isn’t ideal\nfor parsing. With Function Calling, we can teach a model to format this output in a structured\nstyle (like JSON) that’s more convenient for another system to parse. Given the same input\nprompt from the user, an example JSON output from a Function might look like Snippet\n5 instead.\nSeptember 2024 21\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 21, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nUnset\nfunction_call {\nname: "display_cities"\nargs: {\n"cities": ["Crested Butte", "Whistler", "Zermatt"],\n"preferences": "skiing"\n}\n}\nSnippet 5. Sample Function Call payload for displaying a list of cities and user preferences\nThis JSON payload is generated by the model, and then sent to our Client-side server to do\nwhatever we would like to do with it. In this specific case, we’ll call the Google Places API to\ntake the cities provided by the model and look up Images, then provide them as formatted\nrich content back to our User. Consider this sequence diagram in Figure 9 showing the above\ninteraction in step by step detail.\nSeptember 2024 22\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 22, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content="Agents\nFigure 9. Sequence diagram showing the lifecycle of a Function Call\nThe result of the example in Figure 9 is that the model is leveraged to “fill in the blanks” with\nthe parameters required for the Client side UI to make the call to the Google Places API. The\nClient side UI manages the actual API call using the parameters provided by the model in the\nreturned Function. This is just one use case for Function Calling, but there are many other\nscenarios to consider like:\n• You want a language model to suggest a function that you can use in your code, but you\ndon't want to include credentials in your code. Because function calling doesn't run the\nfunction, you don't need to include credentials in your code with the function information.\nSeptember 2024 23\n"),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 23, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content="Agents\n• You are running asynchronous operations that can take more than a few seconds. These\nscenarios work well with function calling because it's an asynchronous operation.\n• You want to run functions on a device that's different from the system producing the\nfunction calls and their arguments.\nOne key thing to remember about functions is that they are meant to offer the developer\nmuch more control over not only the execution of API calls, but also the entire flow of data\nin the application as a whole. In the example in Figure 9, the developer chose to not return\nAPI information back to the agent as it was not pertinent for future actions the agent might\ntake. However, based on the architecture of the application, it may make sense to return the\nexternal API call data to the agent in order to influence future reasoning, logic, and action\nchoices. Ultimately, it is up to the application developer to choose what is right for the\nspecific application.\nFunction sample code\nTo achieve the above output from our ski vacation scenario, let’s build out each of the\ncomponents to make this work with our gemini-1.5-flash-001 model.\nFirst, we’ll define our display_cities function as a simple Python method.\nSeptember 2024 24\n"),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 24, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nPython\ndef display_cities(cities: list[str], preferences: Optional[str] = None):\n"""Provides a list of cities based on the user\'s search query and preferences.\nArgs:\npreferences (str): The user\'s preferences for the search, like skiing,\nbeach, restaurants, bbq, etc.\ncities (list[str]): The list of cities being recommended to the user.\nReturns:\nlist[str]: The list of cities being recommended to the user.\n"""\nreturn cities\nSnippet 6. Sample python method for a function that will display a list of cities.\nNext, we’ll instantiate our model, build the Tool, then pass in our user’s query and tools to\nthe model. Executing the code below would result in the output as seen at the bottom of the\ncode snippet.\nSeptember 2024 25\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 25, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nPython\nfrom vertexai.generative_models import GenerativeModel, Tool, FunctionDeclaration\nmodel = GenerativeModel("gemini-1.5-flash-001")\ndisplay_cities_function = FunctionDeclaration.from_func(display_cities)\ntool = Tool(function_declarations=[display_cities_function])\nmessage = "I’d like to take a ski trip with my family but I’m not sure where\nto go."\nres = model.generate_content(message, tools=[tool])\nprint(f"Function Name: {res.candidates[0].content.parts[0].function_call.name}")\nprint(f"Function Args: {res.candidates[0].content.parts[0].function_call.args}")\n> Function Name: display_cities\n> Function Args: {\'preferences\': \'skiing\', \'cities\': [\'Aspen\', \'Vail\',\n\'Park City\']}\nSnippet 7. Building a Tool, sending to the model with a user query and allowing the function call to take place\nIn summary, functions offer a straightforward framework that empowers application\ndevelopers with fine-grained control over data flow and system execution, while effectively\nleveraging the agent/model for critical input generation. Developers can selectively choose\nwhether to keep the agent “in the loop” by returning external data, or omit it based on\nspecific application architecture requirements.\nSeptember 2024 26\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 26, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nData stores\nImagine a language model as a vast library of books, containing its training data. But unlike\na library that continuously acquires new volumes, this one remains static, holding only the\nknowledge it was initially trained on. This presents a challenge, as real-world knowledge is\nconstantly evolving. Data Stores address this limitation by providing access to more dynamic\nand up-to-date information, and ensuring a model’s responses remain grounded in factuality\nand relevance.\nConsider a common scenario where a developer might need to provide a small amount of\nadditional data to a model, perhaps in the form of spreadsheets or PDFs.\nFigure 10. How can Agents interact with structured and unstructured data?\nSeptember 2024 27\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 27, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nData Stores allow developers to provide additional data in its original format to an agent,\neliminating the need for time-consuming data transformations, model retraining, or fine-\ntuning. The Data Store converts the incoming document into a set of vector database\nembeddings that the agent can use to extract the information it needs to supplement its next\naction or response to the user.\nFigure 11. Data Stores connect Agents to new real-time data sources of various types.\nImplementation and application\nIn the context of Generative AI agents, Data Stores are typically implemented as a vector\ndatabase that the developer wants the agent to have access to at runtime. While we won’t\ncover vector databases in depth here, the key point to understand is that they store data\nin the form of vector embeddings, a type of high-dimensional vector or mathematical\nrepresentation of the data provided. One of the most prolific examples of Data Store usage\nwith language models in recent times has been the implementation of Retrieval Augmented\nSeptember 2024 28\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 28, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nGeneration (RAG) based applications. These applications seek to extend the breadth and\ndepth of a model’s knowledge beyond the foundational training data by giving the model\naccess to data in various formats like:\n• Website content\n• Structured Data in formats like PDF, Word Docs, CSV, Spreadsheets, etc.\n• Unstructured Data in formats like HTML, PDF, TXT, etc.\nFigure 12. 1-to-many relationship between agents and data stores, which can represent various types of\npre-indexed data\nThe underlying process for each user request and agent response loop is generally modeled\nas seen in Figure 13.\n1. A user query is sent to an embedding model to generate embeddings for the query\n2. The query embeddings are then matched against the contents of the vector database\nusing a matching algorithm like SCaNN\n3. The matched content is retrieved from the vector database in text format and sent back to\nthe agent\n4. The agent receives both the user query and retrieved content, then formulates a response\nor action\nSeptember 2024 29\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 29, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\n5. A final response is sent to the user\nFigure 13. The lifecycle of a user request and agent response in a RAG based application\nThe end result is an application that allows the agent to match a user’s query to a known data\nstore through vector search, retrieve the original content, and provide it to the orchestration\nlayer and model for further processing. The next action might be to provide a final answer to\nthe user, or perform an additional vector search to further refine the results.\nA sample interaction with an agent that implements RAG with ReAct reasoning/planning can\nbe seen in Figure 14.\nSeptember 2024 30\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 30, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nFigure 14. Sample RAG based application w/ ReAct reasoning/planning\nSeptember 2024 31\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 31, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nTools recap\nTo summarize, extensions, functions and data stores make up a few different tool types\navailable for agents to use at runtime. Each has their own purpose and they can be used\ntogether or independently at the discretion of the agent developer.\nExtensions Function Calling Data Stores\nExecution Agent-Side Execution Client-Side Execution Agent-Side Execution\nUse Case • Developer wants • Security or Developer wants to\nagent to control Authentication implement Retrieval\ninteractions with the restrictions prevent the Augmented Generation\nAPI endpoints agent from calling an (RAG) with any of the\nAPI directly following data types:\n• Useful when\nleveraging native pre- • Timing constraints or • Website Content from\nbuilt Extensions (i.e. order-of-operations pre-indexed domains\nVertex Search, Code constraints that and URLs\nInterpreter, etc.) prevent the agent\n• Structured Data in\nfrom making API calls\n• Multi-hop planning formats like PDF,\nin real-time. (i.e. batch\nand API calling Word Docs, CSV,\noperations, human-in-\n(i.e. the next agent Spreadsheets, etc.\nthe-loop review, etc.)\naction depends on\n• Relational / Non-\nthe outputs of the • API that is not exposed\nRelational Databases\nprevious action / to the internet, or\nAPI call) non-accessible by • Unstructured Data in\nGoogle systems formats like HTML, PDF,\nTXT, etc.\nSeptember 2024 32\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 32, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content="Agents\nEnhancing model performance with\ntargeted learning\nA crucial aspect of using models effectively is their ability to choose the right tools when\ngenerating output, especially when using tools at scale in production. While general training\nhelps models develop this skill, real-world scenarios often require knowledge beyond the\ntraining data. Imagine this as the difference between basic cooking skills and mastering\na specific cuisine. Both require foundational cooking knowledge, but the latter demands\ntargeted learning for more nuanced results.\nTo help the model gain access to this type of specific knowledge, several approaches exist:\n• In-context learning: This method provides a generalized model with a prompt, tools, and\nfew-shot examples at inference time which allows it to learn ‘on the fly' how and when to\nuse those tools for a specific task. The ReAct framework is an example of this approach in\nnatural language.\n• Retrieval-based in-context learning: This technique dynamically populates the model\nprompt with the most relevant information, tools, and associated examples by retrieving\nthem from external memory. An example of this would be the ‘Example Store’ in Vertex AI\nextensions or the data stores RAG based architecture mentioned previously.\n• Fine-tuning based learning: This method involves training a model using a larger dataset\nof specific examples prior to inference. This helps the model understand when and how to\napply certain tools prior to receiving any user queries.\nTo provide additional insights on each of the targeted learning approaches, let’s revisit our\ncooking analogy.\nSeptember 2024 33\n"),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 33, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\n• Imagine a chef has received a specific recipe (the prompt), a few key ingredients (relevant\ntools) and some example dishes (few-shot examples) from a customer. Based on this\nlimited information and the chef’s general knowledge of cooking, they will need to figure\nout how to prepare the dish ‘on the fly’ that most closely aligns with the recipe and the\ncustomer’s preferences. This is in-context learning.\n• Now let’s imagine our chef in a kitchen that has a well-stocked pantry (external data\nstores) filled with various ingredients and cookbooks (examples and tools). The chef is now\nable to dynamically choose ingredients and cookbooks from the pantry and better align\nto the customer’s recipe and preferences. This allows the chef to create a more informed\nand refined dish leveraging both existing and new knowledge. This is retrieval-based\nin-context learning.\n• Finally, let’s imagine that we sent our chef back to school to learn a new cuisine or set of\ncuisines (pre-training on a larger dataset of specific examples). This allows the chef to\napproach future unseen customer recipes with deeper understanding. This approach is\nperfect if we want the chef to excel in specific cuisines (knowledge domains). This is fine-\ntuning based learning.\nEach of these approaches offers unique advantages and disadvantages in terms of speed,\ncost, and latency. However, by combining these techniques in an agent framework, we can\nleverage the various strengths and minimize their weaknesses, allowing for a more robust and\nadaptable solution.\nSeptember 2024 34\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 34, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nAgent quick start with LangChain\nIn order to provide a real-world executable example of an agent in action, we’ll build a quick\nprototype with the LangChain and LangGraph libraries. These popular open source libraries\nallow users to build customer agents by “chaining” together sequences of logic, reasoning,\nand tool calls to answer a user’s query. We’ll use our gemini-1.5-flash-001 model and\nsome simple tools to answer a multi-stage query from the user as seen in Snippet 8.\nThe tools we are using are the SerpAPI (for Google Search) and the Google Places API. After\nexecuting our program in Snippet 8, you can see the sample output in Snippet 9.\nSeptember 2024 35\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 35, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nPython\nfrom langgraph.prebuilt import create_react_agent\nfrom langchain_core.tools import tool\nfrom langchain_community.utilities import SerpAPIWrapper\nfrom langchain_community.tools import GooglePlacesTool\nos.environ["SERPAPI_API_KEY"] = "XXXXX"\nos.environ["GPLACES_API_KEY"] = "XXXXX"\n@tool\ndef search(query: str):\n"""Use the SerpAPI to run a Google Search."""\nsearch = SerpAPIWrapper()\nreturn search.run(query)\n@tool\ndef places(query: str):\n"""Use the Google Places API to run a Google Places Query."""\nplaces = GooglePlacesTool()\nreturn places.run(query)\nmodel = ChatVertexAI(model="gemini-1.5-flash-001")\ntools = [search, places]\nquery = "Who did the Texas Longhorns play in football last week? What is the\naddress of the other team\'s stadium?"\nagent = create_react_agent(model, tools)\ninput = {"messages": [("human", query)]}\nfor s in agent.stream(input, stream_mode="values"):\nmessage = s["messages"][-1]\nif isinstance(message, tuple):\nprint(message)\nelse:\nmessage.pretty_print()\nSnippet 8. Sample LangChain and LangGraph based agent with tools\nSeptember 2024 36\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 36, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nUnset\n=============================== Human Message ================================\nWho did the Texas Longhorns play in football last week? What is the address\nof the other team\'s stadium?\n================================= Ai Message =================================\nTool Calls: search\nArgs:\nquery: Texas Longhorns football schedule\n================================ Tool Message ================================\nName: search\n{...Results: "NCAA Division I Football, Georgia, Date..."}\n================================= Ai Message =================================\nThe Texas Longhorns played the Georgia Bulldogs last week.\nTool Calls: places\nArgs:\nquery: Georgia Bulldogs stadium\n================================ Tool Message ================================\nName: places\n{...Sanford Stadium Address: 100 Sanford...}\n================================= Ai Message =================================\nThe address of the Georgia Bulldogs stadium is 100 Sanford Dr, Athens, GA\n30602, USA.\nSnippet 9. Output from our program in Snippet 8\nWhile this is a fairly simple agent example, it demonstrates the foundational components\nof Model, Orchestration, and tools all working together to achieve a specific goal. In the\nfinal section, we’ll explore how these components come together in Google-scale managed\nproducts like Vertex AI agents and Generative Playbooks.\nSeptember 2024 37\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 37, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nProduction applications with Vertex\nAI agents\nWhile this whitepaper explored the core components of agents, building production-grade\napplications requires integrating them with additional tools like user interfaces, evaluation\nframeworks, and continuous improvement mechanisms. Google’s Vertex AI platform\nsimplifies this process by offering a fully managed environment with all the fundamental\nelements covered earlier. Using a natural language interface, developers can rapidly\ndefine crucial elements of their agents - goals, task instructions, tools, sub-agents for task\ndelegation, and examples - to easily construct the desired system behavior. In addition, the\nplatform comes with a set of development tools that allow for testing, evaluation, measuring\nagent performance, debugging, and improving the overall quality of developed agents. This\nallows developers to focus on building and refining their agents while the complexities of\ninfrastructure, deployment and maintenance are managed by the platform itself.\nIn Figure 15 we’ve provided a sample architecture of an agent that was built on the Vertex\nAI platform using various features such as Vertex Agent Builder, Vertex Extensions, Vertex\nFunction Calling and Vertex Example Store to name a few. The architecture includes many of\nthe various components necessary for a production ready application.\nSeptember 2024 38\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 38, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nFigure 15. Sample end-to-end agent architecture built on Vertex AI platform\nYou can try a sample of this prebuilt agent architecture from our official documentation.\nSeptember 2024 39\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 39, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\nSummary\nIn this whitepaper we’ve discussed the foundational building blocks of Generative AI\nagents, their compositions, and effective ways to implement them in the form of cognitive\narchitectures. Some key takeaways from this whitepaper include:\n1. Agents extend the capabilities of language models by leveraging tools to access real-\ntime information, suggest real-world actions, and plan and execute complex tasks\nautonomously. agents can leverage one or more language models to decide when and\nhow to transition through states and use external tools to complete any number of\ncomplex tasks that would be difficult or impossible for the model to complete on its own.\n2. At the heart of an agent’s operation is the orchestration layer, a cognitive architecture that\nstructures reasoning, planning, decision-making and guides its actions. Various reasoning\ntechniques such as ReAct, Chain-of-Thought, and Tree-of-Thoughts, provide a framework\nfor the orchestration layer to take in information, perform internal reasoning, and generate\ninformed decisions or responses.\n3. Tools, such as Extensions, Functions, and Data Stores, serve as the keys to the outside\nworld for agents, allowing them to interact with external systems and access knowledge\nbeyond their training data. Extensions provide a bridge between agents and external APIs,\nenabling the execution of API calls and retrieval of real-time information. functions provide\na more nuanced control for the developer through the division of labor, allowing agents\nto generate Function parameters which can be executed client-side. Data Stores provide\nagents with access to structured or unstructured data, enabling data-driven applications.\nThe future of agents holds exciting advancements and we’ve only begun to scratch the\nsurface of what is possible. As tools become more sophisticated and reasoning capabilities\nare enhanced, agents will be empowered to solve increasingly complex problems.\nFurthermore, the strategic approach of ‘agent chaining’ will continue to gain momentum. By\nSeptember 2024 40\n'),
     Document(metadata={'source': 'data/Newwhitepaper_Agents2.pdf', 'file_path': 'data/Newwhitepaper_Agents2.pdf', 'page': 40, 'total_pages': 42, 'CreationDate': "D:20241113100853-07'00'", 'Creator': 'Adobe InDesign 20.0 (Macintosh)', 'ModDate': "D:20241113100858-07'00'", 'Producer': 'Adobe PDF Library 17.0', 'Trapped': 'False', 'filename': 'data/Newwhitepaper_Agents2.pdf'}, page_content='Agents\ncombining specialized agents - each excelling in a particular domain or task - we can create\na ‘mixture of agent experts’ approach, capable of delivering exceptional results across\nvarious industries and problem areas.\nIt’s important to remember that building complex agent architectures demands an iterative\napproach. Experimentation and refinement are key to finding solutions for specific business\ncases and organizational needs. No two agents are created alike due to the generative nature\nof the foundational models that underpin their architecture. However, by harnessing the\nstrengths of each of these foundational components, we can create impactful applications\nthat extend the capabilities of language models and drive real-world value.\nSeptember 2024 41\n')]</pre>



## Dataset Generation
We'll create datasets using ChatOpenAI. Before writing the code, let's define the roles of our objects:
- Dataset Generator: `generator_llm`
- Document Embeddings: `embeddings`

```python
from ragas.llms import LangchainLLMWrapper
from ragas.embeddings import LangchainEmbeddingsWrapper
from langchain_openai import ChatOpenAI
from ragas.testset.graph import KnowledgeGraph
from ragas.testset.graph import Node, NodeType
from ragas.embeddings.base import embedding_factory

# Dataset Generator
generator_llm = LangchainLLMWrapper(ChatOpenAI(model="gpt-4o"))

# Document Embeddings
embeddings = embedding_factory()
```

First, let's initialize the DocumentStore. We'll configure it to use custom LLM and `embeddings`.

```python
# Wrap LangChain's ChatOpenAI model with LangchainLLMWrapper to make it compatible with Ragas
langchain_llm = LangchainLLMWrapper(ChatOpenAI(model="gpt-4o"))

# Create ragas_embeddings
ragas_embeddings = LangchainEmbeddingsWrapper(embeddings)

# Create a KnowledgeGraph object
kg = KnowledgeGraph()

for doc in docs:
   kg.nodes.append(
       Node(
           type=NodeType.DOCUMENT,
           properties={
               "page_content": doc.page_content,
               "document_metadata": doc.metadata
           }
       )
   )
```

### Self Check
Let's check the total number of nodes in the knowledge graph.

```python
print(len(generator.knowledge_graph.nodes))
```

Run this code to verify if knowledge graph nodes have been created. If no nodes were created, there may be issues with executing subsequent code.

```python
for node in generator.knowledge_graph.nodes:
    print(node.properties)
```

```python
# check relationships
print("Total number of nodes:", len(kg.nodes))
print("Total number of relationships:", len(kg.relationships))
```

<pre class="custom">Total number of nodes: 38
    Total number of relationships: 0
</pre>

Now we will establish relationships between nodes in the knowledge graph.

### `Extractor`
The extracted information is used to establish the relationship between the nodes. Before generating relationships between nodes, we will first examine only the three main `extractors`.
1. `KeyphrasesExtractor`
2. `SummaryExtractor`
3. `HeadlinesExtractor`

First, I will import all the necessary modules.

```python
from ragas.testset.transforms.extractors import (
    KeyphrasesExtractor,
    SummaryExtractor,
    HeadlinesExtractor
)

from ragas.testset.transforms import (
    OverlapScoreBuilder
)
```

**1. `KeyphrasesExtractor`**

```python
# [1] Initial version (before refactoring)
# Please run this first to see the original implementation.
keyphrase_extractor = KeyphrasesExtractor()
output = [await keyphrase_extractor.extract(node) for node in kg.nodes]
_ = [node.properties.update({key:val}) for (key,val), node in zip(output, kg.nodes)]
kg.nodes[0].properties
```




<pre class="custom">{'page_content': "Agents\nThis combination of reasoning,\nlogic, and access to external\ninformation that are all connected\nto a Generative AI model invokes\nthe concept of an agent.\nIntroduction\nHumans are fantastic at messy pattern recognition tasks. However, they often rely on tools\n- like books, Google Search, or a calculator - to supplement their prior knowledge before\narriving at a conclusion. Just like humans, Generative AI models can be trained to use tools\nto access real-time information or suggest a real-world action. For example, a model can\nleverage a database retrieval tool to access specific information, like a customer's purchase\nhistory, so it can generate tailored shopping recommendations. Alternatively, based on a\nuser's query, a model can make various API calls to send an email response to a colleague\nor complete a financial transaction on your behalf. To do so, the model must not only have\naccess to a set of external tools, it needs the ability to plan and execute any task in a self-\ndirected fashion. This combination of reasoning, logic, and access to external information\nthat are all connected to a Generative AI model invokes the concept of an agent, or a\nprogram that extends beyond the standalone capabilities of a Generative AI model. This\nwhitepaper dives into all these and associated aspects in more detail.\nSeptember 2024 4\n",
     'document_metadata': {'source': 'data/Newwhitepaper_Agents2.pdf',
      'file_path': 'data/Newwhitepaper_Agents2.pdf',
      'page': 3,
      'total_pages': 42,
      'CreationDate': "D:20241113100853-07'00'",
      'Creator': 'Adobe InDesign 20.0 (Macintosh)',
      'ModDate': "D:20241113100858-07'00'",
      'Producer': 'Adobe PDF Library 17.0',
      'Trapped': 'False',
      'filename': 'data/Newwhitepaper_Agents2.pdf'},
     'keyphrases': ['Generative AI model',
      'reasoning',
      'external information',
      'tailored shopping recommendations',
      'self-directed fashion']}</pre>



```python
# [2] Optimized version (faster execution)
# Comment out the code above and run this version instead to compare execution times.
import asyncio
from multiprocessing.pool import ThreadPool

async def process_batch(batch):
    keyphrase_extractor = KeyphrasesExtractor()
    batch_output = await asyncio.gather(*[keyphrase_extractor.extract(node) for node in batch])
    return batch_output

def process_batch_in_thread(batch):
    return asyncio.run(process_batch(batch))

def process_with_thread_and_async(nodes, batch_size=5, num_threads=4):
    batches = [nodes[i:i + batch_size] for i in range(0, len(nodes), batch_size)]
    
    with ThreadPool(processes=num_threads) as pool:
        all_outputs = pool.map(process_batch_in_thread, batches)
    
    outputs = []
    for batch_output in all_outputs:
        outputs.extend(batch_output)
    
    _ = [node.properties.update({key:val}) for (key,val), node in zip(outputs, nodes)]
    
    return nodes[0].properties

_ = process_with_thread_and_async(kg.nodes)
kg.nodes[0].properties
```




<pre class="custom">{'page_content': "Agents\nThis combination of reasoning,\nlogic, and access to external\ninformation that are all connected\nto a Generative AI model invokes\nthe concept of an agent.\nIntroduction\nHumans are fantastic at messy pattern recognition tasks. However, they often rely on tools\n- like books, Google Search, or a calculator - to supplement their prior knowledge before\narriving at a conclusion. Just like humans, Generative AI models can be trained to use tools\nto access real-time information or suggest a real-world action. For example, a model can\nleverage a database retrieval tool to access specific information, like a customer's purchase\nhistory, so it can generate tailored shopping recommendations. Alternatively, based on a\nuser's query, a model can make various API calls to send an email response to a colleague\nor complete a financial transaction on your behalf. To do so, the model must not only have\naccess to a set of external tools, it needs the ability to plan and execute any task in a self-\ndirected fashion. This combination of reasoning, logic, and access to external information\nthat are all connected to a Generative AI model invokes the concept of an agent, or a\nprogram that extends beyond the standalone capabilities of a Generative AI model. This\nwhitepaper dives into all these and associated aspects in more detail.\nSeptember 2024 4\n",
     'document_metadata': {'source': 'data/Newwhitepaper_Agents2.pdf',
      'file_path': 'data/Newwhitepaper_Agents2.pdf',
      'page': 3,
      'total_pages': 42,
      'CreationDate': "D:20241113100853-07'00'",
      'Creator': 'Adobe InDesign 20.0 (Macintosh)',
      'ModDate': "D:20241113100858-07'00'",
      'Producer': 'Adobe PDF Library 17.0',
      'Trapped': 'False',
      'filename': 'data/Newwhitepaper_Agents2.pdf'},
     'keyphrases': ['Generative AI model',
      'reasoning',
      'external information',
      'tailored shopping recommendations',
      'self-directed fashion']}</pre>




---

**[Note] Refactoring for Performance Improvement!**

In the bonus section of this tutorial, we optimized the code to significantly reduce execution time—from **45 seconds to 1 minute** down to just **3-8 seconds**!  
If you're familiar with **parallel processing** and **asynchronous processing**, you can combine these techniques to further enhance performance.  
We used the `asyncio` module for asynchronous processing and the `multiprocessing` module for parallel processing.  
(**Tested on an M1 CPU with 4 performance cores and 4 efficiency cores.**)

Check out the details in the [Bonus Refactoring Section](#summary-moving-forward-with-generated-and-prepared-datasets)!

---

**2. `SummaryExtractor`**

```python
summary_extractor = SummaryExtractor()
output = [await summary_extractor.extract(node) for node in kg.nodes]
_ = [node.properties.update({key:val}) for (key, val), node in zip(output, kg.nodes)]
kg.nodes[0].properties
```




<pre class="custom">{'page_content': "Agents\nThis combination of reasoning,\nlogic, and access to external\ninformation that are all connected\nto a Generative AI model invokes\nthe concept of an agent.\nIntroduction\nHumans are fantastic at messy pattern recognition tasks. However, they often rely on tools\n- like books, Google Search, or a calculator - to supplement their prior knowledge before\narriving at a conclusion. Just like humans, Generative AI models can be trained to use tools\nto access real-time information or suggest a real-world action. For example, a model can\nleverage a database retrieval tool to access specific information, like a customer's purchase\nhistory, so it can generate tailored shopping recommendations. Alternatively, based on a\nuser's query, a model can make various API calls to send an email response to a colleague\nor complete a financial transaction on your behalf. To do so, the model must not only have\naccess to a set of external tools, it needs the ability to plan and execute any task in a self-\ndirected fashion. This combination of reasoning, logic, and access to external information\nthat are all connected to a Generative AI model invokes the concept of an agent, or a\nprogram that extends beyond the standalone capabilities of a Generative AI model. This\nwhitepaper dives into all these and associated aspects in more detail.\nSeptember 2024 4\n",
     'document_metadata': {'source': 'data/Newwhitepaper_Agents2.pdf',
      'file_path': 'data/Newwhitepaper_Agents2.pdf',
      'page': 3,
      'total_pages': 42,
      'CreationDate': "D:20241113100853-07'00'",
      'Creator': 'Adobe InDesign 20.0 (Macintosh)',
      'ModDate': "D:20241113100858-07'00'",
      'Producer': 'Adobe PDF Library 17.0',
      'Trapped': 'False',
      'filename': 'data/Newwhitepaper_Agents2.pdf'},
     'keyphrases': ['Generative AI model',
      'reasoning',
      'external information',
      'tailored shopping recommendations',
      'self-directed fashion'],
     'summary': 'Generative AI models can function as agents by combining reasoning, logic, and access to external information. They can utilize tools like databases and APIs to perform tasks such as generating tailored recommendations or executing transactions. This capability allows them to operate in a self-directed manner, extending beyond their standalone functions. The whitepaper explores these concepts and their implications in detail.'}</pre>





---

[Note] **Refactoring for Performance Improvement!** 

You can refactor it like the Summary Keyphrases Extractor.

Check out the details in the [Bonus Refactoring Section](#summary-moving-forward-with-generated-and-prepared-datasets)!

---

**3. `HeadlinesExtractor`**

```python
headline_extractor = HeadlinesExtractor()
output = [await headline_extractor.extract(node) for node in kg.nodes]
_ = [node.properties.update({key:val}) for (key,val), node in zip(output, kg.nodes)]
kg.nodes[0].properties
```




<pre class="custom">{'page_content': "Agents\nThis combination of reasoning,\nlogic, and access to external\ninformation that are all connected\nto a Generative AI model invokes\nthe concept of an agent.\nIntroduction\nHumans are fantastic at messy pattern recognition tasks. However, they often rely on tools\n- like books, Google Search, or a calculator - to supplement their prior knowledge before\narriving at a conclusion. Just like humans, Generative AI models can be trained to use tools\nto access real-time information or suggest a real-world action. For example, a model can\nleverage a database retrieval tool to access specific information, like a customer's purchase\nhistory, so it can generate tailored shopping recommendations. Alternatively, based on a\nuser's query, a model can make various API calls to send an email response to a colleague\nor complete a financial transaction on your behalf. To do so, the model must not only have\naccess to a set of external tools, it needs the ability to plan and execute any task in a self-\ndirected fashion. This combination of reasoning, logic, and access to external information\nthat are all connected to a Generative AI model invokes the concept of an agent, or a\nprogram that extends beyond the standalone capabilities of a Generative AI model. This\nwhitepaper dives into all these and associated aspects in more detail.\nSeptember 2024 4\n",
     'document_metadata': {'source': 'data/Newwhitepaper_Agents2.pdf',
      'file_path': 'data/Newwhitepaper_Agents2.pdf',
      'page': 3,
      'total_pages': 42,
      'CreationDate': "D:20241113100853-07'00'",
      'Creator': 'Adobe InDesign 20.0 (Macintosh)',
      'ModDate': "D:20241113100858-07'00'",
      'Producer': 'Adobe PDF Library 17.0',
      'Trapped': 'False',
      'filename': 'data/Newwhitepaper_Agents2.pdf'},
     'keyphrases': ['Generative AI model',
      'reasoning',
      'external information',
      'tailored shopping recommendations',
      'self-directed fashion'],
     'summary': 'Generative AI models can function as agents by combining reasoning, logic, and access to external information. They can utilize tools like databases and APIs to perform tasks such as generating tailored recommendations or executing transactions. This capability allows them to operate in a self-directed manner, extending beyond their standalone functions. The whitepaper explores these concepts and their implications in detail.',
     'headlines': ['Agents', 'Introduction']}</pre>




---

[Note] **Refactoring for Performance Improvement!**

You can refactor it like the Headlines Extractor.

Check out the details in the [Bonus Refactoring Section](#summary-moving-forward-with-generated-and-prepared-datasets)!

---

### Relationship builder
We will define relationships using the extracted information from earlier.
In the case of technology documents, the relationship can be established between the nodes based on the entities present in the nodes. 

Since both the nodes have the same entities, the relationship is established between the nodes based on the entity similarity. let's take a look at this.

```python
print(kg.nodes[0].properties['keyphrases'])
print(kg.nodes[1].properties['keyphrases'])
print(kg.nodes[2].properties['keyphrases'])
print(kg.nodes[3].properties['keyphrases'])
print(kg.nodes[4].properties['keyphrases'])
```

<pre class="custom">['Generative AI model', 'reasoning', 'external information', 'tailored shopping recommendations', 'self-directed fashion']
    ['Generative AI agent', 'autonomous agents', 'cognitive architecture', 'goal achievement', 'decision making']
    ['agent architecture', 'language model', 'instruction based reasoning', 'cognitive architecture', 'specific tools']
    ['foundational models', 'tools', 'agents', 'orchestration layer', 'retrieval augmented generation']
    ['agents vs. models', 'knowledge is limited', 'multi turn inference', 'native cognitive architecture', 'planning execution adjustment']
</pre>

Relationships can be formed using the builder.

```python
%pip install -qU rapidfuzz
```

<pre class="custom">Note: you may need to restart the kernel to use updated packages.
</pre>

```python
from ragas.testset.transforms import apply_transforms

relation_builder = OverlapScoreBuilder(
    property_name="keyphrases",
    new_property_name="overlap_score",
)

transforms = [
    keyphrase_extractor,
    relation_builder
]

apply_transforms(kg,transforms)
```

<pre class="custom">Applying KeyphrasesExtractor:   0%|          | 0/38 [00:00<?, ?it/s]Property 'keyphrases' already exists in node '7a5209'. Skipping!
    Applying KeyphrasesExtractor:   3%|▎         | 1/38 [00:01<00:59,  1.60s/it]Property 'keyphrases' already exists in node '39e9bb'. Skipping!
    Property 'keyphrases' already exists in node '3a4217'. Skipping!
    Property 'keyphrases' already exists in node '3fe9c8'. Skipping!
    Property 'keyphrases' already exists in node '4c2d0a'. Skipping!
    Property 'keyphrases' already exists in node 'f48395'. Skipping!
    Applying KeyphrasesExtractor:  16%|█▌        | 6/38 [00:01<00:06,  4.60it/s]Property 'keyphrases' already exists in node '939cf8'. Skipping!
    Property 'keyphrases' already exists in node '399e66'. Skipping!
    Property 'keyphrases' already exists in node 'e9062c'. Skipping!
    Applying KeyphrasesExtractor:  24%|██▎       | 9/38 [00:02<00:05,  4.90it/s]Property 'keyphrases' already exists in node 'f2153b'. Skipping!
    Property 'keyphrases' already exists in node '5601ec'. Skipping!
    Property 'keyphrases' already exists in node '99c2d9'. Skipping!
    Property 'keyphrases' already exists in node 'c38097'. Skipping!
    Applying KeyphrasesExtractor:  34%|███▍      | 13/38 [00:02<00:03,  8.08it/s]Property 'keyphrases' already exists in node '447508'. Skipping!
    Property 'keyphrases' already exists in node '801029'. Skipping!
    Property 'keyphrases' already exists in node 'd5db0d'. Skipping!
    Property 'keyphrases' already exists in node 'ed9fad'. Skipping!
    Applying KeyphrasesExtractor:  45%|████▍     | 17/38 [00:02<00:02,  7.61it/s]Property 'keyphrases' already exists in node '6d52b0'. Skipping!
    Property 'keyphrases' already exists in node '66343a'. Skipping!
    Property 'keyphrases' already exists in node '3e9a86'. Skipping!
    Property 'keyphrases' already exists in node '387b03'. Skipping!
    Property 'keyphrases' already exists in node 'de6d01'. Skipping!
    Applying KeyphrasesExtractor:  58%|█████▊    | 22/38 [00:03<00:01, 10.91it/s]Property 'keyphrases' already exists in node '95153c'. Skipping!
    Property 'keyphrases' already exists in node '20c2ac'. Skipping!
    Property 'keyphrases' already exists in node '3ea0f0'. Skipping!
    Applying KeyphrasesExtractor:  66%|██████▌   | 25/38 [00:03<00:01,  9.51it/s]Property 'keyphrases' already exists in node '61f9f3'. Skipping!
    Property 'keyphrases' already exists in node 'ff329d'. Skipping!
    Applying KeyphrasesExtractor:  71%|███████   | 27/38 [00:03<00:01, 10.27it/s]Property 'keyphrases' already exists in node '3d521d'. Skipping!
    Property 'keyphrases' already exists in node '6072f2'. Skipping!
    Applying KeyphrasesExtractor:  76%|███████▋  | 29/38 [00:03<00:00,  9.46it/s]Property 'keyphrases' already exists in node 'd06196'. Skipping!
    Property 'keyphrases' already exists in node 'db3d6b'. Skipping!
    Applying KeyphrasesExtractor:  82%|████████▏ | 31/38 [00:04<00:00, 10.55it/s]Property 'keyphrases' already exists in node '519899'. Skipping!
    Property 'keyphrases' already exists in node '408524'. Skipping!
    Applying KeyphrasesExtractor:  87%|████████▋ | 33/38 [00:04<00:00, 10.56it/s]Property 'keyphrases' already exists in node '567957'. Skipping!
    Property 'keyphrases' already exists in node '67a3a3'. Skipping!
    Applying KeyphrasesExtractor:  92%|█████████▏| 35/38 [00:04<00:00, 11.78it/s]Property 'keyphrases' already exists in node '1a2be1'. Skipping!
    Property 'keyphrases' already exists in node 'eec414'. Skipping!
    Applying KeyphrasesExtractor:  97%|█████████▋| 37/38 [00:04<00:00, 11.54it/s]Property 'keyphrases' already exists in node '722e72'. Skipping!
                                                                                 </pre>

```python
from ragas.testset import TestsetGenerator
clusters = kg.find_indirect_clusters()
generator = TestsetGenerator(
    llm=generator_llm,
    embedding_model=ragas_embeddings,
    knowledge_graph=kg, # the graph with newly created relationships will be entered.
)
```

```python
# check relationships
print("Total number of nodes:", len(kg.nodes))
print("Total number of relationships:", len(kg.relationships))
```

<pre class="custom">Total number of nodes: 38
    Total number of relationships: 51
</pre>

## Distribution of Question Types
Before we begin generating questions, let's first define the distribution (frequency) of questions by type. Using the **`SingleHopSpecificQuerySynthesizer`** , **`MultiHopAbstractQuerySynthesizer`** , **`MultiHopSpecificQuerySynthesizer`**  and **`MultiHopQuerySynthesizer`** , we aim to create a test set with the following distribution of question types:

- `simple`: Basic questions (40%) ㅡ **`SingleHopSpecificQuerySynthesizer`**
- `reasoning`: Questions requiring reasoning (20%) ㅡ **`MultiHopAbstractQuerySynthesizer`** 
- `multi_context`: Questions requiring consideration of multiple contexts (20%) ㅡ **`MultiHopSpecificQuerySynthesizer`** 
- `conditional`: Conditional questions (20%) ㅡ **`MultiHopQuerySynthesizer`** 

### Role of the synthesizers Module
The synthesizers module in `Ragas` is a core module responsible for Query Synthesis. It provides functionality to generate various types of questions based on documents stored in the `Knowledge Graph`. This module is used to automatically generate test sets for evaluating RAG (Retrieval-Augmented Generation) systems.

```python
from ragas.testset.synthesizers.multi_hop import (
    MultiHopAbstractQuerySynthesizer,
    MultiHopSpecificQuerySynthesizer,
)
from ragas.testset.synthesizers.single_hop.specific import (
    SingleHopSpecificQuerySynthesizer,
)
from ragas.testset.synthesizers.multi_hop.base import (
    MultiHopQuerySynthesizer,
)
from ragas.testset.synthesizers.base import BaseSynthesizer
```

```python
from dataclasses import dataclass
import typing as t
from ragas.testset.synthesizers.multi_hop.base import (
    MultiHopScenario,
)
from ragas.testset.synthesizers.prompts import (
    ThemesPersonasInput,
    ThemesPersonasMatchingPrompt,
)

@dataclass
class NewMultiHopQuery(MultiHopQuerySynthesizer):

    theme_persona_matching_prompt = ThemesPersonasMatchingPrompt()

    async def _generate_scenarios(
        self,
        n: int,
        knowledge_graph,
        persona_list,
        callbacks,
    ) -> t.List[MultiHopScenario]:

        # query and get (node_a, rel, node_b) to create multi-hop queries
        results = kg.find_two_nodes_single_rel(
            relationship_condition=lambda rel: (
                True if rel.type == "keyphrases_overlap" else False
            )
        )

        num_sample_per_triplet = max(1, n // len(results))

        scenarios = []
        for triplet in results:
            if len(scenarios) < n:
                node_a, node_b = triplet[0], triplet[-1]
                overlapped_keywords = triplet[1].properties["overlapped_items"]
                if overlapped_keywords:

                    # match the keyword with a persona for query creation
                    themes = list(dict(overlapped_keywords).keys())
                    prompt_input = ThemesPersonasInput(
                        themes=themes, personas=persona_list
                    )
                    persona_concepts = (
                        await self.theme_persona_matching_prompt.generate(
                            data=prompt_input, llm=self.llm, callbacks=callbacks
                        )
                    )

                    overlapped_keywords = [list(item) for item in overlapped_keywords]

                    # prepare and sample possible combinations
                    base_scenarios = self.prepare_combinations(
                        [node_a, node_b],
                        overlapped_keywords,
                        personas=persona_list,
                        persona_item_mapping=persona_concepts.mapping,
                        property_name="keyphrases",
                    )

                    # get number of required samples from this triplet
                    base_scenarios = self.sample_diverse_combinations(
                        base_scenarios, num_sample_per_triplet
                    )

                    scenarios.extend(base_scenarios)

        return scenarios
```

```python
query = NewMultiHopQuery(llm=generator_llm)
query
```




<pre class="custom">NewMultiHopQuery(name='NewMultiHopQuery', llm=LangchainLLMWrapper(langchain_llm=ChatOpenAI(...)), generate_query_reference_prompt=QueryAnswerGenerationPrompt(instruction=Generate a multi-hop query and answer based on the specified conditions (persona, themes, style, length) and the provided context. The themes represent a set of phrases either extracted or generated from the context, which highlight the suitability of the selected context for multi-hop query creation. Ensure the query explicitly incorporates these themes.### Instructions:
    1. **Generate a Multi-Hop Query**: Use the provided context segments and themes to form a query that requires combining information from multiple segments (e.g., `<1-hop>` and `<2-hop>`). Ensure the query explicitly incorporates one or more themes and reflects their relevance to the context.
    2. **Generate an Answer**: Use only the content from the provided context to create a detailed and faithful answer to the query. Avoid adding information that is not directly present or inferable from the given context.
    3. **Multi-Hop Context Tags**:
       - Each context segment is tagged as `<1-hop>`, `<2-hop>`, etc.
       - Ensure the query uses information from at least two segments and connects them meaningfully., examples=[(QueryConditions(persona=Persona(name='Historian', role_description='Focuses on major scientific milestones and their global impact.'), themes=['Theory of Relativity', 'Experimental Validation'], query_style='Formal', query_length='Medium', context=['<1-hop> Albert Einstein developed the theory of relativity, introducing the concept of spacetime.', '<2-hop> The bending of light by gravity was confirmed during the 1919 solar eclipse, supporting Einstein’s theory.']), GeneratedQueryAnswer(query='How was the experimental validation of the theory of relativity achieved during the 1919 solar eclipse?', answer='The experimental validation of the theory of relativity was achieved during the 1919 solar eclipse by confirming the bending of light by gravity, which supported Einstein’s concept of spacetime as proposed in the theory.'))], language=english))</pre>



### Implementation of Custom Distribution
I've revamped the distribution setup to make it more flexible. Now it features four query types: `simple`, `reasoning`, `multi_context`, and `conditional`. Users can freely adjust the frequency of each type according to their needs.

```python
import typing as t
from ragas.llms import BaseRagasLLM

QueryDistribution = t.List[t.Tuple[BaseSynthesizer, float]]
```

Due to insufficient cluster size, we were unable to use `MultiHopAbstractQuerySynthesizer`(llm=llm) and `SingleHopSpecificQuerySynthesizer`(llm=llm). We will proceed with implementation using only `NewMultiHopQuery`.

```python
simple_synthesizer = SingleHopSpecificQuerySynthesizer(llm=generator_llm)
reasoning_synthesizer = NewMultiHopQuery(llm=generator_llm)
multi_context_synthesizer = NewMultiHopQuery(llm=generator_llm)
conditional_synthesizer = NewMultiHopQuery(llm=generator_llm)
```

```python
def custom_query_distribution(
   llm: BaseRagasLLM,
   distributions: t.List[float],
   kg: t.Optional[KnowledgeGraph] = None
) -> QueryDistribution:
   default_queries = [
       simple_synthesizer,
       reasoning_synthesizer,
       multi_context_synthesizer,
       conditional_synthesizer
   ]

   if kg is not None:
       available_queries = [q for q in default_queries if q.get_node_clusters(kg)]
   else:
       available_queries = default_queries

   return list(zip(available_queries, distributions))
```

```python
distributions = [0.4, 0.2, 0.2, 0.2]

query_distribution = custom_query_distribution(generator_llm, distributions)
query_distribution
```




<pre class="custom">[(SingleHopSpecificQuerySynthesizer(name='single_hop_specifc_query_synthesizer', llm=LangchainLLMWrapper(langchain_llm=ChatOpenAI(...)), generate_query_reference_prompt=QueryAnswerGenerationPrompt(instruction=Generate a single-hop query and answer based on the specified conditions (persona, term, style, length) and the provided context. Ensure the answer is entirely faithful to the context, using only the information directly from the provided context.### Instructions:
      1. **Generate a Query**: Based on the context, persona, term, style, and length, create a question that aligns with the persona's perspective and incorporates the term.
      2. **Generate an Answer**: Using only the content from the provided context, construct a detailed answer to the query. Do not add any information not included in or inferable from the context.
      , examples=[(QueryCondition(persona=Persona(name='Software Engineer', role_description='Focuses on coding best practices and system design.'), term='microservices', query_style='Formal', query_length='Medium', context='Microservices are an architectural style where applications are structured as a collection of loosely coupled services. Each service is fine-grained and focuses on a single functionality.'), GeneratedQueryAnswer(query='What is the purpose of microservices in software architecture?', answer='Microservices are designed to structure applications as a collection of loosely coupled services, each focusing on a single functionality.'))], language=english), theme_persona_matching_prompt=ThemesPersonasMatchingPrompt(instruction=Given a list of themes and personas with their roles, associate each persona with relevant themes based on their role description., examples=[(ThemesPersonasInput(themes=['Empathy', 'Inclusivity', 'Remote work'], personas=[Persona(name='HR Manager', role_description='Focuses on inclusivity and employee support.'), Persona(name='Remote Team Lead', role_description='Manages remote team communication.')]), PersonaThemesMapping(mapping={'HR Manager': ['Inclusivity', 'Empathy'], 'Remote Team Lead': ['Remote work', 'Empathy']}))], language=english), property_name='entities'),
      0.4),
     (NewMultiHopQuery(name='NewMultiHopQuery', llm=LangchainLLMWrapper(langchain_llm=ChatOpenAI(...)), generate_query_reference_prompt=QueryAnswerGenerationPrompt(instruction=Generate a multi-hop query and answer based on the specified conditions (persona, themes, style, length) and the provided context. The themes represent a set of phrases either extracted or generated from the context, which highlight the suitability of the selected context for multi-hop query creation. Ensure the query explicitly incorporates these themes.### Instructions:
      1. **Generate a Multi-Hop Query**: Use the provided context segments and themes to form a query that requires combining information from multiple segments (e.g., `<1-hop>` and `<2-hop>`). Ensure the query explicitly incorporates one or more themes and reflects their relevance to the context.
      2. **Generate an Answer**: Use only the content from the provided context to create a detailed and faithful answer to the query. Avoid adding information that is not directly present or inferable from the given context.
      3. **Multi-Hop Context Tags**:
         - Each context segment is tagged as `<1-hop>`, `<2-hop>`, etc.
         - Ensure the query uses information from at least two segments and connects them meaningfully., examples=[(QueryConditions(persona=Persona(name='Historian', role_description='Focuses on major scientific milestones and their global impact.'), themes=['Theory of Relativity', 'Experimental Validation'], query_style='Formal', query_length='Medium', context=['<1-hop> Albert Einstein developed the theory of relativity, introducing the concept of spacetime.', '<2-hop> The bending of light by gravity was confirmed during the 1919 solar eclipse, supporting Einstein’s theory.']), GeneratedQueryAnswer(query='How was the experimental validation of the theory of relativity achieved during the 1919 solar eclipse?', answer='The experimental validation of the theory of relativity was achieved during the 1919 solar eclipse by confirming the bending of light by gravity, which supported Einstein’s concept of spacetime as proposed in the theory.'))], language=english)),
      0.2),
     (NewMultiHopQuery(name='NewMultiHopQuery', llm=LangchainLLMWrapper(langchain_llm=ChatOpenAI(...)), generate_query_reference_prompt=QueryAnswerGenerationPrompt(instruction=Generate a multi-hop query and answer based on the specified conditions (persona, themes, style, length) and the provided context. The themes represent a set of phrases either extracted or generated from the context, which highlight the suitability of the selected context for multi-hop query creation. Ensure the query explicitly incorporates these themes.### Instructions:
      1. **Generate a Multi-Hop Query**: Use the provided context segments and themes to form a query that requires combining information from multiple segments (e.g., `<1-hop>` and `<2-hop>`). Ensure the query explicitly incorporates one or more themes and reflects their relevance to the context.
      2. **Generate an Answer**: Use only the content from the provided context to create a detailed and faithful answer to the query. Avoid adding information that is not directly present or inferable from the given context.
      3. **Multi-Hop Context Tags**:
         - Each context segment is tagged as `<1-hop>`, `<2-hop>`, etc.
         - Ensure the query uses information from at least two segments and connects them meaningfully., examples=[(QueryConditions(persona=Persona(name='Historian', role_description='Focuses on major scientific milestones and their global impact.'), themes=['Theory of Relativity', 'Experimental Validation'], query_style='Formal', query_length='Medium', context=['<1-hop> Albert Einstein developed the theory of relativity, introducing the concept of spacetime.', '<2-hop> The bending of light by gravity was confirmed during the 1919 solar eclipse, supporting Einstein’s theory.']), GeneratedQueryAnswer(query='How was the experimental validation of the theory of relativity achieved during the 1919 solar eclipse?', answer='The experimental validation of the theory of relativity was achieved during the 1919 solar eclipse by confirming the bending of light by gravity, which supported Einstein’s concept of spacetime as proposed in the theory.'))], language=english)),
      0.2),
     (NewMultiHopQuery(name='NewMultiHopQuery', llm=LangchainLLMWrapper(langchain_llm=ChatOpenAI(...)), generate_query_reference_prompt=QueryAnswerGenerationPrompt(instruction=Generate a multi-hop query and answer based on the specified conditions (persona, themes, style, length) and the provided context. The themes represent a set of phrases either extracted or generated from the context, which highlight the suitability of the selected context for multi-hop query creation. Ensure the query explicitly incorporates these themes.### Instructions:
      1. **Generate a Multi-Hop Query**: Use the provided context segments and themes to form a query that requires combining information from multiple segments (e.g., `<1-hop>` and `<2-hop>`). Ensure the query explicitly incorporates one or more themes and reflects their relevance to the context.
      2. **Generate an Answer**: Use only the content from the provided context to create a detailed and faithful answer to the query. Avoid adding information that is not directly present or inferable from the given context.
      3. **Multi-Hop Context Tags**:
         - Each context segment is tagged as `<1-hop>`, `<2-hop>`, etc.
         - Ensure the query uses information from at least two segments and connects them meaningfully., examples=[(QueryConditions(persona=Persona(name='Historian', role_description='Focuses on major scientific milestones and their global impact.'), themes=['Theory of Relativity', 'Experimental Validation'], query_style='Formal', query_length='Medium', context=['<1-hop> Albert Einstein developed the theory of relativity, introducing the concept of spacetime.', '<2-hop> The bending of light by gravity was confirmed during the 1919 solar eclipse, supporting Einstein’s theory.']), GeneratedQueryAnswer(query='How was the experimental validation of the theory of relativity achieved during the 1919 solar eclipse?', answer='The experimental validation of the theory of relativity was achieved during the 1919 solar eclipse by confirming the bending of light by gravity, which supported Einstein’s concept of spacetime as proposed in the theory.'))], language=english)),
      0.2)]</pre>



```python
dataset = generator.generate_with_langchain_docs(
   documents=docs, # document data
   testset_size=10, # number of questions to generate
   query_distribution=query_distribution, # distribution by question type 
   with_debugging_logs=True # output debugging logs
)
```

<pre class="custom">Applying SummaryExtractor:   0%|          | 0/36 [00:00<?, ?it/s]</pre>

    Applying CustomNodeFilter:  24%|██▎       | 9/38 [00:01<00:02,  9.70it/s] Node c2855407-9922-456c-b369-34965aebdcaf does not have a summary. Skipping filtering.
    Applying CustomNodeFilter:  50%|█████     | 19/38 [00:02<00:01, 10.46it/s]Node a6843a7c-c3ef-45a3-a2f7-cad5f845d7fa does not have a summary. Skipping filtering.
    Generating personas: 100%|██████████| 3/3 [00:02<00:00,  1.13it/s]                                             
    Generating Scenarios: 100%|██████████| 4/4 [00:06<00:00,  1.66s/it]
    Generating Samples: 100%|██████████| 10/10 [00:07<00:00,  1.29it/s]
    

```python
dataset.to_pandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_input</th>
      <th>reference_contexts</th>
      <th>reference</th>
      <th>synthesizer_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>What Generative AI do?</td>
      <td>[Agents\nThis combination of reasoning,\nlogic...</td>
      <td>Generative AI models can be trained to use too...</td>
      <td>single_hop_specifc_query_synthesizer</td>
    </tr>
    <tr>
      <th>1</th>
      <td>What is the significance of September 2024 in ...</td>
      <td>[Agents\nWhat is an agent?\nIn its most fundam...</td>
      <td>The context does not provide specific informat...</td>
      <td>single_hop_specifc_query_synthesizer</td>
    </tr>
    <tr>
      <th>2</th>
      <td>How does the ReAct framework integrate with la...</td>
      <td>[Agents\nFigure 1. General agent architecture ...</td>
      <td>In the scope of an agent, a model refers to th...</td>
      <td>single_hop_specifc_query_synthesizer</td>
    </tr>
    <tr>
      <th>3</th>
      <td>What DELETE do in tools for agents, how it hel...</td>
      <td>[Agents\nThe tools\nFoundational models, despi...</td>
      <td>DELETE is a common web API method that tools c...</td>
      <td>single_hop_specifc_query_synthesizer</td>
    </tr>
    <tr>
      <th>4</th>
      <td>How does a Generative AI model utilize reasoni...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\nWhat is an agent?\nIn its ...</td>
      <td>A Generative AI model functions as an agent by...</td>
      <td>NewMultiHopQuery</td>
    </tr>
    <tr>
      <th>5</th>
      <td>How does the orchestration layer enhance the c...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\nSummary\nIn this whitepape...</td>
      <td>The orchestration layer enhances the capabilit...</td>
      <td>NewMultiHopQuery</td>
    </tr>
    <tr>
      <th>6</th>
      <td>How does the ReAct framework enable agents to ...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\n• Chain-of-Thought (CoT), ...</td>
      <td>The ReAct framework enables agents to effectiv...</td>
      <td>NewMultiHopQuery</td>
    </tr>
    <tr>
      <th>7</th>
      <td>How does a generative AI model utilize reasoni...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\nWhat is an agent?\nIn its ...</td>
      <td>A generative AI model functions as a generativ...</td>
      <td>NewMultiHopQuery</td>
    </tr>
    <tr>
      <th>8</th>
      <td>How fine-tuning based learning help chef learn...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\n• Imagine a chef has recei...</td>
      <td>Fine-tuning based learning helps the chef lear...</td>
      <td>NewMultiHopQuery</td>
    </tr>
    <tr>
      <th>9</th>
      <td>How does a Generative AI model, like the one u...</td>
      <td>[&lt;1-hop&gt;\n\nAgents\nPython\nfrom vertexai.gene...</td>
      <td>A Generative AI model, such as the one used in...</td>
      <td>NewMultiHopQuery</td>
    </tr>
  </tbody>
</table>
</div>



```python
dataset.to_pandas().to_csv("data/ragas_synthetic_dataset.csv", index=False)
```

## Summary: Moving Forward with Generated and Prepared Datasets
Now that we have generated our dataset or prepared datasets from the data folder, let's move on to the next section: Evaluation using `RAGAS`.

## Bonus: Refactoring Section

This tutorial's bonus section demonstrates how to improve code execution time from at 1 minute to 3-8 seconds.

If you're familiar with parallel and asynchronous processing, you can combine them to improve response time.
We'll use the `asyncio` module for asynchronous processing and `multiprocessing` for parallel processing.

Original code takes at least 50 seconds:

```python
keyphrase_extractor = KeyphrasesExtractor()
output = [await keyphrase_extractor.extract(node) for node in kg.nodes]
_ = [node.properties.update({key:val}) for (key,val), node in zip(output, kg.nodes)]
kg.nodes[0].properties
```

* `output = [await keyphrase_extractor.extract(node) for node in kg.nodes]` - Processing nodes sequentially, waiting for each extract to complete before processing the next node

Let's improve using `ThreadPool`:

```python
import asyncio
from multiprocessing.pool import ThreadPool

def process_node(node):
    keyphrase_extractor = KeyphrasesExtractor()
    return asyncio.run(extractor.keyphrase_extract(node))

def update_nodes_pool(kg_nodes, num_threads=4):
    with ThreadPool(processes=num_threads) as pool:
        outputs = pool.map(process_node, kg_nodes)
    _ = [node.properties.update({key:val}) for (key,val), node in zip(outputs, kg_nodes)]
    return kg_nodes[0].properties

_ = update_nodes_pool(kg.nodes)
kg.nodes[0].properties
```

Improved to approximately 14-15 seconds (14.6s, 15.2s, 14.3s).

Now let's improve using `async` processing:

```python
keyphrase_extractor = KeyphrasesExtractor()
async def process_keyphrase_batch(nodes, batch_size=5):
    outputs = []
    for i in range(0, len(nodes), batch_size):
        batch = nodes[i:i + batch_size]
        batch_output = await asyncio.gather(*[keyphrase_extractor.extract(node) for node in batch])
        outputs.extend(batch_output)
    return outputs
    
outputs = await process_keyphrase_batch(kg.nodes)
_ = [node.properties.update({key:val}) for (key,val), node in zip(outputs, kg.nodes)]
kg.nodes[0].properties
```

Improved to approximately 16 seconds.
Processing nodes in batches of 5 simultaneously using `asyncio.gather`.
The key improvement comes from `asyncio.gather`, which executes multiple coroutines simultaneously and waits for all results. Performance improvement is achieved because extract function includes I/O operations (API calls).

What happens when we combine both approaches?

```python
import asyncio
from multiprocessing.pool import ThreadPool

# Async function to process single batch
async def process_batch(batch):
    keyphrase_extractor = KeyphrasesExtractor()
    batch_output = await asyncio.gather(*[keyphrase_extractor.extract(node) for node in batch])
    return batch_output

# Function to run in thread
def process_batch_in_thread(batch):
    return asyncio.run(process_batch(batch))

def process_with_thread_and_async(nodes, batch_size=5, num_threads=4):
    # Divide data into batches
    batches = [nodes[i:i + batch_size] 
              for i in range(0, len(nodes), batch_size)]
    
    # Process batches using thread pool
    with ThreadPool(processes=num_threads) as pool:
        all_outputs = pool.map(process_batch_in_thread, batches)
    
    outputs = []
    for batch_output in all_outputs:
        outputs.extend(batch_output)
    
    # Update results
    _ = [node.properties.update({key:val}) 
         for (key,val), node in zip(outputs, nodes)]
    
    return nodes[0].properties

_ = process_with_thread_and_async(kg.nodes)
kg.nodes[0].properties
```

By effectively combining parallel and asynchronous processing, we can reduce execution time from 1 minute to approximately 3-8 seconds.
