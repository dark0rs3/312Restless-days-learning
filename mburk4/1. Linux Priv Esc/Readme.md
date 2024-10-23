*Linux Privilege Escalation*

```Welcome```

Privilege escalation, often abbreviated as privesc, is a sophisticated process that entails the transition from an account with lower permissions to one with elevated permissions, which can be achieved through the exploitation of various vulnerabilities, design flaws, or configuration oversights that exist within an operating system or application.

### Enumeration

The term enumeration refers to the meticulous process of gathering comprehensive information regarding a system or device that you are either in the process of exploiting or have already exploited.

### hostname

The hostname command is utilized to return the name of the target machine, for instance, SQL-PROD-01, as the hostname can potentially contain invaluable information pertaining to the specifics of the machine in question.

### uname -a

The uname -a command will output detailed system information, such as the kernel that is currently being utilized, which is particularly useful in the context of identifying kernel exploits that can facilitate privilege escalation.

### proc/version

The /proc/version file provides essential information regarding the processes running on the target system, which may include critical details about the kernel version as well as additional information such as the installed compiler, for instance, gcc.

### etc/issue

The /etc/issue file contains pertinent information regarding the operating system, although it is important to note that this can be easily altered or customized by users or administrators.

### ps command

The ps command serves as an effective method for examining the currently running processes on a Linux system, allowing for insight into system operations and resource utilization.

ps -a //This variant allows for the viewing of all running processes on the system.

ps axjf //This specific option enables the user to view the hierarchical process tree, illustrating parent-child relationships.

### env

The env command displays the environmental variables present within the current session, such as the path variable, which may indicate the presence of a compiler or scripting language that could potentially be leveraged for privilege escalation purposes.

### sudo -l

The target system may be configured in such a way that it allows users to execute certain commands, or even all commands, with root privileges, and this command is specifically used to enumerate the list of commands that the user is permitted to run using the sudo command.

### ls

The ls command is employed to list the contents of a specified directory, which can be particularly useful when searching for potential vectors that may be exploited for privilege escalation.

use ls -la when looking for potential vectors to use for privilege escalation, as this option provides a detailed view of file attributes and permissions.

### id

The id command offers a general overview of the user's privilege level and group memberships, and it can also be utilized to obtain information regarding another user by executing a command such as id bnb.

### etc/passwd

The /etc/passwd file provides a straightforward method for discovering the users that exist on the system, serving as a critical resource for understanding user accounts.

### history

The history command can provide valuable insights regarding the target system in certain scenarios, allowing for the examination of previously executed commands.

### ifconfig

The target system may be pivoting to another network, and the ifconfig command will provide essential information about the network interfaces that are configured on the system.

### netstat

The netstat command can be utilized with various options to gather extensive information regarding existing connections on the system.

netstat -a //This option shows all listening ports and established connections, providing a comprehensive view of network activity.

netstat -at //This command will list all TCP connections that are currently active on the system.

netstat -au //This command will list all UDP connections that are currently established.

netstat -l //This option specifically lists all ports that are in listening mode, indicating services that are available for connection.

netstat -s //This command lists statistics related to network usage, offering insights into data transmission.

netstat -p //This variant lists all connections along with their respective process ID (PID) information.

netstat -i //This command provides statistics related to the network interfaces, offering insights into their operational status.

netstat -ano //This option allows for the display of connections without resolving hostnames, displaying all sockets, and including timers.

### find

The find command is utilized to search the target system for important information and potential privilege escalation vectors, allowing for a thorough examination of the filesystem.

Redirect errors to /dev/null to have a cleaner output, for example:

find / -size +10G -type f 2>/dev/null

For example, to find files that have the SUID bit set, one would execute:

find / -perm -u=s -type f 2>/dev/null

### Automated Enumeration

A variety of automated enumeration tools can be employed to facilitate the enumeration process, including but not limited to:

Linpeas

LinEnum

Linux Exploit Suggestor

Linux Smart Enumeration

Linux Priv Checker

*Privilege Escalation Techniques*
1. **Kernel Exploits**

The kernel orchestrates communication between various system components, including memory and applications.

Consequently, due to the critical nature of its operations, the kernel is endowed with specific privileges; thus, a successful exploit can result in the acquisition of root privileges.

Steps:

Determine the kernel version or identify the vulnerable kernel.

Conduct a search for exploit code applicable to the kernel of the target system.

Execute the exploit on the designated target system.

2. **Sudo**

The sudo command facilitates the execution of commands with superuser privileges.

Ordinary users typically lack superuser permissions; however, certain tasks may necessitate such access.

In these instances, the user may be permitted to execute the application with superuser privileges while the remainder of the system operates without such permissions.

Applications that can be executed in this manner can be identified using the sudo -l command. The gtfobins resource may be consulted for privilege escalation assistance.

Capitalize on the functionality of applications.

In scenarios where an exploit is not available, but the application possesses exploitable functionalities, these can be leveraged instead.

For instance, if the application reads a configuration file, one can configure it to reference a file to which access is restricted, potentially resulting in the unintended disclosure of sensitive information.

Utilize LD_PRELOAD.

The LD_PRELOAD option is available on certain systems.

This option permits any program to utilize shared libraries.

If the "env_keep" option is activated, it becomes feasible to create a shared library that will be loaded and executed prior to the execution of the program.

It is noteworthy that the LD_PRELOAD option is disregarded if the real user ID differs from the effective user ID.

Steps for privilege escalation include:

Verify the presence of preload (with the env_keep option).

Compile a simple code snippet as a shared object ( so extension) file.

Execute the program with sudo privileges while specifying the LD_PRELOAD option pointing to the so file, which will result in the spawning of a root shell.

3. **SUID (Set User Identification)**

The privilege management framework in Linux is predicated upon regulating the interactions between users and files through permissions.

Files may possess read, write, and execute (rwx) permissions.

Files that have the SUID bit enabled can be executed with the permission level of the file owner, denoted by the presence of an "s" bit, which indicates their special permission status.

Identify files that possess the SUID and SGID bits by utilizing the following command:

find / -type f -perm 04000 -ls 2>/dev/null

Employ gtfobins.com to ascertain whether a binary can be exploited via the SUID bit.

For example, to escalate privileges using Python3 with the SUID bit enabled:

python3 -c 'import pty;pty.spawn("\bin\bash -p")';

For instance, using base64:

LFILE=file_to_read

./base64 "$LFILE" | base64 --decode

etc.

4. **Capabilities**

Capabilities facilitate the management of privileges at a granular level.

To identify tools with enabled capabilities, one can utilize the getcap command, for instance:

getcap / -r 2>/dev/null

From the output listing the binaries, select either of them for privilege escalation or file access. The gtfobins resource may be referenced for additional guidance as necessary.

It is important to note that files in this context do not possess the SUID bit set.

In superuser permissions as root.

5. **Cron Jobs**

Cron jobs serve the purpose of executing scripts or binaries at predetermined intervals.

They operate under the authority of their respective owners rather than the current user.

In instances where a cron job is executed with root privileges, it becomes viable to modify the script that is being executed, thus allowing our script to operate with root privileges.

The crontab file is located at /etc/crontab and can be accessed for reading by any user logged into the system.

To edit a crontab for which one possesses write permissions, but that is executed by root, one could potentially obtain a reverse root shell, allowing for the retrieval of desired information, among other actions.

Although a cronjob may have been deleted, the corresponding entry may still persist in the crontab.

By recreating the deleted file and modifying it, one can acquire a reverse root shell, facilitating access to desired information and further actions.

6. **PATH**

The PATH variable is an environmental variable that informs the operating system regarding the locations to search for executable files.

In situations where a directory with write permissions for your user is included in the PATH, it becomes feasible to compromise an application to execute a script.

For any command that is not inherently included in the shell, the Linux operating system will search through the directories specified in the PATH variable.

To identify writable directories, one can utilize the following command:

find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u

For instance, if a program owned by root (e.g., #include h> void main(){ setuid(0); setgid(0); system("boy"); } ) fails to locate the path or program during execution, it is possible to manipulate the path or insert the missing path from which our script can be executed, as follows:

export PATH=/tmp:$PATH

Ultimately, we can hijack the path by inserting our script into the temporary directory, for example:

echo "/bin/bash" > boy

chmod 777 boy

Consequently, when the script owned by root executes and fails to locate 'boy' within the system, it will investigate the directories in the path and execute the 'boy' file we created in /tmp, thereby granting us superuser privileges as root.

7. **NFS (Network File Systems)**

Shared directories and remote management interfaces, such as SSH and Telnet, can be exploited to gain root access to a system, for example:

locating the root SSH key on the target system and utilizing it for login.

Misconfigurations in network shells may also present vulnerabilities.

The configuration for NFS is maintained within the /etc/exports file.

By default, NFS modifies the root user to 'nfsnobody' and revokes any files from operating with root privileges.

In this file, if the 'no_root_squash' option is present on a writable share, it becomes feasible to create an executable with the SUID bit enabled and execute it on the target system.

The following steps outline the procedure:

First, enumerate the mountable shares from the attacking machine:

showmount -e

One would then mount one of the no_root_squash shares to the attacking machine and construct the executable.

mkdir /tmp/backupsonattackermachine

Subsequently, execute the mount command as follows:

mount -o rw :/backups /tmp/backupsonattacker machine

As we are able to establish SUID bits, a straightforward executable designed to initiate /bin/bash on the designated system will suffice for the intended purpose.

```c int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; } ```

Compile the code and assign the SUID bit accordingly.

root@boy:/tmp/backupsonattackermachine# gcc boy c -o boy

root@boy:/tmp/backupsonattackermachine# chmod +s boy

root@boy:/tmp/backupsonattackermachine# ls -l boy

-rwsr-sr-x 1 root root 16712 Feb 10 20:50 boy

Given that it is a mounted system, the files will also be accessible on the target system, allowing us to execute our binary and subsequently acquire superuser privileges.