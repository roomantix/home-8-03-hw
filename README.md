# Домашнее задание к занятию "`GitLab`" - `Романов Алексей`



### Задание 1

```Что нужно сделать:

1. Разверните GitLab локально, используя Vagrantfile и инструкцию, описанные в этом репозитории.
2. Создайте новый проект и пустой репозиторий в нём.
3. Зарегистрируйте gitlab-runner для этого проекта и запустите его в режиме Docker. Раннер можно регистрировать и запускать на той же виртуальной машине, на которой запущен GitLab.
4. В качестве ответа в репозиторий шаблона с решением добавьте скриншоты с настройками раннера в проекте.```


```
Ответ
1. Установка  Vagrant - wget https://hashicorp-releases.yandexcloud.net/vagrant/2.4.9/vagrant_2.4.9-1_amd64.deb &&&
sudo dpkg -i vagrant_2.3.5-1_amd64.deb

2. Далее установил ВиртуалБокс т.к. я начал делать это все на виртуальной машине  и я даже не знал о том что нужно было делать на домашнем пк - sudo apt update && sudo apt install virtualbox virtualbox-ext-pack , после установки произошла ошибка , скриншот 1

3. Далее я установил все на той же машине , код ниже.

```
Поле для вставки кода...
Установка Docker , GitLab, gitlab-runner

apt-get install -y docker.io docker-compose

apt-get install -y curl openssh-server ca-certificates tzdata perl postfix

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

Локальный домен
ip a
echo '192.168.1.9    gitlab.localdomain gitlab' >> /etc/hosts

sudo EXTERNAL_URL="http://gitlab.localdomain" apt-get install gitlab-ee

Установка Докер
1.
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

2.
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
3.
sudo apt update
sudo apt install docker-ce

Установка гитлаб раннер
1.
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
2.
sudo apt install gitlab-runner
3.
Скачиваем образы заранее

docker pull gitlab/gitlab-runner:latest
docker pull sonarsource/sonar-scanner-cli:latest
docker pull golang:1.17
docker pull docker:latest

4.
Узнаем пароль
cat /etc/gitlab/initial_root_password
и заходим в наш новый проект, там в CI/CD находим раннер и нажимаем регистрация

5.
Регистрация

docker run -ti --rm --name gitlab-runner \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest register
```
---

`Скриншот-1 к Установить Gitlab с помощью Vagrant:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/1.png)

`Скриншот-2 к Зарегестрировать Раннер:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/2.png)




### Задание 2

Что нужно сделать:
1. Запушьте репозиторий на GitLab, изменив origin. Это изучалось на занятии по Git.
2. Создайте .gitlab-ci.yml, описав в нём все необходимые, на ваш взгляд, этапы.
В качестве ответа в шаблон с решением добавьте:

файл gitlab-ci.yml для своего проекта или вставьте код в соответствующее поле в шаблоне;
скриншоты с успешно собранными сборками.


```
Ответ
Задание 2 
Пункт 1
git remote -v
И тут оказалось что у меня уже подключен мой репозиторий в который я пишу задания

Я создал отдельно папку , защел внутрь и склонировал 
git clone https://github.com/netology-code/sdvps-materials.git
Исправил на нужный
git remote set-url origin http://gitlab.localdomain/aleksey/8-3.git

git fetch origin

git merge origin/main --allow-unrelated-histories

git add .
git commit -m "Initial commit"

git push origin main
Готово
Задание 2
Пункт 2
К сожалению я не смог запустить сборку , приложу скриншоты ошибок, так и не смог понять в чем причина данной ошибки, приложил скриншоты после расшифровки файла.

После того как дали ответ по исправлению я начал искать ошибки.
Решение проблем:

Первое что у меня было не правильно , первое что сделал удалил сам руннер 
docker rm gitlab-runner - дело именно в нем.
Затем попытался его заного зарегестрировать и наткнулся на вот такую ошибку.

PANIC: decoding configuration file: toml: line 9 (last key "runners.name"): invalid UTF-8 byte: 0xd1 
Я посмотрел кодировку
file -i /etc/gitlab-runner/config.toml

Решил прост удалить сам файл , перед этим я его скопировал на всякий случай.
После этого я его заного зарегистрировал и запустил.
docker run -d --name gitlab-runner --restart always \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest

Запуск

docker run -d --name gitlab-runner --restart always \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest


После этого, у меня не уходила ошибка в самом гитлабе.
Я посмотрел статус руннера
sudo gitlab-runner status

Runtime platform                                    arch=amd64 os=linux pid=706743 revision=139a0ac0 version=18.4.0
gitlab-runner: Service has stopped

Оказалось что он отключен , я его включил
sudo gitlab-runner start
После этого ошибка не исчезла , почему не понятно.
посмотрев 
cat /etc/hosts
там все вопрядке с айпи адресом.

Я решил прописать явно руннеру что за хостом и дописал вот так

   extra_hosts = ["gitlab.localdomain:192.168.1.9"]

После этого всё заработало, скриншот успешного запуска прилагаю ( Скриншоты 6 - 7)

Описание .gitlab-ci.yml приложил в поле ниже
```


```
Файл .gitlab-ci.yml

- этапы , тестирование , сборка
stages:
  - test
  - build
Тут на этапе проверка происходит проверка есть ли язык го
test:
  stage: test
  image: golang:1.16
  script: 
   - go test .
Тут проверка сонара
sonarqube-check:
 stage: test
 image:
  name: sonarsource/sonar-scanner-cli
  entrypoint: [""]
 variables:
 script:
  - sonar-scanner -Dsonar.projectKey=netology-gitlab -Dsonar.sources=. -Dsonar.host.url=http://gitlab.localdomain:9000 -Dsonar.login=a778675a32f0d9d6455a3d502f4e2838e784994d
- после этого происходит полноценная сборка образ
build:
  stage: build
  image: docker:latest
  only:
    - master
  script:
   - docker build .

build:
  stage: build
  image: docker:latest
  when: manual
  -ветка по которой работает 
  except:
    - master
  script:
   - docker build .
```


Скриншот-3 к заданию 2 пункт 2, ошибка jobs:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/3.png)

Скриншот-4 к заданию 2 пункт 2, docker-compose.yaml:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/4.png)

Скриншот-5 к заданию 2 пункт 2, gitlab-ci:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/5.png)


Скриншот-5 к заданию 2 пункт 2, успешный запуск gitlab-ci:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/6.png)

Скриншот-5 к заданию 2 пункт 2, ошибка до насписания extra_hosts в настройках рунне-а:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/7.png)