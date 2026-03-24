#Name - Srividhya S

Task 1 — Classify and Handle PII Fields

The dataset contains the following fields:
full_name, email, date_of_birth, zip_code, job_title, diagnosis_notes

Classify each field as either Direct PII or Indirect PII.
For each field, state whether you would drop it, mask it, or pseudonymize it before sharing, and briefly justify your choice.


Answer -

| Field           | Type           | Action              | Justification                                            |
| --------------- | -------------- | ------------------- | -------------------------------------------------------- |
| full_name       | Direct PII     | Drop / Pseudonymize | Direct identifier; not needed for analysis               |
| email           | Direct PII     | Drop / Pseudonymize | Uniquely identifies individuals                          |
| date_of_birth   | Indirect PII   | Mask (generalize)   | Can re-identify when combined; convert to age range      |
| zip_code        | Indirect PII   | Mask / Generalize   | Location can enable re-identification                    |
| job_title       | Indirect PII   | Keep / Generalize   | Low risk but may identify in rare cases                  |
| diagnosis_notes | Sensitive Data | Mask / Redact       | Contains sensitive health info and possible embedded PII |


Task2 -  Audit the API Script for Ethical Compliance
Your team's data collection script is shown below:

```
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])

# Store all records permanently in company database
save_to_database(records)
```

Identify two ethical or TOS violations present in this script. For each violation, explain what the problem is and suggest a corrected version of the relevant code.

1. Excessive Requests - Ignoring Rate Limits

The script sends 100 continuous requests using a free-tier API key, which may violate API rate limits and TOS.
Need to Add delay to respect rate limits.

```
import time

for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])
    time.sleep(1)  # respect rate limits
```

2. Storing Raw Sensitive Data Without Minimization

The script stores full personal and health data permanently, violating data minimization and privacy principles.
Need to Store only cleaned, non-identifiable data.

```
cleaned_records = []

for r in records:
    cleaned_records.append({
        "user_id": hash(r.get("email")),                 # simple pseudonym
        "age_band": str(r.get("date_of_birth"))[:4],     # keep only birth year
        "zip_prefix": str(r.get("zip_code"))[:3],        # partial zip
        "job_title": r.get("job_title")                  # keep as-is (low risk)
    })

save_to_database(cleaned_records)

```

3. Hardcoded API Key
API key is exposed in code, which is insecure and may violate TOS.
Need to Use environment variables.

```
import os
API_KEY = os.getenv("API_KEY")
```