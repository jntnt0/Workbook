|   |
|---|
|import requests|
|import subprocess|
|import platform|
|from bs4 import BeautifulSoup|
|import re|
||
|# Function to perform traceroute and return list of IPs|
|def perform_traceroute(target):|
|if platform.system() == "Windows":|
|result = subprocess.run(['tracert', target], capture_output=True, text=True)|
|lines = result.stdout.strip().split('\n')[4:] # Skip first 4 lines of tracert output|
|else:|
|result = subprocess.run(['traceroute', target], capture_output=True, text=True)|
|lines = result.stdout.strip().split('\n')[1:] # Skip the first line (traceroute to ...)|
||
|ips = []|
|ip_regex = re.compile(r"\d+\.\d+\.\d+\.\d+")|
|for line in lines:|
|match = ip_regex.search(line)|
|if match:|
|ips.append(match.group())|
|return ips|
||
|# Function to get IP geolocation information|
|def get_ip_info(ip):|
|response = requests.get(f"https://ipinfo.io/{ip}/json")|
|data = response.json()|
|return data|
||
|# Function to scrape and parse HTML using BeautifulSoup|
|def scrape_website(url):|
|response = requests.get(url)|
|soup = BeautifulSoup(response.content, "html.parser")|
||
|# Example: Extract all links from the webpage|
|links = soup.find_all("a")|
|scraped_links = [link.get("href") for link in links]|
||
|return scraped_links|
||
|def main():|
|target_website = "https://brutalist.report"|
||
|# Perform traceroute|
|print(f"Tracerouting to {target_website}...")|
|hop_ips = perform_traceroute(target_website)|
|print(f"Hops found: {hop_ips}")|
||
|# Get IP geolocation information for each hop|
|print("\nIP Geolocation Information:")|
|for ip in hop_ips:|
|info = get_ip_info(ip)|
|print(f"IP: {ip}")|
|print(f"Location: {info.get('city', 'N/A')}, {info.get('country', 'N/A')}")|
|print(f"ISP: {info.get('org', 'N/A')}")|
|print("-----")|
||
|# Scrape target website|
|print(f"\nScraping {target_website}...")|
|scraped_data = scrape_website(target_website)|
|print("\nLinks scraped from the website:")|
|for link in scraped_data:|
|print(link)|
||
|if __name__ == "__main__":|
|main()|