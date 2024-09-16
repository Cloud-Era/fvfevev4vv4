To generate a report listing Docker images with the latest build from your JFrog Artifactory, you can use the Artifactory API to fetch information about Docker repositories and images. Below is a Python script that does this and generates a CSV report containing the Docker image name and the latest build information.

### Python Script to List Docker Images with Latest Build

This script assumes that you are using the JFrog Artifactory REST API, and it uses an identity token for authentication.

```python
import requests
import csv
import os
from urllib.parse import urljoin

# JFrog Artifactory instance
artifactory_instance = "https://frigate.jfrog.io"

# Fetch the Artifactory token from environment variables
artifactory_token = os.getenv("ARTIFACTORY_TOKEN")  # Set this token in your environment

# Output CSV file
output_file = "docker_image_report.csv"

# CSV Headers
csv_headers = ['docker_image_name', 'latest_tag', 'build_date', 'url']

# Function to fetch all repositories of type 'docker'
def fetch_docker_repos():
    url = urljoin(artifactory_instance, "/artifactory/api/repositories")
    headers = {
        "Authorization": f"Bearer {artifactory_token}",
    }
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return [repo['key'] for repo in response.json() if repo['packageType'] == 'docker']

# Function to fetch the latest build tag and date for a Docker image
def fetch_latest_build(repo_name):
    # Using the Docker V2 API to list tags for the repository
    url = urljoin(artifactory_instance, f"/artifactory/api/docker/{repo_name}/v2/_catalog")
    headers = {
        "Authorization": f"Bearer {artifactory_token}",
    }
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    repositories = response.json().get('repositories', [])
    
    latest_builds = []
    
    # Loop through each Docker image in the repository
    for image_name in repositories:
        # Get all tags for the image
        tags_url = urljoin(artifactory_instance, f"/artifactory/api/docker/{repo_name}/v2/{image_name}/tags/list")
        response = requests.get(tags_url, headers=headers)
        response.raise_for_status()
        tags = response.json().get('tags', [])
        
        if tags:
            # Sort tags by date to get the latest build
            latest_tag = sorted(tags)[-1]
            
            # Retrieve tag details (like build date)
            manifest_url = urljoin(artifactory_instance, f"/artifactory/api/docker/{repo_name}/v2/{image_name}/manifests/{latest_tag}")
            response = requests.get(manifest_url, headers=headers)
            response.raise_for_status()
            
            build_date = response.headers.get('Last-Modified', 'N/A')
            
            # Store the latest build details
            latest_builds.append({
                'docker_image_name': image_name,
                'latest_tag': latest_tag,
                'build_date': build_date,
                'url': urljoin(artifactory_instance, f"/artifactory/{repo_name}/{image_name}/{latest_tag}")
            })
    
    return latest_builds

# Main function to generate the report
def generate_report():
    with open(output_file, mode='w', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=csv_headers)
        writer.writeheader()

        # Fetch all Docker repositories
        docker_repos = fetch_docker_repos()

        # Fetch the latest build for each repository and write it to the CSV
        for repo_name in docker_repos:
            latest_builds = fetch_latest_build(repo_name)
            for build in latest_builds:
                writer.writerow(build)

    print(f"Report generated: {output_file}")

# Execute the script
if __name__ == "__main__":
    generate_report()
```

### How the Script Works:
1. **Fetch Docker Repositories**: It uses the `/artifactory/api/repositories` endpoint to fetch a list of repositories and filter those with `packageType='docker'`.
   
2. **Get Docker Image Tags**: For each Docker repository, it calls the `/artifactory/api/docker/{repo_name}/v2/_catalog` to get a list of Docker images. It then retrieves the tags of each image.

3. **Find Latest Build**: The script sorts the tags and selects the latest one. It also fetches metadata like the build date from the Docker manifest headers.

4. **Generate CSV Report**: The data, including Docker image name, latest tag, build date, and URL, are written to a CSV file.

### Running the Script:
1. **Set the Artifactory Token**: Make sure the Artifactory token is set in your environment:
   ```bash
   export ARTIFACTORY_TOKEN="your-identity-token-here"
   ```

2. **Run the Script**: Execute the script:
   ```bash
   python docker_image_report.py
   ```

### CSV Output:
The report will be saved as `docker_image_report.csv` with columns:
- `docker_image_name`: The name of the Docker image.
- `latest_tag`: The latest build tag of the image.
- `build_date`: The date of the latest build.
- `url`: The URL to the latest Docker image in Artifactory.

This script should help you generate the Docker image report with the latest build for each image. Let me know if you need further customizations!
