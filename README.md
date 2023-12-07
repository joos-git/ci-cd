# ci-cd
Что такое DevOps? CI/CD

**DevOps** — это методология разработки ПО, задача которой — наладить взаимодействие программистов и сисадминов в компании. Если ИТ-специалисты из разных отделов недопонимают суть задач друг друга, выпуск новых приложений и обновлений для них затягивается

**CI/CD**

Важными процессами для достижения результатов DevOps являются процессы CI/CD

**CI (Continuous Integration)** — непрерывная интеграция. В задачи этого процесса входит обязанность иметь максимально рабочую версию каждого нового релиза. В первую очередь это сборка и тестирование проекта с каждым внесением изменений

**CD (Continuous Deployment)** — непрерывная поставка. Суть этого процесса заключается в постоянной выкладке ПО, желательно, уже протестированного, на сервера

Процессы CI/CD разрабатываются вместе, и в современном мире не принято их делить

Для тестирования проекта вместе с другими его компонентами — база данных, фронтенд и т. п. — необходимо сделать его деплой, а это уже часть процесса CD. Таким образом, для выполнения CI требуется запуск процесса CD, и наоборот, так как перед деплоем проект надо собрать и провести статические тесты

**Пример CI/CD процесса**

1. Сборка проекта, например, docker build
2. Запуск статических тестов. В зависимости от языка могут вызываться по-разному и запускаться до сборки
3. Деплой проекта на тестовое окружение
4. Запуск тестов для всего тестового окружения — интеграционные тесты
5. Деплой проекта на продакшн окружение

**Инструменты DevOps**

Для запуска всех описанных процессов обычно используется интеграция с Git. При появлении коммита в Git-сервере запускается процесс CI/CD, который в зависимости от разных параметров коммита — ветка, автор, тег и т. п. — выполняет все действия, необходимые для сборки процесса

Примеры систем, используемых для запуска CI/CD:
● Jenkins
● Gitlab CI
● Teamcity

**Jenkins** — программная система с открытым исходным кодом на Java, предназначенная для обеспечения процесса непрерывной интеграции ПО Jenkins можно установить на Windows, macOS, Debian, Ubuntu, CentOS и другие ОС. Также Jenkins можно установить через системные пакеты, Docker или запустить автономно на любом компьютере с настроенной Java Runtime Environment (JRE) https://www.jenkins.io/

**Установка Jenkins**

Сначала устанавливаем Java:
```
sudo apt install openjdk-11-jdk
```
Затем устанавливаем Jenkins из пакета для debian/ubuntu:
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
```
**Первичная настройка**

1. В строке браузера введите IP-адрес сервера и порт 8080 в формате 123.123.123.123:8080
2. На стартовой странице Jenkins указан путь, по которому хранится пароль для входа, обычно:
```
/var/lib/jenkins/secrets/initialAdminPassword
```
3. Введите пароль в браузере для разблокировки Jenkins
4. После предложения установить плагины и создать первого admin пользователя Jenkins доступен для работы

Важно: чтобы получть доступ к Docker из пайплайнов, необходимо добавить пользователя Jenkins в группу Docker и перезапустить Jenkins

**Sonatype Nexus** — интегрированная платформа, с помощью которой разработчики могут хранить и управлять зависимостями Java (Maven), образами Docker, Python, Ruby, NPM, Bower, RPM-пакетами, gitlfs, Apt, Go, Nuget, а также распространять своё ПО. https://help.sonatype.com/repomanager3

Установка с помощью Docker:
```
docker run -d -p 8081:8081 --name nexus -e INSTALL4J_ADD_VM_PARAMS="-Xms273m
-Xmx273m -XX:MaxDirectMemorySize=273m" sonatype/nexus3
```
Обратите внимание на параметры Xms и Xmx. Это объём выделяемой памяти, по умолчанию равен 2703 Mb

**Nexus: создание helm-репозитория**

1. Переходим на страницу настройки репозиориев:
```
http://123.123.123.123:8081/#admin/repository/repositories
```
2. Создаём новый репозиторий типа helm-hosted
3. Выбираем дополнительные параметры
4. Сохраняем

**Настройка простого pipeline**

1. Создаём проект: http://123.123.123.123:8080/view/all/newJob
2. Выбираем freestyle project и пишем имя проекта
3. Подключаем scm и параметры авторизации, если необходимо
4. Переходим в раздел Build, выбираем новый этап сборки типа "Execute shell"
5. Пишем любой скрипт, например docker build
6. Сохраняем проект и запускаем сборку

**Настройка declarative pipeline**

1. Создаём проект: http://123.123.123.123:8080/view/all/newJob
2. Выбираем pipeline и пишем имя проекта
3. Добавляем скрипт в поле Pipeline
```
pipeline {
  agent any
  stages {
    stage('Git') {
      steps {git 'https://github.com/killmeplz/k8s-job-sidekiller.git'}
    }
    stage('Build') {
      steps {
        sh 'docker build .'
        sh 'helm package .helm'
        sh 'curl -u admin:123
http://10.168.10.220:8081/repository/helm-test/ --upload-file
k8s-job-sidekiller-0.1.0.tgz -v'
      }
    }
  }
}
```

