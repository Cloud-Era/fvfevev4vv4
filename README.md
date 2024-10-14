You're encountering the error because I incorrectly assumed that GitHub's file contents are base64-decoded directly via the `decode('base64')` method. However, GitHub's API response contains a `content` field that is already base64-encoded, and you'll need to decode it properly using Python's `base64` module.

Let me fix that and show you how to decode the content correctly.

### Updated Script:

```python
import requests
import json
import base64

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

        # GitHub returns file content as base64 encoded
        if "content" in package_json:
            content = base64.b64decode(package_json['content']).decode('utf-8')  # Decode the base64 content
            package_data = json.loads(content)

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

### Explanation of Fixes:
1. **Base64 Decoding**:
   - GitHub returns file content in base64-encoded form. Instead of using `.decode('base64')`, we use the `base64` module to properly decode it.
   - `base64.b64decode(package_json['content'])` decodes the base64 content.
   - After decoding, it’s converted into a string using `.decode('utf-8')`.

2. **Package Dependency Check**:
   - The decoded content is loaded into a JSON object to check for Angular dependencies.

### Run and Test:
This updated script should now work without the previous `AttributeError`. It will fetch repositories, decode the `package.json` file's base64 content, and check if Angular version 12 is present in the dependencies.

Let me know if you need further modifications!
