# Intro to Tanzu Platform Workshop

**Building and Deploying Mission Ready AI Apps on Tanzu Platform**

This workshop is designed to give developers hands-on experience with Tanzu Platform for Cloud Foundry and show how quickly teams can move from source code to a running, AI-enabled application. Participants will deploy an application with `cf push`, explore the platform developer experience, and progressively add AI capabilities using managed platform services instead of managing infrastructure, credentials, model endpoints, vector databases, or runtime plumbing themselves.

## Agenda

1. **Introductions and Workshop Goals:** Meet the team, review the target use case, and align on what participants will build during the session.
2. **Tanzu Platform Overview:** Briefly introduce Tanzu Platform for Cloud Foundry, the developer workflow, the service marketplace, buildpacks, AI app capabilities and why platform services matter for mission-ready applications.
3. **Environment Onboarding:** Log in to Apps Manager and the CF CLI, target the assigned org and space, and run basic commands such as `cf apps`, `cf services`, `cf routes`, and `cf marketplace`.
4. **Deploy the Chat Application with `cf push`:** Download the workshop application artifact and manifest, customize the application name, push it to Tanzu Platform, and open the generated route in a browser.
5. **Explore the Baseline Application:** Use the chat interface before any AI services are bound and observe how the application reports missing chat, document, agent, MCP, and memory capabilities.
6. **Add an Enterprise LLM Service:** Create an AI Services instance, from the Marketplace, bind it to the application, restart the app, and test AI-powered chat without handling API keys or model endpoint configuration manually.
7. **Implement Document Q&A with RAG:** Create and bind an embeddings model and a PostgreSQL vector database, upload a PDF, and ask questions that are answered using document-specific context.
8. **Extend the Application with MCP Tools:** Demonstrate the limits of a standalone LLM, bind the application to an MCP server, and show how tool access enables real-time or external system interactions.
9. **Discuss A2A Agents and AI Integration Patterns:** Review when to use MCP tools, when to use Agent2Agent interactions, and how Tanzu Platform makes these capabilities composable for application teams.
10. **Architecture Recap and Q&A:** Review the final application architecture, connect the hands-on work back to enterprise delivery patterns, and discuss next steps for building AI-enabled applications on Tanzu Platform.

## Optional Stretch Goals

1.  Review Tanzu app onboarding process
2.  Demo of the Tanzu App Assessment and modernization tools


## Pre-Requisites

You will need a workstation with the following software and access prepared before the workshop. If a participant workstation cannot meet these requirements, a jumpbox with all software pre-installed will be provided to participants.  Jumpbox details will be provided to participants prior to the workshop.

1. Commercial internet access, including access to:
   - `github.com`
   - The provided Tanzu Platform Apps Manager URL
   - The provided Tanzu application route domain
2. Access to the workshop Tanzu Platform foundation, including:
   - Apps Manager URL
   - CF API endpoint
   - Assigned username and password
   - Assigned org and space
3. CF CLI installed locally.
4. `git` CLI installed.
5. A modern web browser.
6. Java 21+ and Maven 3.8+ installed locally.
7. A small PDF document for the RAG exercise. The workshop sample content can be used, or participants may bring an approved non-sensitive document.


