### 📌 **Bash Command Crash Course**

Here’s a quick guide to essential **Bash** commands for navigating, managing files, and working with processes.

---

## 📂 **File and Directory Management**

|**Command**|**Description**|
|---|---|
|`pwd`|Print current directory path.|
|`ls`|List files in the current directory.|
|`ls -l`|List in **long format** (permissions, size).|
|`ls -a`|Show **hidden** files.|
|`cd directory_name`|Change directory.|
|`cd ..`|Move **up** one directory.|
|`mkdir new_folder`|Create a new directory.|
|`rmdir folder_name`|Remove **empty** directory.|
|`rm file.txt`|Delete a file.|
|`rm -r folder_name`|Delete directory **and contents**.|
|`cp file.txt backup.txt`|Copy a file.|
|`cp -r dir1 dir2`|Copy directories recursively.|
|`mv file.txt folder/`|Move/rename a file.|
|`touch newfile.txt`|Create a new **empty** file.|

---

## 📄 **File Viewing & Editing**

|**Command**|**Description**|
|---|---|
|`cat file.txt`|Display the contents of a file.|
|`less file.txt`|View a file (scrollable, faster for large files).|
|`head file.txt`|Show the **first 10 lines**.|
|`tail file.txt`|Show the **last 10 lines**.|
|`nano file.txt`|Open a file in the **Nano** text editor.|
|`vim file.txt`|Open a file in the **Vim** editor.|
|`echo "Hello" > file.txt`|Write **Hello** to a new file (overwrite).|
|`echo "Hello" >> file.txt`|**Append** "Hello" to an existing file.|

---

## 🔍 **Search and Find**

|**Command**|**Description**|
|---|---|
|`grep "text" file.txt`|Search for "text" inside a file.|
|`grep -i "text" file.txt`|Case-insensitive search.|
|`grep -r "text" folder/`|Recursive search inside folders.|
|`find /path -name "*.txt"`|Find all `.txt` files in a directory.|
|`find . -type f -size +10M`|Find files **larger** than 10 MB.|

---

## ⚙️ **Permissions**

|**Command**|**Description**|
|---|---|
|`chmod +x script.sh`|Make a file **executable**.|
|`chmod 644 file.txt`|Set **read-write** for owner, read-only others.|
|`chown user:group file.txt`|Change file **ownership**.|

---

## 🧮 **Processes and System**

|**Command**|**Description**|
|---|---|
|`ps aux`|List all **running processes**.|
|`top` / `htop`|Interactive process viewer (**htop** is user-friendly).|
|`kill PID`|Terminate a process by **PID**.|
|`killall process_name`|Kill processes by **name**.|
|`df -h`|Show **disk usage** (human-readable).|
|`du -sh folder/`|Check **size** of a directory.|
|`free -m`|Display available **memory**.|

---

## 🌐 **Networking**

|**Command**|**Description**|
|---|---|
|`ping example.com`|Check if a website is reachable.|
|`curl example.com`|Fetch content from a URL.|
|`wget url`|Download a file from the web.|
|`ifconfig` or `ip a`|Show **network** information.|

---

## 🧰 **Archives and Compression**

|**Command**|**Description**|
|---|---|
|`tar -cvf archive.tar folder`|Create a **tar** archive.|
|`tar -xvf archive.tar`|Extract a **tar** archive.|
|`tar -czvf archive.tar.gz folder`|Compress folder to `.tar.gz`.|
|`tar -xzvf archive.tar.gz`|Extract `.tar.gz` archive.|
|`zip -r archive.zip folder`|Create a **ZIP** archive.|
|`unzip archive.zip`|Extract a **ZIP** file.|

---

## ⌨️ **Shortcuts & Tricks**

|**Command**|**Description**|
|---|---|
|`CTRL + C`|**Stop** a running process.|
|`CTRL + Z`|**Pause** a process (background it with `bg`).|
|`CTRL + L`|**Clear** the terminal.|
|`!!`|Run the **last command** again.|
|`history`|Show command **history**.|
|`alias ll='ls -lah'`|Create a **shortcut** for a command.|

---

## 📊 **Advanced Commands**

|**Command**|**Description**|
|---|---|
|`|` (pipe)|
|`> file.txt`|**Redirect** output to a file (overwrite).|
|`>> file.txt`|Append output to a file.|
|`command1 && command2`|Run `command2` **if** `command1` succeeds.|
|`command1||
|`command &`|Run command in the **background**.|

---

🎯 **Need more?** Let me know if you want to dive deeper into specific areas!