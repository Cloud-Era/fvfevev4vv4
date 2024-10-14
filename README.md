Thank you for the clarification! If you simply want a report from GitHub that includes:

1. **Repository name**
2. **Eon ID**
3. **Repositories using Angular 12 framework**

We can streamline the process without using local files like SBOMs or a CSV mapping file. Instead, we'll:

1. Fetch repositories from your GitHub organization.
2. For each repository, we'll check its SBOM (or package manager files, if available) to detect if it references Angular version 12.
3. Return the `repo_name` and `eon_id` (if available) in the report.

### Key Assumptions:
- The **SBOM** or dependency information (like `package.json`) is stored in the repository, either in a `package.json` or as part of the repository contents, and contains Angular 12 if used.
- The `eon_id` is fetched using repository properties or stored in some standard way in the repo (this part may vary depending on how you manage eon_id in your repos).

### Simple Script (GitHub API):

Here's a Python script that meets your requirements:

```python
import requests
import json

# GitHub token and organization name
GITHUB_TOKEN = "your_github_token_here"  # Replace with your actual GitHub token
ORG_NAME = "cloud-era"  # Replace with your organization name
ANGULAR_VERSION = "12"

# GitHub headers
HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github+json",
}

# Fetch the list of repositories from the organization
def get_repositories(org_name):
    url = f"https://api.github.com/orgs/{org_name}/repos?per_page=100"
    response = requests.get(url, headers=HEADERS)

    if response.status_code == 200:
        return response.json()
    else:
        print(f"Failed to fetch repositories. Status code: {response.status_code}")
        return []

# Check for Angular 12 in package.json (or another file that lists dependencies)
def check_angular_dependency(repo_name):
    url = f"https://api.github.com/repos/{ORG_NAME}/{repo_name}/contents/package.json"  # Modify if needed
    response = requests.get(url, headers=HEADERS)

    if response.status_code == 200:
        package_json = response.json()
        content = package_json.get('content')
        
        if content:
            package_data = json.loads(content.decode('base64'))  # GitHub sends file content base64 encoded
            dependencies = package_data.get("dependencies", {})

            # Check if Angular version 12 is listed in dependencies
            if "angular" in dependencies and dependencies["angular"].startswith(ANGULAR_VERSION):
                return True
    return False

# Fetch eon_id from repository properties (assume eon_id is stored somewhere like in the properties)
def fetch_eon_id(repo_name):
    # Placeholder: Modify this to fetch the actual eon_id for the repo, if stored somewhere
    # In this example, I'm assuming you have an API or a standard location for eon_id
    return "eon_id_placeholder"  # Replace with actual logic to fetch eon_id

# Generate a report for repositories using Angular 12
def generate_report():
    repositories = get_repositories(ORG_NAME)
    report = []

    for repo in repositories:
        repo_name = repo['name']
        print(f"Checking Angular dependency for {repo_name}...")

        # Check if the repo uses Angular 12
        if check_angular_dependency(repo_name):
            # Fetch eon_id (assuming it's available in some form)
            eon_id = fetch_eon_id(repo_name)
            report.append({"repo_name": repo_name, "eon_id": eon_id})

    return report

# Main function to generate and print the report
def main():
    print(f"Generating report for organization: {ORG_NAME}")
    report = generate_report()

    # Print the results
    if report:
        print("\nRepositories using Angular 12:")
        print("{:<30} {:<20}".format("Repository", "Eon ID"))
        print("-" * 50)
        for entry in report:
            print(f"{entry['repo_name']:<30} {entry['eon_id']:<20}")
    else:
        print("No repositories found using Angular 12.")

if __name__ == "__main__":
    main()
```

### Key Points:

1. **Fetching Repositories:**
   - The function `get_repositories()` retrieves all repositories from the organization using GitHub API.

2. **Checking Angular Dependency:**
   - The function `check_angular_dependency()` looks into the `package.json` file of the repository (or another file you might use) to check if Angular 12 is listed as a dependency.
   - It decodes the content from base64 (GitHub sends file content encoded) and then parses it to find Angular 12.

3. **Fetching `eon_id`:**
   - The function `fetch_eon_id()` is a placeholder. You’ll need to modify it to suit your needs based on where/how the `eon_id` is stored in your repos.
   
4. **Generating Report:**
   - `generate_report()` compiles the information into a list and prints a report showing the repository name and `eon_id` for those using Angular 12.

### Customizing for Your Environment:
- **Modify `fetch_eon_id()`**: Customize this function to retrieve the `eon_id` based on your repository setup.
- **Modify File Locations**: The script looks for a `package.json` file in each repository, but if Angular dependencies are stored elsewhere, adjust the file path or the file being checked.

### Output:
The script will print out repositories that are using Angular 12 and their associated `eon_id`. You can modify the script to save the report to a CSV or any other format as needed.

Let me know if you need any further adjustments or clarifications!
