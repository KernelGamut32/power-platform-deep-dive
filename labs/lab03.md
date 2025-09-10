# Lab 03: Dataverse Security Deep Dive (Business Units, Roles, Teams, CLS, Sharing)

**Duration:** 45–60 minutes  
**Audience:** Makers / Admins who need to design and verify Dataverse security  
**You’ll practice:** Business units, security roles, user/team assignment, column-level security (CLS), app visibility, and record-level sharing

## Learning outcomes

By the end, students will be able to:

1. Explain how Business Units (BUs) scope record access.  
2. Create and tailor Security Roles for least privilege.  
3. Use Owner Teams and Entra ID (Azure AD) Group Teams to assign permissions at scale.  
4. Protect sensitive columns with Column-Level Security.  
5. Control a model-driven app’s visibility with app-level security roles.  
6. Grant one-off access with record-level sharing and confirm the effect.

---

## Prerequisites

- An environment with Dataverse (e.g., a Trial or Dev environment) where you are **System Administrator**.
- At least **two test users** in your tenant (e.g., `learnerA@...`, `learnerB@...`).  
  - *If you only have one account:* you can still complete every step and simulate another user by assigning roles to an Owner Team and temporarily adding/removing yourself from that team to “mimic” access changes.

> **Where you’ll click:**  
> **Power Platform admin center:** `admin.powerplatform.microsoft.com`  
> **Maker portal:** `make.powerapps.com`

---

## Scenario

You work at “Contoso Projects.” Two regional groups (East and West) manage **Projects** and **Timesheets**.

- **Project Managers** need full control over **Project** records *in their region*.  
- **Consultants** should create and edit *only their own* **Timesheets**.  
- The **Billing Rate** column is sensitive: only **Billing Admins** can read it; others see it masked.  
- Only the right users should **see the app**.  
- Sometimes managers must grant a peer access to *just one* project—use **sharing**.

---

## Part 0 — Quick Setup (Solution + Tables + Sample Data) (5–10 min)

1) **Create a solution (unmanaged)**
   - Go to **make.powerapps.com** → top bar **Solutions** → **+ New solution**  
   - **Name:** `DV Security Lab` | **Publisher:** Your default | **Version:** `1.0.0.0` → **Create** → **Open** the solution.

2) **Create tables** (User or team owned!)
   - In the solution → **+ New** → **Table**  
     - **Display name:** `Project` → **Ownership:** *User or team* → **Save**.  
       Add columns:
       - `Project Name` (Text) - edit existing  
       - `Client Name` (Text)  
       - `Billing Rate` (Currency) → **Enable column security** (you can toggle this after creation—see Part D)  
     - **Display name:** `Timesheet` → **Ownership:** *User or team* → **Save**.  
       Add columns:
       - `Timesheet Name` (Text) - edit existing 
       - `Hours` (Whole Number)  
       - `Work Date` (Date only)

3) **Create sample records**
   - In **Tables** list (still inside the solution) → open **Project** → **Data** → **+ New row**:  
     - `Project`: “Apollo (East)”, `Client Name`: “AlphaCo”, `Billing Rate`: `225`  
     - `Project`: “Zephyr (West)”, `Client Name`: “BetaCorp”, `Billing Rate`: `195`  
   - Open **Timesheet** → **Data** → **+ New row** (a couple of rows with Hours: 4–8, Work Date: this week).

4) **Save and exit**

---

## Part A — Business Units & Users (5–10 min)

> **Why:** BU membership + security role privilege depth (User/Business Unit/Parent:Child/Org) determine *how far* a user can see/edit records.

1) **Create Regional BUs**
   - Go to **admin.powerplatform.microsoft.com** → **Environments** → pick your lab environment → **Settings** (at top).  
   - **Users + permissions** → **Business units** → **+ New business unit**.  
     - Create: `East BU` (Parent: Root), then `West BU`.

2) **Move users into BUs** (optional if you only have one account)
   - **Users + permissions** → **Users** → open a test user (e.g., learnerA) → **Change business unit** → pick `East BU`.  
   - Move the second user (learnerB) to `West BU`.  
   - *Note:* Changing a user’s BU removes their roles; you’ll (re)assign roles in Part C.

---

## Part B — Security Roles (least privilege) (10–12 min)

> **Tip:** Build roles for *what users do*, not for who they are. Keep privileges narrow; grant more via teams if needed.

Create **three roles** inside the environment:

1) **Project Manager (Regional)**
   - **Users + permissions** → **Security roles** → **+ New role** → name it `Project Manager (Regional)` assigned to root BU.  
   - On the **Tables** tab, set privileges for the **Project** table:  
     - **Read/Write/Append/Append To/Assign/Share:** **Business Unit** scope.  
     - **Create:** **Business Unit**.  
   - On **Timesheet**, grant **Read** at **Business Unit** (for oversight), but **no write** unless required.  
   - Save.

2) **Consultant (Self Timesheets)**
   - **+ New role** → `Consultant (Self Timesheets)` assigned to root BU.  
   - On **Timesheet**: **Create/Read/Write/Append/Append To** at **User** scope.  
   - On **Project**: **Read** at **User** or **None** (choose **User** so they can see projects they own or are shared).  
   - Save.

3) **Billing Admin (Sensitive Read)**
   - **+ New role** → `Billing Admin` assigned to root BU.  
   - On **Project**: at least **Read** at **Org** (or **BU** if you want regional billing admins).  
   - This role will be paired with a **Column Security Profile** (Part D) to see the sensitive column.  
   - Save.

> **Verification checkpoint:** Open the **Security roles** list and confirm all three exist.

---

## Part C — Teams for Scalable Assignment (Owner Team + Entra Group Team) (8–10 min)

> **Why:** Assign roles to a team once, then manage membership centrally.

1) **Create an Owner Team (East PMs)**
   - **Users + permissions** → **Teams** → **+ New team**  
     - **Name:** `East PMs` | **Type:** *Owner* | **Business unit:** `East BU` | **Administrator:** admin account → **Save**.  
     - **Manage security roles** → add `Project Manager (Regional)`.  
     - **Members** → **+ Add** → add your East user (learnerA).  

2) **Create an Owner Team (Billing)**
   - **Users + permissions** → **Teams** → **+ New team**  
     - **Name:** `Billing Admins` | **Type:** *Owner* | **Business unit:** `East BU` | **Administrator:** admin account → **Save**.  
     - **Manage security roles** → add `Billing Admin`.  
     - **Members** → **+ Add** → add your West user (learnerB).  

3) **Create an Entra (Azure AD) Group Team (West PMs)** *(if you can create groups)*
   - Navigate to **entra.microsoft.com**
   - When prompted for MFA, choose **Postpone MFA**
   - Click **Confirm postponement** and click **Continue sign-in without MFA**
   - In Entra ID, create a **Security group** `West PMs Group`, add `learnerB` as a member.  
   - Back in **Teams** → **+ New team**  
     - **Name:** `West PMs (AAD)` | **Type:** *Microsoft Entra ID Security Group* | **Business unit:** `West BU` | **Group:** select your Entra group | **Membership type:** `Members` | **Administrator:** admin account → **Save**.  
   - Open the team → **Security roles** → add `Project Manager (Regional)`.

4) **Consultant assignment**
   - Create team `Consultants` (Owner team, Business unit of your choice).  
   - Assign the `Consultant (Self Timesheets)` role to the team.  
   - Use admin account for **Administrator**.
   - Add whichever user(s) should act as consultants.

> **Verification checkpoint:**  
> In **Users**, open `learnerA` and `learnerB` → **Security roles** tab shows effective roles via their team(s).  
> In **Teams**, confirm membership and attached roles.

---

## Part D — Column-Level Security (protect “Billing Rate”) (7–8 min)

> **Goal:** Only Billing Admins can view “Billing Rate” on Project.

1) **Enable column security** (if not already)
   - Maker portal → **Solutions** → open `DV Security Lab` → open **Project** table → **Columns** → open `Billing Rate`.  
   - Ensure **Enable column security** is **On** → **Save** and **Publish** all.

2) **Create a Column Security Profile**
   - Admin center → **Users + permissions** → **Column security profiles** → **+ New** → `Billing Rate Viewers`.  
   - Open it → **Teams** → **+ Add** → add the **Billing Admin** team or users who should see it.  
   - **Column permissions** → find **Project • Billing Rate** → set **Read** = **Allowed**, **Update** as desired (usually **Not allowed**), **Create** (not relevant here). **Save**.

> **Verification checkpoint:**  
> As a **non-Billing Admin** user, open any **Project** record in a model-driven app (you’ll build it next). Confirm **Billing Rate** is masked or access denied.  
> As a **Billing Admin** user (or a user in the `Billing Rate Viewers` profile), open the same record and verify the value is visible.

---

## Part E — App Visibility via Security Roles (5–7 min)

> **Why:** Even with table privileges, users shouldn’t see apps they don’t need.

1) **Create a model-driven app**
   - Maker portal → **Solutions** → open `DV Security Lab` → **+ New** → **App → Model-driven**.  
   - **Name:** `Contoso Projects` → **Create**.  
   - In the **Sitemap** (navigation), add **Areas/Groups/Subareas** to include **Project** and **Timesheet** tables. **Save** and **Publish**.

2) **Restrict app to roles**
   - From the solution, open your app → **... (More)** → **Manage roles**.  
   - Select only: `Project Manager (Regional)`, `Consultant (Self Timesheets)`, and anyone who needs to see it (e.g., `Billing Admin`). **Save**.

> **Verification checkpoint:**  
> Sign in as a user **without** those roles: the app should **not** appear in **Apps**.  
> Sign in as a `Consultant`: the app **appears** and opens.

---

## Part F — Record Ownership & Sharing (row-level access) (7–8 min)

> **Goal:** Show how owner scope + BU + sharing combine.

1) **Set ownership for demonstration**
   - In the app, open **Project** → open “Apollo (East)” → ensure **Owner** is an **East** user or team (e.g., `East PMs`).  
   - Open “Zephyr (West)” → set **Owner** to a **West** user/team (`Nestor Wilke`).

2) **Explore behaviors and access based on login**
   - Sign in as **learnerA** (East PM).  
     - You should **Read/Write** projects in your **East BU** (Apollo), but **not** see or edit West (Zephyr) beyond your role’s scope.  
   - Sign in as **learnerB** (West PM) and observe the reverse.

3) **Share one record cross-region**
   - Still as **learnerB** (West PM), open **Zephyr (West)** and share to **learnerA**  
   - Now **learnerA** refreshes and confirms **Apollo (East)** is visible with the granted permissions.

> **Verification checkpoint:**  
> Remove sharing and confirm access is lost.  
> Discuss how **Assign** vs **Share** differs: *Assign* changes owner; *Share* grants additional privileges without changing owner.

---

## Tips & Troubleshooting

- **Don’t see Business units / Security roles?** You might be in the wrong environment or lack **System Administrator**. Switch environment at the top right of the admin center or elevate your role.  
- **Column-level security not taking effect?** Ensure the column has **Enable column security** checked *and* the user isn’t in a profile that allows read. Publish customizations and refresh the app.  
- **App not visible to the right people?** Reopen the app’s **Manage roles** and confirm you granted roles that users actually hold (directly or via a team).  
- **Role depth matters:**  
  - **User** = only records the user owns (or that are shared/teamed).  
  - **Business Unit** = records owned by anyone in the user’s BU.  
  - **Parent:Child BU** = the user’s BU and any child BUs.  
  - **Organization** = all records in the environment.

---

## Quick Knowledge Check (use at the end)

1) A PM in **East BU** with **BU-level** privileges cannot see a **West-owned** record. What are two ways to allow access without changing ownership?  
   - **Answer:** Share the record; or add the PM to a team with appropriate privileges that owns/has access to the record.  
2) Why use **Column Security Profiles** if you already have table roles?  
   - **Answer:** To restrict **specific columns** (e.g., Billing Rate) even when users can view the rest of the record.  
3) What’s the difference between **Assign** and **Share**?  
   - **Answer:** **Assign** changes record **Owner**; **Share** grants access while **keeping the current Owner**.
