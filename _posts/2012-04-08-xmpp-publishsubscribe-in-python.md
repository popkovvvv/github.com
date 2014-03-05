---
layout: post
title: "XMPP Publish/Subscribe in python"
description: ""
category : Programming
---

Доброго времени суток! В данной статье я расскажу о том, как сделать базовую реализацию <a
href="http://en.wikipedia.org/wiki/Publish/subscribe">publish/subscribe</a> с помощью протокола XMPP, используя <a
href="http://twistedmatrix.com/">Twisted</a> и библиотеку <a href="http://wokkel.ik.nu/">Wokkel</a>. XMPP поддерживает
pub/sub благодаря расширению <a href="http://xmpp.org/extensions/xep-0060.html">XEP-0060</a>. Используя pub/sub, можно
решить задачу уведомления всех участников о событии и множество других. Достоверно известно, что Apple использует
основанный на Wokkel'е pub/sub внутри своего notification server'a, но об этом чуть позже.
<habracut />

<h4>Зачем нужен publish/subscribe?</h4>
При разработке проекта возникла следующая задача: построить клиент-серверную архитектуру, в которой клиенты производят
действие с данными, хранящимися на сервере. Данные находятся в общем доступе и необходимо, чтобы клиент производил
действия над новейшей версией данных. Немного объясню как система должна работать. Клиент подписывается на сервис. При
подписке на сервис клиент получает все данные хранящиеся на сервере. Клиент посылает данные на сервер. Сервер сохраняет
присланные данные и пересылает их всем подписчикам. Если подписчик обнаруживает, что целостность данных нарушена, он
запрашивает сервер переслать данные и т.д.

<h4>Инструменты</h4>
Решение было найдено в виде использования XMPP и его расширения XEP-0060. XMPP был выбран в силу своей популярности и
богатых возможностей. В свою очередь Twisted был выбран, как один из самых мощных сетевых framework'ов на python. В
twisted.words есть некоторая базовая поддержка xmpp протокола, но к сожалению, нет поддержки расширения pub/sub. В
качестве библиотеки, добавляющей поддержку pub/sub к Twisted, был взят Wokkel. Wokkel поддерживает не только pub/sub,
но и (<a href="http://xmpp.org/extensions/xep-0030.html">service discovery</a>) и пару других полезных расширений.

<h4>Пример</h4>
Попробуем написать небольшой пример, показывающий то, как работать с библиотекой Wokkel. У нас будет сервис и клиенты,
подписанные на обновления от сервиса. Каждые 5 секунды, клиент генерирует сообщение и отсылает его серверу. Сервер же
отсылает сообщение всем подписанным участникам. Приступим.

Для начала следует создать объект класса XMPPClient, являющийся сервисом (в терминологии Twisted). Он отвечает за
авторизацию на Jabber сервере и управление соединением. Объект класса PubSubClient или PubSubService обеспечит нам
возможность взаимодействия на уровне протокола pub/sub. Вызываем setHandlerParent, чтобы протокол и сервис смогли
взаимодействовать между собой. Запускаем сервис. Протокол начинает слушает поток на предмет сообщений относящихся к
pub/sub, при получении вышеназванных сообщений вызывает соответствующие обработчики. Следующим шагом мы рассмотрим
реализацию класса Service.

{% highlight python %}
def main():
    log.startLogging(sys.stdout)
    options, args = parse_args()
    jid, resource, password = args
    service = options.service
    fulljid = jabber.jid.internJID(jid + '/' + resource)
    transport = XMPPClient(fulljid, password)
    transport.logTraffic = True
    protocol = Client(jabber.jid.internJID(service)) if service else Service()
    protocol.setHandlerParent(transport)
    transport.startService()
    reactor.run()
{% endhighlight %}

В нашем случае задача pub/sub сервиса проста: принять сообщение от подписчика и разослать его всем другим подписчикам.
Клиент начинает взаимодействие с сервисом с того, что он посылает запрос на подписку. Как только сервис распознает, что
пришёл subscribe запрос, он вызывает метод subscribe. В данном методе мы должны создать связанную с клиентом подписку и
вернуть её callback'ом. Мы не будем реализовывать модели доступа и считаем, что все могут подписаться. Когда подписчик
отошлёт нам некоторые данные, вызовется метод publish. В методе sendData мы пересылаем пришедшие данные всем, кто
подписан на наш сервис. Теперь перейдём к рассмотрению клиента.

{% highlight python %}
class Service(PubSubService):
    def __init__(self):
        PubSubService.__init__(self)
        self._subscriptions = {}

    def publish(self, requestor, service, nodeIdentifier, items):
        self.sendData(items)
        return defer.succeed(None)

    def subscribe(self, requestor, service, nodeIdentifier, subscriber):
        if subscriber in self._subscriptions:
            info = self._subscriptions[subscriber]
        else:
            info = Subscription(NODE_NAME, subscriber, 'subscribed')
            self._subscriptions[subscriber] = info
        return defer.succeed(info)

    def sendData(self, items):
        sendList = []
        for subscription in self._subscriptions.values():
            sendList.append([subscription.subscriber, None, items])
        self.notifyPublish(self.parent.jid, NODE_NAME, sendList)
{% endhighlight %}

При создании клиента мы передаём ему адрес сервиса, к которому он будет подписываться. Когда сервис(XMPPClient)
авторизовался у jabber сервера и связь установилась, вызывается метод connectionInitialized у всех протоколов
присоединенных к сервису(XMPPClient). Мы сделаем так, чтобы сразу же после установки соединения происходила подписка на
нужный нам сервис. После того, как подписка удачно завершилась, мы стартуем событие, которое будет происходить каждые 4
секунды и будет генерировать и отсылать сообщение на сервис. Пересылаемые данные должны быть в правильно оформленном
XML. Собственно и всё.

{% highlight python %}
class Client(PubSubClient):
    def __init__(self, service):
        PubSubClient.__init__(self)
        self.__service = service

    def connectionInitialized(self):
        PubSubClient.connectionInitialized(self)
        d = self.subscribe(self.__service, NODE_NAME, self.parent.jid)
        d.addCallback(lambda success: task.LoopingCall(sendGeneratedData, self))
        d.addCallback(lambda lc: lc.start(5, now = True))

    def itemsReceived(self, event):
        for item in event.items:
            log.msg(item.getAttribute('sender') + ' sends ' +
                    item.getAttribute('message') + ' at ' + item.getAttribute('time'))

    def sendData(self, items):
        self.publish(self.__service, NODE_NAME, items)

def sendGeneratedData(protocol):
    element = Element((None, 'item'))
    element.attributes['sender'] = protocol.parent.jid.full()
    element.attributes['time'] = time.strftime("%H:%M:%S", time.localtime())
    element.attributes['message'] = 'Hello!'
        protocol.sendData([element])
{% endhighlight %}

Просмотреть исходный код можно <a href="http://pastebin.com/5WSMJZUU">здесь</a>
Чтобы проверить работу, нужно:
Запустить сервер:
{% highlight sh %}
pubsub.py test@example.com server password
{% endhighlight %}
Запустить двух клиентов:
{% highlight sh %}
pubsub.py --service test@example.com/server test@example.com client1 password
pubsub.py --service test@example.com/server test@example.com client2 password
{% endhighlight %}

<h4>При чем здесь Apple?</h4>
То что мы сейчас сделали - это пример, показывающий одну из сотни возможностей pub/sub. Если обратиться к спецификации,
то можно увидеть, что наш пример соответствует типу узла Leaf, с конфигурацией persistItems=False deliveryPayloads=True
и открытой моделью доступа, без поддержки affiliations, discovery, возможности создания других узлов и т.д. Представьте,
как трудно реализовать всю спецификацию pub/sub или даже необходимую часть. Но <a href="http://ralphm.net/blog/">Ralph
Meijer</a>, автор wokkel, упростил нам задачу и написал <a href="http://idavoll.ik.nu/">Idavoll</a>. Idavoll - это
надстройка над Wokkel. Он почти полностью поддерживает XEP-0060 и имеет возможность взаимодействия по http. Idavoll
используется Apple в качестве notification server'a. Данный факт можно проверить <a
href="http://www.apple.com/opensource/">здесь</a>

<h4>Заключение</h4>
Для изучения Twisted рекомендую официальную документацию и http://krondo.com/?page_id=1327
К сожалению, по Wokkel очень мало информации, а по Idavoll и вовсе нет, поэтому остаётся только сгенерированная
документация.
Успехов в изучении!
