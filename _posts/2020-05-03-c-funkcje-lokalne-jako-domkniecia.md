---
layout: post
title: "C# - Funkcje lokalne jako domknięcia"
date: 2020-05-03
permalink: /2020/05/c-funkcje-lokalne-jako-domkniecia
categories: ["Pozostałe"]
---

Właściwie tytuł powinien brzmieć: funkcje lokalne wywołane z przechwyceniem wartości jako domknięcia.
Rozważmy następujący kod:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace Demo
{
    public delegate string FormatSomethingDelegate<in T1, in T2>(T1 a, T2 b);

    public class ClosuresDemo
    {
        public void Go()
        {
            var delegates = new List<FormatSomethingDelegate<int, int>>
            {
                CreateAlfa<int, int>(),
                CreateBeta<int, int>(),
                CreateAlfa<int, int>()
            };

            delegates
                .Select(d => d.Invoke(7, 12))
                .ToList()
                .ForEach(Console.WriteLine);
        }

        public FormatSomethingDelegate<T1, T2> CreateAlfa<T1, T2>()
        {
            string z = "ALFA";

            string Func(T1 x, T2 y) => ComputeInternal(x, y, z);

            return Func;
        }

        public FormatSomethingDelegate<T1, T2> CreateBeta<T1, T2>()
        {
            string z = "BETA";

            string Func(T1 x, T2 y) => ComputeInternal(x, y, z);

            return Func;
        }

        private string ComputeInternal<T1, T2>(T1 x, T2 y, string z)
        {
            return String.Join("|", x, y, z);
        }
    }
}
```

Wynikiem wywołania metody ``ClosuresDemo.Go`` będzie:

```console
7|12|ALFA
7|12|BETA
7|12|ALFA
```

Ok, ale jak to właściwie się stało? Aby to wyjaśnić należy spojrzeć na metodę ``CreateAlfa``:

```csharp
public FormatSomethingDelegate<T1, T2> CreateAlfa<T1, T2>()
{
    string z = "ALFA";

    string Func(T1 x, T2 y) => ComputeInternal(x, y, z);

    return Func;
}
```

Mamy tu funkcję lokalną Func, której ciałem jest wywołanie prywatnej metody ``ComputeInternal``. Jednak metoda ``ComputeInternal`` przyjmuje jeden parametr więcej niż funkcja lokalna ``Func``. Tym parametrerm jest lokalny parametr metody ``CreateAlfa`` (parametr ``z``), który staje się parametrem wywołania metody ``ComputeInternal``. Funkcja lokalna ``Func`` jest zwracana na zewnątrz w postaci delegata typu ``FormatSomethingDelegate``. A co z parametrem z? Jest on zmienną lokalną znaną tylko w zakresie metody ``CreateAlfa``. Otóż, następuje tu przechwytywanie zmiennych (ang. variable capture). Kompilator tworzy niejawną klasę, która przechowuje funkcję opakowaną w niezbędne parametry wywołania. Zdekompilowana metoda ``CreateAlfa`` wygląda następująco:

```csharp
// Demo.ClosuresDemo
public FormatSomethingDelegate<T1, T2> CreateAlfa<T1, T2>()
{
    <>c__DisplayClass1_0<T1, T2> <>c__DisplayClass1_ = new <>c__DisplayClass1_0<T1, T2>();
    <>c__DisplayClass1_.<>4__this = this;
    <>c__DisplayClass1_.z = "ALFA";
    return new FormatSomethingDelegate<T1, T2>(<>c__DisplayClass1_.<CreateAlfa>g__Func|0);
}
```

Jak widać powstała dodatkowa struktura służąca do obsługi takiego wywołania. [Dokumentacja](https://docs.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/local-functions) mówi o tym, że dzieje się tak tylko wtedy gdy jest taka potrzeba. Czyli gdy lokalna funkcja staje się delegatem i następuje przechwytywanie zmiennych. Czym zatem jest domknięcie? Definicję można sformułować następująco:
**Domknięcie** – struktura danych wiążąca funkcję z kontekstem jej wywołania. Zatem używając w C# funkcji lokalnych i przechwytywania zmiennych otrzymujemy kolejny sposób na stworzenie domknięcia znanego z programowania funkcyjnego.