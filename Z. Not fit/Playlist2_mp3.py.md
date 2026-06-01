|   |
|---|
|import yt_dlp|
|import requests|
|import os|
|import browser_cookie3|
||
|# Function to get the current IP address as seen by Tor|
|def get_tor_ip():|
|session = requests.Session()|
|session.proxies = {|
|'http': 'socks5h://127.0.0.1:9050',|
|'https': 'socks5h://127.0.0.1:9050'|
|}|
|try:|
|response = session.get('http://httpbin.org/ip')|
|return response.json()['origin']|
|except requests.RequestException:|
|return None|
||
|# Function to check if cookies file exists|
|def cookies_file_exists():|
|return os.path.isfile('cookies.txt')|
||
|# Function to load cookies from Tor Browser|
|def load_tor_browser_cookies():|
|try:|
|# Adjust this path to your Tor Browser's profile directory.|
|# For example, on many Linux systems, it might be:|
|# ~/.tor-browser-en/Browser/TorBrowser/Data/Browser/profile.default/cookies.sqlite|
|tor_cookie_path = os.path.expanduser("~/.tor-browser-en/Browser/TorBrowser/Data/Browser/profile.default/cookies.sqlite")|
|cookies = browser_cookie3.firefox(cookie_file=tor_cookie_path)|
|return cookies|
|except Exception as e:|
|print(f"Error loading Tor Browser cookies: {e}")|
|return None|
||
|# Function to configure yt-dlp options|
|def configure_yt_dlp_options():|
|ydl_opts = {|
|'proxy': 'socks5h://127.0.0.1:9050', # Route traffic through Tor|
|'format': 'bestaudio/best', # Best audio quality|
|'postprocessors': [{|
|'key': 'FFmpegExtractAudio', # Extract audio with FFmpeg|
|'preferredcodec': 'mp3', # Convert to MP3|
|'preferredquality': '192',|
|}],|
|'outtmpl': '%(title)s.%(ext)s', # Filename format|
|'quiet': False,|
|'ignoreerrors': True, # Skip problematic videos|
|}|
||
|# Attempt to load cookies from Tor Browser|
|cookies = load_tor_browser_cookies()|
|if cookies:|
|# Write cookies to a temporary file for yt-dlp to use.|
|cookie_file_path = 'cookies.txt'|
|with open(cookie_file_path, 'w') as f:|
|for cookie in cookies:|
|# The Netscape cookie format: domain, flag, path, secure, expiration, name, value|
|f.write(f"{cookie.domain}\t{'TRUE' if cookie.secure else 'FALSE'}\t{cookie.path}\t{'TRUE' if cookie.secure else 'FALSE'}\t{cookie.expires}\t{cookie.name}\t{cookie.value}\n")|
|ydl_opts['cookiefile'] = cookie_file_path|
|elif cookies_file_exists():|
|ydl_opts['cookiefile'] = 'cookies.txt'|
|else:|
|print("No valid cookies found. Proceeding without authentication.")|
||
|return ydl_opts|
||
|def main():|
|ydl_opts = configure_yt_dlp_options()|
||
|while True:|
|playlist_url = input("Enter the YouTube playlist URL to download the audio from (or 'q' to quit): ")|
|if playlist_url.lower() in ['q', 'quit', 'exit']:|
|break|
||
|with yt_dlp.YoutubeDL(ydl_opts) as ydl:|
|try:|
|ydl.download([playlist_url])|
|print("Download completed.")|
|except Exception as e:|
|print("An error occurred:", e)|
||
|print('IP address used for download:', get_tor_ip())|
||
|if __name__ == "__main__":|
|main()|