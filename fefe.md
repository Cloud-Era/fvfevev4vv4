Sure! Based on your request, I'll create a script that fetches repositories from a GitHub organization and checks for the use of Angular 12 or above in their SBOM (Software Bill of Materials). This will include gathering the repository name, `eon_id`, and a flag indicating whether the repo uses Angular 12 or above.

Here’s the updated script:

```python
import requests
import json
import os
import time

# Hard-coded GitHub token (replace with your actual token)
GITHUB_TOKEN = "your_github_token_here"

# Variable to easily switch between "EoNID" and "eon_id"
ID_PROPERTY_NAME = "eon_id"  # Change this to "EoNID" if needed

# Organization name
ORG = "cloud-era"

# Output folder
OUTPUT_FOLDER = f"{ORG}_repos_output"

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

def process_repo(org, repo_name):
    print(f"Processing {repo_name}...")
    sbom = get_repo_sbom(org, repo_name)
    id_value = get_repo_id(org, repo_name)

    if sbom is not None:
        uses_angular = check_angular_version(sbom)
    else:
        uses_angular = False  # Assume no Angular if SBOM fetching fails

    output = {
        "repository": f"{org}/{repo_name}",
        ID_PROPERTY_NAME: id_value,
        "uses_angular": uses_angular
    }

    # Save JSON output to file
    filename = f"{repo_name}_report.json"
    filepath = os.path.join(OUTPUT_FOLDER, filename)
    with open(filepath, 'w') as f:
        json.dump(output, f, indent=2)
    print(f"Output for {repo_name} saved to {filepath}")

def main():
    # Create output folder if it doesn't exist
    if not os.path.exists(OUTPUT_FOLDER):
        os.makedirs(OUTPUT_FOLDER)

    print(f"Fetching repositories for {ORG}...")
    repos = get_org_repos(ORG)

    if repos is None:
        print("Failed to fetch repositories. Exiting.")
        return

    print(f"Found {len(repos)} repositories. Processing each repository...")

    for repo in repos:
        process_repo(ORG, repo['name'])
        time.sleep(1)  # To avoid hitting rate limits

    print(f"\nAll repositories processed. Outputs saved in the '{OUTPUT_FOLDER}' folder.")

if __name__ == "__main__":
    main()
```

### Key Changes:
1. **Check for Angular Version**:
   - The function `check_angular_version` checks if Angular 12 or above is listed in the SBOM by examining the `"components"` field.

2. **Output Structure**:
   - Each repository's report will include:
     - `repository`: The full repository name (e.g., `cloud-era/repo-name`).
     - `eon_id`: The associated Eon ID.
     - `uses_angular`: A boolean indicating whether the repository uses Angular 12 or above.

### Running the Script:
- Replace `your_github_token_here` with your actual GitHub token.
- Ensure your GitHub organization name is correct.
- Run the script. It will process each repository in the specified organization and save the output as JSON files in the designated output folder.

Let me know if you have any further questions or need additional modifications!
