
|                                                                           |
| ------------------------------------------------------------------------- |
| import xml.etree.ElementTree as ET                                        |
| import os                                                                 |
|                                                                           |
| def main():                                                               |
| file_path = input("Enter the full path to your XML file: ").strip()       |
| if not os.path.isfile(file_path):                                         |
| print(f"❌ File not found: {file_path}")                                   |
| return                                                                    |
|                                                                           |
| try:                                                                      |
| tree = ET.parse(file_path)                                                |
| root = tree.getroot()                                                     |
|                                                                           |
| # Define the namespace prefix and URI from your root tag                  |
| ns = {'uslm': 'http://xml.house.gov/schemas/uslm/1.0'}                    |
|                                                                           |
| print("\n📄 Extracting <section> elements...\n")                          |
| print("\nRoot tag:", root.tag)                                            |
| print("Listing top-level child tags:")                                    |
| for child in root:                                                        |
| print(" -", child.tag)                                                    |
|                                                                           |
| # Now find sections using the namespace prefix 'uslm'                     |
| for section in root.findall('.//uslm:section', ns):                       |
| number = section.get('identifier', '[No identifier]')                     |
| heading = section.findtext('uslm:heading', '[No heading]', namespaces=ns) |
| print(f"Section {number}: {heading}")                                     |
|                                                                           |
| except ET.ParseError as e:                                                |
| print(f" XML Parse Error: {e}")                                           |
| except Exception as e:                                                    |
| print(f" Unexpected Error: {e}")                                          |
|                                                                           |
| if __name__ == "__main__":                                                |
| main()                                                                    |
