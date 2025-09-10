# Lab 05 – Advanced Copilot Studio Features and Integration

## Duration

45–60 minutes

## Learning Objectives

- Use topics, variables, and AI integration in conversation flows.  
- Connect Copilot Studio with Power Automate and Dataverse.  
- Understand Copilot management, deployment, and governance.  

---

## Step-by-Step Instructions

### Part A – Advanced Topics and Variables (15–20 min)

1. Open your **Training Assistant Copilot** from Lab 4.  
2. Create a new **Topic**:  
   - Name: “Course Enrollment”  
   - Trigger phrases: “I want to enroll”, “Sign me up”, “Register for training”  
3. Add a **Question node**:  
   - Question: “Which course would you like to enroll in?”  
   - Save response to a **variable**: `courseName`.  
4. Add a **Condition node**:
   - If `courseName` = “Copilot Studio” → Respond: “You’re enrolled in Copilot Studio!”  
   - Else → Respond: “Thanks! We’ll register you for the [courseName] course.”  
5. Test the topic and confirm variable capture in the **Test Bot** window.  

### Part B – AI Integration (10–15 min)

1. From the **AI Capabilities** menu, enable **Generative AI responses**.  
2. Add a new **Topic**:  
   - Name: “Ask Anything”  
   - Trigger phrases: “I have a question”, “Help me with AI”  
3. Add a **Generative Answer node** connected to your **Dataverse knowledge source** or an uploaded FAQ document.  
4. Test by asking: “What is Copilot Studio?” and observe the AI-generated response.  

### Part C – Integration with Power Automate and Dataverse (15–20 min)

1. Create a **Power Automate flow**:
   - Go to Power Automate → **+ Create → Automated Cloud Flow**.  
   - Name: “Save Enrollment Request”.  
   - Trigger: “When an HTTP request is received”.  
   - Action: Add row to Dataverse table **Training Registrations** (columns: Name, Course).  
   - Save and copy the HTTP endpoint.  
2. Back in **Copilot Studio**, update your **Course Enrollment** topic:  
   - After capturing `courseName`, add a **Call an action → Power Automate Flow**.  
   - Select your new flow and map `courseName` to the **Course** column.  
3. Test by entering “I want to enroll in Copilot Studio” → confirm data saved in Dataverse table.  

### Part D – Copilot Management, Deployment, and Governance (5–10 min)

1. Open the **Settings → Security & Governance** menu.  
2. Review how **roles and permissions** affect Copilot access.  
3. Explore the **Analytics** dashboard to monitor usage.  
4. Discuss **licensing considerations**:  
   - Differences between seeded Copilot functionality in M365 vs premium licensing.  
   - Impacts of Dataverse and AI Builder usage on costs.  
