It seems like the logic for checking the Angular version in the SBOM might not be correctly identifying the components. Let's refine the function that checks for Angular dependencies to ensure it correctly identifies packages and versions.

### Updated Script

Here’s the revised version of the script that properly checks for Angular packages:

```python
import asyncio
import aiohttp
import csv
import os
import re

# Hard-coded GitHub token (replace with your actual token)
GITHUB_TOKEN = "your_github_token_here"

# Variable to easily switch between "EoNID" and "eon_id"
ID_PROPERTY_NAME = "eon_id"  # Change this to "EoNID" if needed

# Repository details
ORG = "cloud-era"  # Your organization name
REPO = "your_repo_name_here"  # Specify the repository name

# Output CSV file path
OUTPUT_CSV = f"{REPO}_angular_report.csv"

# Define Angular version to check
ANGULAR_VERSION = 12

async def get_repo_sbom(session, org, repo):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    sbom_url = f"https://api.github.com/repos/{org}/{repo}/dependency-graph/sbom"
    async with session.get(sbom_url, headers=headers) as response:
        if response.status == 200:
            return await response.json()
        else:
            print(f"Error fetching SBOM for {repo}: {response.status}")
            return None

async def get_repo_id(session, org, repo):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    properties_url = f"https://api.github.com/repos/{org}/{repo}/properties/values"
    async with session.get(properties_url, headers=headers) as response:
        if response.status == 200:
            properties_data = await response.json()
            for prop in properties_data:
                if prop.get("property_name") == ID_PROPERTY_NAME:
                    return prop.get("value")
            print(f"{ID_PROPERTY_NAME} not found for {repo}")
        else:
            print(f"Error fetching properties for {repo}: {response.status}")

    return None

def check_angular_version(sbom):
    # Check if Angular 12 or above is listed in the SBOM dependencies
    if "components" in sbom:
        for component in sbom["components"]:
            # Check for Angular in the name and extract version
            if "name" in component and "version" in component:
                # Normalize name by removing spaces and casing
                normalized_name = component["name"].lower().replace(" ", "")
                # Match Angular packages
                if re.match(r"^@angular/(.*)$", normalized_name):
                    version = component["version"]
                    major_version = int(version.split('.')[0])  # Extract major version
                    if major_version >= ANGULAR_VERSION:
                        return True
    return False

async def main():
    async with aiohttp.ClientSession() as session:
        print(f"Fetching SBOM and {ID_PROPERTY_NAME} for {ORG}/{REPO}...")
        
        sbom = await get_repo_sbom(session, ORG, REPO)
        id_value = await get_repo_id(session, ORG, REPO)

        if sbom is not None and check_angular_version(sbom):
            print(f"Angular {ANGULAR_VERSION} or above found in {ORG}/{REPO}.")
            with open(OUTPUT_CSV, mode='w', newline='', encoding='utf-8') as csvfile:
                fieldnames = ['repository', 'eon_id']
                writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
                writer.writeheader()
                writer.writerow({
                    'repository': f"{ORG}/{REPO}",
                    'eon_id': id_value
                })
            print(f"Report generated: '{OUTPUT_CSV}'")
        else:
            print(f"No Angular {ANGULAR_VERSION} or above found in {ORG}/{REPO}.")

if __name__ == "__main__":
    asyncio.run(main())
```

### Key Changes:
1. **Regular Expression Matching**: The `check_angular_version` function now uses a regular expression to identify Angular packages, which should ensure that all packages prefixed with `@angular/` are correctly detected.

2. **Normalization**: The name of the component is normalized by removing spaces and converting it to lowercase, which should help in matching Angular packages accurately.

3. **Version Extraction**: The major version is extracted correctly to compare with `ANGULAR_VERSION`.

### How to Use:
- Replace `your_github_token_here` with your actual GitHub token.
- Set `your_repo_name_here` with the name of the repository you want to check.
- Run the script.

This should help in accurately identifying whether Angular 12 or above is present in the specified repository. If you have further issues or additional requests, feel free to ask!
