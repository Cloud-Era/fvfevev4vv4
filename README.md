Yes, you can retrieve **code scanning alerts** in a similar way as Dependabot alerts using GitHub's API for **Code Scanning Alerts**. The approach involves fetching the code scanning alerts for each repository and organization, processing the data, and writing it into a CSV file while ensuring no duplicates are included.

Here's an updated version of the script to fetch **Code Scanning Alerts** for all repositories within an organization and export the data to a CSV file.

### Code Scanning Alerts Script:

```python
import os
import json
import requests
import csv
import datetime
import warnings

warnings.filterwarnings("ignore")

# Function to get Code Scanning alerts for a repository
def getCodeScanningAlerts(enterprise, token, org, repo, page=1):
    repo_alerts = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    url = f"https://api.github.com/repos/{org}/{repo}/code-scanning/alerts?per_page=100&page={page}"
    res = requests.get(url, headers=headers, verify=False)

    # Debugging output
    print(f"Request URL: {res.url}")
    print(f"Response Status Code: {res.status_code}")

    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")

    if len(res.json()) == 100:
        repo_alerts += res.json()
        repo_alerts += getCodeScanningAlerts(enterprise, token, org, repo, page + 1)
    else:
        repo_alerts += res.json()

    return repo_alerts

# Function to get all repositories in an organization
def getRepositories(org, token):
    repo_list = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    url = f"https://api.github.com/orgs/{org}/repos?per_page=100"
    
    while url:
        res = requests.get(url, headers=headers, verify=False)
        if res.status_code != 200:
            raise Exception(f"Error: {res.status_code}, {res.text}")
        repo_list.extend(res.json())
        # Pagination handling: check if there are more pages
        if 'next' in res.links:
            url = res.links['next']['url']
        else:
            url = None

    return repo_list

# Main function to generate the CSV file for code scanning alerts
def main():
    headers = ["Organization", "Repository_Name", "Alert Number", "Rule Name", "Severity", "Description", "Tool", "Dismissed Reason", "Most Recent Instance", "Alert URL", "Created At"]

    # Retrieve the access token and enterprise name from environment variables or hardcode them
    GHToken = os.getenv("ACCESS_TOKEN")
    enterprise_name = "your_enterprise"  # Replace with your enterprise name

    # List of organizations to collect Code Scanning alerts from
    org_list = ["org1", "org2"]  # Replace with your organization names

    # Get the current working directory to save the CSV in the current folder
    current_directory = os.getcwd()

    # Set the path and name for the CSV file
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = os.path.join(current_directory, f"CodeScanningAlerts_{current_date}.csv")

    # To track unique alerts and avoid duplicates
    unique_alert_ids = set()

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(headers)

        for org in org_list:
            repositories = getRepositories(org, GHToken)

            # Process each repository in the organization
            for repo in repositories:
                repo_name = repo['name']
                codeScanningAlerts = getCodeScanningAlerts(enterprise_name, GHToken, org, repo_name)

                # Write the Code Scanning alerts for each repository to the CSV file
                for alert in codeScanningAlerts:
                    alert_id = alert['number']
                    if alert_id not in unique_alert_ids:
                        unique_alert_ids.add(alert_id)

                        writer.writerow([
                            org,
                            repo_name,
                            alert['number'],
                            alert['rule']['name'],
                            alert['rule']['severity'],
                            alert['rule']['description'],
                            alert['tool']['name'],
                            alert['dismissed_reason'] if alert['dismissed_reason'] else "None",
                            alert['most_recent_instance']['ref'] if 'most_recent_instance' in alert else "N/A",
                            alert['html_url'],
                            alert['created_at']
                        ])
                    else:
                        print(f"Duplicate alert found: {alert_id} in {repo_name}")

    print(f"CSV file generated at: {csv_file_path}")
    print(f"Total unique alerts written: {len(unique_alert_ids)}")

if __name__ == "__main__":
    main()
```

### Explanation of Key Components:

1. **Fetching Repositories**:
   - The `getRepositories` function retrieves all repositories in the organization.
   - It handles pagination for large organizations.

2. **Fetching Code Scanning Alerts**:
   - The `getCodeScanningAlerts` function retrieves code scanning alerts from GitHub’s API for each repository.
   - It uses pagination to handle repositories with more than 100 alerts.

3. **Avoiding Duplicates**:
   - Like the Dependabot alerts script, this script tracks unique alert IDs using the `unique_alert_ids` set to ensure no duplicate alerts are written to the CSV.

4. **CSV Headers**:
   - The CSV headers include details such as `Organization`, `Repository_Name`, `Alert Number`, `Rule Name`, `Severity`, `Description`, `Tool`, `Dismissed Reason`, `Most Recent Instance`, `Alert URL`, and `Created At`.

5. **Logging**:
   - The script prints information about duplicate alerts and successful CSV generation.

### How to Use the Script:
1. Replace `org1` and `org2` in the `org_list` with your organization names.
2. Set the GitHub access token either as an environment variable (`ACCESS_TOKEN`) or hardcode it in the script.
3. Run the script, and it will generate a CSV file in the current folder with all the code scanning alerts for your specified organizations.

### CSV Output:
The CSV will contain the following fields for each code scanning alert:
- `Organization`: The name of the organization.
- `Repository_Name`: The repository where the alert was found.
- `Alert Number`: The unique ID of the code scanning alert.
- `Rule Name`: The name of the rule that triggered the alert.
- `Severity`: The severity level (e.g., high, medium, low).
- `Description`: A short description of the issue.
- `Tool`: The name of the scanning tool (e.g., CodeQL).
- `Dismissed Reason`: The reason the alert was dismissed (if applicable).
- `Most Recent Instance`: The branch or ref where the alert was found.
- `Alert URL`: The GitHub URL of the alert.
- `Created At`: The timestamp when the alert was created.

Let me know if you need any further adjustments or if you encounter any issues!
