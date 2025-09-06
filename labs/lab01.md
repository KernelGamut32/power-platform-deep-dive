# Lab 01: Plan, create, and automate Power Platform environments with Dataverse

> **You’ll practice**

> 1) Using **Dataverse** as the data platform  
> 2) Applying **environment best practices** (types, security groups, capacity)  
> 3) **Automating** environment creation with **PowerShell**

---

## Prerequisites

- You have the **Power Platform admin** or **Dynamics 365 admin** role in the tenant
- You have enough **Dataverse database capacity** to create a **Sandbox** or **Production** environment
- Windows PowerShell 5.1+ (or PowerShell 7+) on your machine.

---

## Section A — Dataverse essentials

**Goal:** Get hands-on with Dataverse tables to see the data platform used inside environments

1. Open your browser to **<https://make.powerapps.com>**.  
2. In the **environment picker** (top-right), select any environment you already have access to.
3. In the left navigation, select **Tables**.
4. Select **+ New table ▸ Create new tables**, click **Start from blank**, and configure:
   - **Display name:** `Course Registrations`  
   - Click **New column ▸ Edit column**
   - Configure:
      - **Display name:** `Registration Name`
      - **Data type:** `Single line of text`
      - **Format:** `Text`
   - Add columns (**+ New column**):
      - **Student Email** – *Text*
      - **Course Code** – *Text*
      - **Registration Date** – *Date only*
      - Select **Save and exit**.
5. Click **Course Registrations**
6. Add sample data - Add 2–3 rows (use realistic values).
7. Explore the table designer:
   - Show where to manage **Columns**, **Relationships**, **Views**, and **Forms**.  
   - (Instructor tip) Point out that Dataverse provides rich types, security, and ALM-friendly behavior.

**Checkpoint ✅** You can see your `Course Registrations` table with a few rows of sample data.

---

## Section B — Environments: create and secure one properly

**Goal:** Create a new environment with a Dataverse database and restrict access via a security group (best practice).

### B1) Create an Entra security group (≈5–7 min)

> You can use either the **Microsoft Entra admin center** or the **Microsoft 365 admin center**. Here we use Entra.

1. Go to **<https://entra.microsoft.com>**.  
2. Navigate to **Groups ▸ All groups ▸ + New group**.
3. **Group type:** *Security*  
   **Group name:** `PP-DEV-Students`  
   (Add a description if desired) → **Create**.
4. Open the new group → **Members ▸ + Add members** → add your student accounts (or test users).  
5. Note the group’s **Object ID** (you’ll need it for the PowerShell section).

### Notes

- You **can** assign a security group to most environment types, but **not** to the **Default** or **Developer** environment.
- Assigning a group ensures only intended members get access to the environment’s maker and Dataverse resources.

### B2) Create a new environment with a Dataverse database (≈10–13 min)

1. Open **<https://admin.powerplatform.com>** (Power Platform admin center, PPAC).  
2. Select **Environments** in the left navigation, then **+ New**.
3. Fill the first page:
   - **Name:** `DEV – Training`  
   - **Region:** choose your tenant’s region (e.g., *United States*)  
   - **Type:** **Sandbox** (recommended for dev/test). If you lack capacity, choose **Trial**.  
   - **Purpose:** `Hands-on lab environment`  
   - **Add a Dataverse data store:** **Yes**  
   - (If prompted for billing) Leave **Pay-as-you-go/Azure subscription** **Off** for this lab.
   - Select **Next** (or **Create** then continue into database configuration, depending on UI).
4. Database settings:
   - **Language** and **Currency** (choose appropriate defaults)  
   - **URL** (unique name for the environment)  
   - **Enable Dynamics 365 apps:** **No** for this lab *(This choice is not reversible later)*  
   - **Deploy sample apps and data:** **No**  
   - **Security group:** select **PP-DEV-Students** (created above)  
   - Select **Save** / **Create**.
5. Wait for the environment status to become **Ready** (refresh if needed).

**Checkpoint ✅** You have an environment `DEV – Training` with a Dataverse database and a security group assigned.

> **Instructor tip:** Briefly show **Settings ▸ Users + permissions** and **Dataverse ▸ Security roles** so students see the difference between environment access (via group) and app/data permissions (via roles/teams).

---

## Section C — Automate environment creation & governance with PowerShell

**Goal:** Script environment creation (with a Dataverse database) and verify your work.

> You’ll use the **Microsoft.PowerApps.Administration.PowerShell** and **Microsoft.PowerApps.PowerShell** modules.

### C1) Install modules and sign in (first time only)

Open **PowerShell** (Run as Administrator is recommended for module install):

```powershell
# Install (first time); harmless to re-run with -Force
Install-Module -Name Microsoft.PowerApps.Administration.PowerShell -Scope CurrentUser -Force
Install-Module -Name Microsoft.PowerApps.PowerShell -Scope CurrentUser -Force

# Sign in (opens a browser prompt)
Add-PowerAppsAccount
```

### C2) Discover valid locations

```powershell
# See available regions/locations for environment creation
Get-AdminPowerAppEnvironmentLocations |
  Sort-Object Name |
  Select-Object Name, Location
```

Pick a value from the Location column (e.g., unitedstates) for the next step.

### C3) Create a Sandbox environment with a Dataverse database

The `-ProvisionDatabase` switch provisions the Dataverse database at creation time.
If you create an environment without a database, you can add one later using `New-AdminPowerAppCdsDatabase`.

```powershell
# --- Customize these variables ---
$displayName     = "DEV – Training (Automated)"
$location        = "unitedstates"       # Use a value from C2
$envSku          = "Sandbox"            # Alternatives: Production, Trial, Developer, etc.
$language        = "English"            # e.g., English
$currency        = "USD"                # e.g., USD
$securityGroupId = "<ObjectId of PP-DEV-Students>"  # From Section B1, step 5
# ---------------------------------

# Create environment WITH Dataverse database
$env = New-AdminPowerAppEnvironment `
  -DisplayName $displayName `
  -LocationName $location `
  -EnvironmentSku $envSku `
  -ProvisionDatabase `
  -LanguageName $language `
  -CurrencyName $currency `
  -SecurityGroupId $securityGroupId `
  -WaitUntilFinished $true

$env
```

Optional (if you created the env without a DB)

```powershell
# Add the Dataverse database after environment creation
New-AdminPowerAppCdsDatabase `
  -EnvironmentName $env.EnvironmentName `
  -LanguageName $language `
  -CurrencyName $currency
```

### C4) Verify (and optional cleanup)

```powershell
# List environments you administer
Get-AdminPowerAppEnvironment |
  Select-Object DisplayName, EnvironmentName, EnvironmentSku, Location

# (Optional) Remove a lab environment later
# Remove-AdminPowerAppEnvironment -EnvironmentName <name from the list above>
```

---

## Debrief & Best Practices

- Use separate Dev/Test/Prod environments and keep Default for personal productivity/prototyping.
- Choose the right environment type:
  - Sandbox (dev/test), Production (live), Trial (short-term evals), Developer (per-maker).
- Assign an Entra security group to control who can access the environment (not supported for Default/Developer).
- Do not enable Dynamics 365 apps unless you intend to deploy Dynamics workloads; the choice is not reversible.
- Consider Pay-as-you-go (Azure subscription) for project-scoped environments that want metered billing.

---

## Troubleshooting

- I don’t see the New environment button: Ensure you have Power Platform admin or Dynamics 365 admin role.
- Capacity error when creating Sandbox/Production: Use Trial for the lab or free up database capacity.
- Security group can’t be set: This is expected for Default and Developer environments.
- PowerShell cmdlet not found: Reopen PowerShell after installation or run `Import-Module Microsoft.PowerApps.Administration.PowerShell`.
- Environment stuck provisioning: Refresh PPAC, confirm region choice, and verify service health in Microsoft 365 admin center ▸ Health.
