# Запуск Jenkins
1. Выполнить git clone https://github.com/MiraD1n/OpsWorks.git
2. перейти в директорию cd ./OpsWorks/Jenkins
3. выполнить команду docker-compose up -d
4. дождатся запуска Jenkins
5. через команду docker logs <CONTAINER ID> взять password обрамленный символами *****
6. перейти по URL http://localhost:8080
7. активировать Jenkins через ввод пароля
8. установить рекомендуемые плагины
9. установить плагин Docker
10. Manage Jenkins -> Configure System -> Add a new cloud *в самом низу страницы
* настроить Pipeline Model Definition
* Docker Label: docker-agent
* Docker registry URL: tcp://DOCKER_HOSTNAME_IP:2375
* настроить плагин Docker (Manage Jenkins -> Configure System -> Add a new cloud *в самом низу страницы)
* Docker cloud details...
* Name: docker
* Docker Host URI: tcp://DOCKER_HOSTNAME_IP:2375
* Выполнить Test Connection: при успешном подключении отображает версию
* в случае ошибки необходимо внести правки в конфигурационные файлы на хосте с docker
* vim /etc/default/docker -> и добавить ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
* vim /lib/systemd/system/docker.service -> и внести правки:
* ExecStart=/usr/bin/dockerd -H fd://                <--- before
* ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375    <--- After
* выполнить рестарт докера
* systemctl daemon-reload
* systemctl restart docker
* провести повторную попытку подключения
* Enabled: true
* Docker Agent templates
* Labels: docker-agent
* Name: docker-agent
* Docker Image: benhall/dind-jenkins-agent:v2
* Открыть Container settings
* Volumes: /var/run/docker.sock:/var/run/docker.sock
* Enabled: true
* Remote File System Root: /home/jenkins
* Connect method: Connect with SSH
* SSH key: Inject SSH key
* User: root
11. Apply -> Save
12. Restart Jenkins

# Создание проекта на деплой NGINX
1. New Item
2. Enter an item name
3. Выбираем Pipeline -> OK
4. Pipeline -> Pipeline script from SCM
* SCM -> Git
* Repository URL: https://github.com/MiraD1n/OpsWorks.git
* Script Path: Jenkins/Build_NGINX.jenkins
5. Apply -> Save

# Запуск деплоя и тестирование:
## Деплой
1. запустить проект
* первый деплой занимает длительное время по причине скачивания агента деплоя и всех исходников
2. после окончания выполнения билда перейти по tcp://DOCKER_HOSTNAME_IP:80 и убедится в доступности статичной страницы HTML
## Тест 1 
1. внести правки в https://github.com/MiraD1n/OpsWorks/blob/master/WebProject/index.html
> можно внести правки и в файл https://github.com/MiraD1n/OpsWorks/blob/master/NGINX/Dockerfile установив новую версию NGINX, но время деплоя увеличится в связи с повторной сборкой
2. запустить билд повторно
3. после окончания выполнения билда перейти по tcp://DOCKER_HOSTNAME_IP:80 и убедится в доступности статичной страницы HTML + внесенные правки
## Тест 2
1. закоментировать строку 36 в файле https://github.com/MiraD1n/OpsWorks/blob/master/Jenkins/Build_NGINX.jenkins (*sh 'docker exec mynginx service nginx start')
  > *в следствии чего после сборки NGINX в новом контейнере не будет запущен
2. внести правки в https://github.com/MiraD1n/OpsWorks/blob/master/WebProject/index.html
3. запустить билд
4. после окончания выполнения билда перейти по tcp://DOCKER_HOSTNAME_IP:80 и убедится в доступности статичной страницы HTML + внесенные правки остались из Тест 1
## Тест 3
1. разкоментировать строку 36 в файле https://github.com/MiraD1n/OpsWorks/blob/master/Jenkins/Build_NGINX.jenkins (*sh 'docker exec mynginx service nginx start')
2. разкоментировать строку 35 в файле https://github.com/MiraD1n/OpsWorks/blob/master/Jenkins/Build_NGINX.jenkins (*sh 'docker exec mynginx rm /usr/local/nginx/html/index.html')
> *в следствии после сборки будет удалена страница index.html из контейнера с NGINX
3. запустить билд
4. после окончания выполнения билда перейти по tcp://DOCKER_HOSTNAME_IP:80 и убедится в доступности статичной страницы HTML + внесенные правки остались из Тест 1
## Тест 4
1. закоментировать строку 35 в файле https://github.com/MiraD1n/OpsWorks/blob/master/Jenkins/Build_NGINX.jenkins (*sh 'docker exec mynginx rm /usr/local/nginx/html/index.html')
2. внести правки в https://github.com/MiraD1n/OpsWorks/blob/master/WebProject/index.html
3. запустить билд повторно
4. после окончания выполнения билда перейти по tcp://DOCKER_HOSTNAME_IP:80 и убедится в доступности статичной страницы HTML + внесенные правки в Тесте 4
