---
layout: post
title: "ArrayList vs LinkedList или как я собеседовался"
category : Programming
---

__UPDATE__: В измерениях обнаружились ошибки, внизу [комментарии](#fn:UPDATE)[^UPDATE].


<img class="left" width="300" src="http://habrastorage.org/files/f95/e64/5df/f95e645dfad54ddfa88fbd92b1167afd.jpg"/>

Сегодня я был на техническом собеседовании в одной известной фирме. Сразу скажу, что собеседование мне понравилось. На собеседовании был стандартный вопрос про коллекции, а именно `ArrayList` vs `LinkedList`. Оказывается я не был готов к данному вопросу. 

В вопросе было стандартное начало про сложность вставки, поиска и тому подобное. Но а в конце, был задан интересный вопрос --- "Какую структуру лучше использовать, если мы проходим по коллекции и поочереди обрабатываем каждый элемент? Все что мы делаем --- это просто итерируем с помощью `foreach`. Первая моя мысль про использование памяти, так как [LinkedList потребляет в 6 раз больше памяти](http://www.javaspecialists.eu/archive/Issue029.html). И какой смысл использовать `LinkedList` в случае обычной итерации? Но собеседующий сказал, что память не важна. То есть надо сделать выбор с точки зрения производительности. Я ответил, что можно использовать любую, потому что производительность будет одинаковая. По всей видимости, не этот ответ хотел услышать собеседующий. Возможно моя ошибка была в том, что я не объяснил свою позицию. А позиция была в следующем: по сути мы сравниваем последовательный доступ к памяти и произвольный и возможно еще что-то. Я полагал, что принципиального различия во временной сложности нет. Мы еще немного пообщались на эту тему (я выяснил детали и понимание мной вопроса) и перешли к следующему вопросу. 

После собеседования мне был выслан отзыв и там было написано, что я не знаю про быстродействие `ArrayList` и `LinkedList`. Конечно я расстроился, что не знаю элементарных вещей. Но хорошо, что время есть и можно проверить, в чем я не прав. Берем в руки молоток и начинаем!

У меня уже был некий опыт написания бенчмарков, поэтому я мог быть уверен, что сделаю это правильно. В качестве молотка берем [JMH](http://openjdk.java.net/projects/code-tools/jmh/). Тест я набросал достаточно быстро:


{% highlight java %} 
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 10)
@Measurement(iterations = 10)
@Fork(3)
@State(Scope.Benchmark)
@Threads(1)
public class JmhBench {
 
    private static int ITERATION_SIZE = 1_000_000;
    private ArrayList<String> arrayList;
    private LinkedList<String> linkedList;
 
    @Setup(Level.Iteration)
    public void setup() {
        arrayList = new ArrayList<>(ITERATION_SIZE);
        for (int i = 0; i < ITERATION_SIZE; i++) {
            arrayList.add(String.valueOf(i));
        }
 
        linkedList = new LinkedList<>();
        for (int i = 0; i < ITERATION_SIZE; i++) {
            linkedList.add(String.valueOf(i));
        }
    }
 
    @GenerateMicroBenchmark
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public void arrayList(BlackHole bh) {
        for (String s : arrayList) {
            bh.consume(s);
        }
    }
 
    @GenerateMicroBenchmark
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public void linkedList(BlackHole bh) {
        for (String s : linkedList) {
            bh.consume(s);
        }
    }
 
}
{% endhighlight %} 

Описывать сам JMH я не буду, но скажу, что прежде чем писать тесты, лучше изучить [примеры использования](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/). Замеры я проводил на разных размерах и получилось следующее:

	ITERATION_SIZE = 1000000
	Benchmark                   Mode   Samples         Mean   Mean error    Units
	f.b.JmhBench.arrayList      avgt        30        5.143        0.282    ms/op
	f.b.JmhBench.linkedList     avgt        30        9.577        1.552    ms/op 
	 
	ITERATION_SIZE = 10000
	Benchmark                   Mode   Samples         Mean   Mean error    Units
	f.b.JmhBench.arrayList      avgt        30       33.602        0.534    us/op
	f.b.JmhBench.linkedList     avgt        30       48.543        0.342    us/op
	 
	ITERATION_SIZE = 100
	Benchmark                   Mode   Samples         Mean   Mean error    Units
	f.b.JmhBench.arrayList      avgt        30      355.468        6.428    ns/op
	f.b.JmhBench.linkedList     avgt        30      400.899        6.362    ns/op

Действительно, итерация по `ArrayList` быстрее. Выигрыш с миллиона итераций 4 миллисекунды. Но какой была цель данного вопроса, я не понял. Миллион записей в `LinkedList` бОльшая проблема, чем 4 миллисекунды.

---

[^UPDATE]: Я померял неправильно. Во-первых, время прогрева было недостаточным, Во-вторых, при каждой итерации создавался новый список и GC сходил с ума (для исправления изменил `Level.Iteration` на `Level.Trial`). Это видно по большому Mean Error в `LinkedList` и по логам GC. В-третьих, по какой-то причине использование String затратно и нужно использовать объекты меньшего размера. Есть догадка, что дело в стоимости чтения и фрагментации памяти. Поменял `String` на `Boolean` (Random.nextBoolean) и в итоге получил лучший Mean Error и преимущество `ArrayList` в 2.5 миллисекунды. И самое интересное, что на 10000 и 100 `LinkedList` уже выигрывает.

		ITERATION_SIZE = 1000000
		Benchmark                   Mode   Samples         Mean   Mean error    Units
		f.b.JmhBench.arrayList      avgt        20        2.854        0.009    ms/op
		f.b.JmhBench.linkedList     avgt        20        5.389        0.017    ms/op

		ITERATION_SIZE = 10000
		Benchmark                   Mode   Samples         Mean   Mean error    Units
		f.b.JmhBench.arrayList      avgt        20       31.892        0.160    us/op
		f.b.JmhBench.linkedList     avgt        20       30.322        0.218    us/op

		ITERATION_SIZE = 100
		Benchmark                   Mode   Samples         Mean   Mean error    Units
		f.b.JmhBench.arrayList      avgt        20      304.192        1.184    ns/op
		f.b.JmhBench.linkedList     avgt        20      291.981        4.535    ns/op


