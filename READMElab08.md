# lab08
## Laboratory work VIII
[![Build Status](https://travis-ci.com/navckin/lab06.svg?branch=main)](https://travis-ci.com/navckin/lab06)
Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [+] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [+] 2. Ознакомиться со ссылками учебного материала
- [+] 3. Выполнить инструкцию учебного материала
- [+] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
1. Присваиваем <имя_пользователя> в переменную GITHUB_USERNAME. Команда export - предназначена для экспорта переменных и функций текущего процесса в дочерний процесс.

```sh
$ export GITHUB_USERNAME=<имя_пользователя> 
```
2. Переходим в репозиторий workspace
   Добавляем в стек текущий каталог. Команда  pushd . - для упрощения переходов между каталогами файловой системы. Используется для запоминания текущего каталога в виртуальном стеке каталогов и переходу в указанный параметром командной строки.
   Выполняем скрипт
```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
3. Копируем репозиторий лаб07 в папку лаб08
   Переходим в дерикторию лаб08
   Клонируем нужные версии сабмодулей
   Удаляем старую ссылку репозитория
   Добавляем ссылку репозитория в управление репозиториями
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```
4.  В файле Dockerfile указывается опрец система, где будем работать, и ее версия.
    Dockerfile -это некий доп. файл, в котором содержится инструкция для заполнения докер-контейнера.
 Сам Docker необходим для программы и скрипта, проекта, чтобы создавать не вдаваясь в подробности, на какой вм они открываются
(те докер - это как вм, только управляется не мышкой, а командами)

```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
5. 1). задаем любые команды, которые мы могли бы записать в баше, конкретно сейчас : обновить систему установки программ
   2). эту строчку трактуем так же, как и если бы писали ее в консоли линукс, те мы устанавливаем пакеты (три штуки)
-yy- говорит о том, что в процессе установки на все вопросы мы отвечаем yes

```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
6. Копируем то, что есть в тек. каталоге в принт и объявлем принт. (те есть все, что мы перенесли в принт- мы объявляем текущим каталогом)
```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
7. RUN'ы для тестирования сборки и инсталляции проекта
(по сути это то, что уже сделано в пр-их работах)

```sh
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
8. env - некая команда, которая позволяет задать какую-то переменную. мы создаем переменную LOG_PATH(ее назавнаие),а ее значение это (путь..) /home/logs/log.txt
примерно то же самое, если бы сделали export

```sh
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```
9. создаем доп.каталог для logs
```sh
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```
10. создаем WORKDIR: он после всего, что сделает с симэйком, перейдаст в _install/bin
```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
11. эта команда вызывает утилиту demo, что была создана в предыдущих лабораторных. 

```sh
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```
Кратко о сделанном: мы создали весь файл, который явл. скриптом в своем собственном формате. Можем вызывать саму утилиту докер, которая будет создавать машину, которая будет "для нас" работать.
12. вызываем утилиту docker. -t это тег. тег - logger
```sh
$ docker build -t logger .
```
13. позволяет вывести (просмотреть) список, тех  ВМ, что есть

```sh
$ docker images
```
14.  Мы находимся сейчас в раб папке,где мы можем вызывать докер. Мы хотим развернуть ВМ и для этого мы 
создаем logs. Затем: делаем docker run - запуск машины, -it - подключаем к машине терминал. после этого нужно подключить каталог, что мы создали внутрь машины. Испольузем -v для этого. logger - это имя машины, которую мы назвали в докер билт.
  Должна быть выведена работа утилиты demo. Все, что проспиано в качестве команд будет выведено на экран.

  
```sh 
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
<C-D>
```
15. docker inspect logger - показывает, что происходит с машиной, оно не информативное, для более сложного управления.
  (говорит упала не упала и тд) 

```sh
$ docker inspect logger
```
16. cat logs/log.txt позволяет посмотреть, что утилита вывела в лог
```sh
$ cat logs/log.txt
```
17. Заменяем README.md 
```sh
$ gsed -i 's/lab07/lab08/g' README.md
```
18. У ВМ-ы есть командыный режим, когда в клавиши вкладываем какие-то комады. это режим по умочанию. Основные команды ввода в текста: i a I o O;
```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC> //  сама клавиша
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
18. Выполянем заполнение репозитория: добавляем Dockerfile, .travis.yml. Выполняем коминт и пуш.
```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
19. Нужно сделать Docker работающим внутри travis'а. Выполянем это.
```sh
$ travis login --auto
$ travis enable
```
*если есть синтаксические ошибки - выдаст ошибки в самом билде.


```

## Links

- [Book](https://www.dockerbook.com)
- [Instructions](https://docs.docker.com/engine/reference/builder/)

```
Copyright (c) 2015-2021 The ISC Authors
```
