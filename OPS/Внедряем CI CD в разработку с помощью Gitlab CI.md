---
tags:
  - data
data_type:
  - DevOps
link: https://habr.com/ru/articles/831110/
source: habr
---
CI/CD — это практика DevOps, ориентированная на автоматизацию развертывания приложений. Она позволяет разработчикам сосредоточиться на решении задач, не тратя время на рутинные действия, связанные с деплоем нового функционала или правок. Это становится возможным благодаря обеспечению автоматизированной системы доставки кода в необходимое окружение, чаще — на продакшн-площадку.

CI/CD пытается устранить человеческий фактор и убрать необходимость публиковать изменения вручную разработчикам.

**Почему GitLab?**

В своей работе мы используем self-hosted версию Gitlab, и поскольку эта платформа дает мощные возможности для организации CI/CD из коробки, мы используем именно его. Тем более, Gitlab CI очень прост для понимания и легко настраивается.

Таким образом, добавить CI/CD на новый проект занимает не более часа, а трудозатраты на публикации и шансы, что что-то сломается, сокращаются многократно.

**Как это выглядит**

Чтобы начать использовать GitLab CI, нужно разместить в корне проекта специальный файл — .gitlab-ci.yml. В этом файле указываются этапы (stages), задания (jobs) и скрипты, которые будут выполняться во время выполнения задания.

Также Gitlab CI позволяет прописать условия срабатывания заданий, например, отложенное выполнение (scheduled jobs) или пуш в указанную ветку.

Набор этапов называется пайплайном. Пайплайны могут представлять из себя что угодно, но чаще это стандартный набор: сборка, тесты, развертывание на площадке. Именно пайплайны вы будете описывать в  .gitlab-ci.yml, если захотите использовать Gitlab CI.

**Gitlab Runner**

Пайплайны выполняются специальными демонами — раннерами.

Раннер может быть размещен как на физическом сервере, так и на виртуальной машине. Обычно раннер загружает код или артефакт и запускает задания — либо локально, либо в контейнере.

Если вы также используете self-hosted решение, вы можете зарегистрировать новый раннер на сервере или использовать уже зарегистрированный. В публичной версии Gitlab должны быть доступны раннеры для Linux, Windows и Mac.

**Реализуем CI/CD**

Приблизительно .gitlab-ci.yml может выглядеть так:

```
stages:  - deploy  - error.branch_stay_actual:  stage: deploy  rules:    - if: ($CI_COMMIT_BRANCH == $BRANCH_NAME)      when: on_success  script:    - make check_vars --makefile=MakefilePipeline -j1    - make notify_start --makefile=MakefilePipeline -j1    - make build --makefile=MakefilePipeline -j1    - make shift_build_versions --makefile=MakefilePipeline -j1    - make notify_success --makefile=MakefilePipeline -j1    - make clear_tmp --makefile=MakefilePipeline -j1  variables:    GIT_STRATEGY: clonedev-stay-actual:  rules:    - if: ($CI_PIPELINE_SOURCE == "schedule")      when: never    - if: ($CI_COMMIT_BRANCH == $BRANCH_NAME)      when: on_success  extends: .branch_stay_actual  variables:    COMPOSE_NAME: docker-compose-dev.yml    FPM_CONTAINER_NAME: backend-dev-fpm    CICD_DIR: /var/www/html/project/path/  environment:    name: "backend-dev"  tags:    - developmentdev-react_error:  stage: error  tags:    - development  environment:    name: "backend-dev"  script:    - sh $TELEGRAM_SCRIPT "❌ <b>FAILURE</b>"  rules:    - if: ($CI_PIPELINE_SOURCE == "schedule")      when: never    - if: ($CI_COMMIT_BRANCH != $BRANCH_NAME)      when: never    - when: on_failureprod-stay-actual:  rules:    - if: ($CI_PIPELINE_SOURCE == "schedule")      when: never    - if: ($CI_COMMIT_BRANCH == $BRANCH_NAME)      when: on_success  extends: .branch_stay_actual  variables:    COMPOSE_NAME: docker-compose-prod.yml    FPM_CONTAINER_NAME: backend-prod-fpm    CICD_DIR:  /var/www/html/project/path/  environment:    name: "backend-prod"  tags:    - productionprod-error:  stage: error  tags:    - production  environment:    name: "backend-prod"  script:    - sh $TELEGRAM_SCRIPT "❌ <b>FAILURE</b>"  rules:    - if: ($CI_PIPELINE_SOURCE == "schedule")      when: never    - if: ($CI_COMMIT_BRANCH != $BRANCH_NAME)      when: never    - when: on_failure
```

**Разберем файл по порядку.**  
  
Блок stages описывает стадии пайплайна. В данном случае, имеем всего две стадии: deploy, в котором выполняются необходимые действия для деплоя проекта на сервере, и error, который выполняется в случае ошибки.

Далее идут задания.

Задание .branch_stay_actual является своеобразным прототипом, который не выполняется напрямую и наследуется в настоящих заданиях ниже (в  dev-stay-actual и prod-stay-actual для деплоя на dev-площадке и prod-площадке соответственно).

Для каждого задания определены правила, при которых задание будет выполнено. Так, задание dev-stay-actual не выполняется при отложенном выполнении, а запускается лишь когда значение специальной предопределенной Gitlab переменной CI_COMMIT_BRANCH будет равно значению переменной BRANCH_NAME, которая указывается вручную. Переменная CI_COMMIT_BRANCH инициализируется автоматически самим Gitlab при пуше в ветку.

Далее идет специальный блок environment, грубо говоря, отвечающий за то, какие переменные стоит передавать в пайплайн. Environments создаются в настройках Operate вашего проекта в Gitlab, после чего в настройках CI/CD проекта можно будет указать переменные для выбранного environment.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/466/821/8df/4668218df092da8b02c80b22dddef902.jpg)

Скрипт отвечает за то, какие команды выполнять. Содержимое скрипта зависит от ваших целей.

Мы применяем Makefile, чтобы уменьшить размер .gitlab-ci.yml и сохранить читаемость. Внутри этого мейкфайла, как и внутри скрипта в целом, как и говорилось, может быть что угодно — главное, чтобы команды были доступны на машине с раннером.

Пример задания Makefile, проверяющего, что все необходимые переменные заданы:

```
check_vars:    ifeq ($(BRANCH_NAME),)        $(error BRANCH_NAME var is required!)    endif    ifeq ($(CICD_DIR),)        $(error CICD_DIR var is required!)    endif    ifeq ($(TELEGRAM_SCRIPT),)        $(error TELEGRAM_SCRIPT var is required!)    endif    ifeq ($(DOTENV), )    	$(error DOTENV var is required)    endif
```

Таким образом, переменные, указанные в блоке variables задания будут переданы в окружение выполняемого скрипта. Список предопределенных переменных CI/CD Gitlab можно найти [здесь](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html).

Немаловажной деталью пайплайна являются теги заданий. Они необходимы, чтобы раннер понимал, нужно ли выполнять данное задание или нет.

Теги настраиваются для каждого раннера индивидуально и указываются у заданий. У заданий и у раннеров может быть ноль или более тегов.

Например, вы можете описать задания, которые подхватываются только раннером, работающим с заданиями помеченными как testing для периодического тестирования системы. Или отметить задания, которые выполняются только на dev-, test- или prod-площадках.

Также, можно наследовать целые пайплайны. Выглядит это примерно так:

#### reusable-pipeline.yml:

```
variables:  DB_USER: user  DB_PASSWORD: passwordproduction:  stage: production  script:    - build    - deploy  environment:    name: production  rules:    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

#### .gitlab-ci.yml:

```
include:   - project: "project-name"        ref: 0.1.0        file: reusable-pipeline.ymlvariables:  DB_USER: root  DB_PASSWORD: real_passwordstages:  - build  - test  - productionproduction:  environment:    name: production
```

В данном случае .gitlab-ci.yml называется reusable-pipeline.yml и находит в другом репозитории – project-name. Так можно описать один общий пайплайн для всех похожих проектов и наследовать при необходимости.

## Заключение

CI/CD помогает разработчикам сократить затраты на развертывание и настройку проектов, позволяя им сконцентрироваться на решении бизнес-задач. Gitlab — чрезвычайно мощная платформа, и мы рекомендуем присмотреться к использованию средств CI/CD, которые она представляет.

В этой статье мы рассмотрели один из самых простых сценариев использования Gitlab CI, но его возможности — куда шире.