---
layout: post
title: "Непрерывное слежение за ротационными логами"
category : Linux
---

Есть стандартная ситуация, когда нужно смотреть логи в реалтайм. Есть конечно много способов делать это: `tail -f имя_файла` или `less имя_файла`, а затем `shift+f`. Второй вариант я не рассматриваю, поскольку не очень люблю less (или пока не полюбил (_UPD: уже полюбил_)).Ротация логов - это механизм позволяющий управлять архивацией логов (далее в википедию). Существует несколько вариантов ротации, нам интересен тот, при котором лог хранится в нескольких файлах и они ротируются при достижении размера в 5 МБ:

	1.1M Aug 15 14:48 server.log
	5.1M Aug 15 14:46 server.log.1
	5.2M Aug 15 14:39 server.log.2
	5.2M Aug 15 14:28 server.log.3
	5.2M Aug 15 14:15 server.log.4
	5.1M Aug 15 14:01 server.log.5

Этот пример взят с сервера с тестовом окружением, на котором пишется много логов. Частота ротации в районе 13 минут в разгар рабочего дня. Мы хотим быть в курсе последних новостей и поэтому следим за `server.log`: `tail -f server.log`. Но проблема в том, что в таком случае `tail` следит не за именем файла, а за его дескриптором, а в силу того, что после каждой ротации `server.log` создаётся заново, то мы начинаем следить за старым файлом. На помощь приходит опция `--follow`, а если быть точнее, то `-F (--follow --retry)`, которая следит за именем файла, а не дескриптора. Вот выдержка из мануала по tail:

>	With --follow (-f), tail defaults to following the file descriptor, which means that even if a tail’ed file is renamed, tail will continue to track its end.  This default behavior  is not desirable when you really want to track the actual name of the file, not the file descriptor (e.g., log rotation).  Use --follow=name in that case.  That causes tail to track the named file by reopening it periodically to see if it has been removed and recreated by some other program.

Так что теперь все используем `tail -F`. 




