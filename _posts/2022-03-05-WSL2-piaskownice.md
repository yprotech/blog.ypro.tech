---
layout: post
title: "WSL2 - wiele piaskownic"
date: 2022-03-05
permalink: /2022/03/WSL2-piaskownice
categories: ["Instrukcje"]
---

#### Czym jest WSL?
To podsystem umożliwiający uruchomienie jądra Linuxa w ramach systemu Windows. Pisałem o tym [wcześniej](https://blog.ypro.tech/2021/04/wsl2-1).


#### Jak wykorzystać?
Jedną z możliwości jest zainstalowanie wielu instancji jednej lub różnych dystrybucji Linuxa i wykorzystanie każdej z nich do innego celu. Każdą z instancji można inaczej skonfigurować, zainstalować inne narzędzia itp.

Można także wykorzystać je do niezaśmiecania bazowego systemu Windows. Wiele programów i narzędzi ma także linuxowe wersje (lub odpowiedniki). Każdą instancję można potraktować jako swoistą piaskownicę - takie wirtualne środowisko, odpowiednik maszyny wirtualnej z wykorzystaniem WSL. Instalując je w różnych instancjach WSL unikamy konieczności ich instalacji bezpośrednio w systemie Windows.

#### Jak to zrobić?
Używając sklepu systemowego (Microsoft Store) można zainstalować kilka dystrybucji np. Ubuntu 20.04 i Ubuntu 18.04, ale nie jest możliwe zainstalowanie kilku instancji jednej dystrybucji (np. Ubuntu 20.04).
Polecam [zainstalować](https://blog.ypro.tech/2021/04/wsl2-1) pierwszą instancję w standardowy sposób. Ta instancja posłuży jako szablon dla następnych. Dlatego można ją wstępnie skonfigurować, zainstalować podstawowe narzędzia (np. mc), zaktualizować itp., według potrzeb i własnego uznania.
Gdy pierwsza instancja jest gotowa należy utworzyć jej kopię:
```console
wsl --export Ubuntu-20.04 D:\WSL\Ubuntu-20.04.tar
```
Należy zachować plik .tar - będzie służył jako szablon dla następnych instancji. Nową instancję można stworzyć importując wyeksportowany wcześniej plik tar. Aby zaimportować nową instancję o nazwie ``Ubuntu-20.04-2`` i umieścić ją w folderze ``D:\WSL\Ubuntu-20.04-2`` należy wykonać:
```console
wsl --import Ubuntu-20.04-2 D:\WSL\Ubuntu-20.04-2 D:\WSL\Ubuntu-20.04.tar --version 2
```

Przed eksportem szablonu można także wskazać domyślnego użytkownika, na rzecz którego nastąpi logowanie do danej instancji. W tym celu należy sprawdzić czy istnieje plik ``/etc/wsl.conf`` z wpisem:
```
[user]
default=abcde
```
gdzie ``abcde`` to nazwa użytkownika.

Uruchomienie konsoli danej dystrybucji:
```console
wsl -d Ubuntu-20.04
```
Wypisanie zawartości pliku:
```console
cat /etc/wsl.conf
```
a jeżeli taki plik nie istnieje lub nie zawiera właściwego wpisu:
```console
echo -e "[user]\ndefault=abcde" >> /etc/wsl.conf
```

W przypadku, gdy mamy do czynienia z instancją, która nie ma właściwego wskazania domyślnego użytkownika, można to skonfigurować także w zaimportowanej instancji:
```console
wsl -d Ubuntu-20.04-2
usermod -aG sudo abcde
echo -e "[user]\ndefault=abcde" >> /etc/wsl.conf
exit
```


#### Inne dystrybucje
Istnieje możliwość zainstalowania innych wersji dystrybucji Linuxa niż te udostępnione w Microsoft Store. Należy pobrać odpowiedni obraz dystrybucji - w przypadku Ubuntu poszukiwania zaczynamy na [stronie](https://cloud-images.ubuntu.com/releases/) i poszukujemy pliku amd64-wsl.rootfs.tar.gz odpowiedniej wersji. Dla 21.10 będzie to [ubuntu-21.10-server-cloudimg-amd64-wsl.rootfs.tar.gz](https://cloud-images.ubuntu.com/releases/21.10/release/ubuntu-21.10-server-cloudimg-amd64-wsl.rootfs.tar.gz).

Należy go pobrać, a następnie zaimportować:
```console
wsl --import Ubuntu-Test D:\WSL\Ubuntu-Test D:\WSL\ubuntu-21.10-server-cloudimg-amd64-wsl.rootfs.tar.gz
wsl -l -v
wsl -d Ubuntu-Test
adduser abcde
usermod -aG sudo abcde
echo -e "[user]\ndefault=abcde" >> /etc/wsl.conf
exit
wsl -t Ubuntu-Test
wsl -d Ubuntu-Test
```


#### Podsumowanie
Z jednego szablonu można utworzyć wiele instancji WSL. W każdej z nich można zainstalować inny zestaw narzędzi i programów lub różne zestawy ich wersji - w zależności od potrzeb.
Jest to szczególnie przydatne, gdy często używamy różnych wersji oprogramowania, a jednoczesna instalacja tych wersji jest kłopotliwa.