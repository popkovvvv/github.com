---
layout: post
title: "Read sequence of chars from console in Java"
category : Programming
---

Послезавтра у меня госэкзамены по различным предметам, в том числе и по программированию, вот я и решил взглянуть на эти задания. Все задания элементарные: определить является ли матрица симметричной, найти наибольше число в последовательности и прочие алгоритмические. Задания предназначены для написания на С и были придуманы еще в 198х. Всё было бы ничего, если бы надо выполнить эти задания на С, но по какой-то причине разрешили использовать любые языки, запретив пользоваться стандартной библиотекой. Я что отвлёкся, перейду к делу. Одно из заданий звучит следующим образом: 

> Имеется непустая последовательность букв, за которой следует точка.
> Напечатать эту же последовательность, удалив из нее все буквы U,
> непосредственно перед которыми находится буква Q.

 Что может быть проще? Реализуем ввод, который останавливается при нажатии `.`:

{% highlight c++ %}
#include <string>
#include <iostream>
using namespace std;
int main() {
  string a;
  while(1) {
    char x;
    cin >> x;
    if(x=='.') break;
    a.push_back(x);
  }
  cout << a
}
{% endhighlight %}

Я не силён в C++ и хочу то же самое сделать с помощью Java. И вот тут то и начинаются проблемы: консоль находится в буферизованном режиме, то есть чтение производится только при нажатии enter. Решение заключается в том, что её необходимо переключить в raw режим. Звучит не страшно, но это переключение не кроссплатформенно и мягко говоря не маленькое. Пример решения под Linux:

{% highlight java %}
private static String ttyConfig;

public static void main(String[] args) {
  try {
    setTerminalToCBreak();
    int i=0;
    while (true) {
      System.out.println( ""+ i++ );
      if ( System.in.available() != 0 ) {
        int c = System.in.read();
        if ( c == 0x1B ) {
          break;
        }
      }
    } // end while
  } catch (IOException e) {
    System.err.println("IOException");
  } catch (InterruptedException e) {
    System.err.println("InterruptedException");
  } finally {
    try {
      stty( ttyConfig.trim() );
     } catch (Exception e) {
      System.err.println("Exception restoring tty config");
     }
  }

}

private static void setTerminalToCBreak() throws IOException, InterruptedException { 
  ttyConfig = stty("-g");
  // set the console to be character-buffered instead of line-buffered
  stty("-icanon min 1");
  // disable character echoing
  stty("-echo");
}

/**
 *  Execute the stty command with the specified arguments
 *  against the current active terminal.
 */
private static String stty(final String args)
                throws IOException, InterruptedException {
  String cmd = "stty " + args + " < /dev/tty";
  return exec(new String[] {"sh","-c", cmd});
}

/**
 *  Execute the specified command and return the output
 *  (both stdout and stderr).
 */
private static String exec(final String[] cmd)
                throws IOException, InterruptedException {
  ByteArrayOutputStream bout = new ByteArrayOutputStream();
  Process p = Runtime.getRuntime().exec(cmd);
  int c;
  InputStream in = p.getInputStream();
  while ((c = in.read()) != -1) {
    bout.write(c);
  }

  in = p.getErrorStream();
  while ((c = in.read()) != -1) {
    bout.write(c);
  }
  p.waitFor();
  String result = new String(bout.toByteArray());
  return result;
}
{% endhighlight %}

Статья, описывающая все эти премудрости - <http://www.darkcoding.net/software/non-blocking-console-io-is-not-possible/>. 

