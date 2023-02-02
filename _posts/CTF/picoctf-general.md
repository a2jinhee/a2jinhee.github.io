---
layout: default
title: picoCTF General
parent: CTF
nav_order: 1
---

# picoCTF General

---

> ## 1. Python Wrangling

- `python filename 명령어(-d, -e 형태)`
- `open(sys.argv[2], "r")` → sys.argv[0] terminal에 넣는 argument 순서 (0부터) : “r” read .txt file, “rb” read non .txt file

<br>

> ## 2. Wave a flag

- `chmod +x filename` : to make file executable(파일을 실행할 수 있는 권한 부여):
- `./fllename` : 파일 실행시키기

<br>

> ## 3. Lets Warm Up

- 0x70 : hexadecimal number → ignore 0x, we only care about what’s after. 70 to decimal.

<br>

> ## 4. what's a net cat

- `nc sitename portname` : access site via port using netcat
- [nc](https://linux.die.net/man/1/nc)

<br>

> ## 5. strings it

- `man page` = manual page를 뜻함
- 리눅스 명령어) strings
- [strings](https://blog.naver.com/PostView.naver?blogId=bomyzzang&logNo=220217101708&redirect=Dlog&widgetTypeCall=true&directAccess=false)

<br>

> ## 6. Tab, Tab, Attack

- `unzip zipfilename`
- `rm -r directoryname` : remove a non-empty directory

<br>

> ## 7. Magikarp Ground Mission

- cat <.txt file> : read txt file
- `cd ~/` : root/home/a2jinhee
- `cd /` : root
- [Linux Bash Shell Cheatsheet](https://oit.ua.edu/wp-content/uploads/2020/12/Linux_bash_cheat_sheet-1.pdf)

<br>

> ## 8. Bases

- Decode Base64
- Example for doing manually (not recommended)

```python
  # The encoded string

  encoded_string = "SGVsbG8sIFdvcmxkIQ=="

  # A lookup table for base64 characters and their corresponding binary values

  base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
  binary_values = [
  "000000", "000001", "000010", "000011", "000100", "000101", "000110", "000111",
  "001000", "001001", "001010", "001011", "001100", "001101", "001110", "001111",
  "010000", "010001", "010010", "010011", "010100", "010101", "010110", "010111",
  "011000", "011001", "011010", "011011", "011100", "011101", "011110", "011111",
  "100000", "100001", "100010", "100011", "100100", "100101", "100110", "100111",
  "101000", "101001", "101010", "101011", "101100", "101101", "101110", "101111",
  "110000", "110001", "110010", "110011", "110100", "110101", "110110", "110111",
  "111000", "111001", "111010", "111011", "111100", "111101", "111110", "111111"
  ]
  base64_to_binary = dict(zip(base64_chars, binary_values))

  # Divide the encoded string into groups of 4 characters

  encoded_chars = [encoded_string[i:i+4] for i in range(0, len(encoded_string), 4)]

  # Convert each group of 4 characters into its corresponding 24-bit binary representation

  binary_data = ""
  for chars in encoded_chars:
  for char in chars:
  binary_data += base64_to_binary[char]

  # Convert the binary data back into its original format (ASCII text in this case)

  decoded_string = ""
  for i in range(0, len(binary_data), 8):
  byte = binary_data[i:i+8]
  decimal_value = int(byte, 2)
  decoded_string += chr(decimal_value)

  print(decoded_string) # prints "Hello, World!"
```

<br>

> ## 9. First Grep

- `cat <filename> | grep <string>` : can find “string” inside file.

<br>

> ## 10. Codebook

- [How does the Internet work?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/How_does_the_Internet_work)
- [troubleshooting] : those are errors from your command shell. you are running code through the shell, not python. try from a python interpreter ;)  
   → ./code.py (x)  
   → python code.py (o)

<br>

> ## 11. fixme1.py

- `nano filename` : to view the file in the webshell
- `ctrl+X` : to exit
- [trouble shooting] : python indentation
