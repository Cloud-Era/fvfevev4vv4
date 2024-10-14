Given that the SBOM JSON output contains entries indicating Angular versions but the script is still reporting no findings, we need to adjust how we search through the SBOM components. 

### Updated Logic

1. **Search for Angular Dependencies**: We will look for dependencies using the `"name"` and `"versionInfo"` fields, which are present in your SBOM. 
2. **Modify the Version Check**: We'll check if the `versionInfo` indicates Angular 12 or above.

Here’s an updated version of the script, focusing on accurately finding Angular dependencies in the SBOM:

### Updated Script

```python
import asyncio
import aiohttp
import csv
import os
import re
import json

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
            if "name" in component and "versionInfo" in component:
                normalized_name = component["name"].lower().replace(" ", "")
                version_info = component["versionInfo"]
                
                # Check for Angular packages
                if re.match(r"^npm:@angular/(.*)$", normalized_name):
                    # Extract the major version from version_info
                    version_parts = version_info.split('.')
                    if len(version_parts) > 0 and version_parts[0].isdigit():
                        major_version = int(version_parts[0])  # Extract major version
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

### Changes Made:
1. **Check for `versionInfo`**: The script now checks for `versionInfo` instead of `version` to correctly extract the version information.
2. **Regex for Angular Packages**: The regex pattern checks for Angular packages specifically from npm with `npm:@angular/` prefix.
3. **Handling Versions**: The script uses `versionInfo` for version checks, ensuring it reads the actual version string where available.

### Usage:
- **GitHub Token**: Ensure you replace `your_github_token_here` with your actual GitHub token.
- **Repository Name**: Replace `your_repo_name_here` with the name of the specific repository you want to analyze.
- **Run the Script**: Execute the script, and it should accurately detect the presence of Angular 12 or higher in the specified repository.

### Debugging Steps:
If you still encounter issues:
1. **Print the SBOM**: Before the version check, print the SBOM to see its contents:
   ```python
   print(json.dumps(sbom, indent=2))  # Print the full SBOM JSON
   ```
   This will help verify that the expected fields are present and contain the correct information.

2. **Validate the Regex**: Ensure the package names in the SBOM match the regex criteria.

Please let me know if this update resolves the issue or if further adjustments are needed!
