# 🏨 Postman Hotel Creation Automation

[![Postman](https://img.shields.io/badge/Postman-v9.0%2B-orange?logo=postman)](https://www.postman.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Prerequisites](#prerequisites)
4. [Quick Start](#quick-start)
5. [Detailed Setup](#detailed-setup)
6. [Usage](#usage)
7. [Data Format](#data-format)
8. [Field Mapping](#field-mapping)
9. [Configuration](#configuration)
10. [Troubleshooting](#troubleshooting)
11. [Advanced Usage](#advanced-usage)
12. [Contributing](#contributing)
13. [License](#license)
14. [Support](#support)

---

## Overview

This Postman automation solution enables **bulk hotel creation** through the CRS (Central Reservation System) API. It replaces the time-consuming, error-prone manual UI process with a data-driven, automated workflow that can create multiple hotel records in a single execution.

The collection reads hotel records from a CSV or JSON data file, constructs the correct API payload for each record, submits the request, validates the response, and produces a summary report — all without manual intervention.

---

## Features

- 📋 **Data-driven testing** — load hotels from CSV or JSON; add new records without touching the collection
- 🔄 **Bulk creation** — process unlimited hotel records in a single Collection Runner execution
- ✅ **Automatic validation** — status code, response time, required fields, and success-code checks per hotel
- 🆔 **Auto-generated IDs** — unique `x-transaction-id` (UUID) and `x-session-id` are generated for every request
- 📊 **Summary report** — consolidated pass/fail report printed to the Postman console after all iterations
- 🛡️ **Error handling** — failed hotels are captured with error code and message; execution continues for remaining records
- 🔧 **Configurable defaults** — environment variables control addresses, credentials, and system IDs without editing the collection
- 🚀 **Newman / CI-CD ready** — run headlessly from the command line or any CI pipeline

---

## Prerequisites

| Requirement | Version / Notes |
|-------------|----------------|
| [Postman](https://www.postman.com/downloads/) | v9.0 or later |
| ****** | Obtain from the CRS admin or your team lead |
| API Access | Network access to `aks-crs-qa.hospitalityrevolution.com` |
| Node.js *(optional)* | v14+ — required only for Newman CLI |

---

## Quick Start

1. **Clone** this repository  
   ```bash
   git clone https://github.com/VishalPalaniAgilysys/Vishal-Repository.git
   ```
2. **Import the collection** — open Postman → *Import* → select `collections/Hotel_Creation_Automation.postman_collection.json`
3. **Import the environment** — *Import* → select `environments/Hotel_Creation_Environment.postman_environment.json`
4. **Configure the token** — select the *Hotel Creation Environment*, edit `bearerToken`, and paste your token
5. **Run** — open Collection Runner, select the collection, choose `data/hotel_test_data.csv` as the data file, and click **Run**

---

## Detailed Setup

### 1. Install Postman

Download and install Postman from [https://www.postman.com/downloads/](https://www.postman.com/downloads/).

### 2. Import the Collection

1. Open Postman.
2. Click **Import** (top-left).
3. Select **File** and browse to `postman-automation/collections/Hotel_Creation_Automation.postman_collection.json`.
4. Click **Import**.

### 3. Import the Environment

1. Click **Import** again.
2. Select `postman-automation/environments/Hotel_Creation_Environment.postman_environment.json`.
3. Click **Import**.

### 4. Configure Authentication

1. In the top-right corner, select **Hotel Creation Environment** from the environment dropdown.
2. Click the **eye icon** to open the environment editor.
3. Find `bearerToken` and replace `YOUR_BEARER_TOKEN_HERE` with your actual token.
4. Click **Save**.

### 5. Verify Configuration

Review and update the following environment variables as needed:

| Variable | Default | Description |
|----------|---------|-------------|
| `baseUrl` | `https://aks-crs-qa.hospitalityrevolution.com` | API base URL |
| `corporateId` | `1` | Corporate identifier |
| `brandId` | `10` | Brand identifier |
| `regionId` | `200` | Region identifier |
| `createdBy` | `42` | User ID performing the creation |

---

## Usage

### Running with Collection Runner

1. Click **Runner** (top-right in Postman).
2. Select **Hotel Creation Automation** collection.
3. Select **Hotel Creation Environment**.
4. Under **Data**, click **Select File** and choose `data/hotel_test_data.csv`.
5. Set **Iterations** to the number of rows in your data file (e.g., `4`).
6. Click **Run Hotel Creation Automation**.

### Running with Newman CLI

```bash
# Install Newman
npm install -g newman

# Run with CSV data
newman run collections/Hotel_Creation_Automation.postman_collection.json \
  --environment environments/Hotel_Creation_Environment.postman_environment.json \
  --iteration-data data/hotel_test_data.csv \
  --reporters cli,json \
  --reporter-json-export results/hotel_creation_results.json

# Run with JSON data
newman run collections/Hotel_Creation_Automation.postman_collection.json \
  --environment environments/Hotel_Creation_Environment.postman_environment.json \
  --iteration-data data/hotel_test_data.json \
  --reporters cli,html \
  --reporter-html-export results/hotel_creation_report.html
```

---

## Data Format

### CSV Format (`data/hotel_test_data.csv`)

```csv
hotelCode,hotelName,status,postalCode,country,timezone,dateFormat,timeFormat,firstName,lastName,countryCode,mobileNumber,currency,roundOff
BNDF,Hotel Fribourg,Active,456543,Switzerland,Europe/Zurich,DD/MM/YYYY,24 hr,Hotel Fribourg,Hotel Manager,41,56787654,EUR,2 Decimals
```

### JSON Format (`data/hotel_test_data.json`)

```json
[
  {
    "hotelCode": "BNDF",
    "hotelName": "Hotel Fribourg",
    "status": "Active",
    "postalCode": "456543",
    "country": "Switzerland",
    "timezone": "Europe/Zurich",
    "dateFormat": "DD/MM/YYYY",
    "timeFormat": "24 hr",
    "firstName": "Hotel Fribourg",
    "lastName": "Hotel Manager",
    "countryCode": "41",
    "mobileNumber": "56787654",
    "currency": "EUR",
    "roundOff": "2 Decimals"
  }
]
```

> **Adding new hotels:** Simply append a new row (CSV) or object (JSON) — no changes to the collection are required.

---

## Field Mapping

| Data File Field | API Request Field | Notes |
|----------------|------------------|-------|
| `hotelCode` | `hotelCode` | Unique hotel identifier |
| `hotelName` | `hotelName` | Display name |
| `status` | `hotelStatus` | e.g., `Active` |
| `postalCode` | `postalCode` | |
| `country` | `stateProvince` | Used as state/province |
| `timezone` | `timezone` | IANA timezone string |
| `dateFormat` | `dateFormat` | e.g., `DD/MM/YYYY` |
| `timeFormat` | `timeFormat` | `24 hr` → `HH:mm`; `12 hr` → `hh:mm a` |
| `firstName` | `firstName` | Contact first name |
| `lastName` | `lastName` | Contact last name |
| `countryCode` | `countryCode`, `dialCountryCode` | Numeric country code |
| `mobileNumber` | `mobileNumber` | |
| `currency` | `currencySymbol` | ISO currency code |
| `roundOff` | `currencyRoundOff` | e.g., `2 Decimals` |

---

## Configuration

All configurable defaults live in the environment file. Key variables:

| Variable | Purpose |
|----------|---------|
| `defaultAddressLine1` | Default street address for all hotels |
| `defaultCity` | Default city |
| `defaultStateProvince` | Default state/province |
| `sourceUserName` | Source system username |
| `sourcePassword` | Source system password (marked secret) |
| `paymentTypes` | Comma-separated payment methods |
| `maxDistributionWindow` | Maximum distribution window (integer) |

---

## Troubleshooting

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for detailed solutions to common problems.

| Error | Quick Fix |
|-------|-----------|
| `401 Unauthorized` | Update `bearerToken` in environment |
| `400 Bad Request` | Check `x-transaction-id` is a valid UUID |
| Data file not loading | Ensure the file path is correct and format matches |
| Timeout | Increase request timeout in Postman settings |

---

## Advanced Usage

### Newman with HTML Reporter

```bash
npm install -g newman newman-reporter-html
newman run collections/Hotel_Creation_Automation.postman_collection.json \
  -e environments/Hotel_Creation_Environment.postman_environment.json \
  -d data/hotel_test_data.csv \
  -r html --reporter-html-export results/report.html
```

### CI/CD Integration (GitHub Actions)

```yaml
- name: Run Hotel Creation Automation
  run: |
    npm install -g newman newman-reporter-html
    newman run postman-automation/collections/Hotel_Creation_Automation.postman_collection.json \
      --environment postman-automation/environments/Hotel_Creation_Environment.postman_environment.json \
      --iteration-data postman-automation/data/hotel_test_data.csv \
      --env-var "bearerToken=${{ secrets.BEARER_TOKEN }}" \
      --reporters cli,html \
      --reporter-html-export newman/report.html
```

---

## Contributing

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/my-improvement`.
3. Make your changes and commit: `git commit -m "Add: description"`.
4. Push and open a Pull Request.

Please ensure all JSON files remain valid and test data follows the established schema.

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## Support

For questions or issues:
- Open a [GitHub Issue](https://github.com/VishalPalaniAgilysys/Vishal-Repository/issues)
- Contact: VishalPalaniAgilysys via GitHub
