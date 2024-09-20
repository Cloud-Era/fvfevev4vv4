It seems like the script might be generating duplicate rows or processing data in a way that leads to an inflated number of rows in the CSV. To troubleshoot the issue, let's focus on a few key areas where the problem might arise:

### Potential Issues:
1. **Duplicate Alerts**: If the script is fetching the same alerts multiple times due to pagination or mis-handling of responses, this could explain the inflated row count.
2. **Multiple Ecosystems or Components**: If each alert includes multiple packages or vulnerabilities, the script may be outputting more rows per alert than necessary.
3. **Pagination**: The script uses pagination, but if not handled properly, it may re-fetch the same alerts across multiple pages, leading to duplication.

### Steps to Investigate and Fix:
1. **Check for Duplicate Data**:
   - Ensure that each alert is only written once to the CSV by keeping track of the unique alert IDs.
   
2. **Simplify the Data Handling**:
   - Review if each alert in the response is being expanded into multiple rows unnecessarily.

3. **Logging for Debugging**:
   - Add some logging to verify what data is being processed and how many unique alerts are actually fetched.

Let’s start by adding logging and making sure each alert is only processed once.

### Updated Script with Duplicate Check and Debug Logging:

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

    # Get the current working directory to save the CSV in the current folder
    current_directory = os.getcwd()

    # Set the path and name for the CSV file
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"Vulnerabilities_{current_date}.csv")

    # To track unique alerts and avoid duplicates
    unique_alert_ids = set()

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        for org in org_list:
            dependaBotAlerts = getDependabotAlertsEnterprise(enterprise_name, GHToken, org)

            # Write the Dependabot alerts for each organization to the CSV file
            for alert in dependaBotAlerts:
                if "dependency" in alert:
                    alert_id = alert['number']
                    repo_name = alert['repository']['full_name']

                    # Check if this alert has already been written
                    if (alert_id, repo_name) not in unique_alert_ids:
                        unique_alert_ids.add((alert_id, repo_name))  # Track the unique alert ID and repo

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
                        print(f"Duplicate alert found: {alert_id} in {repo_name}")
                else:
                    print(f"No valid data for org: {org}, repo: {alert.get('repository', {}).get('full_name')}")

    print(f"CSV file generated at: {csv_file_path}")
    print(f"Total unique alerts written: {len(unique_alert_ids)}")

if __name__ == "__main__":
    main()
```

### Key Updates:
1. **Tracking Unique Alerts**:
   - The script uses a `unique_alert_ids` set to track each unique alert by its `alert_id` and `repo_name`. This ensures that the same alert is not written multiple times.
   
2. **Duplicate Check**:
   - Before writing an alert to the CSV, the script checks if the `(alert_id, repo_name)` combination has already been processed.
   
3. **Logging**:
   - The script prints a message if a duplicate alert is found and skips it. It also prints the total number of unique alerts processed after writing the CSV.

### Next Steps:
- **Run the updated script** and check the CSV file.
- Ensure that the number of rows matches the expected number of unique open alerts in your enterprise.
  
This should resolve the issue of the inflated row count in the CSV. If the problem persists, you may want to check the GitHub API responses to ensure no alerts are being fetched multiple times across different pages.
