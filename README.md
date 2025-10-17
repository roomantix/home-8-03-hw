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



```
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
К сожалению я не смог запустить сборку локльно , приложу скриншоты ошибок, так и не смог понять в чм причина данной ошибки.
Описание .gitlab-ci.yml приложил в поле ниже



```
Файл .gitlab-ci.yml

- этапы , тестирование , сборка
stages:
  - test
  - build
Тут на этапе проверка происходит есть ли язык го
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

`Скриншот-2 к Установить Gitlab с помощью Vagrant:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/3.png)

`Скриншот-3 к Зарегестрировать Раннер:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/4.png)

`Скриншот-4 к Зарегестрировать Раннер:
![Установка машины с помощью Vargant](https://github.com/roomantix/home-8-03-hw/blob/main/img/5.png)




### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.

---
   ### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

P.S Оставил ля себя

# Как вставить скриншот в шаблон с решением

1. Разместите скриншот в локальном репозитории
2. Вставьте ссылку с указанием файла скриншота в следующем формате:

   `![alt text](https://github.com/username/reponame/blob/branch/path/image.png)`  
   где   
   [username] — ваш ник на Github;  
   [reponame] — название репозитория, в котором хранятся скриншоты;  
   [branch] — ветка репозитория;  
   [path] — путь к месту нахождения скриншота.     

Пример вставки изображения:

### Задание 1

Скриншот-1 к заданию 1:
![Скриншот-1](https://github.com/netology-code/sys-pattern-homework/blob/main/img/img15.png)