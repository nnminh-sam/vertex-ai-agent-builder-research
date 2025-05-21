# Vertex AI Agent Builder Use Cases

---

**Table of Content**

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [**Prerequisites**](#prerequisites)
- [**Implementations**](#implementations)
  - [**Data Preparation and Knowledge Base**](#data-preparation-and-knowledge-base)
  - [**Building Watch Searcher Using Vertex AI Search**](#building-watch-searcher-using-vertex-ai-search)
    - [**Create Search Engine**](#create-search-engine)
    - [**AI Search Integration**](#ai-search-integration)
  - [**Building Watch Recommender AI Agent Using Conversational Chat**](#building-watch-recommender-ai-agent-using-conversational-chat)
    - [**Initialize The Agent**](#initialize-the-agent)
    - [**Playbook In Conversational Agent**](#playbook-in-conversational-agent)
    - [**Create Simple Playbook For Watch Recommender**](#create-simple-playbook-for-watch-recommender)
    - [**Tools In Conversational Agent**](#tools-in-conversational-agent)
    - [**Creating Tools For Recommender Playbook**](#creating-tools-for-recommender-playbook)
    - [**Preview Conversational Agent**](#preview-conversational-agent)
    - [**Agent Integration**](#agent-integration)
- [**Summary**](#summary)
- [**References**](#references)

<!-- /code_chunk_output -->

---

In this research, our goals are to create:

1. A custom **product search engine** tailored to the product details.
2. A **conversational generative AI chat agent** that help recommending product to the user via an assistance.

Along these use cases, I'll use a small dataset about watches.

---

## **Prerequisites**

1.  **Google Cloud Project:** An active Google Cloud Project.
2.  **Enable Vertex AI API:** Enabled Vertex AI API.
3.  **Watch Catalog Schema:**
    - `Id` (unique identifier)
    - `Brand`
    - `Model`
    - `Movement_Type` (e.g., Automatic, Quartz, Manual)
    - `Case_Material` (e.g., Stainless Steel, Gold, Titanium)
    - `Strap_Material` (e.g., Leather, Metal, Rubber)
    - `Style` (e.g., Dress, Sport, Dive, Pilot, Casual, Luxury)
    - `Features` (e.g., Chronograph, Date, Moonphase, Water Resistance)
    - `Price_Range` (e.g., Entry-level, Mid-range, Luxury)
    - `Target_Audience` (e.g., Men's, Women's, Unisex)
    - `Description` (a brief, rich description of the watch)
    - `Image_URL` (optional, for front-end display)

---

## **Implementations**

### **Data Preparation and Knowledge Base**

Firstly, we'll need to upload the dataset to the Google Cloud Storage for AI application integration.

1. Go to your **Google Cloud Console**.
1. Go to **Cloud Storage**.
1. Go to **Buckets** tab and choose **Create**.

    ![](/images/buckets-1.png)

1. Then enter the bucket's name. Here I'll go with `watch_catalog`.
1. Then you can choose any region you want. Note that different region will have different cost so choose the region that fit with your bugdet.
1. Next you will want to choose how to store your data. Here you can choose autoclass to let Google Cloud Storage decide how it should manage your data. In my case, I'll go with `Standard default` class only since my data is frequently access.
1. For access controll, by default, Cloud Storage will recommend you to use prevent public access feature. This does not affect the usage of the integrated AI application but will affect other third party application. Here I'll keep that option turn on and choose `Uniform access controll` option.
1. Finally, I'll recommend to choose soft delete for data protection.

After creating process of the bucket, you will see this UI pop up:

![](/images/buckets-2.png)

From now on you can import your data into this bucket. Notes that you will have to use the NDJSON format for your JSON file.

This is the sample watch data:

```json
{"id": "W001", "struct_data": {"Brand": "Rolex", "Model": "Submariner Date", "Movement_Type": "Automatic", "Case_Material": "Oystersteel", "Strap_Material": "Oystersteel", "Style": "Dive", "Features": "Water Resistance (300m), Chronometer, Date, Unidirectional Rotatable Bezel", "Price_Range": "Luxury", "Target_Audience": "Men's", "Description": "An iconic diver's watch, the Rolex Submariner Date combines robust functionality with timeless elegance. Its highly legible Chromalight display and reliable Oyster Perpetual movement make it a favorite for both professionals and enthusiasts.", "Image_URL": "https://example.com/images/rolex_submariner.jpg"}}
{"id": "W002", "struct_data": {"Brand": "Seiko", "Model": "Presage Cocktail Time 'Manhattan'", "Movement_Type": "Automatic", "Case_Material": "Stainless Steel", "Strap_Material": "Leather", "Style": "Dress", "Features": "Date, Hardlex Crystal, Exhibition Caseback", "Price_Range": "Mid-range", "Target_Audience": "Unisex", "Description": "Inspired by classic cocktails, the Seiko Presage 'Manhattan' features a captivating sunburst dial that shimmers with every movement. Its refined aesthetic and reliable automatic movement make it a perfect choice for formal occasions or daily wear.", "Image_URL": "https://example.com/images/seiko_presage_manhattan.jpg"}}
{"id": "W003", "struct_data": {"Brand": "Casio", "Model": "G-Shock GA-2100", "Movement_Type": "Quartz", "Case_Material": "Carbon Core Guard Resin", "Strap_Material": "Resin", "Style": "Sport", "Features": "Shock Resistant, 200m Water Resistance, World Time, Chronograph, Alarm, LED Light", "Price_Range": "Entry-level", "Target_Audience": "Unisex", "Description": "Nicknamed the 'Casioak' for its octagonal bezel, the G-Shock GA-2100 offers unparalleled toughness in a remarkably slim and lightweight design. It's the ultimate tool watch for those who demand durability and functionality.", "Image_URL": "https://example.com/images/casio_ga2100.jpg"}}
{"id": "W004", "struct_data": {"Brand": "Omega", "Model": "Speedmaster Professional Moonwatch", "Movement_Type": "Manual", "Case_Material": "Stainless Steel", "Strap_Material": "Nylon", "Style": "Pilot", "Features": "Chronograph, Tachymeter, Hesalite Crystal, NASA Certified", "Price_Range": "Luxury", "Target_Audience": "Men's", "Description": "The legendary Omega Speedmaster Professional, famously worn on the moon, is a true icon of space exploration and precision timing. Its robust manual-wind movement and historical significance make it a coveted timepiece.", "Image_URL": "https://example.com/images/omega_moonwatch.jpg"}}
{"id": "W005", "struct_data": {"Brand": "Citizen", "Model": "Eco-Drive Promaster Diver", "Movement_Type": "Quartz (Eco-Drive)", "Case_Material": "Stainless Steel", "Strap_Material": "Rubber", "Style": "Dive", "Features": "Solar Powered, 200m Water Resistance, Unidirectional Bezel, Luminous Hands and Markers", "Price_Range": "Mid-range", "Target_Audience": "Unisex", "Description": "The Citizen Eco-Drive Promaster Diver is a robust and reliable dive watch powered by any light source, eliminating the need for battery changes. Its bold design and excellent water resistance make it ideal for aquatic adventures.", "Image_URL": "https://example.com/images/citizen_promaster.jpg"}}
{"id": "W006", "struct_data": {"Brand": "Patek Philippe", "Model": "Calatrava", "Movement_Type": "Automatic", "Case_Material": "Rose Gold", "Strap_Material": "Alligator Leather", "Style": "Dress", "Features": "Date, Sapphire Crystal Caseback, Small Seconds", "Price_Range": "Luxury", "Target_Audience": "Men's", "Description": "Embodying timeless elegance and understated luxury, the Patek Philippe Calatrava is a testament to refined watchmaking. Its minimalist dial, exquisite craftsmanship, and precious metal case make it a pinnacle of dress watches.", "Image_URL": "https://example.com/images/patek_calatrava.jpg"}}
{"id": "W007", "struct_data": {"Brand": "Timex", "Model": "Weekender Chronograph", "Movement_Type": "Quartz", "Case_Material": "Brass", "Strap_Material": "Nylon", "Style": "Casual", "Features": "Chronograph, Indiglo Night-Light, Date", "Price_Range": "Entry-level", "Target_Audience": "Unisex", "Description": "The Timex Weekender Chronograph offers versatile style and reliable functionality at an accessible price. Its interchangeable straps and Indiglo backlight make it perfect for everyday wear and various casual outfits.", "Image_URL": "https://example.com/images/timex_weekender.jpg"}}
{"id": "W008", "struct_data": {"Brand": "Tag Heuer", "Model": "Carrera Calibre Heuer 02", "Movement_Type": "Automatic", "Case_Material": "Stainless Steel", "Strap_Material": "Stainless Steel", "Style": "Sport", "Features": "Chronograph, Date, Sapphire Crystal, Exhibition Caseback", "Price_Range": "Luxury", "Target_Audience": "Men's", "Description": "A sophisticated and powerful chronograph, the TAG Heuer Carrera Calibre Heuer 02 pays homage to motorsports. Its in-house movement and sporty yet elegant design make it a favorite for those who appreciate performance and style.", "Image_URL": "https://example.com/images/tagheuer_carrera.jpg"}}
{"id": "W009", "struct_data": {"Brand": "Longines", "Model": "Master Collection Moonphase", "Movement_Type": "Automatic", "Case_Material": "Stainless Steel", "Strap_Material": "Alligator Leather", "Style": "Dress", "Features": "Moonphase, Chronograph, Date, Day/Month Display", "Price_Range": "Mid-range", "Target_Audience": "Men's", "Description": "The Longines Master Collection Moonphase is a horological marvel, combining classic elegance with complex complications. Its beautiful moonphase display and comprehensive calendar functions make it a truly distinguished timepiece.", "Image_URL": "https://example.com/images/longines_moonphase.jpg"}}
{"id": "W010", "struct_data": {"Brand": "Apple", "Model": "Watch Ultra 2", "Movement_Type": "Smartwatch", "Case_Material": "Titanium", "Strap_Material": "Fluoroelastomer", "Style": "Sport", "Features": "GPS, Heart Rate Monitor, Water Resistance (100m), Always-On Retina Display, Cellular Connectivity, Action Button", "Price_Range": "Luxury", "Target_Audience": "Unisex", "Description": "Designed for adventurers and athletes, the Apple Watch Ultra 2 is a robust and feature-rich smartwatch. Its durable titanium case, extended battery life, and advanced health and fitness tracking make it an indispensable companion for extreme conditions.", "Image_URL": "https://example.com/images/apple_watch_ultra2.jpg"}}
```

Save this data into a json file with `utf-8` encoding then you can upload it to **Cloud Storage**.

A success upload should look like this:

![](/images/buckets-3.png)

---

### **Building Watch Searcher Using Vertex AI Search**

#### **Create Search Engine**

1. Go to the **Google Cloud Console**.
1. Navigate to **Vertex AI \> Vertex AI Search**.
1. Choose **Custom search** option.

    ![](/images/search-1.png)

1. Enter your app name and company name as you want and choose global region.

    ![](/images/search-2.png)

1. Next you'll choose the previous created `watch_catalog` data store.

    ![](/images/search-3.png)

1. Then press **Create** to create your first search app.

Once the app is created, you will be redirected to this UI:

![](/images/search-4.png)

Then you can choose **Preview** to preview your custom search engine for your product catalog. Note that event the UI is shown but the app may need about 5 minutes to create.

![](/images/search-5.png)

There are more configuration you can apply to your search engine. This is an example to add AI answer to the search result with follow up question for users.

![](/images/search-6.png)

#### **AI Search Integration**

For integrating this search engine to your application, you can navigate to the **Integration** tabs and follow the instruction there. These are clear and simple instruction so you can follow along just fine.

![](/images/search-7.png)

---

### **Building Watch Recommender AI Agent Using Conversational Chat**

#### **Initialize The Agent**

1. Go to the **Google Cloud Console**.
1. Navigate to **AI Application \> Apps \> Create App**.
1. In **Conversational Agent**, choose **Create Conversational Agent**.

    ![](/images/chat-2.png)

1. Then when the below UI comes up, you'll choose **Create Agent** and **Build your own** option.

    ![](/images/chat-3.png)

1. Then you will fill in the Display name, location (choose `global`), timezone, and default language for the agent.
1. The goal for this agent is to use Generative AI so we'll choose playbook option.

    ![](/images/chat-4.png)

Choose **Create** and wait a little bit for the app to process.

#### **Playbook In Conversational Agent**

In the general context of Conversational AI agents (which includes chatbots, virtual assistants, etc.), a **Playbook** is essentially a **pre-defined strategy or script that guides the agent's interaction with a user to achieve a specific outcome or handle a particular scenario.**

Think of it as a set of rules, instructions, and potential conversational paths that the AI is programmed to follow. These playbooks ensure that the agent responds appropriately, stays on topic, and effectively assists the user.

Here's what that generally means:

1. **Defined Scope and Goal:** Each playbook is typically designed to address a specific task, topic, or user intent. For example:
    - Answering FAQs about a product.
    - Guiding a user through a troubleshooting process.
    - Collecting information for a lead.
    - Booking an appointment.
    - Handling a complaint.
1. **Conversation Design and Flow:** Playbooks map out how a conversation should ideally proceed. This can include:
    - **Key messages** the agent should deliver.
    - **Questions** the agent should ask.
    - **Possible user responses** and how the agent should react to them (branching logic).
    - **Decision points** where the conversation might take different turns based on user input or other conditions.
1. **Integration with Tools and Data:** Playbooks often define when and how the conversational agent should interact with other systems or data sources. This could be:
    - Looking up information in a knowledge base.
    - Fetching customer data from a CRM.
    - Calling an API to perform an action (e.g., place an order).
1. **Escalation Paths:** They can include instructions on when to hand over the conversation to a human agent if the AI cannot resolve the issue or if the user requests it.
1. **Consistency and Quality Control:** Playbooks help ensure that the conversational agent provides consistent and accurate information and follows company guidelines for interaction.
1. **Efficiency and Automation:** By codifying common interactions into playbooks, businesses can automate responses and processes, freeing up human resources.

**Why are they called "Playbooks"?**

The term "playbook" is borrowed from sports, where a playbook contains a team's strategies and plays for various game situations. In conversational AI, it similarly provides the "plays" or strategies for the agent to use during interactions.

**Key characteristics across different platforms (even if terminology varies):**

- **Goal-Oriented:** Designed to achieve a specific conversational objective.
- **Structured:** Provides a framework for the conversation.
- **Conditional Logic:** Includes rules for how to respond to different user inputs or situations.
- **Modular:** Often, complex agents are built using multiple playbooks that can be triggered as needed.

So, while specific implementations (like Google's with its emphasis on generative AI and LLM instructions) can have unique features, the core concept of a "Playbook" in conversational agents is about having a structured, pre-defined approach to managing conversations effectively to achieve desired outcomes.

#### **Create Simple Playbook For Watch Recommender**

First, you choose **Create** in the **Playbooks** tab.

![](/images/chat-5.png)

Choose **Routine** for keeping the user session data during the conversation.

Then you can name your playbook as you want. I suggest to name is as descriptive as the playbook functionality as you can.

**Agent Goal:**

```
You are a highly knowledgeable and sophisticated watch recommendation expert. Your goal is to understand the user's preferences regarding their needs, characteristics, and style, and then recommend a watch from the available catalog that best fits their description.
```

**Instructions:**

```
- Carefully analyze the user's prompt for keywords related to:
    - **Needs:** (e.g., daily wear, special occasions, sport, durable, waterproof)
    - **Characteristics:** (e.g., sophisticated, rugged, minimalist, elegant, modern, vintage)
    - **Style:** (e.g., formal, casual, adventurous, classic, trendy, understated)
        - **Gender/Target Audience:** (e.g., "for a man," "for a woman," "unisex")
        - **Budget/Price:** (e.g., "affordable," "luxury," "under $500")
- **Use the ${TOOL: watch_catalog} tool exclusively** to find suitable watches based on the extracted preferences. Do NOT make up watch recommendations.
- If multiple watches match, recommend the top 1-3 most relevant options.
- For each recommendation, provide:
    - Watch Name (Brand and Model)
    - Key features that align with the user's request.
    - A brief, compelling reason why it's a good fit.
- If you cannot find a suitable watch, politely inform the user and suggest they try different criteria.
- Maintain a polite, helpful, and expert tone.
```

![](/images/chat-6.png)

In the instruction code, you can see the `${TOOL: watch_catalog}`. This is how Google help you reference to the **Tools** set for creating Agent.

Save your playbook and create this tool to help your agent complete it task.

#### **Tools In Conversational Agent**

In the context of conversational agents (like chatbots, virtual assistants, or AI agents), **Tools** are external resources, services, APIs, or functions that the agent can access and utilize to perform actions or retrieve information that goes beyond its core conversational capabilities and built-in knowledge.

Essentially, tools extend the agent's abilities, allowing it to interact with the outside world, access dynamic data, and perform tasks on behalf of the user.

Here's a breakdown of what "tools" mean for conversational agents:

1. **Extending Capabilities:**
    - Conversational agents, especially those based on Large Language Models (LLMs), have a vast amount of knowledge but it's generally static (up to their last training date) and they can't inherently perform real-world actions.
    - Tools bridge this gap, giving the agent the ability to:
        - **Fetch Real-time Data:** Access current information like weather forecasts, stock prices, flight statuses, news updates, or product availability.
        - **Interact with External Systems:** Connect to databases, Customer Relationship Management (CRM) systems, booking platforms, e-commerce backends, knowledge bases, or other enterprise software.
        - **Perform Actions:** Execute tasks like booking an appointment, placing an order, sending an email, updating a user's profile, or calling another API.
        - **Utilize Specialized Functions:** Leverage capabilities like making calculations, translating languages, or running custom code.

1. **How They Work (Conceptual):**
    - The conversational agent (often guided by its "playbook" or underlying logic) determines when a specific piece of information or an action is needed that it cannot handle on its own.
    - It then "calls" or "invokes" the appropriate tool, often passing necessary parameters (e.g., a city for a weather forecast, a product ID for an inventory check).
    - The tool performs its function (e.g., queries an API, runs a database lookup).
    - The tool returns the result (data or confirmation of an action) to the conversational agent.
    - The agent then uses this result to formulate a response to the user or continue the conversation.

1. **Common Types and Examples of Tools:**
    - **API Integrations:** This is the most common form. The agent calls external APIs (Application Programming Interfaces) for:
        - Weather services (e.g., OpenWeatherMap API)
        - Mapping services (e.g., Google Maps API)
        - Translation services (e.g., Google Translate API)
        - E-commerce platforms (e.g., Shopify API, Magento API)
        - Payment gateways (e.g., Stripe API, PayPal API)
        - CRM systems (e.g., Salesforce API, HubSpot API)
        - Ticketing systems (e.g., Zendesk API, Jira API)
    - **Database Lookups:** Directly querying internal or external databases for specific information.
    - **Knowledge Base Search:** Interfacing with specialized knowledge bases or document repositories to find answers to complex user queries.
    - **Custom Functions/Code Execution:** Running predefined code snippets or serverless functions to perform specific logic or calculations.
    - **Human Handoff Tools:** Though not always a "tool" in the same technical sense, the mechanism to escalate a conversation to a human agent can be considered a way to "tool out" of an automated flow.

1. **Importance in Conversational AI:**
    - **Enhanced Functionality:** Tools make conversational agents significantly more useful and capable.
    - **Personalization:** Accessing user data through tools allows for more tailored and relevant interactions.
    - **Automation of Complex Tasks:** Agents can handle multi-step processes that involve external systems.
    - **Up-to-date Information:** Provides users with current and accurate data.
    - **Increased User Satisfaction:** By successfully resolving queries and performing tasks, agents with good tools deliver a better user experience.

#### **Creating Tools For Recommender Playbook**

First, you go to the **Tools** tab and choose **Create**.

![](/images/chat-7.png)

This tool will use the previous created data store `watch_catalog` for limitting the agent to recommend the product from our database. Scenarios when the product does not match our database, we'll simple return a sorry response and ask for other recommendation.

Then you choose the data store we've created.

![](/images/chat-8.png)

In my scenario, I cannot see the Google pick up the created data store. This is and unknown error that I have not figured out yet. But as a video I've seen other people sucessfully create their own data store and import it into their tools. You can watch the reference video in the Reference section.

Therefore, I'll continue this research with this failure in mind. Below is the rest of the settings for the tool.

![](/images/chat-9.png)

Finally, you choose **Save** to save your tool.

Now, you can go back to your **Playbook** and update the instruction to make sure that is referencing the correct tool.

#### **Preview Conversational Agent**

You can preview your agent by choosing **Toggle Simulator** option on the right top navbar.

![](/images/chat-10.png)

Below is a sample chat conversation using the created agent.

![](/images/chat-11.png)

The last output is failed because the tools did not connect to the data store successfully. Resolving this error will ensure that the agent can returns the correct response to the users need.

Although this implementation did not success but does ensure that creating an AI agent that can play the role of a consultant for a watch store to talk to the customers is possible with this approach. I'll make further researches on this subject and try to fix the data store not found error while creating **Tools**.

#### **Agent Integration**

Integrating the agent to other application does not come easily as the Search engine. Google only allow for some specific applications such as below:

![](/images/chat-12.png)

![](/images/chat-13.png)

---

## **Summary**

This research focusing on building a Search Engine tailored to a specific data store and a conversational generative AI agent for recommending product from a data store to the customer. Despite of the unsuccessfull implementation of the conversational AI agent, the process shows that the approach I've taken during this research is possible to build a no-code AI agent in the future.

---

## **References**

- [Create an Academic Agent using Vertex AI Agent Builder| Conversational Agent](https://www.youtube.com/watch?v=3kZgoNoE8ts)
