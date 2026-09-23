---
layout: post
title: "macOS 27 a drukarki Brother"
date: 2026-09-23
permalink: /2026/09/macos-brother
categories: ["Instrukcje"]
---

### Wstęp
Historia zaczyna się od aktualizacji systemu macOS do wersji 27. Nowa wersja systemu przestała wspierać sterowniki drukarek inne niż AirPrint. W związku z tym straciłem możliwość drukowania na mojej, dość leciwej już, drukarce Brother w dotychczasowej konfiguracji.
Drukarka straciła już wsparcie i producent nie będzie dostarczał sterowników do nowych wersji systemów operacyjnych.

Wszystkie te zmiany związane są wycofywaniem wsparcia dla architektury x86_64.

W rezultacie próba dodania nowej drukarki w systemie kończy się błędem:
![błąd dodanie drukarki](/img/p202609/brother_01.png){: .mx-auto .d-block }

### Rozwiązanie

Gdy nie ma sterowników od producenta należy sobie poradzić samodzielnie. Istnieją otwartoźródłowe, oparte o licencję [GPL-2.0 license]( https://github.com/pdewacht/brlaser?tab=GPL-2.0-1-ov-file) sterowniki [brlaser](https://github.com/pdewacht/brlaser.git). Nie są one już rozwijane (ostatni commit w lutym 2023 roku), ale w moim przypadku okazały się wystarczające.

Wystarczy pobrać kod źródłowy, skompilować i zainstalować w systemie.

```console
brew install cmake pkg-config

git clone https://github.com/pdewacht/brlaser.git
cd brlaser

cmake . -DCMAKE_POLICY_VERSION_MINIMUM=3.5 
make
sudo make install
```

### Problemy

Niestety, nie zawsze wszystko idzie gładko. Tym razem `make install` powoduje błąd:

```
Install the project...
-- Install configuration: "RelWithDebInfo"
-- Installing: /usr/libexec/cups/filter/rastertobrlaser
-- Installing: /usr/share/cups/drv/brlaser.drv
CMake Error at cmake_install.cmake:71 (file):
  file INSTALL cannot copy file
  "/Users/tomek/Projects/tests/brlaser/brlaser.drv" to
  "/usr/share/cups/drv/brlaser.drv": Operation not permitted.

make: *** [install] Error 1
```

Można ten proces dokończyć ręcznie. Sprawdzenie czy wygenerowane są pliki ppd:
```console
ls ppd/*
```

Jeśli nie to zostaną wygenerowane poleceniem:
```console
ppdc brlaser.drv
```
i teraz już powinny być:
```console
ls ppd/*
```

Pozostaje manualne przekopiowanie plików wynikowych we właściwe miejsce:
```console
sudo cp ppd/*.ppd /Library/Printers/PPDs/Contents/Resources/
```

i sparwdzenie czy rzeczywiście tam się znalazły:
```console
ls -la /Library/Printers/PPDs/Contents/Resources/br*.ppd
ls -la /usr/libexec/cups/filter/rastertobrlaser
```

Na wszelki wypadek restart cups-a:
```console
sudo killall -HUP cupsd
```
lub
```console
sudo launchctl kickstart -k system/org.cups.cupsd
```

### Rezultat

W ponownym procesie dodawania drukarki, zamiast standardowych sterowników CUPS w polu ‘Użyj’
![nie CUPS](/img/p202609/brother_02.png){: .mx-auto .d-block }
należy wybrać w tym polu pozycję ‘Wybierz oprogramowanie…’ i wybrać jeden ze sterowników brlaser:
![wybierz brlaser](/img/p202609/brother_03.png){: .mx-auto .d-block }

Z tym sterownikiem dodanie drukarki w systemie nie powinno już zgłaszać błędu z oprogramowaniem drukarki.
