---
layout: post
title: "Server tools and configuration"
tags : [programming]
---

У меня есть небольшой сервер, на котором установлено много всяких программ. 
Легко забыть что там установлено и как это было настроено. Здесь описана конфигурация сервера.
Информация предназначена для меня, но может кому-то захочется меня взломать.

## Services

- serviio
	- web interface
	- ffmpeg from repo
	- media browser(forwarding)
	- plugins
- chkrootkit
	- cron
	- email notifications
- logcheck
	- cron
	- custom ignore rules
	- email notifications
- rkhunter
	- cron
	- email notifications
- transmission 
	- web interface
	- blocklist
	- upnp
- torrentmonitor
	- web interface via lighttpd
	- custom cron
	- email notifications
- i2p
	- web interface
	- proxy
	- upnp
- privoxy
	- web interface
	- i2p proxy
	- tor proxy
- tor
	- relay(forwarding)

## Server and router

- disabled ipv6 (due to conflict with i2p)
- noexec on /home and usb drives
- group "storage" for data
- llvm
- blocked tv upnp
- ddns
- syslogd for router (+logrotate)


	

