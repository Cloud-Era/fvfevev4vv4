I understand your concern about duplicates and the discrepancy between the number of alerts you see in the browser (21k) versus what you're getting in the API results. Let's modify the script to address these issues and try to get a more accurate count of unique alerts. Here's an updated version:

```python
import os
import requests
import urllib3
import csv
import datetime
import warnings

warnings.filterwarnings("ignore")
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

def getCodeScanningAlertsEnterprise(enterprise, token, org, page=1):
    alerts = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    url = f"https://api.github.com/enterprises/{enterprise}/code-scanning/alerts?per_page=100&page={page}&org={org}&state=open"
    
    res = requests.get(url, headers=headers, verify=False)

    print(f"Code Scanning Request URL: {res.url}")
    print(f"Code Scanning Response Status Code: {res.status_code}")

    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")

    data = res.json()
    alerts.extend(data)

    if len(data) == 100:
        alerts.extend(getCodeScanningAlertsEnterprise(enterprise, token, org, page + 1))

    return alerts

def main():
    headers = ["Organization", "Repository_Name", "Alert Number", "Rule ID", "Severity", "State", "Tool Name", "Description", "URL", "Created At", "Updated At"]
    
    GHToken = os.getenv("ACCESS_TOKEN")
    enterprise_name = "your_enterprise"  # Replace with your enterprise name
    org_list = ["org1", "org2"]  # Replace with your organization names

    current_directory = os.getcwd()
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"OpenCodeScanningAlerts_{current_date}.csv")

    unique_alerts = set()
    total_alerts = 0

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        for org in org_list:
            codeScanningAlerts = getCodeScanningAlertsEnterprise(enterprise_name, GHToken, org)
            
            for alert in codeScanningAlerts:
                alert_key = (alert['repository']['full_name'], alert['number'], alert['rule']['id'])
                if alert_key not in unique_alerts:
                    unique_alerts.add(alert_key)
                    total_alerts += 1
                    writer.writerow([
                        org,
                        alert['repository']['full_name'],
                        alert['number'],
                        alert['rule']['id'],
                        alert['rule']['severity'],
                        alert['state'],
                        alert['tool']['name'],
                        alert['rule']['description'],
                        alert['html_url'],
                        alert['created_at'],
                        alert['updated_at']
                    ])

    print(f"Open Code Scanning Alerts CSV file generated at: {csv_file_path}")
    print(f"Total unique open Code Scanning alerts written: {total_alerts}")

if __name__ == "__main__":
    main()
```

Key changes in this version:

1. Removed the severity-specific querying, as it was causing duplicates.
2. Added a `unique_alerts` set to track unique alerts based on repository name, alert number, and rule ID.
3. Only write alerts to the CSV file if they haven't been seen before.
4. Updated the total_alerts counter to reflect the number of unique alerts written.

This script should now:
- Fetch all open code scanning alerts for each organization in your enterprise.
- Deduplicate alerts based on repository name, alert number, and rule ID.
- Write only unique alerts to the CSV file.
- Print the total number of unique open alerts fetched across all organizations.

If you're still seeing a significant discrepancy between the number of alerts in the browser (21k) and what this script retrieves, it could be due to one of the following reasons:

1. API limitations or pagination issues.
2. Differences in how alerts are counted in the UI vs. the API.
3. Potential filtering in the API that's not present in the UI.

To troubleshoot further:
1. Check if the total number of alerts retrieved before deduplication matches closer to what you see in the UI.
2. Verify that your access token has the necessary permissions to view all alerts.
3. Consider reaching out to GitHub support if the discrepancy persists, as there might be underlying differences between the UI and API representations of alerts.

Remember to replace `"your_enterprise"` with your actual enterprise name and update `org_list` with your organization names before running the script.

Citations:
[1] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/managing-code-scanning-alerts-for-your-repository
[2] https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2022-11-28
[3] https://docs.github.com/en/enterprise-server%403.11/rest/code-scanning/code-scanning
[4] https://github.com/orgs/community/discussions/24321
[5] https://docs.github.com/en/rest/secret-scanning/secret-scanning
[6] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts
[7] https://docs.github.com/en/enterprise-server%403.11/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts
[8] https://stackoverflow.com/questions/72520372/how-to-refresh-github-code-scanning-alerts
