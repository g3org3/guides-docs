# Proyecto 1 - Asegurar tu servidor ssh, two factor auth

## Previo
Servidor - computadora que queremos proteger
Host - computadora en donde te conectas a tu servidor por ssh


## Paso 1: Crear tu llave privada y publica
para crear tu llave corre este comando
```sh
ssh-keygen -t rsa -C miuser
# con el -t indicamos el tipo ya sea [dsa, ecdsa, ed25519, rsa, rsa1 ]
# con el -C agregamos comentarios a la llave
```
Respuesta
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/george/.ssh/id_rsa): [ENTER]
Created directory '/home/george/.ssh'.
Enter passphrase (empty for no passphrase): [ENTER]
Enter same passphrase again: [ENTER]
Your identification has been saved in /home/root/.ssh/id_rsa.
Your public key has been saved in /home/root/.ssh/id_rsa.pub.
The key fingerprint is:
68:f4:d9:c7:2e:e2:af:b5:79:d6:55:a5:3a:28:1e:72 miuser
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                .|
|      .        ..|
|     . o o .  . .|
|      o S ..o.  .|
|     .. E .oo   .|
|       +.oo .o . |
|       ..o +o .  |
|        oo+o     |
+-----------------+
```
ya con esto tenemos la llave publica y privada

## Paso 2: Guardar la llave en el servidor
Enviar la llave publica al servidor por un medio seguro SCP

```sh
scp ~/.ssh/id_rsa.pub miserver:authorized_keys
# primero indicamos que archivo queremos mandar
# segundo indicamos donde se va a guardar por default el directorio es $HOME/
```

## Paso 3: Crear la carpeta ssh y proteger
```sh
ssh miuser@miservidor.com
miuser@miservidor:~$ cd ~
miuser@miservidor:~$ mkdir .ssh
miuser@miservidor:~$ mv authorized_keys .ssh/authorized_keys

# proteger
miuser@miservidor:~$ chmod a-wr .ssh; chmod u+rwx .ssh;
miuser@miservidor:~$ chmod a-wrx .ssh/authorized_keys; chmod u+rw .ssh/authorized_keys;

# listo para hacer login sin contrasena!!!
# Un paso extra es eliminar la contrasena
miuser@miservidor:~$ sudo passwd --delete miuser
```

## Paso 4: Agregar two factor auth
en el servidor
```sh
sudo apt-get update
```

instalar pam GA
```sh
sudo apt-get install -y libpam-google-authenticator
```

correr google-auth
```sh
google-authenticator

Do you want authentication tokens to be time-based (y/n) y
Do you want me to update your "~/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

Ahora configuramos ssh para que requiera el nuevo metodo de autenticacion
```sh
sudo nano /etc/pam.d/sshd
```
Agregar hasta el final del archivo ... si es que no lo agrego el GA antes
```
. . .
auth required pam_google_authenticator.so nullok
```

```sh
sudo nano /etc/ssh/sshd_config
. . .
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication yes
. . .
sudo service ssh restart
```

Ahora quitar que pida la contrasena y solo 2 factor auth
```sh
sudo nano /etc/ssh/sshd_config
. . .
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
. . .
```
hasta abajo del archivo agregar esto
```
. . .
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive <-
```

Ahora comentamos para que no pregunte la password
```sh
sudo nano /etc/pam.d/sshd
. . .
# Standard Un*x authentication.
#@include common-auth
. . .

sudo service ssh restart
```

DONE~
```sh
ssh miuser@miservidor

Authenticated with partial success.
Verification code:
```
