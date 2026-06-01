|                                                                                                 |
| ----------------------------------------------------------------------------------------------- |
| def hexdump(filename):                                                                          |
| with open(filename, 'rb') as f:                                                                 |
| content = f.read()                                                                              |
|                                                                                                 |
| hex_str = " ".join(f"{byte:02x}" for byte in content)                                           |
| ascii_str = "".join(chr(byte) if 32 <= byte <= 126 else '.' for byte in content)                |
|                                                                                                 |
| result = []                                                                                     |
| for i in range(0, len(hex_str), 48): # 16 bytes per line, 3 characters (2 hex + space) per byte |
| hex_part = hex_str[i:i+48]                                                                      |
| ascii_part = ascii_str[i//3:i//3 + 16]                                                          |
| result.append(f"{i//3:08x} {hex_part:<48} {ascii_part}")                                        |
|                                                                                                 |
| return "\n".join(result)                                                                        |
|                                                                                                 |
| filename = 'data.bin'                                                                           |
| hexdump_output = hexdump(filename)                                                              |
|                                                                                                 |
| print(hexdump_output)                                                                           |