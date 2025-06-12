# 📨 Absence Email Analyzer with LLM + Google Sheets

This project automates the process of reading absence-related emails from a Yahoo inbox, using an AI model to extract structured absence information, and then writing that information into a centralized **Google Sheet** to track who will miss upcoming meetings.

---

## 📌 What This Project Does

1. **Connects to a Yahoo email inbox**
2. **Parses new emails for absence-related information**
3. **Uses an LLM API (OpenAI or Cohere)** to extract:
   - Whether the sender is absent today
   - Start and end dates of absence
   - The reason for the absence
4. **Writes the structured results into a Google Sheet**
5. **Sorts the sheet by absence date (descending)** so the latest absences appear on top

---

## ⚙️ Configuration Constants (Must Update Before Running)

| Constant           | What It Is | What You Need To Do |
|--------------------|------------|---------------------|
| `IMAP_SERVER`      | The email server to connect via IMAP | ✅ Already correct for Yahoo: `"imap.mail.yahoo.com"` |
| `EMAIL_ACCOUNT`    | The Yahoo email address receiving absence emails | 🔧 Use your dedicated inbox, e.g., `requestabsence@yahoo.com` |
| `EMAIL_PASSWORD` or `APP_PASSWORD` | Password to access the Yahoo inbox | 🔐 Must be a Yahoo **App Password**, not your normal login. Generate from Yahoo Security settings |
| `SPREADSHEET_ID`   | The ID of the Google Sheet used for logging absences | 🔧 Copy this from the Google Sheet URL (`/d/<ID>/`) |
| `SHEET_NAME`       | The tab name inside the spreadsheet | ✅ Must match the sheet tab name (e.g., `"Absence Tracker"`) |
| `SCOPES`           | The level of access granted to the script | ✅ Already set correctly: `['https://www.googleapis.com/auth/spreadsheets']` |

---

## 🧠 Code Structure

### `1. fetch_and_format_yahoo_emails()`
- Connects to the Yahoo inbox using IMAP
- Downloads all emails
- Parses out plain text bodies and returns them as a list of dictionaries

### `2. analyze_email_with_llm()`
- Sends each email body to an LLM (e.g., Cohere or OpenAI)
- Uses a prompt to extract structured JSON info
- Returns fields: `is_absent_today`, `start_time`, `end_time`, `reason`

### `3. append_absence_to_sheet()`
- For every day in the absence range, adds a row to the Google Sheet:
  - `[Date, Sender Name, Reason]`
- Calls `sort_sheet_by_date_desc()` to keep sheet sorted

### `4. sort_sheet_by_date_desc()`
- Sorts the sheet by date (column A) in descending order

### `5. main()`
- Executes the full workflow:
  - Fetch → Analyze → Append → Sort
- Includes logging and error handling

---

## 🔐 Setup Requirements

1. **Yahoo App Password**
   - Go to [Yahoo Account Security](https://login.yahoo.com/account/security)
   - Generate a one-time **App Password**
   - Use this password in the script for your Yahoo account (not your normal login)

2. **Google Sheet + Service Account**
   - Create a new Google Sheet to log absence entries
   - Share the sheet with your service account email (from Google Cloud Console)
   - Download the `service_account_credentials.json` file and place it in the project folder
   - Update the `SPREADSHEET_ID` and `SHEET_NAME` values in the config section

3. **Install Required Python Libraries**

   Depending on which AI model you choose to use for email analysis, install the following:

   ### Option A: Using OpenAI
   ```bash
   pip install openai google-api-python-client pandas
   ```

   ### Option B: Using Cohere
   ```bash
   pip install cohere google-api-python-client pandas
   ```

   _You do not need both unless you are switching between models._

   ### Optional (in case of IMAP decoding issues):
   ```bash
   pip install imaplib2
   ```

---

## 📊 Example Output

Each row in the Google Sheet will look like:
```
| Date        | Name       | Reason               |
|-------------|------------|----------------------|
| 2025-05-22  | John Doe   | attending a wedding  |
| 2025-05-23  | Jane Smith | flu recovery         |
```

---

## 📎 Notes

- This script assumes **every email in the inbox is absence-related** (already filtered by routing to `requestabsence@yahoo.com`)
- Special characters are decoded using fallback encodings to avoid parsing errors
- If LLM cannot parse the email clearly, it returns an empty or skipped result
