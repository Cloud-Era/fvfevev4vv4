import argparse
import concurrent.futures
from tqdm import tqdm

def parse_arguments():
    parser = argparse.ArgumentParser(description="Fetch Dependabot alerts from GitHub Enterprise")
    parser.add_argument("--enterprise", required=True, help="Name of the GitHub Enterprise")
    parser.add_argument("--output", default="vulnerabilities.csv", help="Output CSV file path")
    return parser.parse_args()

def fetch_alerts_with_retry(enterprise, token, page, max_retries=3):
    for attempt in range(max_retries):
        try:
            return get_dependabot_alerts_enterprise(enterprise, token, page)
        except requests.RequestException as e:
            if attempt == max_retries - 1:
                raise
            logging.warning(f"Request failed, retrying (attempt {attempt + 1}/{max_retries})")
            time.sleep(2 ** attempt)  # Exponential backoff

def main():
    args = parse_arguments()
    token = os.getenv("ACCESS_TOKEN")
    if not token:
        raise ValueError("ACCESS_TOKEN environment variable is not set")

    try:
        logging.info(f"Fetching dependabot alerts for enterprise '{args.enterprise}'")
        with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
            futures = [executor.submit(fetch_alerts_with_retry, args.enterprise, token, page) for page in range(1, 6)]
            alerts = []
            for future in tqdm(concurrent.futures.as_completed(futures), total=len(futures), desc="Fetching alerts"):
                alerts.extend(future.result())

        logging.info(f"Fetched {len(alerts)} alerts")

        with open(args.output, 'w', newline='', encoding='utf-8') as csvfile:
            fieldnames = [...]  # Your fieldnames here
            writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
            writer.writeheader()
            
            for alert in tqdm(alerts, desc="Processing alerts"):
                writer.writerow(process_alert(alert))
        
        logging.info(f"CSV file has been saved: {args.output}")
    
    except Exception as e:
        logging.error(f"An error occurred: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
