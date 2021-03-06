
**UTCTF 2019: Regular ZIPS**
==================================

Challenge description:

In this challenge we only have a zip (![RegularZips.zip](RegularZips.zip)) and a txt file that contains : ^ 7 y RU[A-Z]KKx2 R4\d[a-z]B N$

The first step was to extract the zip file to know what is inside it, so we created a list of passwords that match with the regex in the first txt file.
After extracting the first zip we got another zip with another txt that contains another regex, so we decided to automate the process in order to generate the passwords with the extracted regex, to finally get the flag.
After several attempts we realized that the first zip contained a total of 1000 zips so we developed a script in python for the process.
For this we use the exrex library (https://github.com/asciimoo/exrex).

```python
#!/usr/bin/env python3

import exrex
import zipfile

INITIAL_FILENAME = 'RegularZips.zip'
INITIAL_HINT =  "^  7  y  RU[A-Z]KKx2 R4\d[a-z]B  N$"

def generate_dict(regex):
    return list(exrex.generate(regex))

def get_hint(iteration):
    with open(str(iteration) + "/hint.txt") as f:
        return f.read()

def unzip_and_get_hint(iteration, filename, regex_dict):
    zip_ref = zipfile.ZipFile(filename, 'r')
    for passwd in regex_dict:
        try:
            zip_ref.extractall(str(iteration), pwd=bytes(passwd,'UTF-8'))
        except Exception as e:
            continue
        print("[{}] Password found! - {}".format(iteration, passwd))
        zip_ref.close()
        hint = get_hint(iteration)
        print("\tHint: - {}".format(hint))
        return hint

if __name__ == "__main__":
    dictionary = generate_dict(INITIAL_HINT)
    filename = INITIAL_FILENAME
    i = 0
    while True:
        hint = unzip_and_get_hint(i, filename, dictionary)
        dictionary = generate_dict(hint)
        filename = "{}/archive.zip".format(i)
        i += 1
```        
Finally at the end we obtained the flag : utflag{bean_pure_omission_production_rally}
