---
layout: post
title: "Redmine Development Environment"
description: "Installation guide"
category : Programming
---

Доброе время суток! Суббота, болею, изучаю Ruby on Rails. Решил написать, то как правильно готовить окружение для разработки.
В своё время мне не хватало такого туториала.  Всё достаточно легко, если у вас Linux или MacOS. И так, начнём.

Давайте поставим Redmine (<http://www.redmine.org>).

{% highlight bash %}
$ git clone https://github.com/edavis10/redmine.git
$ cd redmine
$ git checkout -b 1.3.0 1.3.0
{% endhighlight %}

Рулить будем при помощи rvm (<https://rvm.beginrescueend.com>).

{% highlight bash %}
$ rvm install 1.8.7
$ rvm use 1.8.7
$ rvm gemset create redmine-1.3
$ rvm gemset use redmine-1.3
$ vim .rvmrc
	rvm use 1.8.7@redmine-1.3
$ rvm rvmrc trust
{% endhighlight %}

Заходим на <http://www.redmine.org/projects/redmine/wiki/RedmineInstall> и смотрим на зависимости.

{% highlight bash %}
$ gem install rake -v=0.8.7
$ gem install rails -v=2.3.14
$ gem install i18n -v=0.4.2
$ gem install sqlite3
{% endhighlight %}

И еще парочку, которые забыли упомянуть:

{% highlight bash %}
$ gem install rubytree
$ gem install tree
$ gem install coderay
{% endhighlight %}

Какие то проблемы с rubygems >= 1.8, поэтому удаляем строку:

{% highlight bash %}
$ vim config/environment.rb
       config.gem 'rubytree', :lib => 'tree'
{% endhighlight %}

Указываем, что будем использовать sqlite

{% highlight bash %}
$ vim config/database.yml
       development:
           adapter: sqlite3
           database: db/development.sqlite3
{% endhighlight %}

И финальный шаг: генерируем ключи и делаем миграцию

{% highlight bash %}
$ RAILS_ENV=development rake config/initializers/session_store.rb
$ RAILS_ENV=development rake db:migrate
$ RAILS_ENV=development rake redmine:load_default_data
{% endhighlight %}

И запускаем:
{% highlight bash %}
$  RAILS_ENV=development ./script/server
{% endhighlight %}

Заходим в redmine по адресу <http://localhost:3000/>, пароль admin/admin

Надеюсь получилось. Советую прочитать по rvm, очень полезная штука, в том случае, когда есть несколько проектов использующих разные версии ruby и gem'ов.