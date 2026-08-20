# Setup Guide — Hotel Creation Automation

This guide walks you through every step required to configure and run the Hotel Creation Automation in Postman.

---

## System Requirements

| Item | Requirement |
|------|------------|
| Operating System | Windows 10+, macOS 10.14+, or Ubuntu 18.04+ |
| Postman | v9.0 or later |
| Node.js (optional) | v14 LTS or later (for Newman CLI) |
| Network | Access to `aks-crs-qa.hospitalityrevolution.com` |

---

## Step 1 — Install Postman

1. Visit [https://www.postman.com/downloads/](https://www.postman.com/downloads/).
2. Download the installer for your operating system.
3. Run the installer and follow the on-screen instructions.
4. Launch Postman and sign in (or continue without signing in).

---

## Step 2 — Clone the Repository

```bash
git clone https://github.com/VishalPalaniAgilysys/Vishal-Repository.git
cd Vishal-Repository/postman-automation
```

---

## Step 3 — Import the Collection

1. Open Postman.
2. Click **Import** (top-left toolbar).
3. Drag and drop `collections/Hotel_Creation_Automation.postman_collection.json` into the import dialog, or click **Upload Files** and browse to the file.
4. Click **Import**.
5. You should now see **Hotel Creation Automation** listed under *Collections* in the left sidebar.

---

## Step 4 — Import the Environment

1. Click **Import** again.
2. Upload `environments/Hotel_Creation_Environment.postman_environment.json`.
3. Click **Import**.
4. Verify that **Hotel Creation Environment** appears under *Environments* (the globe icon in the left sidebar).

---

## Step 5 — Configure Authentication

1. Click the **Environments** icon (🌐) in the left sidebar.
2. Select **Hotel Creation Environment**.
3. Find the `bearerToken` variable.
4. Click the value field and replace `YOUR_BEARER_TOKEN_HERE` with your actual ******
5. Click **Save** (Ctrl+S / Cmd+S).

> ⚠️ **Security:** Never commit real tokens to source control. The `bearerToken` variable is marked as `secret` so Postman masks its value.

---

## Step 6 — Review Environment Variables

Open the environment editor and confirm the following values match your target environment:

| Variable | Expected Value |
|----------|---------------|
| `baseUrl` | `https://aks-crs-qa.hospitalityrevolution.com` |
| `apiVersion` | `v1.0` |
| `corporateId` | Your corporate ID (default: `1`) |
| `brandId` | Your brand ID (default: `10`) |
| `regionId` | Your region ID (default: `200`) |
| `createdBy` | Your user ID (default: `42`) |
| `sourcePassword` | Source system password — **must be set before running** |

---

## Step 7 — Prepare the Test Data

The data files are already included in the `data/` folder:

- `hotel_test_data.csv` — 4 sample hotels (recommended for Collection Runner)
- `hotel_test_data.json` — same data in JSON format

To add more hotels, append rows to the CSV or objects to the JSON array — no changes to the collection are needed.

---

## Step 8 — Run with Collection Runner

1. Click **Runner** (top-right, looks like a play button with lines).
2. Drag **Hotel Creation Automation** into the runner, or select it from the list.
3. Select **Hotel Creation Environment** from the environment dropdown.
4. Under **Iterations**, enter the number of hotels in your data file (e.g., `4`).
5. Under **Data**, click **Select File** and choose `data/hotel_test_data.csv`.
6. Click **Run Hotel Creation Automation**.
7. Watch the results panel — each request shows pass/fail status.
8. After all iterations, open the Postman Console (View → Postman Console) to see the summary report.

---

## Step 9 — Run with Newman CLI

```bash
# Install Newman globally
npm install -g newman

# Run the collection
newman run collections/Hotel_Creation_Automation.postman_collection.json \
  --environment environments/Hotel_Creation_Environment.postman_environment.json \
  --iteration-data data/hotel_test_data.csv \
  --env-var "bearerToken=<YOUR_TOKEN>" \
  --reporters cli,json \
  --reporter-json-export results/hotel_creation_results.json
```

---

## Step 10 — Verify Results

### In Postman Console

Open **View → Postman Console** (or Alt+Ctrl+C). Look for the summary block:

```
========================================
       HOTEL CREATION SUMMARY REPORT    
========================================
Total Processed : 4
✓ Successful    : 4
✗ Failed        : 0
```

### In Newman Output

Newman prints a table of all tests with pass/fail status. The JSON report is saved to `results/hotel_creation_results.json`.

### Via Environment Variables

After a run, `createdHotels` and `failedHotels` environment variables contain JSON arrays with the outcome of each hotel.

---

## Common Configurations

### Targeting a Different Environment

Change `baseUrl` in the environment to point to staging or production:

| Target | baseUrl |
|--------|---------|
| QA | `https://aks-crs-qa.hospitalityrevolution.com` |
| Staging | `https://aks-crs-staging.hospitalityrevolution.com` |
| Production | `https://aks-crs.hospitalityrevolution.com` |

### Changing Default Address

Update `defaultAddressLine1`, `defaultAddressLine2`, `defaultCity`, and `defaultStateProvince` in the environment.

### Changing Payment Types

Update `paymentTypes` with a comma-separated list, e.g., `CreditCard,Cash,BankTransfer`.
