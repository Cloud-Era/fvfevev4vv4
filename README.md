To retrieve **code scanning alerts** in a similar way to how the **Dependabot alerts** are fetched at the enterprise level, you can create a script that aggregates code scanning alerts across all repositories in an organization or enterprise. However, you'll still need to fetch them one repository at a time since the GitHub API for code scanning alerts is not available at the enterprise level directly.

Here's a revised script that collects code scanning alerts for each repository in an organization and handles pagination, similar to the Dependabot alerts script.

### Code Scanning Alerts Script

```python
import os
import json
import requests
import csv
import datetime

# Function to get all repositories for the organization
def getRepositories(org, token, page=1):
    repos = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    res = requests.get(f"https://api.github.com/orgs/{org}/repos?per_page=100&page={page}", headers=headers)
    
    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")
    
    repos += res.json()
    
    if len(res.json()) == 100:
        repos += getRepositories(org, token, page + 1)
    
    return repos

# Function to get code scanning alerts for a repository
def getCodeScanningAlertsRepo(org, repo, token, page=1):
    alerts = []
    headers = {'Authorization': f"Bearer {token}", "Accept": "application/vnd.github+json"}
    res = requests.get(f"https://api.github.com/repos/{org}/{repo}/code-scanning/alerts?per_page=100&page={page}", headers=headers)
    
    if res.status_code == 404:
        print(f"No analysis found for repository: {repo}. Skipping.")
        return alerts
    
    if res.status_code != 200:
        raise Exception(f"Error: {res.status_code}, {res.text}")
    
    alerts += res.json()
    
    if len(res.json()) == 100:
        alerts += getCodeScanningAlertsRepo(org, repo, token, page + 1)
    
    return alerts

# Main function to aggregate alerts and write to CSV
def main():
    # Retrieve the access token and organization name from environment variables or hardcode them
    GHToken = os.getenv("ACCESS_TOKEN")
    org_name = "cloud-era"  # Replace with your organization name

    # Get all repositories in the organization
    repositories = getRepositories(org_name, GHToken)

    total_alerts = []
    
    for repo in repositories:
        repo_name = repo['name']
        alerts = getCodeScanningAlertsRepo(org_name, repo_name, GHToken)
        total_alerts.extend(alerts)

    # Write alerts to CSV
    current_date = datetime.datetime.now().strftime("%m-%d-%Y")
    csv_file_path = f"./CodeScanningAlerts_{current_date}.csv"

    with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Repository_Name", "Alert_ID", "Description", "Severity", "State", "URL"])
        
        for alert in total_alerts:
            writer.writerow([
                alert['repository']['full_name'],
                alert['number'],
                alert['description'],
                alert['severity'],
                alert['state'],
                alert['html_url']
            ])
    
    print(f"Code scanning alerts saved to {csv_file_path}")

if __name__ == "__main__":
    main()
```

### Key Points:
1. **Repository Retrieval**:
   - The script first fetches all repositories in the specified organization using the `getRepositories` function, handling pagination.

2. **Fetching Code Scanning Alerts**:
   - For each repository, it retrieves code scanning alerts using the `getCodeScanningAlertsRepo` function, which also handles pagination.
   - If a repository has no analysis (404), it logs a message and continues to the next repository.

3. **CSV Output**:
   - All collected alerts are written to a CSV file named `CodeScanningAlerts_<current_date>.csv` in the current directory.

4. **Access Token**:
   - Ensure you set the `ACCESS_TOKEN` environment variable with your GitHub token that has the necessary permissions.

This script efficiently collects all code scanning alerts across an organization while properly managing pagination and error handling.
