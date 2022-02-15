---
layout: post
title: "Ciemny motyw strony internetowej"
date: 2022-02-15
permalink: /2022/02/dark
categories: ["Pozostałe"]
---

Aby zmienić wygląd strony internetowej, w zależności od tego, czy preferowany jest jasny czy ciemny schemat kolorystyczny, można wykorzystać cssową dyrektywę ``@media (prefers-color-scheme:``
```css
@media (prefers-color-scheme: dark) {
    ...
}

@media (prefers-color-scheme: light) {
    ...
}
```


Zasada jest taka, że wartość ``dark`` jest aktywowana, gdy preferowany jest ciemny schemat kolorystyczny, natomiast wartość ``light``, gdy preferowany jest schemat jasny lub preferencje nie są ustalone.
Oczywiście wewnątrz dyrektyw ``@media`` umieszczane są te definicje, które są charakterystyczne dla danego schematu. Definicje ogólne, niezależne od schematu, należy umieszczać poza tymi dyrektywami.

[Przykładowa strona](../../examples/2022/02/dark.html) oraz jej źródło:

```html
<!DOCTYPE html>
<html lang="pl">

<head>
    <meta charset="utf-8">

    <style type="text/css">
        @media (prefers-color-scheme: dark) {
            body {
                background-color: #111;
            }

            .box1 {
                background-color: #222;
                color: white;
            }

            .box2 {
                background-color: #222;
                color: blue;
            }
        }

        @media (prefers-color-scheme: light) {
            body {
                background-color: #eee;
            }

            .box1 {
                background-color: #fff;
                color: black;
            }

            .box2 {
                background-color: #e8e8e8;
                color: #222;
            }
        }

        .box1 {
            font: 20px "Helvetica, sans-serif";
            border: 1px solid;
            margin: 5px;
            padding: 3px;
        }

        .box2 {
            font: 20px "Georgia, serif";
            border: 1px dotted;
            margin: 5px;
            padding: 3px;
        }
    </style>
</head>

<body>
    <div class="box1">
        Tekst przykładowy.
    </div>

    <div class="box2">
        Inny tekst.
    </div>
</body>

</html>
```