# Lab 02: Modeling Data in Dataverse — Tables, Columns, and Relationships

## What you’ll build

A **Training Management** data model using Microsoft Dataverse:

- Custom tables for **Course**, **Session**, and **Enrollment**
- A mix of **standard** tables (**Account**, **Contact**) and **custom** tables
- Columns across data types, plus **Formula** (calculated) and **Rollup** columns
- Relationships: **1:N**, **N:N**, **self-referencing hierarchy**, cascade behaviors
- A **polymorphic (multi-table) “Customer”** lookup to **Account _or_ Contact**

> 🧭 This lab uses the Power Apps **maker portal** at `https://make.powerapps.com`.

---

## Prerequisites

- Environment with a **Dataverse database**
- Security role with maker/customizer rights (e.g., **System Administrator**)
- Familiarity with Solutions and model-driven apps

---

## Part 0 — Create a solution

1. In the maker portal, confirm you’re in the correct **Environment** (top-right).
2. Go to **Solutions** → **+ New solution**  
   - **Display name:** _Dataverse Data Modeling Lab_  
   - **Publisher:** Use an existing publisher (or create a new one)  
   - **Version:** `1.0.0.0` → **Create**
3. Open the solution to keep all lab components together.

---

## Part 1 — Tables: standard vs. custom (8–10 min)

### 1A) Inspect standard tables

1. In your solution, **+ New** → **Table** → search for and **Add existing table**:
   - **Account**, **Contact** (these are **standard** system tables).
2. Open **Account** → **Columns** to scan common system columns (e.g., _Primary Name_, _Owner_).  
   > Note: Adding a **lookup** column automatically creates a **1:N relationship**. Microsoft Learn lists the supported relationship types as **one-to-many** and **many-to-many** (there’s no native 1:1). :contentReference[oaicite:0]{index=0}

### 1B) Create custom tables

Create three **custom** tables in your solution:

- **Course**  
  - **Primary column (Text):** _Course Name_  
  - Uncheck **“Enable attachments (notes)”** (optional) → **Save**

- **Session**  
  - **Primary column (Text):** _Session Name_ → **Save**

- **Enrollment**  
  - **Primary column (Text):** _Enrollment Number_ (or use Autonumber if you prefer) → **Save**

---

## Part 2 — Columns: data types, formula (calculated), and rollup (15–20 min)

> This section uses **Formula** columns (Power Fx) for calculated values, and **Rollup** columns for aggregates. Formula columns are first-class in Dataverse and support Power Fx; see guidelines (length/depth, function limits) in Learn. :contentReference[oaicite:1]{index=1}

### 2A) Add core columns (various data types)

**Course** (open table → **Columns** → **+ New column**):

- **Base Price** — _Currency_
- **Capacity** — _Whole number_
- **Is Virtual?** — _Yes/No_
- **Difficulty** — _Choice_ (create simple choices: _Intro_, _Intermediate_, _Advanced_)

**Session**:

- **Start Date** — _Date and time_ (behavior _Date Only_)
- **Duration (days)** — _Whole number_ (Default 2)

**Enrollment**:

- **Status** — _Choice_ (e.g., _Pending_, _Confirmed_, _Cancelled_)

### 2B) Create a **Formula** (calculated) column

**Session** → **Columns** → **+ New column**:

- **Display name:** _End Date (calc)_  
- **Data type:** **fx Formula**  
- **Formula:**  

`DateAdd(ThisRecord.'Start Date', ThisRecord.'Duration (days)', TimeUnit.Days)`

- **Save**

> Tip: Formula columns use Power Fx; see official limits (e.g., max expression length 1,000 chars, depth 10). Currency handling has caveats; review Learn notes if using currency in formulas. :contentReference[oaicite:2]{index=2}

### 2C) Create a **Rollup** column

**Course** → **Columns** → **+ New column**:

- **Display name:** _Enrollment Count (rollup)_  
- **Data type:** _Whole number_  
- **Column type:** **Rollup** → **Edit** rollup:
- **Source table:** _Course_ (no hierarchy)
- **Related table:** _Enrollment_ (via the relationship you’ll add in Part 3; you can return to complete this after 3A if editor doesn’t yet show it)
- **Aggregation:** **COUNT** of related _Enrollment_ where **Status = Confirmed**
- **Save** the rollup definition → **Save** column

> About rollups: They’re recalculated by **system jobs** (one **Mass Calculate** job, then recurring **Calculate Rollup Field** job ~**hourly**). You can **manually Recalculate** from the form. Online recalculation is limited to **50,000** related rows; hierarchical depth limit **10**. Max **50 rollups per table** (and default **200 per environment**). :contentReference[oaicite:3]{index=3}

---

## Part 3 — Relationships: 1:N, N:N, hierarchy & cascade (15–18 min)

### 3A) **1:N** (Course → Session) and **Session → Enrollment**

Create two lookups (child → parent), which creates **1:N**:

1. **Session** → **Columns** → **+ New column**

- **Display name:** _Course_  
- **Data type:** **Lookup** → **Related table:** _Course_ → **Save**

2. **Enrollment** → **Columns** → **+ New column**

- **Display name:** _Session_  
- **Data type:** **Lookup** → **Related table:** _Session_ → **Save**

> Creating a lookup adds the underlying **1:N** (and reverse **N:1**) relationship automatically. :contentReference[oaicite:4]{index=4}

### 3B) Configure **cascade** behavior (complex behaviors)

1. **Course** → **Relationships** → open the **Course (one) → Session (many)** relationship.  
2. Expand **Advanced options** → **Type of behavior**:

- Set to **Referential, Restrict Delete** (prevents deleting a Course if Sessions exist).

3. **Session (one) → Enrollment (many)**:

- Set to **Parental** (deleting a Session **deletes** its Enrollments).

4. **Save** each relationship.

> Behavior types: **Parental** (cascades all actions), **Referential**, **Referential, Restrict Delete**, and **Configurable (Custom)** with per-action choices like _Cascade All/Active/None_ and _Remove Link/Restrict_ on Delete. :contentReference[oaicite:5]{index=5}

### 3C) **N:N** (many-to-many) — Course ↔ Contact (Instructors)

1. **Course** → **Relationships** → **+ New relationship** → **Many-to-many**  

- **Related table:** _Contact_  
- **Name:** _course_contact_instructor_ → **Save**

2. Later, you’ll use the associated grid to add/remove **Instructor** contacts on a Course.

### 3D) **Self-referencing hierarchy** (complex)

1. **Course** → **Columns** → **+ New column**  

- **Display name:** _Parent Course_  
- **Data type:** **Lookup** → **Related table:** _Course_ → **Save**

2. **Course** → **Settings** (… menu) → enable **Hierarchical** if available, or use the out-of-box hierarchy settings in the environment.  
3. Add **Parent Course** to the Course main form.

> Hierarchies work over a **self-referencing 1:N**; rollups can optionally aggregate over hierarchies (mind the depth/row limits if you toggle that later). :contentReference[oaicite:6]{index=6}

---

## Part 4 — Polymorphic lookups (Customer to Account/Contact) (6–8 min)

Create a **Customer** type column on **Enrollment** so an enrollment can be tied to either an **Account** or a **Contact**.

1. **Enrollment** → **Columns** → **+ New column**  

- **Display name:** _Customer_  
- **Data type:** **Customer** (multi-table lookup: Account or Contact) → **Save**

2. Add **Customer** to the Enrollment main form.

> **Customer** is a native **polymorphic** lookup (multi-table) in Dataverse (like **Owner**), and tables can include zero, one, or more Customer columns. :contentReference[oaicite:7]{index=7}

> 🔎 FYI: Dataverse supports **1:N** and **N:N** relationships. A true **1:1** isn’t a native type; it’s modeled via patterns (e.g., required 1:N plus custom logic). Keep this in mind if your requirements call for strict 1:1. :contentReference[oaicite:8]{index=8}

---

## Part 5 — Model-driven app to verify the model (8–10 min)

1. In your **solution** → **+ New** → **App** → **Model-driven app** (modern)  

- **Name:** _Training Management_ → **Create**

2. In the app designer, **Add pages**:

- **Course**, **Session**, **Enrollment**, **Account**, **Contact**

3. **Course** main form:

- Add fields: _Course Name, Base Price, Capacity, Is Virtual?, Difficulty, Enrollment Count (rollup), Parent Course_  
- Add a **related** subgrid for **Sessions** (shows the 1:N)
- Add a **related** subgrid for **Contacts** (Instructors) from the N:N relationship

4. **Session** main form:

- Add: _Session Name, Course, Start Date, Duration (days), End Date (calc)_  
- Add a **related** subgrid for **Enrollments**

5. **Enrollment** main form:

- Add: _Enrollment Number, Session, Customer, Status_

6. **Publish** the app and **Play**.

---

## Part 6 — Enter data & validate behaviors (10–12 min)

### 6A) Seed data

1. **Accounts**: create _Adventure Works_.
2. **Contacts**: create _Ava Adams_ (Adventure Works), _Ben Brown_ (no account).
3. **Course**:  

- _Power Platform Basics_ — Base Price `500`, Capacity `20`, Is Virtual `Yes`  
- _Power Platform Advanced_ — Base Price `900`, Capacity `15`, Is Virtual `No`

4. **Self-reference**: set _Advanced_ → **Parent Course** = _Basics_.
5. **Sessions** (for _Basics_):  

- _Basics – Cohort 1_: Start Date today, Duration 2  
- _Basics – Cohort 2_: Start Date next week, Duration 3

6. **N:N Instructors**: open _Power Platform Basics_ → **Contacts** tab/grid → **+ New existing** → select **Ava Adams** (now she’s an Instructor).
7. **Enrollments**:  

- Open _Basics – Cohort 1_ → **+ New Enrollment**:  
  - _Enrollment Number:_ `E-0001`  
  - _Customer:_ **Adventure Works** (Account)  
  - _Status:_ **Confirmed**  
- Add another: _Customer:_ **Ben Brown** (Contact), _Status:_ **Confirmed**

### 6B) Test **Formula** & **Rollup**

1. Open **Session** _Basics – Cohort 1_ → verify **End Date (calc)** = Start Date + Duration.
2. Open **Course** _Power Platform Basics_ → locate **Enrollment Count (rollup)**.

- Click the **calculator** icon near the field → **Recalculate** to update on demand (rather than waiting for background jobs). :contentReference[oaicite:9]{index=9}
- Confirm it shows **2**.

### 6C) Test **cascade** behaviors

1. Try to delete **Course → Power Platform Basics**  

- Expect **blocked delete** (because **Course → Session** is _Referential, Restrict Delete_).

2. Delete **Session → Basics – Cohort 1**  

- Confirm related **Enrollments** are **deleted** (Parental on Session→Enrollment).

### 6D) Polymorphic lookup behavior

1. Open either **Enrollment** you created.
2. Change **Customer** from _Adventure Works (Account)_ to _Ben Brown (Contact)_ and **Save**.  

- Verify the same column supports **either** table type.

---

## Instructor notes & talking points

- **Calculated vs. Formula vs. Rollup**  
- Modern guidance favors **Formula** (Power Fx) columns for business calculations; review limits (e.g., depth, currency caveats). **Rollup** aggregates are computed by **system jobs** (~hourly), can be **manually recalculated**, and have **row/depth** limits for the manual recalculation path. :contentReference[oaicite:10]{index=10}
- **Relationship types**  
- Dataverse supports **1:N** and **N:N** (no native **1:1**). If you truly require 1:1, discuss patterns (e.g., one child record per parent enforced by business logic or plugins). :contentReference[oaicite:11]{index=11}
- **Cascade behavior**  
- Show how **Referential (Restrict Delete)** prevents deletes; **Parental** cascades deletes; **Custom** lets you fine-tune Assign/Share/Unshare/Reparent/Delete/Merge. :contentReference[oaicite:12]{index=12}
- **Polymorphic lookups**  
- Use **Customer** or **Owner** as native examples; they are **multi-table** lookups (e.g., Account or Contact; User or Team). :contentReference[oaicite:13]{index=13}

---

## Clean up (optional, 2–3 min)

- Delete test records from **Enrollment**, **Session**, and **Course**.
- Keep the solution if you plan to extend it; otherwise **export** or **delete**.

---

## Checklist (quick review)

- [ ] Created **Course**, **Session**, **Enrollment** (custom) and added **Account**, **Contact** (standard)
- [ ] Added variety of **column** types, plus **Formula** and **Rollup**
- [ ] Built **1:N** (Course→Session, Session→Enrollment), **N:N** (Course↔Contact), **self-referencing** (Course→Course)  
- [ ] Set **cascade** behaviors and validated **Restrict Delete** and **Parental** effects
- [ ] Added **Customer** (polymorphic) on **Enrollment** and verified Account/Contact switching
- [ ] Verified **formula** and **rollup** results (including manual **Recalculate**)

---

## Extension ideas (if you finish early)

- Add a **rollup** on **Course** to **AVG** Session Duration across its Sessions. (Remember: rollups only aggregate over **1:N**.) :contentReference[oaicite:14]{index=14}
- Add a **model-driven app chart** grouping Enrollments by **Customer**.
- Add **business rules** to default Session **Duration** based on Course **Difficulty**.
