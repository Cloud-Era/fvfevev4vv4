Here’s the full script for fetching Code Scanning alerts, including logging to help you debug the number of alerts retrieved:

```python
import os
import requests
import urllib3
import csv
import datetime
import warnings

warnings.filterwarnings("ignore")
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Function to get Code Scanning alerts for the enterprise
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
    print(f"Fetched {len(data)} alerts from page {page} for {org}")

    # Check if there's another page of alerts
    if len(data) == 100:
        alerts.extend(getCodeScanningAlertsEnterprise(enterprise, token, org, page + 1))

    return alerts

# Main function to generate CSV file for code scanning alerts
def main():
    headers = ["Organization", "Repository_Name", "Alert Number", "Rule ID", "Severity", "State", "Tool Name", "Description", "URL", "Created At", "Updated At"]
    
    GHToken = os.getenv("ACCESS_TOKEN")
    enterprise_name = "your_enterprise"  # Replace with your enterprise name
    org_list = ["org1", "org2"]  # Replace with your organization names

    current_directory = os.getcwd()
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"CodeScanning_Alerts_{current_date}.csv")

    unique_alert_ids = set()

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        for org in org_list:
            codeScanningAlerts = getCodeScanningAlertsEnterprise(enterprise_name, GHToken, org)

            for alert in codeScanningAlerts:
                alert_id = alert['number']
                repo_name = alert['repository']['full_name']

                if (alert_id, repo_name) not in unique_alert_ids:
                    unique_alert_ids.add((alert_id, repo_name))
                    
                    severity = alert['rule']['severity'] if 'rule' in alert and 'severity' in alert['rule'] else 'Unknown'

                    writer.writerow([
                        org,
                        repo_name,
                        alert['number'],
                        alert['rule']['id'],
                        severity,
                        alert['state'],
                        alert['tool']['name'],
                        alert['rule']['description'],
                        alert['html_url'],
                        alert['created_at'],
                        alert['updated_at']
                    ])
                else:
                    print(f"Duplicate Code Scanning alert found: {alert_id} in {repo_name}")

    print(f"Code Scanning CSV file generated at: {csv_file_path}")
    print(f"Total unique Code Scanning alerts written: {len(unique_alert_ids)}")

if __name__ == "__main__":
    main()
```

### Key Components:
- **Logging**: The script logs the request URL, response status, number of alerts fetched per page, and checks for duplicates.
- **CSV Output**: Alerts are written to a CSV file in the current directory, including various details for each alert.
- **Pagination**: The function handles pagination automatically.

### Usage:
1. Set your GitHub token in the `ACCESS_TOKEN` environment variable.
2. Update `enterprise_name` and `org_list` with your actual enterprise and organization names.
3. Run the script, and it will generate a CSV file in the current directory.

If you have any further questions or need additional modifications, feel free to ask!
