Certainly! Here's the full script that combines both Dependabot and Code Scanning alerts retrieval:

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

    print(f"Dependabot Request URL: {res.url}")
    print(f"Dependabot Response Status Code: {res.status_code}")

    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")

    if len(res.json()) == 100:
        repo_urls += res.json()
        repo_urls += getDependabotAlertsEnterprise(enterprise, token, org, page + 1)
    else:
        repo_urls += res.json()

    return repo_urls

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

    if len(res.json()) == 100:
        alerts += res.json()
        alerts += getCodeScanningAlertsEnterprise(enterprise, token, org, page + 1)
    else:
        alerts += res.json()

    return alerts

# Main function to generate CSV files for multiple organizations in the enterprise
def main():
    dependabot_headers = ["Organization", "Repository_Name", "Alert ID", "ComponentName", "Package Name", "Ecosystem", "Manifest_Path", "Vulnerability Rating", "ShortDescription", "Description", "Vulnerability ID", "First_Patched_Version", "Unique ID", "CVSS Rating", "CVSS Version", "Vulnerabilities List", "Identifiers", "Vulnerable Version Range", "Github URL", "Date Discovered", "Base_Repo_Name"]
    code_scanning_headers = ["Organization", "Repository_Name", "Alert Number", "Rule ID", "Severity", "State", "Tool Name", "Description", "URL", "Created At", "Updated At"]
    
    # Retrieve the access token and enterprise name from environment variables or hardcode them
    GHToken = os.getenv("ACCESS_TOKEN")
    enterprise_name = "your_enterprise"  # Replace with your enterprise name
    
    # List of organizations to collect alerts from
    org_list = ["org1", "org2"]  # Replace with your organization names

    # Get the current working directory to save the CSVs in the current folder
    current_directory = os.getcwd()

    # Set the paths and names for the CSV files
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    dependabot_csv_path = os.path.join(current_directory, f"Dependabot_Vulnerabilities_{current_date}.csv")
    code_scanning_csv_path = os.path.join(current_directory, f"CodeScanning_Alerts_{current_date}.csv")

    # To track unique alerts and avoid duplicates
    unique_dependabot_ids = set()
    unique_code_scanning_ids = set()

    # Process Dependabot alerts
    with open(dependabot_csv_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(dependabot_headers)

        for org in org_list:
            dependaBotAlerts = getDependabotAlertsEnterprise(enterprise_name, GHToken, org)

            for alert in dependaBotAlerts:
                if "dependency" in alert:
                    alert_id = alert['number']
                    repo_name = alert['repository']['full_name']

                    if (alert_id, repo_name) not in unique_dependabot_ids:
                        unique_dependabot_ids.add((alert_id, repo_name))

                        Vuln_ID = alert['security_advisory']['cve_id'] if alert['security_advisory']['cve_id'] else alert['security_advisory']['ghsa_id']
                        CVSS_Version = re.search(r'CVSS:(.*?)V(\d)', str(alert['security_advisory']['cvss']['vector_string']))
                        if CVSS_Version:
                            CVSS_Version = CVSS_Version.group(1)
                        writer.writerow([
                            org,
                            repo_name,
                            f"GHASID-{alert_id}",
                            f"{alert['dependency']['package']['ecosystem']}:{alert['dependency']['package']['name']}",
                            alert['dependency']['package']['name'],
                            alert['dependency']['package']['ecosystem'],
                            alert['dependency']['manifest_path'],
                            alert['security_advisory']['severity'],
                            alert['security_advisory']['summary'].replace('\n', '').replace('\r', ''),
                            alert['security_advisory']['description'].replace('\n', ' ').replace('\r', ' '),
                            Vuln_ID,
                            alert['security_vulnerability']['first_patched_version']['identifier'] if alert['security_vulnerability']['first_patched_version'] else "Not patched",
                            f"GHASID-{alert_id}_{alert['dependency']['package']['name']}_{repo_name.replace('/', '_')}",
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
                        print(f"Duplicate Dependabot alert found: {alert_id} in {repo_name}")
                else:
                    print(f"No valid Dependabot data for org: {org}, repo: {alert.get('repository', {}).get('full_name')}")

    print(f"Dependabot CSV file generated at: {dependabot_csv_path}")
    print(f"Total unique Dependabot alerts written: {len(unique_dependabot_ids)}")

    # Process Code Scanning alerts
    with open(code_scanning_csv_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(code_scanning_headers)

        for org in org_list:
            codeScanningAlerts = getCodeScanningAlertsEnterprise(enterprise_name, GHToken, org)

            for alert in codeScanningAlerts:
                alert_id = alert['number']
                repo_name = alert['repository']['full_name']

                if (alert_id, repo_name) not in unique_code_scanning_ids:
                    unique_code_scanning_ids.add((alert_id, repo_name))

                    writer.writerow([
                        org,
                        repo_name,
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
                else:
                    print(f"Duplicate Code Scanning alert found: {alert_id} in {repo_name}")

    print(f"Code Scanning CSV file generated at: {code_scanning_csv_path}")
    print(f"Total unique Code Scanning alerts written: {len(unique_code_scanning_ids)}")

if __name__ == "__main__":
    main()
```

This script combines both Dependabot and Code Scanning alert retrieval into a single program. It will generate two separate CSV files:

1. A CSV file for Dependabot alerts
2. A CSV file for Code Scanning alerts

The script does the following:

1. Defines functions to retrieve both Dependabot and Code Scanning alerts from the GitHub API.
2. Sets up the necessary headers and file paths for both types of alerts.
3. Iterates through the specified organizations, retrieving both types of alerts for each.
4. Processes the alerts, avoiding duplicates, and writes them to their respective CSV files.
5. Provides console output for debugging and information purposes.

To use this script:

1. Ensure you have the necessary Python libraries installed (`requests`, `urllib3`).
2. Set your GitHub access token as an environment variable named `ACCESS_TOKEN`.
3. Replace `"your_enterprise"` with your actual enterprise name.
4. Update the `org_list` with the names of your organizations.
5. Run the script.

The script will generate two CSV files in your current working directory, one for Dependabot alerts and another for Code Scanning alerts, both dated with the current date.
