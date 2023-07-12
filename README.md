## Урок 2. Механизмы контрольных групп

### **Информация о проекте**

Задание 1:
1) запустить контейнер с ubuntu, используя механизм LXC;
2) ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает;
3) добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно;
4) при создании указать файл, куда записывать логи;
5) после перезагрузки проанализировать логи.

**Задание**

* Установим LXC и шаблоны:
```
sudo apt install lxc lxc-templates uidmap
```

Затем командой "$ lxc version" проверяем уствновленную версию. Видим как система подсказывает, что для начала необходимо инициализировать LXD на машине. Вводим команду:
```
sudo apt install lxd-installer
```

После инициализации снова убеждаемся, что все нормально установлено, путем проверки версии LXC:
```
lxc version
```

![lxc version](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/1.PNG)

* Следующей командой создаем новый контейнер с именем testOne и задаем путь для файла конфигурации:
```
lxc-create -n testOne -t ubuntu -f /usr/share/doc/lxc/lxc-veth.conf 
```

Видим большое количество текста, значит команда введена правильно. В тексте указана ошибка открытия файла конфигурации указанного нами. На запуск контейнера это не повлияет, а случается из=за того, что такого файла на данном этапе не существует.

![lxc create](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/2.PNG)


После того, как контейнер установился мы видим следующий текст:

![lxc creates](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/3.PNG)

Система оповещает нас о том, что контейнер создан, у него заданы стандартные параметры (логин: ubuntu, пароль: ubuntu).


* Теперь необходимо запустить созданный нами контейнер. Для этого вводим следующую команду:
```
sudo lxc-start -d -n testOne
```

![lxc-start](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/4.PNG)

Если ситема никак не оповестила нас ни о чем, значит команда успешно выполнена. Проверить это можно, например, путем входа в контейнер:
```
sudo lxc-attach -n testOne
```

Видим, что мы зашли в testOne под root.

![lxc-attach](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/5.PNG)


* Вводим следующую команду для просмотра выделенной и свободной памяти:
```
free -m
```

![free -m](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/6.PNG)


Для удобства сразу перейдем в нужную папку:
```
cd sys/fs/cgroup
```

Считываем данные из файла, в котором уграничивается потребление памяти контейнером:
```
cat memory.max
```
Здесь мы можем наблюдать, что ограничение памяти установлено на максимальное значение.

![cat memory.max](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/7.PNG)

Далее выполним тоже самое из папки .lxc:

![cat memory.max](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/8.PNG)

Убеждаемся в максимальном значении. Выходим из контейнера:
```
exit
```


Для того, что бы ограничить потребление памяти контейнером, необходимо добавить строку в файл конфигурации. Для начала считаем информацию из него и убедимся, что ограничений там нет командой:
```
sudo cat /var/lib/lxc/testOne/config
```

![cat memory.max](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/9.PNG)


После этого любым удобным редактором (я делал через редактор nano) добавляем в этот файл конфигурации вниз текста следующую строку:

```
lxc.cgroup2.memory.max = 256M
```

![memory.max = 256M](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/10.PNG)


Таким образом мы ограничили потребление памяти на более 256 мб.
Останавливаем контейнер и снова запускаем его.
Вводим следующую команду, что бы убедиться, что ограничение работает:
```
sudo cat /sys/fs/cgroup/lxc.payload.testOne/mempry.max
```

![memory.max = 256 mb](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/11.PNG)

Как мы видим, система пересчитала и выдала результат, где много цифр. Не пугаемся, это значение примерно равно 256 мб.


* Далее просмотрим список имеющихся в нашей системе контейнеров:
```
sudo lxc-ls -f
```
 У нас имеется один контейнер testOne, он запущен и в автозапуске стоит 0 (что означает, что автозапуск контейнера отключен).
 Нам необходимо включить автозапуск, для этого октрываем файл конфигурации в редакторе nano:
```
sudo nano /var/lib/lxc/testOne/config
```

![sudo nano /var/lib/lxc/testOne/config](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/12.PNG)

Добавляем в конец текста файла конфигурации следующую строку:
```
lxc.start.auto = 1
```

Сохраняем, закрываем редактор и перезагружаем систему, например командой '$ reboot'.

![reboot](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/13.PNG)


После перезагрузки снова выполняем команду:

```
sudo lxc-ls -f
```

Вводим пароль root и убеждаемся, что контейнер запущен и автозапуск включен.

![autorun](https://github.com/Mihon99/Containerization-Seminar_2/blob/main/source/14.PNG)


* При запуске контейнера можно в параметрах команды указать, где хранить логи, например:
```
lxc-start -n testOne --logfile log.log
```


* Для удаления контейнера необходмио воспользоваться следующей командой:
```
lxc-destroy -n testOne
```