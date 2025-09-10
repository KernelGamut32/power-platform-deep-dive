# Lab 04 – Getting Started with Copilot Studio

## Duration

45–60 minutes

## Learning Objectives

- Understand the Copilot Studio interface and architecture.  
- Build and test your first Copilot.  
- Explore core use cases and how copilots fit into the Power Platform.  

---

## Step-by-Step Instructions

### Part A – Explore Copilot Studio (10–15 min)

1. **Sign in** to [Copilot Studio](https://copilotstudio.microsoft.com) with your Microsoft 365 credentials.  
2. On the **Home page**, review the following:
   - The **left navigation panel**: Home, Copilots, Data, Settings, Analytics.  
   - The **environment selector** in the top-right corner (note how environments connect to Dataverse).  
3. Open the **Architecture Overview** page (Help → Documentation → Architecture) and review how copilots use:
   - Topics and conversation flows.  
   - Natural language understanding (NLU).  
   - Integration with Power Automate and Dataverse.  

### Part B – Create Your First Copilot (20 min)

1. From the **Copilots** tab, select **+ New Copilot**.  
2. Provide the following details:  
   - **Name**: “Training Assistant Copilot”  
   - **Language**: English  
   - **Environment**: Use your training environment.  
3. After creation, explore the **Copilot canvas** where you’ll build conversations.  
4. Add a **Topic**:  
   - Name: “Greeting”  
   - Trigger phrases: “Hi”, “Hello”, “Good morning”, “Good afternoon”  
   - Add a message node: “Hello! I’m your Training Assistant Copilot. How can I help today?”  
   - Save and test it in the **Test Bot** pane on the right.  
5. Add another **Topic**:  
   - Name: “Course Info”  
   - Trigger phrases: “Tell me about courses”, “What training is available?”  
   - Add a message node: “We offer courses on Copilot Studio, Power Automate, and Dataverse. Which would you like to know more about?”  
   - Test in the **Test Bot** window.  

### Part C – Publishing Your Copilot (10–15 min)

1. In the top navigation, click **Publish**.  
2. Review the deployment options:
   - **Web link** for testing.  
   - **Embed** in a website.  
   - **Teams app** deployment.  
3. Copy the **web link** and test the Copilot in a browser tab.  
4. Reflect: discuss how this could be used in customer service, HR, or IT helpdesk scenarios.  
