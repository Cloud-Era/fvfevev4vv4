Yes, you can generate a report of repositories that reference Angular 12 and retrieve the associated `eon_id` from the latest SBOMs. The process involves two main tasks:

1. **Fetching SBOMs for the repositories**: Assuming your repositories have generated SBOMs that are accessible via an API or stored in a directory, you would need to parse the SBOMs to check for dependencies like Angular 12.
2. **Mapping `eon_id`**: You would then map the repositories from these SBOMs with their respective `eon_id`.

Here’s an outline of how you can approach this:

### Steps:
1. **Retrieve the latest SBOMs**: Fetch the SBOM files (CycloneDX format or any other format you're using). This can be done via an API call to where the SBOMs are hosted or by reading them from a local directory.
2. **Search for Angular 12 in SBOM**: Parse the SBOMs to find repositories that list Angular 12 as a dependency.
3. **Get the `eon_id`**: Associate each repository with its corresponding `eon_id` (as you might have in a CSV mapping file).
4. **Generate a report**: Output the repositories that reference Angular 12, along with their `eon_id`, in a CSV file.

### Example Python Script:
This script assumes that you have the SBOMs stored in a local directory and a CSV mapping file for `eon_id`.

```python
import os
import json
import csv

# Path to the SBOM folder and the CSV mapping
SBOM_FOLDER_PATH = "c:\\path\\to\\sboms"
MAPPING_CSV = "c:\\path\\to\\sbom_mapping.csv"
ANGULAR_VERSION = "12"

# Load the mapping of repositories to eon_ids
def load_eon_id_mapping(csv_file):
    repo_to_eon_id = {}
    with open(csv_file, mode='r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            repo_to_eon_id[row['sbom_name']] = row['eon_id']
    return repo_to_eon_id

# Parse SBOM to check if Angular 12 is listed as a dependency
def check_angular_dependency(sbom_file):
    with open(sbom_file, 'r') as file:
        sbom_data = json.load(file)

        # Look for components section in CycloneDX format
        components = sbom_data.get("components", [])
        for component in components:
            if component.get("name") == "angular" and component.get("version", "").startswith(ANGULAR_VERSION):
                return True
    return False

# Generate a report of repositories referencing Angular 12
def generate_report(sbom_folder, mapping_csv):
    eon_id_mapping = load_eon_id_mapping(mapping_csv)

    report = []
    for sbom_file in os.listdir(sbom_folder):
        if sbom_file.endswith(".json"):  # Assuming SBOMs are in JSON format
            repo_name = os.path.splitext(sbom_file)[0]  # Use the filename (without extension) as repo name

            sbom_file_path = os.path.join(sbom_folder, sbom_file)
            if check_angular_dependency(sbom_file_path):
                eon_id = eon_id_mapping.get(repo_name, "N/A")
                report.append({"repository": repo_name, "eon_id": eon_id})

    return report

def save_report_to_csv(report, output_file):
    with open(output_file, 'w', newline='') as csvfile:
        fieldnames = ['repository', 'eon_id']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

        writer.writeheader()
        for row in report:
            writer.writerow(row)

def main():
    sbom_folder = SBOM_FOLDER_PATH
    mapping_csv = MAPPING_CSV
    output_csv = "c:\\sre\\angular12_repos_with_eon_id.csv"  # Define output file path

    print("Generating report for repositories using Angular 12...")
    report = generate_report(sbom_folder, mapping_csv)
    save_report_to_csv(report, output_csv)

    print(f"Report saved to {output_csv}")

if __name__ == "__main__":
    main()
```

### Breakdown of the Script:
1. **Load Mapping**: The `load_eon_id_mapping` function reads the CSV file that contains the mapping between SBOM names and `eon_id`s.
2. **Check Angular Dependency**: The `check_angular_dependency` function checks if a repository (SBOM) has Angular 12 as a dependency by parsing the SBOM (CycloneDX) JSON.
3. **Generate Report**: The `generate_report` function loops through the SBOMs in the specified folder, checks each for Angular 12, and associates the repository with its `eon_id` if Angular 12 is found.
4. **Save to CSV**: The `save_report_to_csv` function writes the results into a CSV file.

### Assumptions:
- SBOMs are stored in the `SBOM_FOLDER_PATH` directory as JSON files.
- The mapping of repositories to `eon_id` is in a CSV file (`MAPPING_CSV`), structured like this:
  ```csv
  sbom_name,eon_id
  repo1,297157
  repo2,297158
  repo3,297159
  ...
  ```

### Output:
The output will be a CSV file (`angular12_repos_with_eon_id.csv`) listing repositories that reference Angular 12, along with their associated `eon_id`.

Make sure to modify the script to fit your exact directory structure, SBOM format, and the way the `eon_id` is associated with repositories.
