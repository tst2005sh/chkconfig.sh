chkconfig
=========

I like the the original chkconfig script, made in perl, because it was easy to use.


Enable a service
----------------

```sh
chkconfig apache2 on
```

Disable a service
-----------------

```sh
chkconfig apache2 off
```

List all services
-----------------

```sh
chkconfig -l
```

List on service
---------------

```sh
chkconfig -l |grep apach
```
or
```sh
chkconfig -l apache2
```
or
```sh
chkconfig apache2
```


What is implemented
===================

* service on/off
* list all service (-l|--list)
* list one service


What is not implemented
=======================

* the inetd and xinetd support
* some more complexe options


License
=======

Free software licensed under MIT License
