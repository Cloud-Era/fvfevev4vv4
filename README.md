Certainly! I'll simplify the script to retrieve code scanning alerts for just one organization. Here's the updated version:

```python
import os
import requests
import urllib3
import csv
import datetime
import warnings

warnings.filterwarnings("ignore")
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

def getCodeScanningAlertsOrg(token, org, page=1):
    alerts = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    url = f"https://api.github.com/orgs/{org}/code-scanning/alerts?per_page=100&page={page}&state=open"
    
    res = requests.get(url, headers=headers, verify=False)

    print(f"Code Scanning Request URL: {res.url}")
    print(f"Code Scanning Response Status Code: {res.status_code}")

    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")

    data = res.json()
    alerts.extend(data)

    if len(data) == 100:
        alerts.extend(getCodeScanningAlertsOrg(token, org, page + 1))

    return alerts

def main():
    headers = ["Repository_Name", "Alert Number", "Rule ID", "Severity", "State", "Tool Name", "Description", "URL", "Created At", "Updated At"]
    
    GHToken = os.getenv("ACCESS_TOKEN")
    org_name = "your_org_name"  # Replace with your organization name

    current_directory = os.getcwd()
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"{org_name}_OpenCodeScanningAlerts_{current_date}.csv")

    unique_alerts = set()
    total_alerts = 0

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        codeScanningAlerts = getCodeScanningAlertsOrg(GHToken, org_name)
        
        for alert in codeScanningAlerts:
            alert_key = (alert['repository']['full_name'], alert['number'], alert['rule']['id'])
            if alert_key not in unique_alerts:
                unique_alerts.add(alert_key)
                total_alerts += 1
                writer.writerow([
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

1. Removed the `org_list` and replaced it with a single `org_name` variable.
2. Simplified the main function to fetch alerts for only one organization.
3. Updated the CSV file name to include the organization name.
4. Removed the "Organization" column from the CSV headers since we're only dealing with one organization.

To use this script:

1. Ensure your GitHub access token is set as an environment variable named `ACCESS_TOKEN`.
2. Replace `"your_org_name"` with the name of your organization.
3. Run the script.

This version will retrieve all open code scanning alerts for the specified organization, deduplicate them, and write them to a CSV file. The file will be named `<org_name>_OpenCodeScanningAlerts_<current_date>.csv` and will be saved in the current working directory.

The script will print the total number of unique open alerts fetched for the organization and the path to the generated CSV file.
