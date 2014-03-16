---
layout: post
title: "Concurrency в Java: Shared State"
category : Programming
---

<img class="left" width="200" src="http://habrastorage.org/files/f49/241/bdd/f49241bdd24849e6b7c040b7b83cb082.jpg">


В многопоточном программировании много сложностей, основными из которых являются работа c разделяемым состоянием и эффективное использование предоставляемых ядер. Статья разбита на 2 части. В первой части я рассмотрю подходы к работе с разделяемым состоянием.

Shared State
==

Работу с разделяемым состоянием я покажу на примере вычисления чисел фибоначчи. 
Формула для вычисления выглядит так: 

	f(0) = 0
	f(1) = 1
	f(n) = f(n - 1) + f(n - 2) , n >= 2


В первую очередь определим интерфейс:

{% highlight java %}
public interface FibonacciGenerator<T> {
    /**
     * Следующее сгенерированное значение
     */
    T next();

    /**
     * Текущее значение в генераторе
     */
    public T val();
}
{% endhighlight %}	

Наши классы будут реализовать интерфейс FibonacciGenerator\<BigInteger\>. Из формулы видно, что для предоставления следующего числа, надо хранить два предыдущих --- это и будет разделяемое состояние, за которое будет проходить конкуренция. Наша реализация должна быть потоко-безопасной. То есть независимо от одновременного количества потребителей, она будет сохранять свои инварианты и придерживаться контракта. И так, приступим.


Locking
=====

Самый простой способ ---  это объявить все методы synchronized. Примером может служить класс Vector.

{% highlight java %}
public class IntrinsicLock implements FibonacciGenerator<BigInteger> {

    private BigInteger curr = BigInteger.ONE;
    private BigInteger next = BigInteger.ONE;

    @Override
    public synchronized BigInteger next() {
        BigInteger result = curr;
        curr = next;
        next = result.add(next);
        return result;
    }

    @Override
    public synchronized BigInteger val() {
        return curr;
    }

}
{% endhighlight %}

Мы получили класс, который корректно работает в многопоточной среде, затратив при этом минимум усилий. В первую очередь мы жертвуем производительностью. Производительность класса такая же, как если бы он запускался в однопоточной среде. К тому же использование локов приносит проблемы такие как: deadlock, livelock и т.п.


Fine-Grained locking
=====

Следующий способ --- оградить локами только те секции, в которых происходит доступ к разделяемому состоянию. Примером такого подхода является СoncurrentHashMap. В нем все данные разбиваются на несколько сегментов. И при доступе блокируется только тот сегмент, в котором происходит обращение к данным. Также есть вариант использовать более функциональные локи, такие как: StampedLock, ReadWriteLock. Пример с использованием ReadWriteLock:

{% highlight java %}
public class FineGrainedLock implements FibonacciGenerator<BigInteger> {

    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private BigInteger curr = BigInteger.ONE;
    private BigInteger next = BigInteger.ONE;

    @Override
    public BigInteger next() {
        BigInteger result;
        lock.writeLock().lock();
        try {
            // Вход другим запрещен
            result = curr;
            curr = next;
            next = result.add(next);
            return result;
        } finally {
            lock.writeLock().unlock();
        }
    }

    @Override
    public BigInteger val() {
        lock.readLock().lock();
        try {
            // При отпущенном write lock
            // Допуст`им вход множества потоков
            return curr;
        } finally {
            lock.readLock().unlock();
        }
    }
}
{% endhighlight %}

Мы усовершенствовали класс и добились более высокой пропускной способности на чтение. У такой реализации есть все минусы локов. К тому же из минусов добавилось то, что алгоритм усложнился (добавилось шума, не связанного с логикой работы) и вероятность некорректной реализации возросла.


Lock-free algorithms
=====

Использование локов влечет массу проблем с производительностью и корректностью. Существует альтернатива в виде неблокирующих алгоритмов. Такие алгоритмы построены на атомарных операциях, предоставляемых процессорами. Примером служит метод get в ConcurrentHashMap. Чтобы писать неблокирующие алгоритмы имеет смысл воспользоваться существующими неблокирующими классами: ConcurrentLinkedQueue, ConcurrentHashMap и т.п. Напишем неблокирующую реализацию нашего класса.

{% highlight java %}
public class LockFree implements FibonacciGenerator<BigInteger> {

    // Инкапсулируем состояние генератора в класс
    private static class State {
        final BigInteger next;
        final BigInteger curr;

        public State(BigInteger curr, BigInteger next) {
            this.next = next;
            this.curr = curr;
        }
    }

    // Сделаем состояние класса атомарным
    private final AtomicReference<State> atomicState = new AtomicReference<>(
            new State(BigInteger.ONE, BigInteger.ONE));

    public BigInteger next() {
        BigInteger value = null;
        while (true) { 
            // сохраняем состояние класса 
            State state = atomicState.get();
            // то что возвращаем если удалось изменить состояние класса
            value = state.curr; 
            // конструируем новое состояние
            State newState = new State(state.next, state.curr.add(state.next));
            // если за время пока мы конструировали новое состояние
            // оно осталось прежним, то заменяем состояние на новое
            // иначе пробуем сконструировать еще раз
            if (atomicState.compareAndSet(state, newState)) {break;}
        }
        return value;
    }

    @Override
    public BigInteger val() {
        return atomicState.get().curr;
    }
}
{% endhighlight %}

Из плюсов такого подхода следует отметить увеличение производительности (по крайней мере для данного класса и при определенных параметрах). А также что не менее важно избавление от болезней локов. Минус в том, что неблокирующий придумать гораздо сложнее. 


Software Transactional Memory
=====

Альтернативой неблокирующим алгоритмам является применение программной транзакционной памяти. Её использование похоже на использование транзакций при работе с базами данных. Концепция довольно таки новая (1995) и пока что нативная поддержка есть только в Clojure. Поддержка STM на уровне библиотек есть во многих языках, в том числе и Java. Я буду использовать Akka STM.

{% highlight java %}
public class STM implements FibonacciGenerator<BigInteger> {

    // Оборачиваем переменные в транзакционные ссылки для участия в транзакциях
    private final Ref<BigInteger> curr = new Ref<>(BigInteger.ONE);
    private final Ref<BigInteger> next = new Ref<>(BigInteger.ONE);

    @Override
    public BigInteger next() {
        // Создаем и выполняем транзакцию
        return new Atomic<BigInteger>() {
            @Override
            public BigInteger atomically() {
                // Производим вычисления не думая о многопоточности
                // STM все сделает за нас
                BigInteger result = curr.get();
                curr.set(next.get());
                next.set(result.add(next.get()));
                return result;
            }
        }.execute();
    }

    @Override
    public BigInteger val() {
        // Транзакция создается неявно
        return curr.get();
    }

}
{% endhighlight %}

Код легко понимается и нет шума, создаваемого неблокирующими алгоритмами и использованием локов. Но за это мы платим более низкой производительностью.


Immutability
=====

Проблемы с общим доступом к состоянию от того, что оно изменяемое. То есть сделав класс неизменяемым, можно без ограничений работать с ним в многопоточной среде, так как он также будет потоко-безопасным. Но неизменяемые структуры данных требуют соответствующих подходов и алгоритмов, так как расходы на память могут быть очень велики. 

{% highlight java %}
public class Immutable {

    private final BigInteger next;
    // Текущее значение
    public final BigInteger val;

    private Immutable(BigInteger next, BigInteger val) {
        this.next = next;
        this.val = val;
    }

    public Immutable next() {
        return new Immutable(val.add(next), next);
    }

    public static Immutable first() {
        return new Immutable(BigInteger.ONE, BigInteger.ONE);
    }

}
{% endhighlight %}

Как вы заметили, интерфейс класса изменился и это потребует других способов работы с данным классом. 

Итог
===

У каждого подхода есть свои минусы и плюсы и нельзя дать универсального совета. Надеюсь, что при следующем проектировании системы, модуля или класса, вы вспомните подходы, описанные в данной статье, и выберите подходящий.


