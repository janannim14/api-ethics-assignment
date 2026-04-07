# Data Ethics & PII Assignment

## Task 1 — Classify and Handle PII Fields

The following table classifies the fields in the dataset as Direct or Indirect PII and outlines the necessary handling actions before sharing the data with research partners.

| Field | Classification | Action | Justification |
| :--- | :--- | :--- | :--- |
| **full_name** | Direct PII | **Pseudonymize** | Directly identifies the individual. Replaced with a unique hash (SHA-256) to maintain data utility while protecting identity. |
| **email** | Direct PII | **Drop** | Highly sensitive contact information that is not required for general health research. Removed entirely. |
| **date_of_birth** | Indirect PII | **Generalize** | Can be used for re-identification. Converted to "Birth Year" or "Age" to reduce risk while keeping demographic value. |
| **zip_code** | Indirect PII | **Mask** | Full zip codes can pinpoint locations. Truncated to the first 3 digits (e.g., 600***) to protect geographic privacy. |
| **job_title** | Indirect PII | **Keep** | General socio-economic data that is usually safe, though must be monitored for very rare/unique titles. |
| **diagnosis_notes** | Sensitive Data | **Manual Review** | Contains critical research data but must be screened to ensure no PII (like names) was accidentally entered by staff. |

---

## Task 2 — Audit the API Script for Ethical Compliance

### Ethical & TOS Violations Identified:

1. **Security Risk (Hardcoded API Key):** The `API_KEY` was stored directly in the source code. This is a major security vulnerability as it would be exposed if the code is shared or pushed to a version control system like GitHub.
2. **Aggressive Data Scraping (TOS Violation):** The script attempts to fetch 100 pages in a rapid loop without any delay. This "hammering" of the API can be seen as a Denial of Service (DoS) attempt and likely violates the "free tier" Terms of Service (TOS).
3. **Lack of Error Handling:** The original script did not account for network failures or API downtime, which could lead to incomplete data collection or script crashes.

### Corrected Python Script:

```python
import requests
import time
import os

# VIOLATION 1 FIX: Use environment variables instead of hardcoding the key
API_URL = "[https://healthstats-api.example.com/records](https://healthstats-api.example.com/records)"
API_KEY = os.getenv("HEALTH_STATS_API_KEY", "secure_fallback_key")

records = []

# VIOLATION 2 FIX: Added range and error handling
for page in range(1, 101):
    try:
        # VIOLATION 3 FIX: Added timeout to prevent hanging
        response = requests.get(API_URL, params={"page": page, "key": API_KEY}, timeout=10)
        
        # Check if request was successful
        response.raise_for_status()
        
        data = response.json()
        if "results" in data:
            records.extend(data["results"])
            print(f"Successfully fetched page {page}")
        
        # VIOLATION 2 FIX: Added 1-second delay to respect API Rate Limits (Ethical Coding)
        time.sleep(1) 

    except requests.exceptions.RequestException as e:
        print(f"Stopping at page {page} due to error: {e}")
        break

# Function to save data securely
# save_to_database(records)
print(f"Total records collected: {len(records)}")
