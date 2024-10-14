Thank you for providing that information. I'll update the script to include the functionality for capturing the EON_ID value using the custom property approach you've described. Here's the updated script that incorporates this change:

```python
import requests
import json
import os
import logging
from github import Github
from github import GithubException
from datetime import datetime
import pytz
import re
import csv

# Set up logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Variable to easily switch between "EoNID" and "eon_id"
ID_PROPERTY_NAME = "eon_id"  # Change this to "EoNID" if needed

def get_dependencies(owner, repo, access_token):
    logging.info(f"Fetching dependencies for repo: {owner}/{repo}")
    url = f"https://api.github.com/repos/{owner}/{repo}/dependency-graph/sbom"
    headers = {
        "Authorization": f"token {access_token}",
        "Accept": "application/vnd.github+json"
    }
    response = requests.get(url, headers=headers)
    logging.info(f"GitHub API response status: {response.status_code}")
    response.raise_for_status()
    logging.info("Successfully fetched dependencies.")
    return response.json()

def get_latest_release_version(repo):
    try:
        latest_release = repo.get_latest_release()
        version = latest_release.tag_name
        return version[1:] if version.startswith('v') else version
    except GithubException:
        logging.warning(f"No releases found for {repo.full_name}")
        return None

def clean_version(version):
    return re.sub(r'^[^0-9]*', '', version)

def get_repo_id(owner, repo, access_token):
    headers = {
        "Authorization": f"token {access_token}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"
    }

    properties_url = f"https://api.github.com/repos/{owner}/{repo}/properties/values"
    response = requests.get(properties_url, headers=headers)
    
    if response.status_code == 200:
        properties_data = response.json()
        for prop in properties_data:
            if prop.get("property_name") == ID_PROPERTY_NAME:
                return prop.get("value")
        logging.info(f"{ID_PROPERTY_NAME} not found for {repo}")
    else:
        logging.error(f"Error fetching properties for {repo}: {response.status_code}")
    
    return None

def check_angular_12(dependencies):
    for package in dependencies['sbom']['packages']:
        if package['name'] == 'npm:@angular/core' and package['versionInfo'].startswith('12.'):
            return True
    return False

def process_organization(org_name, access_token, output_file):
    g = Github(access_token)
    
    try:
        org = g.get_organization(org_name)
        logging.info(f"Successfully accessed organization: {org_name}")
        
        with open(output_file, 'w', newline='') as csvfile:
            csvwriter = csv.writer(csvfile)
            csvwriter.writerow(['Repository Name', ID_PROPERTY_NAME, 'Uses Angular 12'])
        
            for repo in org.get_repos():
                if repo.archived:
                    logging.info(f"Skipping archived repository: {repo.full_name}")
                    continue

                logging.info(f"Processing repository: {repo.full_name}")
                try:
                    eon_id = get_repo_id(org_name, repo.name, access_token)
                    dependencies = get_dependencies(org_name, repo.name, access_token)
                    uses_angular_12 = check_angular_12(dependencies)
                    
                    csvwriter.writerow([repo.name, eon_id, uses_angular_12])
                except requests.exceptions.HTTPError as http_err:
                    logging.error(f"HTTP error occurred for {repo.name}: {http_err}")
                except Exception as err:
                    logging.error(f"An error occurred for {repo.name}: {err}")
    
    except GithubException as e:
        logging.exception(f"Error accessing organization {org_name}")
    except Exception as e:
        logging.exception("Error processing organization")

if __name__ == "__main__":
    org_name = os.environ.get('GITHUB_ORG_NAME')
    access_token = os.environ.get('GITHUB_TOKEN')
    output_file = 'github_report.csv'
    
    process_organization(org_name, access_token, output_file)
```

Key changes in this updated script:

1. Added the `ID_PROPERTY_NAME` variable to easily switch between "EoNID" and "eon_id".

2. Implemented the `get_repo_id` function to fetch the EON_ID value from the repository's custom properties.

3. Updated the `process_organization` function to use `get_repo_id` instead of the previous method of getting the EON_ID.

4. The CSV report now includes the repository name, the EON_ID value (using the specified property name), and whether the repository uses Angular 12.

To use this script:

1. Set the following environment variables:
   - `GITHUB_ORG_NAME`: Your GitHub organization name
   - `GITHUB_TOKEN`: Your GitHub personal access token with necessary permissions

2. Ensure that the `ID_PROPERTY_NAME` is set to the correct value ("eon_id" or "EoNID") at the top of the script.

3. Run the script. It will generate a file named `github_report.csv` in the same directory.

This script will now correctly capture the EON_ID value from the repository's custom properties, along with checking for Angular 12 usage, for all non-archived repositories in the specified organization.

Citations:
[1] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/11902716/6c18de5e-e231-414b-a391-fd96ab507e14/paste.txt
[2] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/11902716/60a4ea87-4981-40b7-814b-d33852662eb0/paste-2.txt
