import asyncio
from playwright.async_api import async_playwright

URL = "https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-virtual-networks/"

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        print("Loading page...")
        await page.goto(URL, wait_until="networkidle")
        html = await page.content()
        print("--- HTML START ---")
        print(html[:5000])
        print("--- HTML END ---")
        await browser.close()

asyncio.run(main())
