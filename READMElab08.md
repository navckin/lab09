## Laboratory work IX

Данная лабораторная работа посвещена изучению процесса создания артефактов на примере **Github Release**

```sh
$ open https://help.github.com/articles/creating-releases/
```
Возникли такие проблемы: 
не было команды go ---> полезная инф. была найдена на таком сайте: https://www.digitalocean.com/community/tutorials/how-to-install-go-and-set-up-a-local-programming-environment-on-ubuntu-18-04-ru
не работала команда гитхаб-релиз ---> https://github.com/github-release/github-release 
## Tasks

- [х] 1. Создать публичный репозиторий с названием **lab09** на сервисе **GitHub**
- [х]  2. Ознакомиться со ссылками учебного материала
- [х] 3. Получить токен для доступа к репозиториям сервиса **GitHub**
- [х]  4. Выполнить инструкцию учебного материала
- [х] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_TOKEN=<полученный_токен>         #присваиваем сгенирированный токен в переменную GITHUB_TOKEN
$ export GITHUB_USERNAME=<имя_пользователя>     #присваиваем имя пользователя GitHub в переменную GITHUB_USERNAME
$ export PACKAGE_MANAGER=<пакетный менеджер>   #указываем используемый пакетный менеджер
$ export GPG_PACKAGE_NAME=<gpg2|gpg>          #указываем, в какой утилите будет создаваться ключ
```

```sh
# for *-nix system
$ $PACKAGE_MANAGER install xclip                     #скачиваем утилиту xclip
Updating Homebrew...
==> Auto-updated Homebrew!
....................................
$ alias gsed=sed                                    #заменяем команду sed на gsed
$ alias pbcopy='xclip -selection clipboard'        #заменяем команду pbcopy на xclip -selection clipboard
$ alias pbpaste='xclip -selection clipboard -o'   #заменяем команду pbpaste на xclip -selection clipboard -o
```

```sh
$ cd ${GITHUB_USERNAME}/workspace                #спускаемся в workspace
$ pushd .                                       #добавляем в стек текущий каталог
$ source scripts/activate                      #выполняем скрипт
$ go get github.com/aktau/github-release      #скачивание и установка пакета Go, для работы с релизами Github
go: downloading github.com/aktau/github-release v0.10.0
go: downloading github.com/dustin/go-humanize v1.0.0
go: downloading github.com/voxelbrain/goptions v0.0.0-20180630082107-58cddc247ea2
go: downloading github.com/github-release/github-release v0.10.0
go: downloading github.com/tomnomnom/linkheader v0.0.0-20180905144013-02ca5825eb80
go: downloading github.com/kevinburke/rest v0.0.0-20210506044642-5611499aa33c
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab081 projects/lab09   #клонируем репозиторий из lab08 в директорию projects/lab09
$ cd projects/lab09                                                     #переходим директорию projects/lab09
$ git remote remove origin                                            #удаляем старую ссылку репозитория
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab09  #добавляем ссылку репозитория в управление репозиториями
```

```sh
$ gsed -i -e 's/lab081/lab09/g' README.md                #поменяем в README.md все строки lab08 на lab09
```

```sh
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}         #устанавливаем программу GPG для шифрования информации и создания электронных цифровых подписей
==> Downloading https://ghcr.io/v2/homebrew/core/gmp/manifests/6.2.1
######################################################################## 100.0%
==> Downloading ................................................................
$ gpg --list-secret-keys --keyid-format LONG           #проверяем на наличие ранее созданных ключей
gpg: создан каталог '/Users/user/.gnupg'
gpg: создан щит с ключами '/Users/user/.gnupg/pubring.kbx'
gpg: /Users/user/.gnupg/trustdb.gpg: создана таблица доверия
$ gpg --full-generate-key                                 #гененрируем ключ
$ gpg --list-secret-keys --keyid-format LONG     #выводим в консоль сгенерированный ключ
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') # cоздаем переменной окружения с сохраненным в ней публичным ключом
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') #cоздаем переменной окружения с сохраненным в ней секретным ключом
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy  #выводим ключ в ASCII
$ pbpaste                                    #копируем ключ в буфер
$ open https://github.com/settings/keys         #открываем Github ключ
$ git config user.signingkey ${GPG_SEC_KEY_ID} #добавляем секретный ключ в Github
$ git config gpg.program gpg
```

```sh
# Настраиваем скрипт для добавления сообщения к тегу
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"    #генерируем пакет с раширением .tar.gz
$ cmake --build _build --target package        #запускаем сборку package
```

```sh
$ travis login --auto         #авторизируемся на travis
$ travis enable              #делаем проект доступным
```

```sh
$ git tag -s v0.1.0.0                   #создаем тега с сообщением с информацией
$ git tag -v v0.1.0.0                  #верифицируем тег
$ git show v0.1.0.0                   #посмотрим изменения
$ git push origin master --tags      #отправляем измения на Github
```

```sh
$ github-release --version              #узнаем версию
github-release v0.10.0
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ github-release release \              #создаем релиза
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```

```sh
#Добавим артефакт с указанием ОС и архитектуры, на которых происходила компиляция библиотеки
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m`        #
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz       #
$ github-release upload \       #
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09 
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}  #скачаем артефакт
$ tar -ztf ${PACKAGE_FILENAME}                                                                    #выводим на экран содержимого архива
```
