# Docker Swarm Cluster Provisioning

## Начало работы

1. Перед начало работы установите ansible:

``` sudo apt get ansible```

2. Далее скопируйте файл `hosts.yml.dist` в корень проекта. Файл один из главных для запуска кластера, в нем заполняется секция `manager` поля `ansible_host`: публичный ip сервера и `ansible_port`: можем оставить по умолчанию 22 порт, а так же `vars` поле `hostname`: имя сервера. Секция `workers` заполняется опциально, при наличии воркера и использовании микросервисной архитектуры.

```cp hosts.yml.dist hosts.yml```

3. Далее заполняется файл `authorize.yml` секция `vars_prompt`: указываем путь до своего ssh-файла.

## Команды make

Запуск ansible-playbook — создание сервера, установка всех необходимых пакетов:

```make site```

Обновление пакетов на сервере, удаление неиспользуемых:

```make upgrade```

Авторизация на сервере. Отправляет ssh ключ пользователя на сервер:

```make authorize```

Команда для генерации ssh ключа пользователя deploy:

```make generate-deploy-key```

Отправляет ssh ключ пользователя deploy на сервер. Производит операцию входа в container registry для сервера. Аналогично тому, что делается на локальной машине, но это команда для сервера.

```make authorize-deploy```

Необходима для того, чтобы при обновлении docker images во время deploy сервер имел доступ к docker registry репозиторию и мог скачать обновленные packages, иначе обновление не произойдет.

```make docker-login```

### После истечения срока действия токена доступа необходимо перевыпустить токен
- Если registry на github: https://github.com/settings/tokens
- Если registry Selectel: https://docs.selectel.ru/cloud/craas/quickstart/

После перевыпуска токена необходимо запустить команду `make docker-login` и заново авторизироваться в `cr.selcloud.ru/<registry_name>` — ввести имя пользователя и новый токен.

Это необходимо для того, чтобы при обновлении docker images во время deploy сервер имел доступ к docker registry репозиторию и мог скачать обновленные packages, иначе обновление не произойдет.

### Авторизоваться в Docker CLI
Для работы с Docker CLI нужен токен и данные для авторизации, которые вы получили на предыдущем шаге.

Откройте терминал и введите команду:

```docker login cr.selcloud.ru```

Введите логин (username) и пароль (password) для реестра.

## Возможные ошибки

Для устранения ошибки после установки докера:

```
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: unable to apply apparmor profile: apparmor failed to apply profile: write /proc/self/attr/apparmor/exec: no such file or directory: unknown.
```
Добавлена установка apparmor. https://github.com/docker/for-linux/issues/1199

`roles/docker/tasks/install_docker.yml`

Секция name: Install dependencies

### Возможно понадобиться перезагрузка сервера после установки apparmor

1. Запускаем `make update` для обновления dependencies в хост системе
2. Запускаем `make authorize` для создания пользователя deploy

## Документация по подъёму кластера Docker Swarm с Ansible

https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/

https://docs.ansible.com/ansible/latest/collections/community/docker/docker_swarm_module.html#ansible-collections-community-docker-docker-swarm-module