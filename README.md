If you need to capture Dependabot alerts from multiple organizations under an **enterprise** account, you should adjust the script to loop through the organizations within the enterprise and collect the Dependabot alerts for each one. Here's the updated script to handle multiple organizations under an enterprise account:

### Updated Script

```python
import os
import json
import requests
import urllib3
import csv
import datetime
import re
import warnings

warnings.filterwarnings("ignore")
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Function to get Dependabot alerts for the enterprise
def getDependabotAlertsEnterprise(enterprise, token, org, page=1):
    repo_urls = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    url = f"https://api.github.com/enterprises/{enterprise}/dependabot/alerts?per_page=100&page={page}&org={org}&state=open"
    res = requests.get(url, headers=headers, verify=False)

    # Debugging output
    print(f"Request URL: {res.url}")
    print(f"Response Status Code: {res.status_code}")

    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")

    if len(res.json()) == 100:
        repo_urls += res.json()
        repo_urls += getDependabotAlertsEnterprise(enterprise, token, org, page + 1)
    else:
        repo_urls += res.json()

    return repo_urls

# Main function to generate the CSV file for multiple organizations in the enterprise
def main():
    headers = ["Organization", "Repository_Name", "Alert ID", "ComponentName", "Package Name", "Ecosystem", "Manifest_Path", "Vulnerability Rating", "ShortDescription", "Description", "Vulnerability ID", "First_Patched_Version", "Unique ID", "CVSS Rating", "CVSS Version", "Vulnerabilities List", "Identifiers", "Vulnerable Version Range", "Github URL", "Date Discovered", "Base_Repo_Name"]
    
    # Retrieve the access token and enterprise name from environment variables or hardcode them
    GHToken = os.getenv("ACCESS_TOKEN")
    enterprise_name = "your_enterprise"  # Replace with your enterprise name
    
    # List of organizations to collect Dependabot alerts from
    org_list = ["org1", "org2"]  # Replace with your organization names

    # Set the path and name for the CSV file
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = f"//Vulnerabilities/current/Vulnerabilities_{current_date}.csv"

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        for org in org_list:
            dependaBotAlerts = getDependabotAlertsEnterprise(enterprise_name, GHToken, org)

            # Write the Dependabot alerts for each organization to the CSV file
            for alert in dependaBotAlerts:
                if "dependency" in alert:
                    Vuln_ID = alert['security_advisory']['cve_id'] if alert['security_advisory']['cve_id'] else alert['security_advisory']['ghsa_id']
                    CVSS_Version = re.search(r'CVSS:(.*?)V(\d)', str(alert['security_advisory']['cvss']['vector_string']))
                    if CVSS_Version:
                        CVSS_Version = CVSS_Version.group(1)
                    writer.writerow([
                        org,
                        alert['repository']['full_name'],
                        f"GHASID-{alert['number']}",
                        f"{alert['dependency']['package']['ecosystem']}:{alert['dependency']['package']['name']}",
                        alert['dependency']['package']['name'],
                        alert['dependency']['package']['ecosystem'],
                        alert['dependency']['manifest_path'],
                        alert['security_advisory']['severity'],
                        alert['security_advisory']['summary'].replace('\n', '').replace('\r', ''),
                        alert['security_advisory']['description'].replace('\n', ' ').replace('\r', ' '),
                        Vuln_ID,
                        alert['security_vulnerability']['first_patched_version']['identifier'] if alert['security_vulnerability']['first_patched_version'] else "Not patched",
                        f"GHASID-{alert['number']}_{alert['dependency']['package']['name']}_{alert['repository']['full_name'].replace('/', '_')}",
                        alert['security_advisory']['cvss']['score'],
                        CVSS_Version,
                        str(alert['security_advisory']['vulnerabilities']),
                        str(alert['security_advisory']['identifiers']),
                        alert['security_vulnerability']['vulnerable_version_range'],
                        alert['repository']['html_url'],
                        alert['created_at'],
                        alert['repository']['full_name'].split("/", 1)[0]
                    ])
                else:
                    print(f"No valid data for org: {org}, repo: {alert.get('repository', {}).get('full_name')}")

if __name__ == "__main__":
    main()
```

### Key Changes:
1. **Enterprise-Level Alerts**: 
   - The function `getDependabotAlertsEnterprise` retrieves alerts for each organization under the enterprise. It takes the `enterprise` name, `token`, and `org` as parameters.
   - The URL structure is adjusted for the enterprise-level API: `https://api.github.com/enterprises/{enterprise}/dependabot/alerts?org={org}`.
   
2. **Organization List**:
   - You can specify multiple organizations under `org_list = ["org1", "org2"]`. This way, the script captures Dependabot alerts for all specified organizations.
   
3. **Output CSV**: 
   - Each alert is written to the CSV with an additional column, `Organization`, to identify which organization the alert came from.

This will ensure that you capture and export Dependabot alerts for all organizations under your enterprise.
