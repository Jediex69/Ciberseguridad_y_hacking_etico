
# 🚩 Reto CTF04: Pwned "CryptoR€To24"

Se trata de un reto para repasar fundamentos de criptografía, pasando por codificaciones básicas, cifrados clásicos, modernos y finalmente hashing.
El objetivo era escalar usuario por usuario hasta llegar al final.

---

## 🟢 Fase 0: Acceso Inicial

Iniciamos la conexión SSH con las credenciales proporcionadas para el usuario `ubuntu` .

- **Target:** `10.0.3.12`
- **User/Pass:** `ubuntu` / `ubuntu`

```
└─$ ssh ubuntu@10.0.3.12      
ubuntu@10.0.3.12's password:
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-141-generic x86_64)
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.
To restore this content, you can run the 'unminimize' command.
Last login: Thu Jun 12 19:13:49 2025 from 10.0.3.6
```

Tras el acceso, identificamos un archivo `info.txt` que nos indica la estructura del reto, dividida en cuatro fases: Codificación, Cifrado Simétrico, Cifrado Asimétrico y Funciones Hash .

```
ubuntu@ubuntuserver:~$ ls -la
total 36
drwxr-x---  3 ubuntu ubuntu 4096 Feb 17  2023 .
drwxr-xr-x 30 root   root   4096 May 16  2023 ..
-rw-------  1 ubuntu ubuntu  103 Jun 12 23:37 .bash_history
-rw-r--r--  1 ubuntu ubuntu  220 Jan  6  2022 .bash_logout
-rw-r--r--  1 ubuntu ubuntu 3771 Jan  6  2022 .bashrc
drwx------  2 ubuntu ubuntu 4096 Jan 27  2023 .cache
-rw-r--r--  1 ubuntu ubuntu  807 Jan  6  2022 .profile
-rw-------  1 ubuntu ubuntu 1071 Jan 27  2023 .viminfo
-rw-r--r--  1 root   root    458 Feb 17  2023 info.txt

ubuntu@ubuntuserver:~$ cat info.txt
Bienvenido,
Este reto consta de varias lineas de trabajo:
        - Codificación
        - Cifrado simétrico
        - Cifrado asimétrico
        - Funciones hash
Deberás ir superando los niveles descifrando las contraseñas de los siguientes usuarios.
Cuando alcances el nivel mínimo en cada apartado, se desbloqueará una nueva vía de trabajo para continuar con los siguientes (flag_mid.txt).
Puedes empezar con el primer usuario:

        Usuario: encode1
        Pass: encode1

Suerte
```

---
## 🟡 Fase 1: Encoding

En esta primera fase, abordamos la ofuscación de datos. Confirmamos que la codificación no provee confidencialidad real, ya que los algoritmos son reversibles sin necesidad de una clave secreta.

#### Nivel Encode 1 (Reverse)
Encontramos el archivo `contraseña.txt` con el texto: `D3srever se 2edocne ed añesartnoc Al`. El texto está simplemente invertido y utilizamos el comando `rev` para leerlo:

```
ubuntu@ubuntuserver:/home$ su encode1
Password:

encode1@ubuntuserver:/home$ cd encode1

encode1@ubuntuserver:~$ ll
total 28
drwxr-x---  2 encode1 encode1 4096 Apr  2  2024 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode1 encode1  234 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode1 encode1  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode1 encode1 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encode1 encode1  807 Feb 16  2023 .profile
-rw-r--r--  1 encode1 encode1   38 Feb 16  2023 contraseña.txt

encode1@ubuntuserver:~$ cat contraseña.txt
D3srever se 2edocne ed añesartnoc Al

encode1@ubuntuserver:~$ rev contraseña.txt
La contraseña de encode2 es revers3D
```

**Flag:** `revers3D`

#### Nivel Encode 2 (Base64)
Nos logamos como `encode2`. 

```
encode1@ubuntuserver:~$ su encode2
Password:

encode2@ubuntuserver:/home/encode1$ cd ../encode2

encode2@ubuntuserver:~$ ll
total 28
drwxr-x---  2 encode2 encode2 4096 Apr  2  2024 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode2 encode2  302 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode2 encode2  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode2 encode2 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encode2 encode2  807 Feb 16  2023 .profile
-rw-r--r--  1 encode2 encode2   49 Feb 16  2023 contraseña.txt

encode2@ubuntuserver:~$ cat contraseña.txt
TGEgY29udHJhc2XDsWEgZGUgZW5jb2RlMyBlcyBCNHNlNjQK
```

Detectamos una cadena con el conjunto de caracteres típico de la codificación Base64 (`A-Z, a-z, 0-9, +, /`). Transferimos el archivo via SSH a la máquina Kali y lo decodificamos:

```
┌──(kali㉿kali)-[~]
└─$ scp encode2@10.0.3.12:/home/encode2/contraseña.txt /home/kali/

encode2@10.0.3.12's password:
contraseña.txt                                                                                                                                                                  100%   49    33.6KB/s   00:00   

┌──(kali㉿kali)-[~]
└─$ base64 -d contraseña.txt

La contraseña de encode3 es B4se64
```

**Flag:** `B4se64`

#### Nivel Encode 3 (Base64 Invertido)
Encontramos una cadena que comienza con el carácter de relleno `=`, lo cual es anómalo en Base64 estándar. Deducimos que la cadena esta invertida y procedemos a revertirla y posteriormente decodificarla.

```
┌──(kali㉿kali)-[~]
└─$ scp encode3@10.0.3.12:/home/encode3/contraseña.txt /home/kali

encode3@10.0.3.12's password:
contraseña.txt      100%   61    36.0KB/s   00:00 

┌──(kali㉿kali)-[~]
└─$ rev contraseña.txt > contraseña2.txt

┌──(kali㉿kali)-[~]
└─$ base64 -d contraseña2.txt

La contraseña de encode4 es D3sreveR46es4B
```

**Flag:** `D3sreveR46es4B`

#### Nivel Encode 4 (Hexadecimal)
Observamos una secuencia de bytes representados en hexadecimal.

```
encode3@ubuntuserver:~$ su encode4
Password:

encode4@ubuntuserver:/home/encode3$ cd ../encode4

encode4@ubuntuserver:~$ ll
total 36
drwxr-x---  2 encode4 encode4 4096 Jun 12 16:34 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode4 encode4  393 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode4 encode4  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode4 encode4 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encode4 encode4  807 Feb 16  2023 .profile
-rw-------  1 encode4 encode4 1667 Jun 12 16:34 .python_history
-rw-r--r--  1 encode4 encode4  111 Feb 16  2023 contraseña.txt
-rw-rw-r--  1 encode4 encode4   37 Jun 12 16:30 resultado.txt

encode4@ubuntuserver:~$ cat contraseña.txt
4C 61 20 63 6F 6E 74 72 61 73 65 F1 61 20 64 65 20 65 6E 63 6F 64 65 35 20 65 73 20 48 33 78 54 6F 54 33 78 74
```

 Mediante un script en Python, convertimos estos valores a su representación ASCII, revelando la credencial.

```
>>> hex_string = "4C 61 20 63 6F 6E 74 72 61 73 65 F1 61 20 64 65 20 65 6E 63 6F 64 65 35 20 65 73 20 48 33 78 54 6F 54 33 78 74"
... bytes_object = bytes.fromhex(hex_string.replace(" ", ""))
... decoded = bytes_object.decode('latin-1')
... print(decoded)
...
La contraseña de encode5 es H3xToT3xt
```

**Flag:** `H3xToT3xt`

#### Nivel Encode 5 (Decimal / ASCII)
```
encode4@ubuntuserver:~$ su encode5
Password:

encode5@ubuntuserver:/home/encode4$ cd ../encode5

encode5@ubuntuserver:~$ ll
total 36
drwxr-x---  3 encode5 encode5 4096 Jun 12 16:38 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode5 encode5  307 Jul 19 22:48 .bash_history
-rw-r--r--  1 encode5 encode5  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode5 encode5 3771 Feb 16  2023 .bashrc
drwx------  2 encode5 encode5 4096 Apr  2  2024 .cache/
-rw-r--r--  1 encode5 encode5  807 Feb 16  2023 .profile
-rw-------  1 encode5 encode5  656 Jun 12 16:38 .python_history
-rw-r--r--  1 encode5 encode5  140 Feb 16  2023 contraseña.txt

encode5@ubuntuserver:~$ cat contraseña.txt
76 97 32 99 111 110 116 114 97 115 101 195 177 97 32 100 101 32 101 110 99 111 100 101 54 32 101 115 32 65 83 68 51 99 105 109 97 108 67 73
```

Nos encontramos con una lista de enteros decimales. Cada número corresponde a un carácter en la tabla ASCII. Usamos Python de nuevo para convertir el array de números a bytes y luego a string.

```
>>> decimal_values = [
...     76, 97, 32, 99, 111, 110, 116, 114, 97, 115, 101, 195, 177, 97, 32,
...     100, 101, 32, 101, 110, 99, 111, 100, 101, 54, 32, 101, 115, 32,
...     65, 83, 68, 51, 99, 105, 109, 97, 108, 67, 73
... ]
... # Convert to bytes
... byte_array = bytes(decimal_values)
... # Decode
... text = byte_array.decode("utf-8")
... print(text)
... exit()
... decimal_values = [
...     76, 97, 32, 99, 111, 110, 116, 114, 97, 115, 101, 195, 177, 97, 32,
...     100, 101, 32, 101, 110, 99, 111, 100, 101, 54, 32, 101, 115, 32,
...     65, 83, 68, 51, 99, 105, 109, 97, 108, 67, 73
... ]
... # Convert to bytes
... byte_array = bytes(decimal_values)
... # Decode
... text = byte_array.decode("utf-8")
... print(text)
...
La contraseña de encode6 es ASD3cimalCI
```

**Flag:** `ASD3cimalCI`

#### Nivel Encode 6 (URL Encode)
```
encode5@ubuntuserver:~$ su encode6
Password:

encode6@ubuntuserver:/home/encode5$ cd ../encode6

encode6@ubuntuserver:~$ ll
total 32
drwxr-x---  2 encode6 encode6 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode6 encode6  118 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode6 encode6  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode6 encode6 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encode6 encode6  807 Feb 16  2023 .profile
-rw-------  1 encode6 encode6  170 Jun 11 17:32 .python_history
-rw-r--r--  1 encode6 encode6   70 Feb 16  2023 contraseña.txt

encode6@ubuntuserver:~$ cat contraseña.txt
La%20contrase%C3%B1a%20de%20encode7%20es%20URL%3C%27%23%27%3Eencoding
```

La contraseña presenta caracteres de escape porcentual (`%20`, `%C3%B1`), que son espacios en codificación URL. Utilizamos la librería `urllib.parse` para normalizar la cadena:

```
└─$ python3
Python 3.13.3 (main, Apr 10 2025, 21:38:51) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import urllib.parse
... texto = "La%20contrase%C3%B1a%20de%20encode7%20es%20URL%3C%27%23%27%3EEncoding"
... decodificado = urllib.parse.unquote(texto)
... print(decodificado)
...
La contraseña de encode7 es URL<'#'>Encoding
```

**Flag:** `URL<'#'>Encoding`

#### Nivel Encode 7 (Binario)
```
encode6@ubuntuserver:~$ su encode7
Password:

encode7@ubuntuserver:/home/encode6$ cd ../encode7

encode7@ubuntuserver:~$ ll
total 32
drwxr-x---  2 encode7 encode7 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode7 encode7  129 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode7 encode7  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode7 encode7 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encode7 encode7  807 Feb 16  2023 .profile
-rw-------  1 encode7 encode7  590 Jun 11 17:34 .python_history
-rw-r--r--  1 encode7 encode7  360 Feb 16  2023 contraseña.txt

encode7@ubuntuserver:~$ cat contraseña.txt
01001100 01100001 00100000 01100011 01101111 01101110 01110100 01110010 01100001 01110011 01100101 11000011 10110001 01100001 00100000 01100100 01100101 00100000 01100101 01101110 01100011 01101111 01100100 01100101 00111000 00100000 01100101 01110011 00100000 01010100 01100101 01111000 01110100 00110010 01000010 01101001 01101110 01100001 01110010 01111001
```

La contraseña está codificada en binario y agrupada en octetos. Para decodificarla usamos el siguiente script de Python:

```
>>> binary_string = """
... 01001100 01100001 00100000 01100011 01101111 01101110 01110100 01110010 01100001 01110011 01100101 11000011 101\
10001
... 01100001 00100000 01100100 01100101 00100000 01100101 01101110 01100011 01101111 01100100 01100101 00111000
... 00100000 01100101 01110011 00100000 01010100 01100101 01111000 01110100 00110010 01000010 01101001 01101110 011\
00001 01110010 01111001
... """.replace("\n", "").replace(" ", "")
... # Convertimos los bits en bytes
... chars = [binary_string[i:i+8] for i in range(0, len(binary_string), 8)]
... text = ''.join([chr(int(b, 2)) for b in chars])
... print(text)
...
La contraseÃ±a de encode8 es Text2Binary
```

**Flag:** `Text2Binary`

#### Nivel Encode 8 (Checkpoint)
Al finalizar esta fase en el usuario `encode8`, localizamos la bandera `flag_mid.txt` que nos autorizaba a proceder a la fase de cifrado .

```
encode7@ubuntuserver:~$ su encode8
Password:

encode8@ubuntuserver:/home/encode7$ cd ../encode8

encode8@ubuntuserver:~$ ll
total 32
drwxr-x---  3 encode8 encode8 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root    root    4096 May 16  2023 ../
-rw-------  1 encode8 encode8  141 Jun 12 19:13 .bash_history
-rw-r--r--  1 encode8 encode8  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encode8 encode8 3771 Feb 16  2023 .bashrc
drwxrwxr-x  3 encode8 encode8 4096 Apr  3  2024 .local/
-rw-r--r--  1 encode8 encode8  807 Feb 16  2023 .profile
-rw-r--r--  1 root    root     260 Apr  3  2024 flag_mid.txt

encode8@ubuntuserver:~$ cat flag_mid.txt
Enhorabuena.

Has alcanzado el nivel que te permite avanzar al siguiente bloque, abriendo una nueva línea en el reto.

Para comenzar con los ejercicios de cifrado, haremos uso del siguiente usuario:
        - Usuario: encrypt1
        - Contraseña: encrypt1
```

---
## 🟠 Fase 2: Cifrado Simétrico (Clásico y Moderno)

En esta etapa, nos enfrentamos a mecanismos que requieren una clave secreta o un algoritmo de sustitución para recuperar la información.

### Nivel Encrypt 1 (ROT13)
```
encode8@ubuntuserver:~$ su encrypt1
Password:

encrypt1@ubuntuserver:/home/encode8$ cd ../encrypt1

encrypt1@ubuntuserver:~$ ll
total 32
drwxr-x---  2 encrypt1 encrypt1 4096 Jun 12 16:53 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt1 encrypt1  167 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt1 encrypt1  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt1 encrypt1 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt1 encrypt1  807 Feb 16  2023 .profile
-rw-------  1 encrypt1 encrypt1 1149 Jun 12 16:53 .python_history
-rw-r--r--  1 encrypt1 encrypt1   41 Feb 16  2023 contraseña.txt

encrypt1@ubuntuserver:~$ cat contraseña.txt
Cr tfekirjvñr uv vetipgk2 vj TrvjriBe0n
```

Analizamos el criptograma y detectamos un patrón de desplazamiento simple. La contraseña tiene un cifrado de sustitución **ROT-13** (variante del cifrado César), y para decodificarla usamos el siguiente script de Python:

```
>>> def rot(text, shift):
...     result = ''
...     for c in text:
...         if c.isalpha():
...             base = ord('A') if c.isupper() else ord('a')
...             result += chr((ord(c) - base + shift) % 26 + base)
...         else:
...             result += c
...     return result
... mensaje = "Cr tfekirjvñr uv vetipgk2 vj TrvjriBe0n"
... for i in range(1, 26):
...     print(f"ROT {i:2}: {rot(mensaje, i)}")
... exit()
... def rot(text, shift):
...     result = ''
...     for c in text:
...         if c.isalpha():
...             base = ord('A') if c.isupper() else ord('a')
...             result += chr((ord(c) - base + shift) % 26 + base)
...         else:
...             result += c
...     return result
... mensaje = "Cr tfekirjvñr uv vetipgk2 vj TrvjriBe0n"
... for i in range(1, 26):
...     print(f"ROT {i:2}: {rot(mensaje, i)}")
... exit()
... def rot(text, shift):
...     result = ''
...     for c in text:
...         if c.isalpha():
...             base = ord('A') if c.isupper() else ord('a')
...             result += chr((ord(c) - base + shift) % 26 + base)
...         else:
...             result += c
...     return result
... mensaje = "Cr tfekirjvñr uv vetipgk2 vj TrvjriBe0n"
... for i in range(1, 26):
...     print(f"ROT {i:2}: {rot(mensaje, i)}")
...
ROT  1: Ds ugfljskwps vw wfujqhl2 wk UswksjCf0o
ROT  2: Et vhgmktlxqt wx xgvkrim2 xl VtxltkDg0p
ROT  3: Fu wihnlumyru xy yhwlsjn2 ym WuymulEh0q
ROT  4: Gv xjiomvnzsv yz zixmtko2 zn XvznvmFi0r
ROT  5: Hw ykjpnwoatw za ajynulp2 ao YwaownGj0s
ROT  6: Ix zlkqoxpbux ab bkzovmq2 bp ZxbpxoHk0t
ROT  7: Jy amlrpyqcvy bc clapwnr2 cq AycqypIl0u
ROT  8: Kz bnmsqzrdwz cd dmbqxos2 dr BzdrzqJm0v
ROT  9: La contrasexa de encrypt2 es CaesarKn0w
ROT 10: Mb dpousbtfyb ef fodszqu2 ft DbftbsLo0x
ROT 11: Nc eqpvtcugzc fg gpetarv2 gu EcguctMp0y
ROT 12: Od frqwudvhad gh hqfubsw2 hv FdhvduNq0z
ROT 13: Pe gsrxvewibe hi irgvctx2 iw GeiwevOr0a
ROT 14: Qf htsywfxjcf ij jshwduy2 jx HfjxfwPs0b
ROT 15: Rg iutzxgykdg jk ktixevz2 ky IgkygxQt0c
ROT 16: Sh jvuayhzleh kl lujyfwa2 lz JhlzhyRu0d
ROT 17: Ti kwvbziamfi lm mvkzgxb2 ma KimaizSv0e
ROT 18: Uj lxwcajbngj mn nwlahyc2 nb LjnbjaTw0f
ROT 19: Vk myxdbkcohk no oxmbizd2 oc MkockbUx0g
ROT 20: Wl nzyecldpil op pyncjae2 pd NlpdlcVy0h
ROT 21: Xm oazfdmeqjm pq qzodkbf2 qe OmqemdWz0i
ROT 22: Yn pbagenfrkn qr rapelcg2 rf PnrfneXa0j
ROT 23: Zo qcbhfogslo rs sbqfmdh2 sg QosgofYb0k
ROT 24: Ap rdcigphtmp st tcrgnei2 th RpthpgZc0l
ROT 25: Bq sedjhqiunq tu udshofj2 ui SquiqhAd0m
```

Observando el resultado, vemos que en la línea ROT 9 se revela el texto.

**Flag:** `CaesarKn0w`

#### Nivel Encrypt 2 (Sustitución Monoalfabética)
Contámos con un archivo cifrado y un ejemplo de texto claro con su contraparte cifrada.

```
encrypt1@ubuntuserver:~$ su encrypt2
Password:

encrypt2@ubuntuserver:/home/encrypt1$ cd ../encrypt2

encrypt2@ubuntuserver:~$ ll
total 40
drwxr-x---  2 encrypt2 encrypt2 4096 Jun 12 17:06 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt2 encrypt2  477 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt2 encrypt2  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt2 encrypt2 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt2 encrypt2  807 Feb 16  2023 .profile
-rw-------  1 encrypt2 encrypt2 2896 Jun 12 17:06 .python_history
-rw-r--r--  1 encrypt2 encrypt2   50 Feb 16  2023 contraseña.txt.enc
-rw-r--r--  1 encrypt2 encrypt2  103 Feb 16  2023 ejemplo.txt
-rw-r--r--  1 encrypt2 encrypt2  103 Feb 16  2023 ejemplo.txt.enc

encrypt2@ubuntuserver:~$ cat contraseña.txt.enc
Iu wlkpeuoyku xyi qoquefl ykwemcp3 yo OqopfpqmyJY

encrypt2@ubuntuserver:~$ cat ejemplo.txt
Esto es un texto de prueba para confirmar que funciona bien el cifrado de sustitucion y que es robusto

encrypt2@ubuntuserver:~$ cat ejemplo.txt.enc
Yopl yo qk pytpl xy ceqyvu cueu wlkzfejue dqy zqkwflku vfyk yi wfzeuxl xy oqopfpqwflk m dqy yo elvqopl
```

Esto nos permitió realizar un **ataque de texto claro conocido**. Comparando el texto original con el cifrado, construimos un diccionario de sustitución (A=X, B=Y, etc.) usando Python y lo aplicamos al archivo de la contraseña para descifrarla.

```
└─$ python3
Python 3.13.3 (main, Apr 10 2025, 21:38:51) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> def construir_diccionario(claro, cifrado):
...     """Construye un diccionario de sustitución letra a letra."""
...     mapa = {}
...     for c, e in zip(claro, cifrado):
...         if c.isalpha() and e.isalpha():
...             mapa[e.lower()] = c.lower()
...     return mapa
...
... def descifrar(texto_cifrado, mapa):
...     """Usa el diccionario de sustitución para descifrar un texto."""
...     resultado = ""
...     for c in texto_cifrado:
...         if c.lower() in mapa:
...             letra_descifrada = mapa[c.lower()]
...             # Respeta mayúsculas
...             resultado += letra_descifrada.upper() if c.isupper() else letra_descifrada
...         else:
...             resultado += c
...     return resultado
...
... # Textos de ejemplo
... texto_claro = "Esto es un texto de prueba para confirmar que funciona bien el cifrado de sustitucion y que es r\
obusto"
... texto_cifrado = "Yopl yo qk pytpl xy ceqyvu cueu wlkzfejue dqy zqkwflku vfyk yi wfzeuxl xy oqopfpqwflk m dqy yo\
 elvqopl"
...
... # Construir mapa de sustitución
... diccionario = construir_diccionario(texto_claro, texto_cifrado)
...
... # Texto cifrado real
... texto_cifrado_real = "Iu wlkpeuoyku xyi qoquefl ykwemcp3 yo OqopfpqmyJY"
...
... # Descifrar
... descifrado = descifrar(texto_cifrado_real, diccionario)
...
... print("Texto descifrado:")
... print(descifrado)
...
Texto descifrado:
La contrasena del usuario encrypt3 es SustituyeME
```

**Flag:** `SustituyeME`

#### Nivel Encrypt 3 (Transposición)
```
ubuntu@ubuntuserver:~$ su encrypt3
Password:

encrypt3@ubuntuserver:/home/ubuntu$ cd ../encrypt3

encrypt3@ubuntuserver:~$ ll
total 36
drwxr-x---  2 encrypt3 encrypt3 4096 Jun 12 17:08 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt3 encrypt3  201 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt3 encrypt3  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt3 encrypt3 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt3 encrypt3  807 Feb 16  2023 .profile
-rw-------  1 encrypt3 encrypt3 2676 Jun 12 17:08 .python_history
-rw-r--r--  1 encrypt3 root       33 Feb 16  2023 contraseña.txt.enc
-rw-r--r--  1 encrypt3 root      138 Feb 16  2023 info.txt

encrypt3@ubuntuserver:~$ cat info.txt
Parece que el fichero ha sido cifrado mediante un algoritmo de Transposición.

¿Podrías verificar si la clave de cifrado es TRANSPOSE?

encrypt3@ubuntuserver:~$ cat contraseña.txt.enc
pr tesy ay Sd4astEa co spoeesLnN
```

Basándonos en las indicaciones sobre un "algoritmo de Transposición" y la clave sugerida "TRANSPOSE", implementamos un script para realizar una **Transposición Columnar** y reordenar las columnas del texto cifrado.

```
┌──(kali㉿kali)-[~]
└─$ python3                          
Python 3.13.3 (main, Apr 10 2025, 21:38:51) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import math
... def descifrar_transposicion_columnar(ciphertext, clave):
...     # Eliminar espacios si los hubiera
...     ciphertext = ciphertext.replace(" ", "")
...     n_col = len(clave)
...     n_row = math.ceil(len(ciphertext) / n_col)
...     # Orden de las columnas basado en la clave
...     orden_columnas = sorted(list(enumerate(clave)), key=lambda x: x[1])
...     orden_indices = [idx for idx, _ in orden_columnas]
...     # Determinar número de letras por columna (algunas pueden tener una menos)
...     num_full_cols = len(ciphertext) % n_col
...     if num_full_cols == 0:
...         num_full_cols = n_col
...     # Crear una lista para almacenar las columnas
...     columnas = [''] * n_col
...     i = 0
...     for idx in orden_indices:
...         largo_col = n_row if orden_indices.index(idx) < num_full_cols else n_row - 1
...         columnas[idx] = ciphertext[i:i+largo_col]
...         i += largo_col
...     # Reconstruir el texto leyendo por filas
...     texto_plano = ''
...     for fila in range(n_row):
...         for col in range(n_col):
...             if fila < len(columnas[col]):
...                 texto_plano += columnas[col][fila]
...     return texto_plano
... # Texto cifrado y clave
... texto_cifrado = "pr tesy ay Sd4astEa co spoeesLnN"
... clave = "TRANSPOSE"
... # Descifrar
... texto_descifrado = descifrar_transposicion_columnar(texto_cifrado, clave)
... print("Texto descifrado:")
... print(texto_descifrado)
...
Texto descifrado:
Lapassdeencrypt4esNotSoEasy
```

**Flag:** `NotSoEasy`

#### Nivel Encrypt 4 (Vigenère)
```
encrypt3@ubuntuserver:~$ su encrypt4
Password:

encrypt4@ubuntuserver:/home/encrypt3$ cd ../encrypt4

encrypt4@ubuntuserver:~$ ll
total 36
drwxr-x---  2 encrypt4 encrypt4 4096 Jun 12 17:24 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt4 encrypt4  280 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt4 encrypt4  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt4 encrypt4 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt4 encrypt4  807 Feb 16  2023 .profile
-rw-------  1 encrypt4 encrypt4 1468 Jun 12 17:24 .python_history
-rw-r--r--  1 encrypt4 encrypt4   41 Feb 16  2023 contraseña.txt.enc
-rw-r--r--  1 encrypt4 encrypt4  148 Feb 16  2023 info.txt

encrypt4@ubuntuserver:~$ cat info.txt
Parece que el archivo ha sido cifrado.

¿Podrías verificar si la contraseña es alguna de las siguientes?

- Caesar
- AlKindi
- Vigenere
- Enigma

encrypt4@ubuntuserver:~$ cat contraseña.txt.enc
Gi vefw ui zvivltk5 in MtgecgxDBBmtieimm
```

Identificamos un cifrado polialfabético. Probamos una por una las claves sugeridas en `info.txt` ("Caesar", "AlKindi", "Vigenere", "Enigma") contra el texto cifrado con un script de Python. También podemos usar la web Cyberchef.

```
def limpiar_texto(texto):
    return ''.join([c.upper() for c in texto if c.isalpha()])
def descifrar_vigenere(texto_cifrado, clave):
    texto_cifrado = limpiar_texto(texto_cifrado)
    texto_plano = ''
    clave = clave.upper()
    for i, c in enumerate(texto_cifrado):
        c_val = ord(c) - ord('A')
        k_val = ord(clave[i % len(clave)]) - ord('A')
        p_val = (c_val - k_val) % 26
        texto_plano += chr(p_val + ord('A'))
    return texto_plano
texto_cifrado = "Gi vefw ui zvivltk5 in MtgecgxDBBmtieimm"
claves = ["Caesar", "AlKindi", "Vigenere", "Enigma"]
for clave in claves:
    descifrado = descifrar_vigenere(texto_cifrado, clave)
    print(f"Clave: {clave}")
    print(descifrado)
    print("-" * 40)

Clave: Caesar
EIRMFFSIVDIEJTGQNVRGAKGGBBXUTRCIIU
----------------------------------------
Clave: AlKindi
GXLWSTMIOLAIILKXDEGDWCVNVOYETXUAZJ
----------------------------------------
Clave: Vigenere
LAPASSDEENCRYPTESENCRYPTITVIGENERE
----------------------------------------
Clave: Enigma
CVNYTWQVRPWVHGCCBMPTWWUXZOTGHIAVEG
----------------------------------------
```

**Flag:** `EncryptITVigenere`

#### Nivel Encrypt 5 (3DES - Legacy)
```
encrypt4@ubuntuserver:~$ su encrypt5
Password:

encrypt5@ubuntuserver:/home/encrypt4$ cd ../encrypt5

encrypt5@ubuntuserver:~$ ll
total 40
drwxr-x---  2 encrypt5 encrypt5 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt5 encrypt5  543 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt5 encrypt5  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt5 encrypt5 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt5 encrypt5  807 Feb 16  2023 .profile
-rw-------  1 encrypt5 encrypt5  786 Jun 11 19:06 .viminfo
-rw-rw-r--  1 encrypt5 encrypt5   46 Jun 12 17:31 contraseña.txt
-rw-r--r--  1 encrypt5 encrypt5   64 Feb 16  2023 contraseña.txt.des3
-rw-r--r--  1 encrypt5 encrypt5  161 Feb 16  2023 info.txt

encrypt5@ubuntuserver:~$ cat info.txt
Parece que el fichero ha sido cifrado.

¿Podrías verificar si la contraseña es alguna de las siguientes?

- RC4Encryption
- DES3Rules
- Symmetric
- NotSoEasy
```

El archivo tiene extensión `.des3`. El archivo `info.txt` sugiere probar contraseñas, por lo que usamos `openssl` para descifrar usando **Triple DES** (3DES). La contraseña correcta fue "Symmetric".

```
encrypt5@ubuntuserver:~$ openssl des3 -d -in contraseña.txt.des3 -out contraseña.txt -pbkdf2
enter DES-EDE3-CBC decryption password: Symmetric

encrypt5@ubuntuserver:~$ cat contraseña.txt
La contraseña de encrypt6 es 3DESEncription!
```

**Flag:** `3DESEncription!`

#### Nivel Encrypt 6 (AES-256)
```
encrypt5@ubuntuserver:~$ su encrypt6
Password:

encrypt6@ubuntuserver:/home/encrypt5$ cd ../encrypt6

encrypt6@ubuntuserver:~$ ll
total 36
drwxr-x---  2 encrypt6 encrypt6 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt6 encrypt6  622 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt6 encrypt6  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt6 encrypt6 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt6 encrypt6  807 Feb 16  2023 .profile
-rw-r--r--  1 encrypt6 encrypt6   64 Feb 16  2023 contraseña.txt.aes2
-rw-r--r--  1 encrypt6 encrypt6  190 Feb 16  2023 info.txt

encrypt6@ubuntuserver:~$ cat info.txt
Parece que el fichero ha sido cifrado.

¿Podrías verificar si la contraseña es alguna de las siguientes?

- RC4Encryption
- CBCRules
- NotSoEasy
- AES256Symmetric
- Symmetric
- ZenAES256
```

Similar al caso anterior, pero con el estándar **AES-256**. Probamos las claves sugeridas en `info.txt`.

```
encrypt6@ubuntuserver:~$ openssl aes-256-cbc -d -in contraseña.txt.aes2 -out contraseña.txt -pbkdf2
enter AES-256-CBC decryption password:

encrypt6@ubuntuserver:~$ cat contraseña.txt
La contraseña de encrypt7 es AESEncrypt256
```

**Flag:** `AESEncrypt256`

#### Nivel Encrypt 7
Llegamos a `encrypt7` y encontramos otra `flag_mid.txt` con las credenciales para comenzar la fase de cifrado asimétrico: `pkencrypt1` / `pkencrypt1`.

```
encrypt6@ubuntuserver:~$ su encrypt7
Password:

encrypt7@ubuntuserver:/home/encrypt6$ cd ../encrypt7

encrypt7@ubuntuserver:~$ ll
total 32
drwxr-x---  2 encrypt7 encrypt7 4096 Jun 12 18:29 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 encrypt7 encrypt7  278 Jun 12 19:13 .bash_history
-rw-r--r--  1 encrypt7 encrypt7  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 encrypt7 encrypt7 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 encrypt7 encrypt7  807 Feb 16  2023 .profile
-rw-------  1 encrypt7 encrypt7 1240 Jun 12 18:29 .python_history
-rw-r--r--  1 encrypt7 encrypt7  444 Apr  3  2024 flag_mid.txt
encrypt7@ubuntuserver:~$ cat flag_mid.txt
  GNU nano 7.2                                                                   New Buffer
Enhorabuena.

Has alcanzado el nivel que te permite avanzar al siguiente bloque, abriendo una nueva línea en el reto.

Para comenzar con los ejercicios de cifrado asimétrico, haremos uso del siguiente usuario:

        - Usuario: pkencrypt1
        - Contraseña: pkencrypt1
```

---
## 🔵 Fase 3: Cifrado Asimétrico (RSA)

![Imagen de Public Key Cryptography diagram](https://encrypted-tbn0.gstatic.com/licensed-image?q=tbn:ANd9GcT34zpIaR_epOS7isTABxJ_jw-_CfQN3fo63Hhmu0wGPkjqRJUTL-ydAmWH1KEO68ZNQL0AimBZfIhsWFHUwsotmzm3PO6LoKQpF6ghNbLzNFXZdTw)

Shutterstock

#### Nivel Pkencrypt 1 (Clave Privada Expuesta)
Accedemos a ``pkencrypt1`` y encontré un directorio `keys` con los archivos `privada.pem` y `publica.pem`.

```
encrypt7@ubuntuserver:~$ su pkencrypt1
Password:

pkencrypt1@ubuntuserver:/home/encrypt7$ cd ../pkencrypt1

pkencrypt1@ubuntuserver:~$ ll
total 40
drwxr-x---  3 pkencrypt1 pkencrypt1 4096 Jun 12 18:36 ./
drwxr-xr-x 30 root       root       4096 May 16  2023 ../
-rw-------  1 pkencrypt1 pkencrypt1  590 Jun 12 19:13 .bash_history
-rw-r--r--  1 pkencrypt1 pkencrypt1  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 pkencrypt1 pkencrypt1 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 pkencrypt1 pkencrypt1  807 Feb 16  2023 .profile
-rw-r--r--  1 pkencrypt1 pkencrypt1  256 Feb 16  2023 contraseña.txt.enc
drwxr-xr-x  2 pkencrypt1 pkencrypt1 4096 Feb 16  2023 keys/

pkencrypt1@ubuntuserver:~$ cd keys

pkencrypt1@ubuntuserver:~/keys$ ll
total 16
drwxr-xr-x 2 pkencrypt1 pkencrypt1 4096 Feb 16  2023 ./
drwxr-x--- 3 pkencrypt1 pkencrypt1 4096 Jun 12 18:36 ../
-rw------- 1 pkencrypt1 pkencrypt1 1704 Feb 16  2023 privada.pem
-rw-r--r-- 1 pkencrypt1 pkencrypt1  451 Feb 16  2023 publica.pem

pkencrypt1@ubuntuserver:~/keys$ cat privada.pem

-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCa2IWDBD1CJ7Lt
5rkau/qdsQQjnlycmkwuP4tKb6nOU5NuQJNxKrRu8TRj/rKI3z/JywATP8kzGUUx
colo9KJ8SGc3sxD9g92K2eueTxwcCx+QwS09VNx9G4HaWwXSYoFJqbRE09VmWb93
uAMIEidXwDt19lgilLoLxZ/b2L31gicbweW+YnCZE1Lq7r9M+E8doCmaaFqVT1RW
NKILmDRO7lVNQfX+vJmC7JoKs4XB9lWKwNbf0dvebec+jeSELiaOb26gi+oSLhep
M1Q/U6G48C0oXo7OQZmERB1msvwFgmSyhnA9jvrErd0qJyr1X7lmsK2YOlbDkLt7
jkGDd8BNAgMBAAECggEAB256Lz42LRqip8WkVKLOSwstMQMaBsS7q0HkJCpqcpCS
9Cj9P+KAzmJH1WoVP3nCIfkwLmb9TQqFcPNs94XmK84ocK/ovNdl++gBkD4pEd98
IFe6csXD30KWtNgcK2thhnDdSgBbWZItHPMj53CXmsqLEATxWJeIEPs8dNyuIT45
aH8Ov0+x6mPhMRhpsNPx040MawhrKh8RC/PRLgjZ5EkBIwb0AUmy38i+uutATRh2
egor81RLBn4lz+bxt4rLUSW9V58/EnoitO0+SXa8W8tZ6GFx4LzgNtAt5L1VUSBA
lMGi/wWCk+K8mFzHvviQWTu8SLIjDGXurmX64Zx8AQKBgQC+BFzsVYnSuJATFDic
Fn8ooAyzR9dmpcRoCyU+k274E43O8CVhTQzCld5BhqgTO0hxTrYIpRvKpvpZyiak
eCn61pT73hdCk1U+JK89HtnqkeKm3RcP1bh+pLVl5qjjY/hCkhRVjswBjj1p8zNa
akwCtb5rge9GPISNYo6oJGl/DQKBgQDQnZixDPKP0RCviRjhrITfIAOsyxCAsnsl
kDnWIifwZGEPbDBNRJfzq/mFsA0jfrHxDaayN6cd52BR0Y+YuRCijY2FiAYR0Fsx
7gCE1n1dhpMVWJJMw+Vg5POVWZPbxJKRRzlePMYZVGR5QbFIXsaAnDZJGxaPuW4I
MnJrEPj2QQKBgQCQtJfjbxzfeahWrz6RN9ysnn4thdd3F2Rka6B4cCTBDXsgDegZ
mmjOQv2YXyjeRHZdu7iLCtoIUXM0L+uPsucdXI7m5HJIRBVVlvBRFo6TwXee5Z4r
c/HlmB+As9EIIlissbyEj5Oy15TTe98uyuaJ5chW7QPANFQpq9XCHMCufQKBgQCh
wMCG71IYPvNgF74qJTk1RD51OVJHZ5xiiMy/guZS15IGgk2Fa90h+8NSbCoTzoWs
MXiCEPLMFf4yEnnz4fLLB1SnJ8wE/ffn4/GVDjZQUSs0TuPJD8+H7J4NvFIQAf/f
E0mhDyBOvYfWGSCby5jAWd8hmhZJRG7TfkIHUDapwQKBgA+TGgfIubd0ZaePGEEm
+LRR0GBKDLhG02qEKPhlMtY9+lYupvh5qdPbsfFxU14gZAfARBy5mV4XsKf6GPTP
otWmX2NYw3+d3pJc/k0VtTLPQxs0rNt4XygO/PCy11GU4oYDD+cXvyWmv+ynbhgN
B11uunywQrQq7e6jGX2Hommn
-----END PRIVATE KEY-----

pkencrypt1@ubuntuserver:~/keys$ cat publica.pem

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAmtiFgwQ9Qiey7ea5Grv6
nbEEI55cnJpMLj+LSm+pzlOTbkCTcSq0bvE0Y/6yiN8/ycsAEz/JMxlFMXKJaPSi
fEhnN7MQ/YPditnrnk8cHAsfkMEtPVTcfRuB2lsF0mKBSam0RNPVZlm/d7gDCBIn
V8A7dfZYIpS6C8Wf29i99YInG8HlvmJwmRNS6u6/TPhPHaApmmhalU9UVjSiC5g0
Tu5VTUH1/ryZguyaCrOFwfZVisDW39Hb3m3nPo3khC4mjm9uoIvqEi4XqTNUP1Oh
uPAtKF6OzkGZhEQdZrL8BYJksoZwPY76xK3dKicq9V+5ZrCtmDpWw5C7e45Bg3fA
TQIDAQAB
-----END PUBLIC KEY-----
```

Usamos `openssl` para descifrar el archivo usando la clave privada directamente.

```
pkencrypt1@ubuntuserver:~$ openssl pkeyutl -decrypt -inkey ./keys/privada.pem -in contraseña.txt.enc -out contraseña.txt

pkencrypt1@ubuntuserver:~$ cat contraseña.txt
La contraseña de pkencrypt2 es Dec0deASPrivate
```

**Flag:** `Dec0deASPrivate`

#### Nivel Pkencrypt 2 (Cifrado Híbrido)
Al acceder nos encontramos con un esquema híbrido de clave efimera RSA y cifrado asimétrico AES-256:

```
pkencrypt1@ubuntuserver:~/keys$ su pkencrypt2
Password:

pkencrypt2@ubuntuserver:/home/pkencrypt1/keys$ cd /home/pkencrypt2

pkencrypt2@ubuntuserver:~$ ll
total 48
drwxr-x---  3 pkencrypt2 pkencrypt2 4096 Jun 12 18:43 ./
drwxr-xr-x 30 root       root       4096 May 16  2023 ../
-rw-------  1 pkencrypt2 pkencrypt2  827 Jun 12 19:13 .bash_history
-rw-r--r--  1 pkencrypt2 pkencrypt2  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 pkencrypt2 pkencrypt2 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 pkencrypt2 pkencrypt2  807 Feb 16  2023 .profile
-rw-r--r--  1 pkencrypt2 pkencrypt2   64 Feb 16  2023 contraseña.txt.aes2
-rw-r--r--  1 pkencrypt2 pkencrypt2  256 Feb 16  2023 ephemereal_key.enc
drwxr-xr-x  2 pkencrypt2 pkencrypt2 4096 Feb 16  2023 keys/

pkencrypt2@ubuntuserver:~$ cd keys

pkencrypt2@ubuntuserver:~/keys$ ll
total 16
drwxr-xr-x 2 pkencrypt2 pkencrypt2 4096 Feb 16  2023 ./
drwxr-x--- 3 pkencrypt2 pkencrypt2 4096 Jun 12 18:43 ../
-rw------- 1 pkencrypt2 pkencrypt2 1704 Feb 16  2023 privada.pem
-rw-r--r-- 1 pkencrypt2 pkencrypt2  451 Feb 16  2023 publica.pem

pkencrypt2@ubuntuserver:~/keys$ cat privada.pem
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCib38SNC3en7yZ
czk8isOc1csfpyK4j16gW9PClzEPnKGKS+JkNLZaX0owQpsOhWJxdPKtShYZN15Q
rddfTpA1ooLddS+9Nb0W3eQ5/KBD1xasR5s+CcrAubJzVfG1PHWTuokDLZuUiooW
KDXWUJvip0hrJ7NxDB//m2ohAXDGcvCBT/hffL9M9pt7ycqNq+IarQT1ho625nEh
WMlkJ304Y3X2l1yGByB3Ew6Wi99wqt9FSDKBWvkTjDouE5ThkzQBnCJr7aAj8+8j
kplZcxT3xEEd2SFQzQ/kuTwR5vXlWUqFEDSsJ81+XMlgPvHecBAnxNaOJpDPl5L9
FrZqmy4xAgMBAAECggEAIQZzOvPB6bPnZ/l9xxndaIstJ6vXCtgXYJoa5ULRFj/9
tfz0s/PlqV0bW9GG7f5fg/rHhkT8VjckJVoa4kU6W7VFTxpO0PTWk4ocp09+FBzs
fq12WjgqcWGv7vQn1vXKX/U6extwONSr+5JEU+UCtKPXPPYO7SqObO0fuEMvNyF+
2kVBibvAwWdLgRoTAMwvykKR0Ncr7QzcACtA+QkemotXgHvKkY3r0vPxDaR98V0L
blFaFATUqteEXl3vUVcLcqw3bgHwO+sPKM6mnqn19IjJDcHMnowB/0GZUNaEuyv/
YJIgNMDGk5/VJ7hzU4YweiLlV1NWgg+aVZ5xN8cLeQKBgQC1igerUE4XiopMraS/
QRL3wPCYU9Wsip62U5KwyRzl3be2KHM5z2/mwsnnV/5rfi4FjLnT+i+ITv1N83Zh
yAOy4YU2/fERE5nripO8+Y9lz9oR1aUfOLrKsuI6r/ZWnPX9kvkXa9KkMy70eRqR
Lu8X1j/f/vzhQyHsVDlrupQ/DQKBgQDlD4vHoIpH8U0ZORJWqeGOnFvei6xkZvXB
800NnrgFl/rRfnDtiQ6HeF2iSmfrUZTSgloBx/J4n1vhTwp817YqG1JVQjZ/8jrj
UC7WH9ebUaFvGx0b5lFL+zdbGP4M8gM+3vuxP9Bz4SBjEQOjZvm2XU8P71uG2TPj
Sm21FV6CtQKBgHnZxR4LD++jMQMYxm0NK8MaQSOtmc1vWep9nAeHZhswQABHlFfo
UW7trgHXQVE7Z36YH58V3dO7WTB6SyqEy17FGtp3hth0dKrx4ApG5CZtZiz0Xxne
xRoLCehkdY9bWY2zmfhLih5msIytwNRUUW2JhGRATdKRcfKj8crKeHj9AoGBAKKY
43dETYR+FGV4Lr1X/+XUth4Gdcwbjg4sICEv7p8B4Ch/obfr12Vwmr7OJHBVS9gW
cb/b6BGZxYXtLpuqIARJuqsMwlUWZJjhXS3gEpONYZPV4lbgqgrOe9/toMEdthIW
BQIwM09emjfYZwXB3jaGi83a8dSKMBwCxFeLoLgVAoGAXYpZHUn+r2uKh2Vcmmdi
KP6/MFnYjtvsgoH59V+QxzRfjtl9e31l4NJQPEtR72JTKB8D8/tJcETcT20eMXtl
TbxDyh3RuhpaGZdp+zsQuzegSYQf2aqISKFPzusHNOAqiKpgsyGKG1PZW/vosrW6
7atRIFxhda1LcaaqfcM4OFA=

-----END PRIVATE KEY-----

pkencrypt2@ubuntuserver:~/keys$ cat publica.pem
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAom9/EjQt3p+8mXM5PIrD
nNXLH6ciuI9eoFvTwpcxD5yhikviZDS2Wl9KMEKbDoVicXTyrUoWGTdeUK3XX06Q
NaKC3XUvvTW9Ft3kOfygQ9cWrEebPgnKwLmyc1XxtTx1k7qJAy2blIqKFig11lCb
4qdIayezcQwf/5tqIQFwxnLwgU/4X3y/TPabe8nKjaviGq0E9YaOtuZxIVjJZCd9
OGN19pdchgcgdxMOlovfcKrfRUgygVr5E4w6LhOU4ZM0AZwia+2gI/PvI5KZWXMU
98RBHdkhUM0P5Lk8Eeb15VlKhRA0rCfNflzJYD7x3nAQJ8TWjiaQz5eS/Ra2apsu
MQIDAQAB
-----END PUBLIC KEY-----
```

1. Utilizamos la clave privada RSA para descifrar la **"clave efímera"**
	
	- Clave recuperada: `VyX76Dnmsny6534jjDM`.
	
2. Posteriormente, utilizamos esa clave efímera para descifrar el archivo simétrico AES-256 final.

```
pkencrypt2@ubuntuserver:~$ openssl pkeyutl -decrypt -inkey ./keys/privada.pem -in ephemereal_key.enc -out ephemereal_key.txt

pkencrypt2@ubuntuserver:~$ ls -la
total 36
drwxr-x---  3 pkencrypt2 pkencrypt2 4096 Jun 11 19:24 .
drwxr-xr-x 30 root       root       4096 May 16  2023 ..
-rw-r--r--  1 pkencrypt2 pkencrypt2  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 pkencrypt2 pkencrypt2 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 pkencrypt2 pkencrypt2  807 Feb 16  2023 .profile
-rw-r--r--  1 pkencrypt2 pkencrypt2   64 Feb 16  2023 contraseña.txt.aes2
-rw-r--r--  1 pkencrypt2 pkencrypt2  256 Feb 16  2023 ephemereal_key.enc
-rw-rw-r--  1 pkencrypt2 pkencrypt2   20 Jun 11 19:24 ephemereal_key.txt
drwxr-xr-x  2 pkencrypt2 pkencrypt2 4096 Feb 16  2023 keys

pkencrypt2@ubuntuserver:~$ cat ephemereal_key.txt
VyX76Dnmsny6534jjDM

pkencrypt2@ubuntuserver:~$ openssl aes-256-cbc -d -in contraseña.txt.aes2 -out contraseña.txt -pbkdf2
enter AES-256-CBC decryption password:

pkencrypt2@ubuntuserver:~$ ls -la
total 40
drwxr-x---  3 pkencrypt2 pkencrypt2 4096 Jun 11 19:26 .
drwxr-xr-x 30 root       root       4096 May 16  2023 ..
-rw-r--r--  1 pkencrypt2 pkencrypt2  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 pkencrypt2 pkencrypt2 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 pkencrypt2 pkencrypt2  807 Feb 16  2023 .profile
-rw-rw-r--  1 pkencrypt2 pkencrypt2   47 Jun 11 19:26 contraseña.txt
-rw-r--r--  1 pkencrypt2 pkencrypt2   64 Feb 16  2023 contraseña.txt.aes2
-rw-r--r--  1 pkencrypt2 pkencrypt2  256 Feb 16  2023 ephemereal_key.enc
-rw-rw-r--  1 pkencrypt2 pkencrypt2   20 Jun 11 19:24 ephemereal_key.txt
drwxr-xr-x  2 pkencrypt2 pkencrypt2 4096 Feb 16  2023 keys

pkencrypt2@ubuntuserver:~$ cat contraseña.txt
La contraseña de pkencrypt3 es KeyExchangeEPH
```

**Flag:** `KeyExchangeEPH`

#### Pkencrypt3
Al llegar a `pkencrypt3`, obtenemos la bandera ``flag_mid.txt`` y pasamos a la última fase de Hashing con las credenciales facilitadas.

```
pkencrypt2@ubuntuserver:~/keys$ su pkencrypt3
Password:

pkencrypt3@ubuntuserver:/home/pkencrypt2/keys$ cd /home/pkencrypt3

pkencrypt3@ubuntuserver:~$ ll
total 28
drwxr-x---  2 pkencrypt3 pkencrypt3 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root       root       4096 May 16  2023 ../
-rw-------  1 pkencrypt3 pkencrypt3  119 Jun 12 19:13 .bash_history
-rw-r--r--  1 pkencrypt3 pkencrypt3  220 Feb 16  2023 .bash_logout
-rw-r--r--  1 pkencrypt3 pkencrypt3 3771 Feb 16  2023 .bashrc
-rw-r--r--  1 pkencrypt3 pkencrypt3  807 Feb 16  2023 .profile
-rw-r--r--  1 pkencrypt3 pkencrypt3  272 Apr  3  2024 flag_mid.txt

pkencrypt3@ubuntuserver:~$ cat flag_mid.txt
Enhorabuena.

Has alcanzado el nivel que te permite avanzar al siguiente bloque, abriendo una nueva línea en el reto.

Para comenzar con los ejercicios de funciones hash, haremos uso del siguiente usuario:

        - Usuario: hashing1 
        - Contraseña: hashing1
```

---
## 🟣 Fase 4: Hashing (Integridad y Cracking)

Finalmente, abordamos la verificación de **integridad** y el **cracking** de contraseñas mediante hashes.

### Niveles Hashing 1 a 4 (Identificación de Hash)
En estos niveles la mecánica era la misma. Tenemos una carpeta con muchos archivos (`contraseña1.txt`, `contraseña2.txt`, etc.) y el `info.txt` que nos da un hash específico. La tarea consistió en identificar qué archivo coincidía con el hash proporcionado. Utilizamos herramientas de sumatoria (`md5sum`, `sha1sum`, `sha256sum`, `sha512sum`) para filtrar los resultados:

###### Hashing 1 (MD5)
```
pkencrypt3@ubuntuserver:~$ su hashing1
Password:

hashing1@ubuntuserver:/home/pkencrypt3$ cd ../hashing1

hashing1@ubuntuserver:~$ ll
total 32
drwxr-x---  3 hashing1 hashing1 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing1 hashing1  789 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing1 hashing1  220 Feb 17  2023 .bash_logout
-rw-r--r--  1 hashing1 hashing1 3771 Feb 17  2023 .bashrc
-rw-r--r--  1 hashing1 hashing1  807 Feb 17  2023 .profile
drwxr-xr-x  2 hashing1 hashing1 4096 Feb 17  2023 Creds/
-rw-r--r--  1 hashing1 hashing1  379 Feb 17  2023 info.txt

hashing1@ubuntuserver:~$ cat info.txt
Tenemos una carpeta con todas las claves de la organización.

Sabemos que el fichero que guarda la clave correcta para el usuario hashing2 tiene el siguiente hash:

        9f75f653a20dba0796f5011dddc34aaa

¿Podrías decirnos qué clave es la correcta para el usuario hashing2?

IMPORTANTE: Sólo hay una condición, únicamente puedes probar 2 claves o el usuario se bloqueará.

hashing1@ubuntuserver:~$ cd Creds

hashing1@ubuntuserver:~/Creds$ ll
total 48
drwxr-xr-x 2 hashing1 hashing1 4096 Feb 17  2023 ./
drwxr-x--- 3 hashing1 hashing1 4096 Jun 11 23:25 ../
-rw-r--r-- 1 hashing1 hashing1   36 Feb 17  2023 contraseña1.txt
-rw-r--r-- 1 hashing1 hashing1   34 Feb 17  2023 contraseña10.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña2.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña3.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña4.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña5.txt
-rw-r--r-- 1 hashing1 hashing1   38 Feb 17  2023 contraseña6.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña7.txt
-rw-r--r-- 1 hashing1 hashing1   37 Feb 17  2023 contraseña8.txt
-rw-r--r-- 1 hashing1 hashing1   35 Feb 17  2023 contraseña9.txt

hashing1@ubuntuserver:~/Creds$ md5sum contraseña*.txt | grep 9f75f653a20dba0796f5011dddc34aaa
9f75f653a20dba0796f5011dddc34aaa contraseña7.txt

hashing1@ubuntuserver:~/Creds$ cat contraseña7.txt
La pass de hashing2 es Check1ngMD5
```

- Archivo: `contraseña7.txt`. **Pass:** `Check1ngMD5`.

###### Hashing2 (SHA-1)
```
hashing1@ubuntuserver:~$ su hashing2
Password:

hashing2@ubuntuserver:/home/hashing1$ cd ../hashing2

hashing2@ubuntuserver:~$ ll
total 32
drwxr-x---  3 hashing2 hashing2 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing2 hashing2  426 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing2 hashing2  220 Feb 17  2023 .bash_logout
-rw-r--r--  1 hashing2 hashing2 3771 Feb 17  2023 .bashrc
-rw-r--r--  1 hashing2 hashing2  807 Feb 17  2023 .profile
drwxr-xr-x  2 hashing2 hashing2 4096 Feb 17  2023 Creds/
-rw-r--r--  1 hashing2 hashing2  387 Feb 17  2023 info.txt

hashing2@ubuntuserver:~$ cat info.txt
Tenemos una carpeta con todas las claves de la organización.

Sabemos que el fichero que guarda la clave correcta para el usuario hashing3 tiene el siguiente hash:

        26ed6139d311e851d4efa906bfc78e90f970cedd

¿Podrías decirnos qué clave es la correcta para el usuario hashing3?

IMPORTANTE: Sólo hay una condición, únicamente puedes probar 2 claves o el usuario se bloqueará.

hashing2@ubuntuserver:~$ cd Creds

hashing2@ubuntuserver:~/Creds$ ll
total 48
drwxr-xr-x 2 hashing2 hashing2 4096 Feb 17  2023 ./
drwxr-x--- 3 hashing2 hashing2 4096 Jun 11 23:25 ../
-rw-r--r-- 1 hashing2 hashing2   36 Feb 17  2023 contraseña1.txt
-rw-r--r-- 1 hashing2 hashing2   34 Feb 17  2023 contraseña10.txt
-rw-r--r-- 1 hashing2 hashing2   35 Feb 17  2023 contraseña2.txt
-rw-r--r-- 1 hashing2 hashing2   35 Feb 17  2023 contraseña3.txt
-rw-r--r-- 1 hashing2 hashing2   36 Feb 17  2023 contraseña4.txt
-rw-r--r-- 1 hashing2 hashing2   35 Feb 17  2023 contraseña5.txt
-rw-r--r-- 1 hashing2 hashing2   38 Feb 17  2023 contraseña6.txt
-rw-r--r-- 1 hashing2 hashing2   35 Feb 17  2023 contraseña7.txt
-rw-r--r-- 1 hashing2 hashing2   37 Feb 17  2023 contraseña8.txt
-rw-r--r-- 1 hashing2 hashing2   35 Feb 17  2023 contraseña9.txt

hashing2@ubuntuserver:~/Creds$ sha1sum contraseña*.txt | grep 26ed6139d311e851d4efa906bfc78e90f970cedd
26ed6139d311e851d4efa906bfc78e90f970cedd  contraseña4.txt

hashing2@ubuntuserver:~/Creds$ cat contraseña4.txt
La pass de hashing3 es Check1ngSHA1
```

- Archivo: `contraseña4.txt`. **Pass:** `Check1ngSHA1`.

###### Hashing3 (SHA-256)
```
hashing2@ubuntuserver:~/Creds$ su hashing3
Password:

hashing3@ubuntuserver:/home/hashing2/Creds$ cd /home/hashing3

hashing3@ubuntuserver:~$ ll
total 32
drwxr-x---  3 hashing3 hashing3 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing3 hashing3  478 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing3 hashing3  220 Feb 17  2023 .bash_logout
-rw-r--r--  1 hashing3 hashing3 3771 Feb 17  2023 .bashrc
-rw-r--r--  1 hashing3 hashing3  807 Feb 17  2023 .profile
drwxr-xr-x  2 hashing3 hashing3 4096 Feb 17  2023 Creds/
-rw-r--r--  1 hashing3 hashing3  411 Feb 17  2023 info.txt

hashing3@ubuntuserver:~$ cat info.txt
Tenemos una carpeta con todas las claves de la organización.

Sabemos que el fichero que guarda la clave correcta para el usuario hashing4 tiene el siguiente hash:

        c5f8d03cab180bffb6268f096ebd44840d5d2f5481a75ad588ca02000f572e7c

¿Podrías decirnos qué clave es la correcta para el usuario hashing4?

IMPORTANTE: Sólo hay una condición, únicamente puedes probar 2 claves o el usuario se bloqueará.

hashing3@ubuntuserver:~$ sha256sum ./Creds/contraseña*.txt | grep c5f8d03cab180bffb6268f096ebd44840d5d2f5481a75ad588ca02000f572e7c
c5f8d03cab180bffb6268f096ebd44840d5d2f5481a75ad588ca02000f572e7c  ./Creds/contraseña8.txt

hashing3@ubuntuserver:~$ cat ./Creds/contraseña8.txt
La contraseña del usuario hashing4 es BDHey23dsfad890bSHDYsm
```

- Archivo: `contraseña8.txt`. **Pass:** `BDHey23dsfad890bSHDYsm`.

###### Hashing4 (SHA-512)
```
hashing3@ubuntuserver:~$ su hashing4
Password:

hashing4@ubuntuserver:/home/hashing3$ cd ../hashing4

hashing4@ubuntuserver:~$ ll
total 32
drwxr-x---  3 hashing4 hashing4 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing4 hashing4  518 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing4 hashing4  220 Feb 17  2023 .bash_logout
-rw-r--r--  1 hashing4 hashing4 3771 Feb 17  2023 .bashrc
-rw-r--r--  1 hashing4 hashing4  807 Feb 17  2023 .profile
drwxr-xr-x  2 hashing4 hashing4 4096 Feb 17  2023 Creds/
-rw-r--r--  1 hashing4 hashing4  475 Feb 17  2023 info.txt

hashing4@ubuntuserver:~$ cat info.txt
Tenemos una carpeta con todas las claves de la organización.

Sabemos que el fichero que guarda la clave correcta para el usuario hashing5 tiene el siguiente hash:

        8a2f1de3b96eac2e0687ab9980d450b147aa3cb46ac891c724abaf757495518211ac71b16f59b92e7704ab1f6553e6f9609a977f723abca0f29b10089fe5db44

¿Podrías decirnos qué clave es la correcta para el usuario hashing5?

IMPORTANTE: Sólo hay una condición, únicamente puedes probar 2 claves o el usuario se bloqueará.

hashing4@ubuntuserver:~$ sha512sum ./Creds/contraseña*.txt | grep 8a2f1de3b96eac2e0687ab9980d450b147aa3cb46ac891c724abaf757495518211ac71b16f59b92e7704ab1f6553e6f9609a977f723abca0f29b10089fe5db44

8a2f1de3b96eac2e0687ab9980d450b147aa3cb46ac891c724abaf757495518211ac71b16f59b92e7704ab1f6553e6f9609a977f723abca0f29b10089fe5db44**  ./Creds/contraseña9.txt

hashing4@ubuntuserver:~$ cat ./Creds/contraseña9.txt
La contraseña del usuario hashing5 es BDHasDFHsydnbSHDYsm
```

- Archivo: `contraseña9.txt`. **Pass:** `BDHasDFHsydnbSHDYsm`.

#### Nivel Hashing 5 (Cracking Final)
```
hashing4@ubuntuserver:~$ su hashing5
Password:

hashing5@ubuntuserver:/home/hashing4$ cd ../hashing5

hashing5@ubuntuserver:~$ ll
total 28
drwxr-x---  2 hashing5 hashing5 4096 Jun 11 23:25 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing5 hashing5  489 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing5 hashing5  220 Feb 17  2023 .bash_logout
-rw-r--r--  1 hashing5 hashing5 3771 Feb 17  2023 .bashrc
-rw-r--r--  1 hashing5 hashing5  807 Feb 17  2023 .profile
-rw-r--r--  1 hashing5 hashing5   33 Feb 17  2023 contraseña_hashing6.md5

hashing5@ubuntuserver:~$ cat contraseña_hashing6.md5
0192023a7bbd73250516f069df18b500
```

El último desafío presentó un hash MD5 sin archivo de referencia. Dado que MD5 está criptográficamente roto y vulnerable a ataques de fuerza bruta o diccionario, consultamos bases de datos de hashes conocidos. El hash `0192023a7bbd73250516f069df18b500` correspondió a la contraseña trivial `admin123`

---
## 🏁 Conclusión

Al logarnos en `hashing6` con la contraseña crackeada, leemos el `flag_mid.txt` final.

```
hashing5@ubuntuserver:~$ su hashing6
Password:

hashing6@ubuntuserver:/home/hashing5$ cd ../hashing6

hashing6@ubuntuserver:~$ ll
total 36
drwxr-x---  3 hashing6 hashing6 4096 Apr  3  2024 ./
drwxr-xr-x 30 root     root     4096 May 16  2023 ../
-rw-------  1 hashing6 hashing6  123 Jun 12 19:13 .bash_history
-rw-r--r--  1 hashing6 hashing6  220 May 16  2023 .bash_logout
-rw-r--r--  1 hashing6 hashing6 3771 May 16  2023 .bashrc
drwx------  2 hashing6 hashing6 4096 May 22  2023 .cache/
-rw-r--r--  1 hashing6 hashing6  807 May 16  2023 .profile
-rw-------  1 hashing6 hashing6 1054 May 22  2023 .viminfo
-rw-r--r--  1 root     root      285 Apr  3  2024 flag_mid.txt

hashing6@ubuntuserver:~$ cat flag_mid.txt
Enhorabuena.

Has conseguido resolver el reto por completo, lo que demuestra que has adquirido los conocimientos necesarios y que eres capaz de poner en práctica todo lo aprendido.

Vamos a por el siguiente módulo!!
```

**Aprendizajes clave del reto:**

Desde una perspectiva técnica, este reto ha evidenciado la importancia crítica de:

1. No confundir codificación con cifrado.   
2. Proteger rigurosamente las claves privadas asimétricas.    
3. Utilizar algoritmos de hashing robustos (como bcrypt o Argon2) en lugar de funciones rápidas como MD5 para el almacenamiento de credenciales.

