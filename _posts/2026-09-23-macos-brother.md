---
layout: post
title: "macOS 27 a drukarki Brother: jak rozwiązać problem ze sterownikami?"
date: 2026-09-23
edited: 2026-09-25
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
# 1. Instalacja narzędzi programistycznych
brew install cmake pkg-config

# 2. Sklonowanie repozytorium projektu brlaser
git clone https://github.com/pdewacht/brlaser.git
cd brlaser

# 3. Przygotowanie i kompilacja kodu źródłowego
cmake . -DCMAKE_POLICY_VERSION_MINIMUM=3.5
make

# 4. Próba automatycznej instalacji w systemie
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


### Jakie modele obsługuje sterownik brlaser?

Sterownik wspiera szereg monochromatycznych drukarek laserowych i urządzeń wielofunkcyjnych Brother bez AirPrint. Pełną listę oraz szczegóły można znaleźć w [dokumentacji źródłowej](https://github.com/pdewacht/brlaser/blob/master/README.md):

- Seria HL: m.in. HL-1110, HL-1200, HL-2030, HL-2140, HL-2220
- Seria HL-L: m.in. HL-L2300D, HL-L2320D, HL-L2340D, HL-L2375DW
- Seria DCP: m.in. DCP-1510, DCP-1600, DCP-7030, DCP-7055
- Seria DCP-L: m.in. DCP-L2500D, DCP-L2520DW, DCP-L2540DW
- Seria MFC: m.in. MFC-1910W, MFC-7240, MFC-7360N, MFC-L2710DW
