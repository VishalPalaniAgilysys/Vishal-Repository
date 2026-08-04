# Postman Automation: Create one hotel for every `alpha-3` country code

This file contains a copy-paste-ready Postman Collection Runner solution to validate the CreateHotel API by creating exactly one hotel for every unique `alpha-3` country code returned by:

```text
https://aks-crs-qaint.hospitalityrevolution.com/assets/Country_Code.json
```

## Collection structure

Create a Postman collection with these 3 requests in this exact order and with these exact request names:

1. `Login`
2. `Load Country Codes`
3. `Create Hotel For Current Country`

Run the collection through **Collection Runner** starting from `Login`.

---

## Collection variables required

Create these variables at the **collection** level. Credentials may also be set as environment variables with the same names.

| Variable | Initial value / example | Required | Notes |
|---|---:|:---:|---|
| `base_url` | `https://aks-crs-qa.hospitalityrevolution.com/api/v1.0` | Yes | API base URL for login and create hotel. |
| `country_dataset_url` | `https://aks-crs-qaint.hospitalityrevolution.com/assets/Country_Code.json` | Yes | Country code dataset URL. |
| `login_username` | `jdoe` | Yes | Replace with valid username. |
| `login_password` | `P@ssw0rd!` | Yes | Replace with valid password. |
| `login_corporate_id` | `12345` | Yes | Corporate ID required by login request. The CreateHotel request uses `corporateId` from the login response, not this value directly. |
| `brand_code` | `QA` | Yes | Mandatory brand code value for CreateHotel. Replace if your API expects a specific brand. |
| `default_dial_code` | `+1` | Yes | Used only as mandatory payload data. The test validates only `alpha-3`. |
| `default_dial_country_code` | `USA` | Yes | Used only as mandatory payload data. The test validates only `alpha-3`. |
| `default_timezone` | `UTC` | Yes | Mandatory payload data. |
| `default_currency_symbol` | `USD` | Yes | Mandatory payload data. |
| `default_currency_round_off` | `2` | Yes | Mandatory payload data. |
| `default_time_format` | `HH:mm` | Yes | Mandatory payload data. |
| `default_date_format` | `YYYY-MM-DD` | Yes | Mandatory payload data. |
| `createHotel_request_name` | `Create Hotel For Current Country` | Yes | Must exactly match the request name. |
| `auth_header_name` |  | No | Optional. Set only if CreateHotel requires an auth/session header. Example: `sessionId`. |
| `auth_header_value` |  | No | Optional. Can be set by login tests if your login response returns a token/session value. |

The following variables are generated automatically during execution:

| Auto variable | Purpose |
|---|---|
| `login_user_id` | `userId` from login response. Used as `createdBy` and `requestedBy`. |
| `login_corporate_id_from_response` | `corporateId` from login response. Used as CreateHotel `corporateId`. |
| `login_session_id` | `sessionId` from login response, if present. |
| `country_alpha3_codes` | JSON array of unique extracted `alpha-3` values. |
| `country_alpha3_total` | Total number of unique country codes to process. |
| `country_alpha3_index` | Current zero-based loop index. |
| `current_alpha3` | Current `alpha-3` code being processed. |
| `current_hotel_code` | Generated unique hotel code for the current request. |
| `current_hotel_name` | Generated hotel name for the current request. |
| `createHotel_processed_count` | Number of CreateHotel requests completed. |
| `createHotel_passed_count` | Number of successful CreateHotel requests. |
| `createHotel_failed_count` | Number of failed CreateHotel requests. |
| `createHotel_failed_codes` | JSON array of failed `alpha-3` codes with details. |
| `createHotel_summary` | Final execution summary JSON. |

---

# Request 1: `Login`

## Method and URL

```text
POST {{base_url}}/Auth/login
```

## Headers

```text
Content-Type: application/json
```

## Request body

```json
{
  "username": "{{login_username}}",
  "password": "{{login_password}}",
  "corporateId": {{login_corporate_id}}
}
```

## Tests script

```javascript
pm.test("Login HTTP status is 200", function () {
    pm.response.to.have.status(200);
});

let loginJson = {};
try {
    loginJson = pm.response.json();
} catch (error) {
    throw new Error("Login response is not valid JSON: " + error.message);
}

pm.test("Login response contains corporateId", function () {
    pm.expect(loginJson).to.have.property("corporateId");
    pm.expect(loginJson.corporateId, "corporateId from login response").to.not.be.oneOf([null, undefined, 0, ""]);
});

pm.test("Login response contains userId", function () {
    pm.expect(loginJson).to.have.property("userId");
    pm.expect(loginJson.userId, "userId from login response").to.not.be.oneOf([null, undefined, 0, ""]);
});

pm.collectionVariables.set("login_user_id", String(loginJson.userId));
pm.collectionVariables.set("login_corporate_id_from_response", String(loginJson.corporateId));

if (loginJson.sessionId) {
    pm.collectionVariables.set("login_session_id", String(loginJson.sessionId));
}

// Optional token/session header support.
// If your CreateHotel API requires a session header, set these collection variables:
// auth_header_name=sessionId
// auth_header_value={{login_session_id}}
if (loginJson.sessionId && !pm.collectionVariables.get("auth_header_value")) {
    pm.collectionVariables.set("auth_header_value", String(loginJson.sessionId));
}

console.log("Login completed. corporateId:", loginJson.corporateId, "userId:", loginJson.userId);
postman.setNextRequest("Load Country Codes");
```

---

# Request 2: `Load Country Codes`

## Method and URL

```text
GET {{country_dataset_url}}
```

## Tests script

```javascript
pm.test("Country dataset HTTP status is 200", function () {
    pm.response.to.have.status(200);
});

let countryData;
try {
    countryData = pm.response.json();
} catch (error) {
    throw new Error("Country dataset response is not valid JSON: " + error.message);
}

function collectAlpha3Values(node, output) {
    if (Array.isArray(node)) {
        node.forEach(item => collectAlpha3Values(item, output));
        return;
    }

    if (node && typeof node === "object") {
        if (Object.prototype.hasOwnProperty.call(node, "alpha-3")) {
            const value = String(node["alpha-3"] || "").trim().toUpperCase();
            if (value) {
                output.push(value);
            }
        }

        Object.keys(node).forEach(key => collectAlpha3Values(node[key], output));
    }
}

const extractedCodes = [];
collectAlpha3Values(countryData, extractedCodes);

const uniqueCodes = Array.from(new Set(extractedCodes));

pm.test("At least one alpha-3 country code is extracted", function () {
    pm.expect(uniqueCodes.length, "unique alpha-3 country code count").to.be.above(0);
});

pm.test("Every extracted alpha-3 value uses 3 uppercase letters", function () {
    const invalidCodes = uniqueCodes.filter(code => !/^[A-Z]{3}$/.test(code));
    pm.expect(invalidCodes, "invalid alpha-3 values").to.eql([]);
});

// Reset loop and counters for a clean Collection Runner execution.
pm.collectionVariables.set("country_alpha3_codes", JSON.stringify(uniqueCodes));
pm.collectionVariables.set("country_alpha3_total", String(uniqueCodes.length));
pm.collectionVariables.set("country_alpha3_index", "0");
pm.collectionVariables.set("createHotel_processed_count", "0");
pm.collectionVariables.set("createHotel_passed_count", "0");
pm.collectionVariables.set("createHotel_failed_count", "0");
pm.collectionVariables.set("createHotel_failed_codes", JSON.stringify([]));
pm.collectionVariables.unset("createHotel_summary");
pm.collectionVariables.unset("current_alpha3");
pm.collectionVariables.unset("current_hotel_code");
pm.collectionVariables.unset("current_hotel_name");

console.log("Unique alpha-3 country codes loaded:", uniqueCodes.length);
console.log(uniqueCodes);

postman.setNextRequest(pm.collectionVariables.get("createHotel_request_name") || "Create Hotel For Current Country");
```

---

# Request 3: `Create Hotel For Current Country`

## Method and URL

```text
POST {{base_url}}/Hotels/createhotel
```

## Headers

```text
Content-Type: application/json
{{auth_header_name}}: {{auth_header_value}}
```

> If your API does not require an auth/session header for CreateHotel, leave `auth_header_name` and `auth_header_value` blank or remove this header from the request.

## Pre-request script

```javascript
function getCollectionNumber(name, defaultValue) {
    const raw = pm.collectionVariables.get(name);
    const parsed = Number(raw);
    return Number.isFinite(parsed) ? parsed : defaultValue;
}

const codesRaw = pm.collectionVariables.get("country_alpha3_codes");
if (!codesRaw) {
    throw new Error("country_alpha3_codes is missing. Run the collection from the Login request so Load Country Codes executes first.");
}

let codes;
try {
    codes = JSON.parse(codesRaw);
} catch (error) {
    throw new Error("country_alpha3_codes is not valid JSON: " + error.message);
}

if (!Array.isArray(codes) || codes.length === 0) {
    throw new Error("No alpha-3 country codes are available to process.");
}

const index = getCollectionNumber("country_alpha3_index", 0);

if (index >= codes.length) {
    console.log("No remaining country codes. Stopping CreateHotel loop.");
    postman.setNextRequest(null);
    return;
}

const alpha3 = String(codes[index]).trim().toUpperCase();
if (!/^[A-Z]{3}$/.test(alpha3)) {
    throw new Error(`Invalid alpha-3 code at index ${index}: ${alpha3}`);
}

const sequence = index + 1;
const sequencePadded = String(sequence).padStart(3, "0");
const timestampSuffix = String(Date.now()).slice(-6);

// Hotel code must be mandatory and unique.
// Example: TC001DZA123456
const hotelCode = `TC${sequencePadded}${alpha3}${timestampSuffix}`;

// Hotel names follow TC_001, TC_002, TC_003, ...
const hotelName = `TC_${sequencePadded}`;

pm.collectionVariables.set("current_alpha3", alpha3);
pm.collectionVariables.set("current_hotel_code", hotelCode);
pm.collectionVariables.set("current_hotel_name", hotelName);
pm.collectionVariables.set("current_sequence", String(sequence));

console.log(`Processing country ${sequence}/${codes.length}: alpha-3=${alpha3}, hotelCode=${hotelCode}, hotelName=${hotelName}`);
```

## Request body JSON template

```json
{
  "corporateId": {{login_corporate_id_from_response}},
  "brandId": null,
  "brandCode": "{{brand_code}}",
  "hotelGroupId": null,
  "regionId": null,
  "hotelCode": "{{current_hotel_code}}",
  "hotelName": "{{current_hotel_name}}",

  "firstName": "Auto",
  "lastName": "Tester",
  "contactPersonFirstName": "Auto",
  "contactPersonLastName": "Tester",

  "addressLine1": "Automation Address 1",
  "addressLine2": null,
  "city": "Automation City",
  "stateProvince": "Automation State",
  "postalCode": "000000",

  "countryCode": "{{current_alpha3}}",
  "dialCode": "{{default_dial_code}}",
  "dialCountryCode": "{{default_dial_country_code}}",

  "mobileNumber": "9876543210",
  "contactPersonPhone": "9876543210",
  "email": "createhotel.{{current_alpha3}}.{{current_sequence}}@example.com",
  "contactPersonEmail": "createhotel.{{current_alpha3}}.{{current_sequence}}@example.com",

  "timezone": "{{default_timezone}}",
  "hotelStatus": "Active",
  "currencySymbol": "{{default_currency_symbol}}",
  "currencyRoundOff": "{{default_currency_round_off}}",
  "paymentTypes": "AX",
  "dateFormat": "{{default_date_format}}",
  "timeFormat": "{{default_time_format}}",

  "hotelDescription": null,
  "sourceGroup": null,
  "sourceSubGroup": null,
  "sourceUserName": null,
  "sourcePassword": null,
  "systemId": null,

  "createdBy": {{login_user_id}},
  "requestedBy": {{login_user_id}}
}
```

## Tests script

```javascript
function getCollectionNumber(name, defaultValue) {
    const raw = pm.collectionVariables.get(name);
    const parsed = Number(raw);
    return Number.isFinite(parsed) ? parsed : defaultValue;
}

function getJsonBodySafely() {
    try {
        return pm.response.json();
    } catch (error) {
        return null;
    }
}

function responseContainsFailure(body) {
    if (!body || typeof body !== "object") {
        return false;
    }

    const errorCode = body.errorCode;
    const errorMessage = body.errorMessage;
    const success = body.success;
    const succeeded = body.succeeded;
    const isSuccess = body.isSuccess;
    const successCode = body.successCode;
    const status = typeof body.status === "string" ? body.status.toLowerCase() : "";
    const message = typeof body.message === "string" ? body.message.toLowerCase() : "";

    if (success === false || succeeded === false || isSuccess === false) {
        return true;
    }

    if (errorCode && String(errorCode).trim() !== "" && String(errorCode).trim() !== "0") {
        return true;
    }

    if (errorMessage && String(errorMessage).trim() !== "") {
        return true;
    }

    if (status.includes("fail") || status.includes("error")) {
        return true;
    }

    if (message.includes("fail") || message.includes("error")) {
        return true;
    }

    // If successCode is present, treat common failure values as failure.
    if (successCode !== undefined && successCode !== null) {
        const normalizedSuccessCode = String(successCode).trim().toLowerCase();
        if (["false", "failed", "failure", "error"].includes(normalizedSuccessCode)) {
            return true;
        }
    }

    return false;
}

const alpha3 = pm.collectionVariables.get("current_alpha3");
const hotelCode = pm.collectionVariables.get("current_hotel_code");
const hotelName = pm.collectionVariables.get("current_hotel_name");
const codes = JSON.parse(pm.collectionVariables.get("country_alpha3_codes") || "[]");
const total = getCollectionNumber("country_alpha3_total", codes.length);
let index = getCollectionNumber("country_alpha3_index", 0);

const responseBody = getJsonBodySafely();
const httpStatusValid = [200, 201].includes(pm.response.code);
const explicitFailure = responseContainsFailure(responseBody);
const createSucceeded = httpStatusValid && !explicitFailure;

pm.test(`HTTP status is 200 or 201 for alpha-3 ${alpha3}`, function () {
    pm.expect(pm.response.code, `HTTP status for ${alpha3}`).to.be.oneOf([200, 201]);
});

pm.test(`Hotel creation succeeded for supplied alpha-3 ${alpha3}`, function () {
    pm.expect(createSucceeded, `CreateHotel failed for alpha-3=${alpha3}. Response: ${pm.response.text()}`).to.eql(true);
});

// Only validate alpha-3 country code field if the API echoes countryCode back in the response.
if (responseBody && Object.prototype.hasOwnProperty.call(responseBody, "countryCode")) {
    pm.test(`Response countryCode matches supplied alpha-3 ${alpha3}`, function () {
        pm.expect(String(responseBody.countryCode).trim().toUpperCase()).to.eql(alpha3);
    });
}

// If the API echoes hotelCode, confirm the response belongs to this request.
if (responseBody && Object.prototype.hasOwnProperty.call(responseBody, "hotelCode")) {
    pm.test(`Response hotelCode matches generated hotelCode for ${alpha3}`, function () {
        pm.expect(String(responseBody.hotelCode).trim()).to.eql(hotelCode);
    });
}

let processedCount = getCollectionNumber("createHotel_processed_count", 0) + 1;
let passedCount = getCollectionNumber("createHotel_passed_count", 0);
let failedCount = getCollectionNumber("createHotel_failed_count", 0);
let failedCodes;

try {
    failedCodes = JSON.parse(pm.collectionVariables.get("createHotel_failed_codes") || "[]");
    if (!Array.isArray(failedCodes)) {
        failedCodes = [];
    }
} catch (error) {
    failedCodes = [];
}

if (createSucceeded) {
    passedCount += 1;
    console.log(`PASS: alpha-3=${alpha3}, hotelCode=${hotelCode}, hotelName=${hotelName}`);
} else {
    failedCount += 1;
    const failureDetail = {
        alpha3,
        hotelCode,
        hotelName,
        status: pm.response.code,
        response: pm.response.text()
    };
    failedCodes.push(failureDetail);
    console.error("FAIL:", failureDetail);
}

pm.collectionVariables.set("createHotel_processed_count", String(processedCount));
pm.collectionVariables.set("createHotel_passed_count", String(passedCount));
pm.collectionVariables.set("createHotel_failed_count", String(failedCount));
pm.collectionVariables.set("createHotel_failed_codes", JSON.stringify(failedCodes));

index += 1;
pm.collectionVariables.set("country_alpha3_index", String(index));

if (index < total) {
    postman.setNextRequest(pm.collectionVariables.get("createHotel_request_name") || "Create Hotel For Current Country");
} else {
    const summary = {
        totalCountryCodes: total,
        processed: processedCount,
        passed: passedCount,
        failed: failedCount,
        failedCountryCodes: failedCodes.map(item => item.alpha3),
        failedDetails: failedCodes
    };

    pm.collectionVariables.set("createHotel_summary", JSON.stringify(summary, null, 2));

    pm.test("Total hotel creation requests equals total unique alpha-3 country codes", function () {
        pm.expect(processedCount).to.eql(total);
    });

    console.log("CreateHotel country automation completed.");
    console.log("SUMMARY:", summary);
    console.table(summary.failedDetails);

    postman.setNextRequest(null);
}
```

---

## Execution instructions

1. Open Postman.
2. Create a collection.
3. Add the 3 requests exactly as named:
   - `Login`
   - `Load Country Codes`
   - `Create Hotel For Current Country`
4. Add the collection variables listed above.
5. Replace these required values:
   - `login_username`
   - `login_password`
   - `login_corporate_id`
   - `brand_code`, if your API requires a specific brand code
6. Open **Collection Runner**.
7. Select this collection.
8. Start the run from `Login`.
9. Do not provide an external data file. The `Load Country Codes` request loads the JSON dataset automatically.
10. Review the Postman Console for progress and final summary.
11. After completion, inspect the collection variable `createHotel_summary`.

---

## Expected behavior

- The `Load Country Codes` request extracts every unique `alpha-3` value from the country JSON dataset.
- The `Create Hotel For Current Country` request loops automatically using `postman.setNextRequest()`.
- Exactly one CreateHotel request is sent for every unique `alpha-3` value.
- `corporateId` is fetched from the login response.
- `createdBy` and `requestedBy` are fetched from the login response `userId`.
- Hotel names are generated as `TC_001`, `TC_002`, `TC_003`, and so on.
- Hotel codes are generated uniquely using the sequence number, `alpha-3` code, and timestamp suffix.
- Failures are logged with the corresponding `alpha-3` country code.
- Final summary includes:
  - Total country codes processed
  - Passed count
  - Failed count
  - Failed country codes list

---

## Notes

- This automation validates only the `alpha-3` country code field, as requested.
- No validation is performed for country name, region, dial code, alpha-2 code, ISO codes, or any other country metadata.
- If the CreateHotel API requires an auth token/session header, configure `auth_header_name` and `auth_header_value` based on the actual login response contract.
- If your CreateHotel API accepts only the sample field names and rejects alias fields, remove `contactPersonFirstName`, `contactPersonLastName`, `contactPersonPhone`, and `contactPersonEmail` from the body. They are included because those fields were listed as mandatory in the requirement.
