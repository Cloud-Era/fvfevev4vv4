If the script still outputs that no Angular 12 or above dependencies are found despite seeing them in the SBOM, we need to double-check how we’re parsing the SBOM and the version comparison logic. Let's ensure we're correctly accessing the components in the SBOM JSON and handling the version string appropriately.

Here’s a refined approach to the script, focusing on correctly extracting and validating the Angular dependencies from the SBOM:

### Updated Script

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
                    # Handle cases where the version might include a range or additional info
                    version_parts = version.split('.')
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

### Key Adjustments:
1. **Version Handling**: The `check_angular_version` function now splits the version string to extract the major version. It also checks if the first part of the version is a digit to prevent errors.

2. **Output Structure**: We ensure that the output CSV only lists repositories that have Angular 12 or above.

### How to Use:
1. **Token and Repository**: Make sure to replace `your_github_token_here` with your actual GitHub token and set `your_repo_name_here` with the repository you want to check.
   
2. **Run the Script**: Execute the script and check the output. It should now accurately identify if Angular 12 or above is present in the SBOM.

### Debugging Steps
If the script still reports no Angular dependencies found:
- **Print SBOM**: Add a print statement to output the SBOM contents for inspection:
   ```python
   print(json.dumps(sbom, indent=2))  # Print the full SBOM JSON
   ```
   This will help verify that the SBOM structure is as expected and confirm the presence of Angular dependencies.

Let me know if you need any further adjustments or if you want to extend the functionality!
