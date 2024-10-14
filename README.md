In this script, since you're working with **local SBOM files** stored in a folder, you don't need to pass the organization name or GitHub token. However, if your plan is to **fetch SBOMs directly from GitHub or through an API** for each repository under an organization, you'll need to integrate GitHub API calls to get those SBOMs.

Here’s how you can modify the script to pass the **GitHub organization name** and **token**, fetch the SBOMs from GitHub repositories dynamically, and then check for the presence of Angular 12 dependencies:

### Modifying the Script to Fetch SBOMs from GitHub:

1. **Set GitHub Organization Name and Token**.
2. **Fetch SBOM files from GitHub** for each repository in the organization.

Here’s the updated script with GitHub API integration:

```python
import os
import json
import csv
import requests

# GitHub token and org name
GITHUB_TOKEN = "your_github_token_here"
ORG_NAME = "cloud-era"  # Replace with the name of your organization

# Constants for paths
CURRENT_DIR = os.getcwd()

# Use the current directory for SBOM_FOLDER_PATH and MAPPING_CSV if you're saving any local files
SBOM_FOLDER_PATH = os.path.join(CURRENT_DIR, "sboms")
MAPPING_CSV = os.path.join(CURRENT_DIR, "sbom_mapping.csv")
ANGULAR_VERSION = "12"

# GitHub headers with authentication token
HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github+json",
}

# Fetch list of repositories from GitHub for the given organization
def get_repositories(org_name):
    url = f"https://api.github.com/orgs/{org_name}/repos?per_page=100"
    response = requests.get(url, headers=HEADERS)

    if response.status_code == 200:
        return response.json()
    else:
        print(f"Failed to fetch repositories. Status code: {response.status_code}")
        return []

# Fetch the latest SBOM for a repository (this assumes you're storing SBOMs in repo releases or similar)
def fetch_sbom_for_repo(org_name, repo_name):
    sbom_url = f"https://api.github.com/repos/{org_name}/{repo_name}/contents/sbom.json"  # Assuming sbom.json is used
    response = requests.get(sbom_url, headers=HEADERS)

    if response.status_code == 200:
        sbom_data = response.json()
        sbom_content = sbom_data['content']
        return json.loads(sbom_content)
    else:
        print(f"Failed to fetch SBOM for {repo_name}. Status code: {response.status_code}")
        return None

# Check for Angular 12 dependency in the SBOM
def check_angular_dependency(sbom_data):
    components = sbom_data.get("components", [])
    for component in components:
        if component.get("name") == "angular" and component.get("version", "").startswith(ANGULAR_VERSION):
            return True
    return False

# Load the mapping of repositories to eon_ids
def load_eon_id_mapping(csv_file):
    repo_to_eon_id = {}
    with open(csv_file, mode='r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            repo_to_eon_id[row['sbom_name']] = row['eon_id']
    return repo_to_eon_id

# Generate a report of repositories referencing Angular 12
def generate_report(org_name, mapping_csv):
    repositories = get_repositories(org_name)
    eon_id_mapping = load_eon_id_mapping(mapping_csv)
    report = []

    for repo in repositories:
        repo_name = repo['name']
        print(f"Fetching SBOM for {repo_name}...")
        sbom_data = fetch_sbom_for_repo(org_name, repo_name)
        
        if sbom_data and check_angular_dependency(sbom_data):
            eon_id = eon_id_mapping.get(repo_name, "N/A")
            report.append({"repository": repo_name, "eon_id": eon_id})

    return report

# Save the report to a CSV file
def save_report_to_csv(report, output_file):
    with open(output_file, 'w', newline='') as csvfile:
        fieldnames = ['repository', 'eon_id']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

        writer.writeheader()
        for row in report:
            writer.writerow(row)

# Main function to generate the report
def main():
    mapping_csv = MAPPING_CSV
    output_csv = os.path.join(CURRENT_DIR, "angular12_repos_with_eon_id.csv")  # Output in current directory

    if not os.path.exists(mapping_csv):
        print(f"Mapping CSV '{mapping_csv}' does not exist. Please check the path.")
        return

    print(f"Generating report for organization: {ORG_NAME}...")
    report = generate_report(ORG_NAME, mapping_csv)
    save_report_to_csv(report, output_csv)

    print(f"Report saved to {output_csv}")

if __name__ == "__main__":
    main()
```

### Key Changes:
1. **GitHub Organization and Token**:
   - `GITHUB_TOKEN`: You need to replace `"your_github_token_here"` with your actual GitHub token that has permission to access the organization repositories.
   - `ORG_NAME`: Set your organization name.

2. **Fetching Repositories**:
   - The function `get_repositories` fetches all repositories for the given organization using the GitHub API.

3. **Fetching SBOM**:
   - The function `fetch_sbom_for_repo` retrieves the SBOM for a given repository. It assumes the SBOM is stored as a file in the repository (e.g., `sbom.json`). You might need to adjust this path based on where SBOMs are stored.

4. **Checking for Angular 12**:
   - `check_angular_dependency` checks if Angular 12 is listed as a dependency in the SBOM.

5. **Generate and Save Report**:
   - The `generate_report` function creates a report for repositories that reference Angular 12 and associates them with the corresponding `eon_id` from the mapping CSV.
   - `save_report_to_csv` saves the report to a CSV file.

### Output:
- The script will output a CSV file `angular12_repos_with_eon_id.csv`, listing all repositories using Angular 12 along with their associated `eon_id` from the mapping.

### Assumptions:
- SBOMs are stored in each GitHub repository under a file (e.g., `sbom.json`). If SBOMs are stored elsewhere (like in releases or other artifacts), you'll need to adjust the `fetch_sbom_for_repo` function accordingly.
