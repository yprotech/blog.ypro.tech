---
layout: post
title: "Użycie procesorów logoicznych przez uruchomione zadania"
date: 2021-10-16
permalink: /2021/10/tasks
---

Postanowiłem zbadać jak wykorzystywane są poszczególne rdzenie wielordzeniowego i wielowątkowego procesora przez aplikację napisaną w C# z użyciem różnej liczby uruchomionych przez nią zadań (*Task*). Do testów postanowiłem użyć bardzo prostej konsolowej aplikacji (źródło poniżej).
Nie badałem wydajności poszczególnych procesorów. Moim celem było jedynie zaobserwowanie jak system przydziela zadania do poszczególnych rdzeni i w jakim stopniu są one obciążane.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

namespace MultiThreadTest
{
    class Program
    {
        static void Main(string[] args)
        {
            int n = (args.Length > 0 && Int32.TryParse(args[0], out int v)) ? v : 8;

            bool useConsole = args.Length > 1 && "C".Equals(args[1].ToUpper());

            Console.WriteLine($"Threads:{n}, use console:{useConsole}");

            for (int i = 0; i < n; i++)
            {
                var task = new Task(() => (new Tester()).Do(useConsole));
                task.Start();
            }

            Thread.Sleep(20000);
        }
    }

    internal class Tester
    {
        internal void Do(bool useConsole)
        {
            int a = 3;
            int b = 5;

            while (true)
            {
                a = a + b;
                b = b + a;

                if (a > 1000)
                {
                    a = 5;
                    b = 3;
                }

                if (useConsole)
                    Console.Write($"{a}-{b}|");
            }
        }
    }
}
```

Testy przeprowadziłem na trzech procesorach:
- i5 10-tej generacji – 6 rdzeni, 12 wątków
- i3 10-tej generacji – 2 rdzenie, 4 wątki
- i5 7-mej generacji – 2 rdzenie, 4 wątki

Pierwszy test to wykonywanie zadań bez wyprowadzania wyników na konsolę (``useConsole: false``). Wyniki przedstawia poniższa tabela. Wyniki procentowe to ogólne wykorzystanie procesora.

|procesor|wątki|rdzenie|liczba&nbsp;tasków→|1|2|3|4|5|6|7|8|9|10|11|12|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| i5 (10) | 6 |12 ||12%|24%|36%|47%|63%|68%|79%|91%|100%|100%|100%|100%|
| i3 (10) | 2 | 4 ||73%|100%|100%|100%|
| i5 (7)  | 2 | 4 ||33%| 62%| 89%|100%|

Drugi test to wykonywanie obliczeń i wyprowadzanie ich rezultatów na konsolę.

|procesor|wątki|rdzenie|liczba&nbsp;tasków→|1|2|3|4|5|6|7|8|9|10|11|12|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| i5 (10) | 6 |12 ||15%|27%|38%|49%|60%|71%|82%|94%|100%|100%|100%|100%|
| i3 (10) | 2 | 4 ||85%|100%|100%|100%|
| i5 (7)  | 2 | 4 ||44%| 72%| 96%|100%|

Jak widać procentowe wykorzystanie procesora w obu przypadkach daje dość zbliżone wyniki. Jednak różnice pojawiają się gdy w rozkładzie obciążenia poszczególnych rdzeni.
Pierwsze zestawienie: procesor sześciordzeniowy i program testowy z uruchomionym jednym taskiem. Wykres dla trybu bez wyprowadzania danych na konsolę:

![](/img/p202110/i5_10-1x1.png)

oraz z ich wypisywaniem:

![](/img/p202110/i5_10-1c1.png)

Drugie zestawienie to ten sam procesor i program testowy z sześcioma uruchomionymi taskami. Dla lepszego zobrazowania program testowy uruchomiony kilka razy z rzędu. Wykres w trybie bez wyprowadzania danych na konsolę:

![](/img/p202110/i5_10-6xm.png)

oraz z ich wypisywaniem:

![](/img/p202110/i5_10-6cm.png)

W przypadku wcześniejszej generacji procesora i5 z tylko dwoma rdzeniami wartości procentowe obciążenia kształtują się inaczej, ale ich charakter wygląda podobnie do powyżej przedstawionych. Wykres bez wykorzystania konsoli wygląda następująco:

![](/img/p202110/i5_7-1xm.png)

a z ich wypisywaniem:

![](/img/p202110/i5_7-1cm.png)

Ja widać użycie w toku wykonywanego algorytmu takich operacji jak np. operacje wejścia-wyjścia, znacznie zmienia rozkład zadań na poszczególne procesory logiczne. W jednolitym algorytmie, który działa w nieskończonej pętli i nie ma w nim naturalnych miejsc przełączenia zadań (jak operacje we/wy) system przerzuca wątki pomiędzy różnymi procesorami logicznymi. Natomiast w drugim przypadku przydział poszczególnych zadań jest bardziej jednolity, mniej jest przeskoków, a jednocześnie przydzielony procesor logiczny nie jest w pełni obciążony.


Na koniec jeszcze jedna refleksja. Poniżej wykres dla dwurdzeniowego i3 z uruchomionym programem testowym z dwoma taskami z wykorzystaniem konsoli.

![](/img/p202110/i3_10-2cm.png)

W tym teście manager zadań pokazuje sumaryczne wykorzystanie procesora na poziomie 100%. Jednak widać, że ostatni procesor logiczny nie jest w pełni wykorzystany. Wynika z tego, że całkowite obciążenie procesora jest wyliczane w bardziej złożony sposób niż tylko średnia arytmetyczna wykorzystania poszczególnych rdzeni.