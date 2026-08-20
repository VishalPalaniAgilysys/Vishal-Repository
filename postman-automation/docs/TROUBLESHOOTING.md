# Troubleshooting Guide — Hotel Creation Automation

---

## Authentication Errors

### 401 Unauthorized

**Symptoms:** Every request returns HTTP 401.

**Causes & Solutions:**

1. **Token not set** — Open the environment editor and verify `bearerToken` contains your actual token (not `YOUR_BEARER_TOKEN_HERE`).
2. **Token expired** — Obtain a fresh token from the CRS admin or re-authenticate. Tokens typically expire after a fixed period.
3. **Wrong environment selected** — Ensure **Hotel Creation Environment** is selected in the Postman top-right dropdown, not *No Environment*.
4. **Token copied with extra spaces** — Paste the token into a text editor first to remove leading/trailing whitespace, then set it in the environment.

### 403 Forbidden

**Symptoms:** Requests return HTTP 403.

**Causes & Solutions:**

1. Your account does not have permission to create hotels — contact the CRS admin to grant the required role.
2. The `corporateId`, `brandId`, or `regionId` values are not accessible to your account — verify these with the admin.

---

## Invalid Request Format (400 Bad Request)

**Symptoms:** Requests return HTTP 400 with an error message about a missing or invalid field.

**Common Causes:**

| Field | Cause | Fix |
|-------|-------|-----|
| `x-transaction-id` | Not a valid UUID | The pre-request script auto-generates a UUID via `{{$guid}}`. Verify the script is not disabled. |
| `hotelCode` | Missing or empty in data file | Ensure every row in your CSV/JSON has a non-empty `hotelCode`. |
| `corporateId` / `brandId` / `regionId` | Non-integer value | These must be integers. Check environment variable values. |
| `timeFormat` | Unexpected value | Only `HH:mm` or `hh:mm a` are accepted. The script maps `24 hr` → `HH:mm`; anything else → `hh:mm a`. |

---

## Transaction ID Issues

**Symptoms:** API returns an error about an invalid or duplicate `x-transaction-id`.

**Solutions:**

1. The pre-request script uses Postman's built-in `{{$guid}}` to generate a fresh UUID per request. Confirm the script is enabled (not commented out).
2. If you see duplicate UUID errors in rapid successive runs, add a small delay between iterations using the Newman `--delay-request` flag:
   ```bash
   newman run ... --delay-request 500
   ```

---

## Data File Loading Problems

**Symptoms:** Collection Runner shows 0 iterations or "No data file loaded".

**Solutions:**

1. **File format mismatch** — CSV files must use comma delimiters and include a header row. JSON files must be an array of objects.
2. **Encoding issue** — Save the CSV file as UTF-8. Special characters (e.g., `â` in "Neuchâtel") can cause parse errors in non-UTF-8 files.
3. **Empty rows** — Remove any trailing blank rows from the CSV.
4. **Iterations count** — Set *Iterations* in the Collection Runner to match the number of data rows (excluding the header).
5. **Newman data flag** — Ensure you use `--iteration-data` (not `--data`) with Newman v5+.

---

## Network and Timeout Errors

**Symptoms:** Requests time out or fail with `ECONNREFUSED` / `ENOTFOUND`.

**Solutions:**

1. **VPN required** — The QA environment may only be accessible from the corporate network or VPN. Connect and retry.
2. **Firewall** — Ensure outbound HTTPS (port 443) is allowed to `aks-crs-qa.hospitalityrevolution.com`.
3. **Increase timeout** — In Postman: *Settings → General → Request timeout* — set to `30000` ms (30 seconds).
4. **Newman timeout flag:**
   ```bash
   newman run ... --timeout-request 30000
   ```

---

## Response Parsing Errors

**Symptoms:** Test script logs "Failed to parse response" or `SyntaxError: Unexpected token`.

**Solutions:**

1. The API may have returned an HTML error page (e.g., 502 Bad Gateway). Check the raw response in the Postman Response panel.
2. Confirm `Content-Type: application/json` header is being sent (it is set in the collection by default).
3. If the server is temporarily unavailable, wait and retry.

---

## Enabling Verbose Logging

### In Postman Console

1. Open **View → Postman Console** (Alt+Ctrl+C on Windows/Linux, Alt+Cmd+C on macOS).
2. All `console.log` statements from the pre-request and test scripts appear here.

### In Newman

Add the `--verbose` flag:

```bash
newman run ... --verbose
```

---

## Retrying Failed Hotels

After a run, the `failedHotels` environment variable contains a JSON array of all failures. To retry:

1. Create a new data file containing only the failed hotel records.
2. Run the Collection Runner again with the new file.

With Newman:

```bash
# Extract failed hotel codes from environment (save as retry_hotels.json)
newman run collections/Hotel_Creation_Automation.postman_collection.json \
  --environment environments/Hotel_Creation_Environment.postman_environment.json \
  --iteration-data data/retry_hotels.json \
  --env-var "bearerToken=<YOUR_TOKEN>"
```

---

## Resetting Environment State

Between test runs, clear accumulated data by resetting these environment variables to empty strings:

- `createdHotels`
- `failedHotels`
- `transactionId`
- `sessionId`

In Postman, open the environment editor, clear the values, and save.

---

## Contact Support

If you cannot resolve the issue with this guide:

- Open a [GitHub Issue](https://github.com/VishalPalaniAgilysys/Vishal-Repository/issues) with:
  - Steps to reproduce
  - Error message / screenshot
  - Postman version
  - Anonymized request/response (remove the token)
- Contact: **VishalPalaniAgilysys** via GitHub
