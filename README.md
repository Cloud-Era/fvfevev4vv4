Here's a revised version of the script that focuses solely on retrieving code scanning alerts, addressing the issue of missing severity levels:

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

### Key Points:

- **Severity Handling**: The script now checks for the presence of a `severity` field within the `rule` dictionary of each alert. If it's not present, it defaults to 'Unknown'.
  
- **Pagination**: The function `getCodeScanningAlertsEnterprise` handles pagination by recursively fetching additional pages if there are more than 100 alerts.

- **CSV Output**: The script writes the retrieved data to a CSV file named `CodeScanning_Alerts_<current_date>.csv`.

Ensure you replace `"your_enterprise"` and the organization names in `org_list` with actual values. Also, make sure your GitHub access token is set as an environment variable named `ACCESS_TOKEN`. This script should help you capture all code scanning alerts along with their severity levels.

Citations:
[1] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts
[2] https://learn.microsoft.com/en-us/azure/devops/repos/security/github-advanced-security-code-scanning?view=azure-devops
[3] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/managing-code-scanning-alerts-for-your-repository
[4] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests
[5] https://github.blog/changelog/2021-07-19-codeql-code-scanning-new-severity-levels-for-security-alerts/
[6] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts
[7] https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/tracking-code-scanning-alerts-in-issues-using-task-lists
[8] https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning
