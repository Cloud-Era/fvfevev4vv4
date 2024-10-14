Understood! Let's simplify the script to generate a CSV report containing only the repository name and the associated `eon_id` for repositories that use Angular 12 or above, while excluding archived repositories. The output will be saved in a CSV format instead of JSON files.

Here’s the updated script:

```python
import requests
import csv
import os
import time

# Hard-coded GitHub token (replace with your actual token)
GITHUB_TOKEN = "your_github_token_here"

# Variable to easily switch between "EoNID" and "eon_id"
ID_PROPERTY_NAME = "eon_id"  # Change this to "EoNID" if needed

# Organization name
ORG = "cloud-era"

# Output CSV file path
OUTPUT_CSV = f"{ORG}_repos_with_angular.csv"

# Define Angular version to check
ANGULAR_VERSION = 12

def get_org_repos(org):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    repos = []
    page = 1
    while True:
        url = f"https://api.github.com/orgs/{org}/repos?per_page=100&page={page}"
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            page_repos = response.json()
            if not page_repos:
                break
            repos.extend(page_repos)
            page += 1
        else:
            print(f"Error fetching repositories: {response.status_code}")
            print(response.text)
            return None
        time.sleep(1)  # To avoid hitting rate limits
    return repos

def get_repo_sbom(org, repo):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    sbom_url = f"https://api.github.com/repos/{org}/{repo}/dependency-graph/sbom"
    response = requests.get(sbom_url, headers=headers)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error fetching SBOM for {repo}: {response.status_code}")
        return None

def get_repo_id(org, repo):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    properties_url = f"https://api.github.com/repos/{org}/{repo}/properties/values"
    response = requests.get(properties_url, headers=headers)
    
    if response.status_code == 200:
        properties_data = response.json()
        for prop in properties_data:
            if prop.get("property_name") == ID_PROPERTY_NAME:
                return prop.get("value")
        print(f"{ID_PROPERTY_NAME} not found for {repo}")
    else:
        print(f"Error fetching properties for {repo}: {response.status_code}")
    
    return None

def check_angular_version(sbom):
    # Check if Angular 12 or above is listed in the SBOM dependencies
    if "components" in sbom:
        for component in sbom["components"]:
            if "name" in component and "version" in component:
                if "angular" in component["name"].lower() and int(component["version"].split('.')[0]) >= ANGULAR_VERSION:
                    return True
    return False

def main():
    print(f"Fetching repositories for {ORG}...")
    repos = get_org_repos(ORG)

    if repos is None:
        print("Failed to fetch repositories. Exiting.")
        return

    print(f"Found {len(repos)} repositories. Processing each repository...")

    with open(OUTPUT_CSV, mode='w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['repository', 'eon_id']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()

        for repo in repos:
            if repo['archived']:
                continue  # Skip archived repositories

            sbom = get_repo_sbom(ORG, repo['name'])
            id_value = get_repo_id(ORG, repo['name'])

            if sbom is not None and check_angular_version(sbom):
                writer.writerow({
                    'repository': f"{ORG}/{repo['name']}",
                    'eon_id': id_value
                })
                print(f"Added {repo['name']} to report.")

            time.sleep(1)  # To avoid hitting rate limits

    print(f"\nReport generated: '{OUTPUT_CSV}'")

if __name__ == "__main__":
    main()
```

### Key Changes:
1. **CSV Output**: The script now generates a CSV file named `cloud-era_repos_with_angular.csv` containing:
   - `repository`: The repository name.
   - `eon_id`: The associated `eon_id`.

2. **Archiving Check**: The script skips any repositories that are archived.

3. **Removed JSON Output**: The script no longer saves individual repository data in JSON format.

### Running the Script:
- Replace `your_github_token_here` with your actual GitHub token.
- Ensure the organization name (`ORG`) is correct.
- Run the script, and it will create a CSV file in the same directory as the script.

Let me know if you need any further adjustments or assistance!
