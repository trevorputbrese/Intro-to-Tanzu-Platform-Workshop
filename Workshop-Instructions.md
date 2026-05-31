# Getting Started with Tanzu Platform Chat: A Guide to Cloud Foundry AI/ML Services

## Welcome! 🚀

Welcome to your journey into AI-powered applications on Cloud Foundry. In this hands-on guide, you'll build and deploy a chat application that progressively gains AI capabilities. Starting with a simple interface, you'll add intelligent chat responses, document understanding, external tool usage, and persistent memory—all without managing any infrastructure yourself.

What makes this special? Cloud Foundry's marketplace transforms complex AI and data services into simple, self-service building blocks. Instead of provisioning GPU clusters, configuring vector databases, or managing model deployments, you'll use simple `cf create-service` and `cf bind-service` commands. Cloud Foundry handles all the heavy lifting—SSL certificates, load balancing, scaling, updates—letting you focus on building great applications.

By the end of this guide, you'll have:
- 💬 An AI chatbot powered by enterprise LLMs
- 📚 Document Q&A using Retrieval Augmented Generation (RAG)
- 🔧 Tool-using AI via Model Context Protocol (MCP) servers
- 🤖 Direct communication with specialized AI agents via A2A protocol
- 🧠 Persistent conversation memory across sessions

Let's turn infrastructure complexity into developer simplicity!

## Prerequisites

A Windows jumpbox will be provided to you for this workshop with all the pre-requisite software installed.  You're only only pre-requisite is to have an RDP client in order to establish a remote desktop connection to the Windows jumpbox.  

Your instructor will provide you credentials and connection details to access your jumpbox prior to the workshop.

The jumpbox will have the following software installed:
- CF CLI installed
- Java 21+ and Maven 3.8+ installed locally
- Git installed for cloning repositories
- VSCode for your IDE
- Several repositories of demo applications will be cloned into the C:\Users\Administrator\Projects folder.  


### Step 1: Deploy to Cloud Foundry

Now let's deploy Chat client application to Cloud Foundry:
You need to download the precompiled jar file and manifest.yml
make a directory and cd into it and copy both files to this folder.

```bash
cd C:\Users\Administrator\Projects\cf-mcp-client

```
Change the name of the application in the manifest.yml. Append your initials and then run cf push command from your terminal. 
- name: ai-tool-chat-[YOUR-INITIALS]

```bash
# Login using CF login. User apps manager url. You can find it in env spreadsheet
cf login -a https://appsmanager-url -u youruser -p yourpassword
# run few cf commands to check the env. cf apps list all the apps
cf apps
# cf services, list all the services like postgress
cf services
# cf routes, list all the routes(urls)
cf routes
# now we are ready to push chat app. Make sure you are in the right folder where you have downloaded chat client jar file and manifest.yml file.
cf push
```

This command reads the `manifest.yml` file and deploys your application with 1GB of memory and Java 21 runtime. After about a minute, you'll see:

```
Showing health and status for app ai-tool-chat...

requested state: started
routes:          ai-tool-chat.apps.your-cf-domain.com
```

Open the provided URL in your browser to see your chat application. Run the below command to get the url of your application

```bash
cf routes
```

## Understanding the Basic Chat Interface

### Step 2: Explore the Application Without AI

When you first open the application, you'll see:
- A clean chat interface at the center
- Five control buttons on the navigation rail: Chat 💬, Documents 📄, Agents 🤖, MCP Servers 🔌, and Memory 🧠

Let's see what happens without any AI services:

1. Type a message like "Hello, how are you?" in the chat box
2. Press Send or hit Enter
3. **Result:** The bot responds with "No chat model available"

Click each button on the navigation rail to explore the status panels:
- **Chat 💬**: Shows "Chat Model: Not Available ❌"
- **Documents 📄**: Shows "Vector Store: Not Available ❌" and "Embed Model: Not Available ❌"
- **Agents 🤖**: Shows "No A2A agents configured" (for Agent2Agent protocol)
- **MCP Servers 🔌**: Shows "Status: Not Available ❌" (for Model Context Protocol tools)
- **Memory 🧠**: Shows "Memory: Transient ⚠️" (only lasts during the session)

This demonstrates that your application is running but has no AI capabilities yet. Let's fix that.

## Enabling AI Chat with an LLM

### Step 3: Create and Bind a Chat Model

Large Language Models (LLMs) are AI systems trained on vast amounts of text that can understand and generate human-like responses. Cloud Foundry makes accessing these powerful models as easy as binding a database.

First, check what GenAI plans are available:

```bash
cf marketplace -e genai
```

You'll see various model options like `tanzu-gpt-multi-model-v1025`, `tanzu-nomic-embed-text-v1025`, or others. Choose one and create a service instance:

```bash
# Create a chat LLM service instance (replace 'gpt-4o-mini' with your chosen plan)
cf create-service genai tanzu-gpt-multi-model-v1025 chat-llm

# Bind the service to your application. Make sure you use your own app name.
cf apps
# check name of your app first before you execute cf bind
cf bind-service ai-tool-chat-yourinitials chat-llm

# Restart the application to pick up the new service
cf restart ai-tool-chat-yourinitials
```

**What just happened?** Cloud Foundry:
- Provisioned access to an enterprise LLM
- Generated secure API credentials
- Injected them into your application's environment
- All without you touching a single API key or endpoint URL!

### Step 4: Test AI-Powered Chat

1. Refresh your browser
2. Click the Chat button (💬) to verify the model is connected
   - You should see: Chat Model: [your-model-name] ✅
3. Send a message like "What is Cloud Foundry?"

**What you'll see:** The AI will now provide intelligent responses about Cloud Foundry, demonstrating that the LLM is working correctly.

## Implementing RAG (Retrieval Augmented Generation)

RAG is a powerful technique that lets AI answer questions about YOUR documents. Instead of relying only on its training data, the AI can access and reference specific information you provide. Think of it as giving the AI a personal library card to your document collection.

### Step 5: Set Up Vector Database and Embeddings

To implement RAG, we need two key components:
- **Embeddings Model**: Converts text into numerical representations (vectors) that capture meaning
- **Vector Database**: Stores and searches these vectors to find relevant information quickly

Let's set them up:

```bash
# Check available embedding models
cf marketplace -s genai

# Create an embeddings model service (e.g., text-embedding-3-small)
cf create-service genai tanzu-nomic-embed-text-v1025 embeddings-llm

# Create a PostgreSQL database with vector support
cf create-service postgres on-demand-postgres-db vector-db
```
**Note:** The database provisioning might take 2-3 minutes. You can check its status with:
```bash
cf service vector-db
```

```bash
# Bind both services to your application. Make sure to use your app name
cf bind-service ai-tool-chat-yourinitials embeddings-llm
cf bind-service ai-tool-chat-yourinitials vector-db

# Restart the application
cf restart ai-tool-chat-yourinitials
```


### Step 6: Upload and Query Documents

Now let's give your AI some documents to work with:

1. Click the **Documents** button (📄) on the right side
2. Verify both services are connected:
   - Vector Store: PgVectorStore ✅
   - Embed Model: text-embedding-3-small ✅
3. Click "Upload File" and select a PDF document
   - **Suggestion**: Use a technical document, user manual, or documentation PDF. You can also find spring health assessment pdf in our Lab's git repo.
   - The file will be processed and indexed automatically
   - You'll see a progress bar during upload

**Behind the scenes:**
1. Your PDF is split into digestible chunks
2. Each chunk is converted to a vector (a list of numbers representing its meaning)
3. These vectors are stored in PostgreSQL using the pgvector extension
4. When you ask a question, it's converted to a vector too
5. The database finds the most similar document chunks
6. These relevant chunks are given to the LLM as context

This all happens in milliseconds, thanks to Cloud Foundry's optimized infrastructure!

### Step 7: Test Document-Based Q&A

Send a question about your uploaded document:
- "What does this document say about [topic]?"
- "Can you summarize the main points?"

The AI will now answer based on the document content, providing accurate information even if the LLM wasn't originally trained on your specific document.

## Extending Capabilities with MCP (Model Context Protocol)

While LLMs are great at generating text, they can't naturally interact with external systems or access real-time data. MCP solves this by giving AI models the ability to use tools, invoking software functions in real time. This opens up possibilities like checking current data, querying databases, or calling APIs.

### Step 8: See the Limitation, Then Fix It

First, let's see what happens without MCP:

1. Click "Clear Files" in the Documents panel to remove distractions
2. Ask the chat: "What is the latest bitcoin price?"
3. The AI will explain it cannot access real-time information

Now let's give it the ability to tell bitcoin price!

### Step 9: Bind to a Bitcoin MCP Server

MCP servers are lightweight applications that expose specific tools to AI models. There's a bitcoin MCP server already deployed in your space. Let's bind your chat application to it. Then ask the same question about bitcoin price.

```bash
cf bind-service ai-tool-chat-yourinitials bitcoin-mcp-server

# Restart the chat application
cf restart ai-tool-chat-yourinitials
```



### When to Use A2A vs MCP

**Use MCP when:**
- You want your LLM to have access to tools and data sources
- The capability should be automatically invoked during conversation
- You need real-time data, API access, or system integration
- Example: Current time, database queries, file system access

**Use A2A when:**
- You want to consult a specialized AI agent
- You need a second opinion or specialized processing
- The agent has domain-specific expertise
- You want explicit agent responses shown to the user
- Example: Document summarization, translation, code analysis

**Use Both when:**
- You want your LLM to have tools (MCP) AND
- You want to directly consult specialized agents (A2A)
- Maximum flexibility and capability!

## Persistent Memory with Vector Storage

When both a vector database and embeddings model are available, your chat application gains a superpower: it remembers conversations even after restarts! This transforms a stateless application into one with long-term memory.

### Step 10: Experience Persistent Memory

1. Click the **Memory** button (🧠) to check your setup
   - Memory: Persistent ✅ (thanks to vector store + embeddings)
   - Note your Conversation ID (this links your chat history)

2. Have a personal conversation with the AI:
   - "My name is Alex and I work on Cloud Foundry automation"
   - "I'm particularly interested in AI/ML integration"
   - "My favorite programming language is Python"

3. Ask a question that requires context:
   - "Based on what you know about me, what AI projects might I enjoy?"

4. Restart the application:
   ```bash
   cf restart ai-tool-chat-yourinitials
   ```

5. Once restarted, open the app and ask:
   - "What do you remember about me?"
   - "What's my name and what do I work on?"

**How it works:**
- Each message is converted to a vector and stored with your conversation ID
- On restart, the app retrieves relevant past conversations from the vector database
- The LLM uses this retrieved context to maintain continuity
- Your conversation history persists as long as you use the same browser session

This is all handled automatically by Cloud Foundry's managed services—no session management code needed!

## 🎉 Congratulations!

You've built a sophisticated AI application using Cloud Foundry's platform services! Let's recap your achievements:

- ✅ **Deployed** an AI chat application with zero infrastructure setup
- ✅ **Connected** enterprise LLMs with simple service bindings
- ✅ **Implemented** RAG for intelligent document Q&A
- ✅ **Extended** your AI with tool-using capabilities via MCP
- ✅ **Connected** to specialized AI agents via A2A protocol
- ✅ **Enabled** persistent memory that survives restarts

### Your Application Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Angular UI     │────▶│ Spring Boot  │────▶│  Chat LLM       │
│  - Chat         │     │  Backend     │     │  (GPT/Claude)   │
│  - Documents    │     │              │     └─────────────────┘
│  - Agents       │     │              │
│  - Memory       │     │              │
└─────────────────┘     └──────┬───────┘
                               │
                    ┌──────────┼──────────┐
                    │          │          │
              ┌─────▼──────┐   │    ┌─────▼──────┐
              │ Vector DB  │   │    │ MCP Server │
              │ + Embeddings│   │    │  (Tools)   │
              └────────────┘   │    └────────────┘
                               │
                         ┌─────▼──────┐
                         │ A2A Agents │
                         │ (AI Agents)│
                         └────────────┘
```

All managed by Cloud Foundry—you just focus on your application logic!

### Troubleshooting Tips

**Chat not responding?**
- Check the Chat panel (💬) - ensure a model shows as available
- View logs: `cf logs ai-tool-chat --recent`
- Verify the service binding: `cf env ai-tool-chat`

**Documents not uploading?**
- Confirm both Vector Store and Embed Model show as available (📄)
- Check that your PDF is valid and under 10MB
- Ensure the database is fully provisioned: `cf service vector-db`



Cloud Foundry turned complex AI infrastructure into simple, composable services. You spent your time building features, not fighting infrastructure. That's the power of a true cloud platform!

Happy building! 🚀