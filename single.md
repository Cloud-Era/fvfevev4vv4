Certainly! If you'd like to check the `package.json` file in the repository instead of relying on the SBOM, we can modify the script to directly fetch and parse the `package.json` file for Angular dependencies.

Here's how we can do that:

### Steps to Update the Script

1. **Fetch `package.json`**: Retrieve the `package.json` file from the repository.
2. **Check for Angular Dependencies**: Look for dependencies with names starting with `@angular/` and ensure their versions are 12 or above.
3. **Output the Results**: Generate a CSV report with the repository name and the associated EoN ID if Angular 12 or higher is found.

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

async def get_package_json(session, org, repo):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    # URL to fetch package.json file
    package_json_url = f"https://api.github.com/repos/{org}/{repo}/contents/package.json"
    async with session.get(package_json_url, headers=headers) as response:
        if response.status == 200:
            # Decode the base64 content
            content = await response.json()
            package_json_content = json.loads(content['content'])
            return package_json_content
        else:
            print(f"Error fetching package.json for {repo}: {response.status}")
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

def check_angular_version(dependencies):
    # Check if Angular 12 or above is listed in the dependencies
    for package, version in dependencies.items():
        if package.startswith("@angular/"):
            # Extract the major version from version string
            version_parts = version.split('.')
            if len(version_parts) > 0 and version_parts[0].isdigit():
                major_version = int(version_parts[0])  # Extract major version
                if major_version >= ANGULAR_VERSION:
                    return True
    return False

async def main():
    async with aiohttp.ClientSession() as session:
        print(f"Fetching package.json and {ID_PROPERTY_NAME} for {ORG}/{REPO}...")
        
        package_json = await get_package_json(session, ORG, REPO)
        id_value = await get_repo_id(session, ORG, REPO)

        if package_json is not None and "dependencies" in package_json:
            if check_angular_version(package_json["dependencies"]):
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
        else:
            print(f"Package.json not found or does not contain dependencies for {ORG}/{REPO}.")

if __name__ == "__main__":
    asyncio.run(main())
```

### Explanation of the Script

1. **Fetching `package.json`**: The script retrieves the `package.json` file from the specified repository using the GitHub API.
2. **Checking Angular Dependencies**: It iterates through the `dependencies` section of the `package.json` to find any Angular packages and checks if their major version is 12 or higher.
3. **CSV Reporting**: If Angular 12 or higher is found, it generates a CSV report with the repository name and EoN ID.

### Usage
- **Replace the Token**: Ensure you replace `your_github_token_here` with your actual GitHub token.
- **Specify the Repository**: Replace `your_repo_name_here` with the actual repository name you want to analyze.
- **Run the Script**: Execute the script, and it should check the `package.json` for Angular dependencies.

### Note
- Make sure the `package.json` file exists in the root directory of your repository; otherwise, the script will not find it.
- If the repository is private, ensure your GitHub token has the required permissions to access it.

Please let me know if you have any questions or if you'd like further modifications!
