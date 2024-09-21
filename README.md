Great! To ensure that all alerts are captured and there are no duplicates, I've made a few enhancements to the script to handle potential issues like duplicates and pagination correctly.

Here’s the refined version of the script, which ensures no duplicates and proper pagination:

### Enhanced Script for One Organization:
```python
import os
import requests
import urllib3
import csv
import datetime
import warnings

warnings.filterwarnings("ignore")
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Function to get Code Scanning alerts for a single organization
def getCodeScanningAlertsOrg(org, token, page=1):
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
    print(f"Fetched {len(data)} alerts from page {page} for {org}")

    # Recursively fetch all pages
    if len(data) == 100:
        alerts.extend(getCodeScanningAlertsOrg(org, token, page + 1))

    return alerts

# Main function to generate CSV file for code scanning alerts for one organization
def main():
    headers = ["Organization", "Repository_Name", "Alert Number", "Rule ID", "Severity", "State", "Tool Name", "Description", "URL", "Created At", "Updated At"]

    GHToken = os.getenv("ACCESS_TOKEN")  # Ensure GitHub token is set in environment variables
    org = "your_org_name"  # Replace with the actual organization name

    current_directory = os.getcwd()
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"CodeScanning_Alerts_{current_date}.csv")

    unique_alert_ids = set()

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        # Get code scanning alerts for the specified organization
        codeScanningAlerts = getCodeScanningAlertsOrg(org, GHToken)

        for alert in codeScanningAlerts:
            alert_id = alert['number']
            repo_name = alert['repository']['full_name']

            # Ensure no duplicates
            if (alert_id, repo_name) not in unique_alert_ids:
                unique_alert_ids.add((alert_id, repo_name))

                # Handle severity - default to 'Unknown' if not found
                severity = alert.get('rule', {}).get('severity', 'Unknown')

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
                print(f"Duplicate alert skipped: {alert_id} in {repo_name}")

    print(f"Code Scanning CSV file generated at: {csv_file_path}")
    print(f"Total unique Code Scanning alerts written: {len(unique_alert_ids)}")

if __name__ == "__main__":
    main()
```

### Key Changes to Ensure Accuracy:
1. **Single Organization**: The `org` variable is set to fetch alerts for just one organization. Replace `"your_org_name"` with the actual organization name.
   
2. **No Duplicates**: The `unique_alert_ids` set ensures that no duplicate alerts are written to the CSV. Each alert is identified by its `alert_id` and `repo_name` combination.

3. **Pagination**: The script uses recursion to handle multiple pages of alerts (if more than 100 alerts exist).

4. **Severity Handling**: The script defaults to `"Unknown"` if the severity is missing, ensuring that all alerts are accounted for.

5. **Logs**: The script will print useful information about duplicate alerts and how many alerts were fetched per page.

### How to Run:
1. Set your GitHub token in an environment variable:
   - On Linux/macOS: `export ACCESS_TOKEN=your_github_token`
   - On Windows: `set ACCESS_TOKEN=your_github_token`
   
2. Run the script.

3. It will generate a CSV file in the current directory named `CodeScanning_Alerts_<current_date>.csv`.

This should capture all code scanning alerts for the specified organization and avoid duplicates. Let me know how it works or if further adjustments are needed!
