---
tags:
  - data
link: https://habr.com/ru/companies/lanit/articles/830446/
data_type:
  - MLOps
company: Lanit
author: Lanit
source: habr
---
В новой статье [CleverData](https://cleverdata.ru/?ysclid=lwt3yj0yt9756257032) мы расскажем о проектировании ML-пайплайна предсказания целевого действия с помощью Yandex Cloud. Пайплайн необходим для автоматического обмена данными с CDP CleverData Join - использования информации с платформы для обучения ML-моделей и формирования прогнозов поведения каждого пользователя. На примерах рассмотрим использование API сервисов Yandex Cloud, коснемся алгоритмов обработки данных и обучения ML-модели, а также расскажем о возникших проблемах. Под катом делимся кодом. 

![Источник](https://habrastorage.org/r/w780/getpro/habr/upload_files/31c/69d/b39/31c69db394a3e866907c0139b4d52d59.jpeg "Источник")

[Источник](https://www.pinterest.com.au/pin/72550243980936850/)

## Что такое CDP и зачем нужны предиктивные атрибуты

Платформа управления клиентскими данными (Customer Data Platform) - это инструмент, помогающий компаниям собирать, анализировать и использовать информацию о своих клиентах. CDP объединяет данные из таких источников, как сайты, мобильные приложения, CRM-системы и т.п., чтобы сформировать полное представление о портрете клиента и истории его действий.

CDP CleverData Join содержит два основных типа данных: клиентские профили и события, привязанные к этим профилям. В состав профиля входят идентификаторы клиента и атрибуты, содержащие информацию о пользователе (его имя, демографические характеристики, предпочтения и многое другое). Атрибуты событий представляют собой информацию о конкретных действиях клиентов. Например, посещения страниц сайта, факты покупки товаров, реакции на маркетинговые кампании. Но CDP - это не только база данных. Это инструмент, который помогает делать полезные выводы о клиентах и повышать эффективность маркетинга.

Основная часть данных собирается из внешних систем, интегрированных с CDP. Однако при наличии такого массива данных возникает желание получить новую, предиктивную характеристику профиля, сформированную методом машинного обучения на собранных сведениях. Знать не то, что человек уже сделал, а предугадывать его следующие шаги. 

В CDP предиктивные атрибуты, как правило, используются для прогнозирования поведения клиентов, определения их потребностей и предсказания вероятности совершения определенных действий. Например, для маркетологов может быть полезно иметь прогноз того, что клиент возьмет кредит или оформит подписку. Это можно вычислить при помощи ML-модели, обученной на исторических данных о поведении клиента. Предиктивные атрибуты играют важную роль в персонализации маркетинговых кампаний, улучшении обслуживания клиентов.

Конечно же, нужно понимать, что для моделирования предиктивных атрибутов требуется:

- иметь качественные, структурированные данные для моделирования,
    
- обеспечить достаточный объем таких данных,
    
- обладать качественными алгоритмами предобработки и обучения. 
    

Не менее важным вопросом является обслуживание моделей - их надо регулярно обновлять и дообучать на свежих данных, иначе качество прогнозирования будет неизбежно падать. Построению пайплайна, который обеспечивает автоматизированное взаимодействие с CDP, и посвящена эта статья.

## MVP и его цели

Мы решили построить MVP пайплайна, реализующего передачу данных для обучения, само обучение и передачу предсказаний обратно в CDP CleverData Join, используя готовые инструменты, предоставляемые Yandex Cloud. 

В качестве бизнес-задачи мы для себя определили предсказание вероятности совершения клиентом целевого действия на основании данных о переходах по страницам сайта - кликстриме. Это типовая задача бинарной классификации (классификация по определенным признакам в две группы) массива клиентов. Под целевым действием подразумевается посещение нужного нам URL (например, страницы оформления покупки или кредита). Поскольку кликстрим является цепочкой переходов пользователей по ссылкам, в качестве архитектуры модели была выбрана рекуррентная нейронная сеть (вид нейронных сетей, где связи между элементами образуют направленную последовательность).

Целью проекта стало построение полностью автоматизированного пайплайна, который приносил бы ценность заказчику в течение длительного времени. Для этого нам нужно было реализовать следующие цепочки операций.

- Предобработка новых данных для обучения и инференса (непрерывной работы нейронной сети на устройствах конечного пользователя) и преобразование их из формата хранения в CDP в формат, требуемый для ML-модели.
    
- Первичное обучение модели под задачу конкретного заказчика на кликстриме его клиентов.
    
- Периодическое дообучение модели на новых данных, поступающих в CDP. Дообучение на новых данных крайне важно, поскольку с течением времени модель может потерять свою актуальность, а, значит, точность прогнозов значительно уменьшится.
    
- Периодический инференс (обеспечение непрерывной работы нейронной сети на устройствах конечного пользователя) для клиентов с обновлениями в кликстриме и отгрузку результатов обратно в CDP.
    

## Архитектура компонентов MVP в Yandex Cloud

Итак, нам потребовалось разработать и развернуть в Yandex Cloud сервисы, реализующие все необходимые для построения MVP цепочки операций. После анализа требований была предложена следующая архитектура решения:

![Схема взаимодействия компонентов решения](https://habrastorage.org/r/w1560/getpro/habr/upload_files/0ce/bd9/803/0cebd980359caf75e5a9dd2a4ff3cd17.png "Схема взаимодействия компонентов решения")

_Схема взаимодействия компонентов решения_

На стороне CleverData Join находятся сервисы Model Manager, отвечающие за автоматизированное создание инфраструктуры под пайплайн в Yandex Cloud, и Data Processor, обеспечивающий периодическую первичную обработку новых данных и загрузку их в бакет S3. По сути Model Manager является Control Plane, который автоматизирует операции по развертыванию компонентов в Yandex Cloud при появлении потребности создать новый предиктивный атрибут.  Остальные компоненты находятся в инфраструктуре Yandex Cloud.

- В бакете Yandex Object Storage накапливаются данные, представляющие собой первично обработанный кликстрим. Там же должны располагаться артефакты и метаданные, полученные при обучении моделей, а также результаты инференса моделей.
    
- В сервисе Yandex DataSphere автоматически создаётся проект, содержащий ноутбуки для обучения и инференса. При поступлении новых данных в Yandex Object Storage должен запускаться необходимый ноутбук. В ноутбуке обучения реализованы вторичная обработка данных на кластере Yandex DataProc, обучение модели непосредственно в сервисе, загрузка артефактов модели на S3. В ноутбуке инференса - получение предсказаний по новым данным, отгруженным на S3.
    
- В сервисе Yandex Cloud Functions располагаются функции, необходимые для запуска ноутбуков, и триггеры, которые привязаны к обновлению данных в Yandex Object Storage по определенным при их создании префиксам. Таким образом, после отгрузки новых данных на бакет производится, в зависимости от расположения данных, обучение/дообучение модели, инференс модели и отгрузка предсказаний в REST API CleverData Join.
    

Так как важно обеспечить одновременную работу нескольких моделей совершения целевого действия для разных тенантов в системе CD Join, для каждой задачи нужен свой отдельный проект в DataSphere со своими шаблонными ноутбуками, облачными функциями и триггерами. Чтобы процесс был максимально автоматизирован, необходимо, чтобы каждый проект был преднастроен: 

- в него были загружены шаблонные ноутбуки,
    
- был активирован готовый Docker-образ для проекта DataSphere.
    

Забегая вперёд, отметим, что с автоматизацией преднастройки проекта возникли некоторые затруднения, но об этом мы расскажем позднее.

## Немного о предобработке данных

Data Processor запускается по расписанию и производит первичную обработку данных и записывает результат в хранилище Yandex Object Storage.

События кликстрима хранятся в CleverData Join в формате AVRO. Предобработка происходит в два этапа.

- На первом этапе из событий кликстрима извлекаются идентификатор клиента, метка времени и URL перехода. Ссылки очищаются от технических доменов и обрезаются до верхнеуровневых доменов. Данные сохраняются в формате Parquet.
    
- Далее для каждого идентификатора клиента формируется цепочка из посещенных URL в хронологическом порядке и дельт, то есть временных интервалов, прошедших между посещениями каждого URL.
    

![Схема предобработки данных](https://habrastorage.org/r/w1560/getpro/habr/upload_files/171/7f3/908/1717f390884ad3bfb592aef2c7196b7b.png "Схема предобработки данных")

_Схема предобработки данных_

Процесс, организованный в два этапа, позволяет избегать повторной первичной обработки старых событий. Старые события хранятся в формате Parquet в S3 бакете и периодически дополняются первично обработанными событиями за новые периоды. После этого происходит пересборка цепочек для клиентов, по которым есть обновления.

## Автоматизированное создание инфраструктуры

У сервисов Yandex Cloud есть API, позволяющий взаимодействовать с ними без использования UI, что критически важно для реализации управления процессом в Model Manager. В этом разделе мы пройдём по всем автоматизированным через API операциям, а именно:

- созданию сервисного аккаунта;
    
- созданию и настройке сети VPC для кластера Data Proc;
    
- созданию проекта в DataSphere;
    
- созданию S3 бакета и получению статического ключа доступа;
    
- созданию и настройке Cloud Functions и их триггеров.
    

Сначала создадим сервисный аккаунт, от имени которого будет происходит взаимодействие с ресурсами Yandex Cloud:

```
def create_service_account(folder_id: str,                           name: str,                           iam_token: str,                           description: str = ''):  url = 'https://iam.api.cloud.yandex.net/iam/v1/serviceAccounts'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {           "folderId": folder_id,           "name": name,           "description": description,         }  response = requests.post(url, headers=headers, json=data)  return responsesa_response = create_service_account(folder_id = folder_id,                                     name = 'ta-service-account',                                     iam_token = iam,                                     description = 'ta-service-account')
```

Теперь, чтобы у сервисного аккаунта хватало прав на взаимодействие с необходимыми сервисами Yandex Cloud (DataSphere, Object Storage, Data Proc, Cloud Functions, VPC), выдадим ему необходимые роли. Для каталога это:

- editor - нужна для чтения, записи и управления бакетом S3;
    
- vpc.user, vpc.admin - необходимы для настройки доступа кластеру в интернет;
    
- dataproc.agent - для использования кластера Data Proc;
    
- dataproc.admin - для создания и удаления кластера Data Proc.  
    

```
def update_access_bindings(resource_id : str,                           folder_id: str,                           roles: list = ['iam.serviceAccounts.user',                                          'editor',                                          'vpc.user',                                          'vpc.admin',                                          'dataproc.admin',                                          'dataproc.agent'],                           iam_token: str = iam):   url = f'https://resource-manager.api.cloud.yandex.net/resource-manager/v1/folders/{folder_id}:updateAccessBindings'   headers = {"Authorization": f"Bearer {iam_token}"}   access_binding_deltas = [       {           "action": "ADD",           "accessBinding": {               "roleId": role,               "subject": {                   "id": resource_id,                   "type": "serviceAccount"               }           }       } for role in roles   ]   payload = {       "accessBindingDeltas": access_binding_deltas   }   response = requests.post(url, json=payload, headers=headers)   return responseresponse_update_access_bindings = update_access_bindings(                                                  resource_id=service_account_id,                                                  folder_id=folder_id)
```

Также необходимо назначить роль сервисному аккаунту, чтобы можно было им управлять. Роль должна быть назначена как на ресурс:

```
def update_access_bindings_1(resource_id: str,roles: list = ['datasphere.community-projects.editor','iam.serviceAccounts.user','functions.functionInvoker'],iam_token: str = iam):url = f"https://iam.api.cloud.yandex.net/iam/v1/serviceAccounts/{resource_id}:updateAccessBindings"headers = {"Authorization": f"Bearer {iam_token}"}access_binding_deltas = [{"action": "ADD","accessBinding": {"roleId": role,"subject": {"id": resource_id,"type": "serviceAccount"}}} for role in roles]payload = {"accessBindingDeltas": access_binding_deltas}response = requests.post(url, json=payload, headers=headers)return responseupdate_access_response_1 = update_access_bindings_1(resource_id=service_account_id)
```

Для того, чтобы кластер Data Proc мог общаться с внешним миром, нам нужно создать сеть VPC, указать шлюз и сформировать таблицу маршрутизации. В этой сети необходимо выделить подсеть для настройки доступа в интернет при помощи созданного шлюза.

Итак, создаём сеть:

```
def create_network(folder_id: str,                   network_name: str,                   description: str,                   iam_token: str = iam,                   ):  url = 'https://vpc.api.cloud.yandex.net/vpc/v1/networks'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "folderId": folder_id,    "name": network_name,    "description": description,  }  response = requests.post(url, headers=headers, json=data)  return responsecreate_network_response = create_network(folder_id = folder_id,                                         network_name = 'network-1',                                         description = ''                                         )
```

…, шлюз:

```
def create_gateway(folder_id: str,                   gateway_name: str,                   description: str,                   iam_token: str = iam,                   ):  url = 'https://vpc.api.cloud.yandex.net/vpc/v1/gateways'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "folderId": folder_id,    "name": gateway_name,    "description": description,    "sharedEgressGatewaySpec": {}  }  response = requests.post(url, headers=headers, json=data)  return responseresponse_create_gateway = create_gateway(folder_id = folder_id,                                         gateway_name = 'gateway',                                         description = ''                                         )
```

 …, таблицу маршрутизации:

```
def create_route_table(folder_id: str,                         route_table_name: str,                         description: str,                         gateway_id: str,                         network_id: str,                         destination_prefix: str = '0.0.0.0/0',                         iam_token: str = iam,                         ):  url = 'https://vpc.api.cloud.yandex.net/vpc/v1/routeTables'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "folderId": folder_id,    "name": route_table_name,    "description": description,    "networkId": network_id,    "staticRoutes": [      {        "destinationPrefix": destination_prefix,        "gatewayId": gateway_id,      }    ]  }  response = requests.post(url, headers=headers, json=data)  return responseresponse_create_route_table = create_route_table(folder_id = folder_id,                                           route_table_name = 'route-table',                                           description = '',                                           gateway_id = gateway_id,                                           network_id = network_id,                                           destination_prefix = '0.0.0.0/0'                                                      )
```

 …, и подсеть с доступом в интернет:

```
def create_subnet(folder_id: str,                  subnet_name: str,                  network_id: str,                  route_table_id: str,                  description: str,                  ip_range: str,                  zone_id: str,                  dhcp_options: dict = {},                  iam_token: str = iam,                  ):  url = 'https://vpc.api.cloud.yandex.net/vpc/v1/subnets'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "folderId": folder_id,    "name": subnet_name,    "description": description,    "networkId": network_id,    "zoneId": zone_id,    "v4CidrBlocks": [      ip_range    ],    "routeTableId": route_table_id,    "dhcpOptions": dhcp_options  }  response = requests.post(url, headers=headers, json=data)  return responseresponse_create_subnet_a = create_subnet(folder_id = folder_id,                subnet_name = 'subnet-ru-central1-a',                network_id = network_id,                route_table_id = route_table_id,                description = 'default subnet for zone ru-central1-a',                ip_range = '10.128.0.0/24',                zone_id = 'ru-central1-a',                dhcp_options = {},                )
```

В процессе работы мы будем создавать кластер Data Proc, на котором производится вторичная обработка данных.

При создании кластера важно не забыть передать идентификатор подсети, для которой мы ранее настраивали доступ в интернет. Нам нужно создать объект конфигурации с информацией о подкластерах, их ролях, типе и размере диска, количестве хостов. Этот объект будет передан в  функцию создания кластера. 

```
def create_cluster(folder_id: str,                   cluster_name: str,                   description: str,                   config_spec: dict,                   service_account_id: str,                   zone_id: str,                   bucket: str,                   ui_proxy: bool,                   security_group_ids: str,                   host_group_ids: str,                   deletion_protection: bool,                   log_group_id: str,                   iam_token: str = iam,                   ):  url = 'https://dataproc.api.cloud.yandex.net/dataproc/v1/clusters'  headers = {"Authorization": "Bearer {}".format(iam_token)}  data = {  "folderId": folder_id,  "name": cluster_name,  "description": description,  "configSpec": config_spec,  "zoneId": zone_id,  "serviceAccountId": service_account_id,  "uiProxy": ui_proxy,  "deletionProtection": deletion_protection,  }  response = requests.post(url, headers=headers, json=data)  return responseconfig_spec = {  "versionId": "2.0",  "hadoop": {  "services": [              'HDFS',              'LIVY',              'MAPREDUCE',              'SPARK',              'TEZ',              'YARN',              'ZEPPELIN'  ],  "sshPublicKeys": [      public_key  ],  },  "subclustersSpec": [  {      "name": "master",      "role": "MASTERNODE",      "resources": {      "resourcePresetId": "s2.micro",      "diskTypeId": "network-ssd",      "diskSize": "137438953472"      },      "subnetId": a_subnet_id,      "hostsCount": "1",      "assignPublicIp": False,  },  {      "name": "data",      "role": "DATANODE",      "resources": {      "resourcePresetId": "s2.small",      "diskTypeId": "network-hdd",      "diskSize": "274877906944"      },      "subnetId": a_subnet_id,      "hostsCount": "1",      "assignPublicIp": False,  },  ]}response_create_cluster = create_cluster(folder_id = folder_id,                                         cluster_name = 'xs-proc',                                         description = '',                                         config_spec = config_spec,                                         service_account_id = service_account_id,                                         zone_id = zone_id,                                         bucket = '',                                         ui_proxy = False,                                         security_group_ids = '',                                         host_group_ids = '',                                         deletion_protection = False,                                         log_group_id = '')cluster_id = response_create_cluster.json()['metadata']['clusterId']
```

Так как кластер будет считывать данные с s3, нужно предоставить ему Credentials для доступа к S3:

```
#!spark --cluster=xs-proc --session preparing_session --variables key --variables sctsc._jsc.hadoopConfiguration().set("fs.s3a.endpoint", "storage.yandexcloud.net")sc._jsc.hadoopConfiguration().set("fs.s3a.signing-algorithm", "")sc._jsc.hadoopConfiguration().set("fs.s3a.aws.credentials.provider", "org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider")sc._jsc.hadoopConfiguration().set("fs.s3a.access.key",key)sc._jsc.hadoopConfiguration().set("fs.s3a.secret.key",sct)
```

Теперь мы готовы создать проект в DataSphere, в нем будут происходить все операции, связанные с обучением и инференсом ML-моделей:

```
def create_project(project_name: str,                   community_id: str,                   service_account_id: str,                   description: str,                   subnet_id: str,                   commit_mode: str,                   default_folder_id: str,                   vm_inactivity_timeout: str,                   default_dedicated_spec: str,                   data_proc_cluster_id: str = '',                   security_group_ids: list = [],                   limits: dict = {},                   early_access: bool = False,                   ide: str = 'JUPYTER_LAB',                   iam_token: str = iam): url = 'https://datasphere.api.cloud.yandex.net/datasphere/v2/projects' headers = {"Authorization": f"Bearer {iam_token}"} data = {   "communityId": community_id,   "name": project_name,   "description": description,   "settings": {     "serviceAccountId": service_account_id,     "subnetId": subnet_id,     "dataProcClusterId": data_proc_cluster_id,     "commitMode": commit_mode,     "securityGroupIds": security_group_ids,     "earlyAccess": early_access,     "ide": ide,     "defaultFolderId": default_folder_id,     "vmInactivityTimeout": vm_inactivity_timeout,     "defaultDedicatedSpec": default_dedicated_spec   },   "limits": limits } response = requests.post(url, headers=headers, json=data) return responseresponse_create_project = create_project(project_name = 'ta-service-project',                                         community_id = community_id,                                         service_account_id = service_account_id,                                         subnet_id = a_subnet_id,                                         description = 'ta model project',                                         commit_mode = 'AUTO',                                         default_folder_id = folder_id,                                         vm_inactivity_timeout = '1800s',                                         default_dedicated_spec = 'c1.4')
```

Сервисный аккаунт, который из Cloud Functions будет запускать ноутбук, должен быть в участниках проекта DataSphere. Выдадим ему соответствующую роль:

```
def update_access_bindings_2(resource_id : str,                            project_id: str,                            roles: list = ['datasphere.community-projects.editor'],                            iam_token: str = iam):   url = f"https://datasphere.api.cloud.yandex.net/datasphere/v2/projects/{project_id}:updateAccessBindings"   headers = {"Authorization": f"Bearer {iam_token}"}   access_binding_deltas = [       {           "action": "ADD",           "accessBinding": {               "roleId": role,               "subject": {                   "id": resource_id,                   "type": "serviceAccount"               }           }       } for role in roles   ]   payload = {       "accessBindingDeltas": access_binding_deltas   }   response = requests.patch(url, json=payload, headers=headers)   return responseresponse_update_access_bindings_2 = update_access_bindings_2(resource_id=service_account_id,                         project_id=project_id)
```

Вы можете использовать для обмена данными уже существующий бакет S3. Но, если его нет, то его создание также не представляет сложности:

```
def create_bucket(bucket_name: str,                  folder_id: str,                  default_storage_class: str,                  max_size: str,                  anonymous_access_flags: dict,                  acl: dict,                  versioning: str,                  tags: list,                  iam_token: str = iam,                  ):  url = 'https://storage.api.cloud.yandex.net/storage/v1/buckets'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "name": bucket_name,    "folderId": folder_id,    "defaultStorageClass": default_storage_class,    "maxSize": max_size,    "anonymousAccessFlags": anonymous_access_flags,    "versioning": versioning,    "acl": acl,    "tags": tags  }  response = requests.post(url, headers=headers, json=data)  return responsebucket_name = 'test-storage-042'response_create_bucket = create_bucket(bucket_name = bucket_name,                           folder_id = folder_id,                           default_storage_class = 'STANDARD',                           max_size = '3221225472',                           anonymous_access_flags = {'read': False, 'list': False},                           acl = {},                           versioning = 'VERSIONING_DISABLED',                           tags = [])
```

Однако крайне важно создать статический ключ доступа к бакету. Ключ позволяет читать данные с S3 как кластеру Data Proc, так и при помощи SDK boto3. Конечно же, у сервисного аккаунта service_account_id, указанного в функции, должна быть роль в каталоге с правами чтения и записи S3 бакета, в нашем случае это editor. 

```
def create_static_key(service_account_id: str,                      description: str,                      iam_token: str = iam,                     ):   url = 'https://iam.api.cloud.yandex.net/iam/aws-compatibility/v1/accessKeys'   headers = {"Authorization": f"Bearer {iam_token}"}   data = {     "serviceAccountId": service_account_id,     "description": description   }   response = requests.post(url, headers=headers, json=data)   return responseresponse_create_static_key = create_static_key(                               service_account_id = service_account_id,                               description = '')key = response_create_static_key.json()['accessKey']['keyId']secret = response_create_static_key.json()['secret']
```

Обязательно сохраняем идентификатор статического ключа и секретный ключ в защищенном месте. Мы передадим их через параметр запроса InputVariables метода execute при запуске ноутбуков.

Создаем функцию в Cloud Functions, которая будет запускать обучение:

```
def create_cloud_function(folder_id: str,                          function_name: str,                          description: str,                          iam_token: str = iam):   url = 'https://serverless-functions.api.cloud.yandex.net/functions/v1/functions'   headers = {"Authorization": f"Bearer {iam_token}"}   data = {     "folderId": folder_id,     "name": function_name,     "description": description,   }   response = requests.post(url, headers=headers, json=data)   return responseresponse_create_cloud_function = create_cloud_function(folder_id = folder_id,                                                       function_name = 'fit',                                                       description = 'start fit')
```

Далее нужно собрать версию функции, в которую передадим сам код скрипта в виде строки. Чтобы получить эту строку, нужно заархивировать код функции в zip-архив и закодировать в base64. Строка передается в переменную content:

```
def create_cloud_function_version(function_id: str,                                 resources: dict,                                 runtime: str,                                 entrypoint: str,                                 log_options: dict,                                 description: str,                                 execution_timeout: str,                                 service_account_id: str,                                 content: str,                                 iam_token: str = iam                                 ):   url = 'https://serverless-functions.api.cloud.yandex.net/functions/v1/versions'   headers = {"Authorization": f"Bearer {iam_token}"}   data = {     "functionId": function_id,     "runtime": runtime,     "description": description,     "entrypoint": entrypoint,     "resources": resources,     "executionTimeout": execution_timeout,     "serviceAccountId": service_account_id,     "logOptions": log_options,     "content": content,   }   response = requests.post(url, headers=headers, json=data)   return responseresponse_create_cloud_function_version = create_cloud_function_version(function_id = fit_cf_id,resources = {'memory': '134217728'},runtime = 'python312',entrypoint = 'index.handler',log_options = {'disabled': False, 'folderId': folder_id},description = 'the function will execute fit notebook',      execution_timeout = '1s',                                                                    service_account_id = service_account_id,content = str(encoded)[2:-1],iam_token = iam)
```

Функция и ее версия для запуска инференса добавляется полностью аналогичным образом.

Осталось создать триггеры, которые после появления новых данных в S3 запустят функции, которые в свою очередь запустят обучение, инференс или же отправку предсказаний в CleverData Join.

Мы будем отслеживать появление нового объекта на S3 с заданным префиксом, поэтому используем соответствующий event_type и передадим нужный prefix:

```
def create_trigger(folder_id: str,                  trigger_name: str,                  description: str,                  event_type: str,                  bucket_id: str,                  prefix: str,                  cloud_function_id: str,                  service_account_id: str,                  retry_attempts: str,                  interval: str,                  function_tag: str = '$latest',                  iam_token: str = iam                  ):  url = 'https://serverless-triggers.api.cloud.yandex.net/triggers/v1/triggers'  headers = {"Authorization": f"Bearer {iam_token}"}  data = {    "folderId": folder_id,    "name": trigger_name,    "description": description,    "rule": {      "objectStorage": {        "eventType": [          event_type        ],        "bucketId": bucket_id,        "prefix": prefix,        "invokeFunction": {          "functionId": cloud_function_id,          "functionTag": function_tag,          "serviceAccountId": service_account_id,          "retrySettings": {            "retryAttempts": retry_attempts,            "interval": interval          },        },      },    }  }  response = requests.post(url, headers=headers, json=data)  return responseresponse_create_fit_trigger = create_trigger(                         folder_id = folder_id,                         trigger_name = 'fit-test-trigger',                         description = '',                         event_type = 'OBJECT_STORAGE_EVENT_TYPE_CREATE_OBJECT',                         bucket_id = bucket_id,                         prefix = fitting_trigger_prefix,                         cloud_function_id = fit_cf_id,                         service_account_id = service_account_id,                         retry_attempts = '5',                         interval = '20s',                         function_tag = '$latest')
```

Поговорим немного о возврате данных в CDP. В CleverData Join имеются готовые адаптеры на пакетную загрузку данных. При этом, все операции взаимодействия с CDP также превосходно автоматизируются через API.

Мы создаем экземпляр входящего адаптера для того, чтобы передавать и записывать в атрибут клиентского профиля CDP предсказания модели:

```
def create_adapter_instance_input(adapter_instance_id: str,                                  source: str,                                  cid: str,                                  access_token: str,                                  value: dict={}):   headers = {"Authorization": "Bearer {}".format(access_token),              "x-dmpkit-onbehalf-of": cid}   url = f'https://domain_name.cleverdata.ru/integration-manager/v1/adapters/1/instances/{adapter_instance_id}/inputs'   data = {       "value":value,       "source":source,       "adapterInstanceId":adapter_instance_id   }   response = requests.post(url, json=data, headers=headers)   return responseresponse = create_adapter_instance_input(adapter_instance_id=adapter_instance_id,                                         source='predictions_task.json',                                         access_token=access_token,                                         cid=cid)
```

Все, что останется сделать, - реализовать в теле функции загрузку данных в файловый менеджер CleverData Join. Это тоже можно реализовать через API. Загруженные данные будут обработаны ранее созданным адаптером.

## Обучение и дообучение модели

После запуска ноутбука обучения или инференса в проекте DataSphere происходит вторичная обработка с использованием кластера Dataproc. Новые сведения о событиях кликстрима добавляются в цепочки действий клиента. Далее на полученных цепочках происходит обучение модели.

Код вторичной обработки и код обучения во время первичного и повторного запуска ноутбуков немного отличается. Один и тот же код в зависимости от состояния объектов в бакете обрабатывает следующие несколько ситуаций. Рассмотрим их по отдельности.

![Первичное обучение и первичный инференс модели](https://habrastorage.org/r/w1560/getpro/habr/upload_files/837/1ac/128/8371ac128bc56e81b1df35291c37c9de.png "Первичное обучение и первичный инференс модели")

_Первичное обучение и первичный инференс модели_

В ситуации, когда в бакете находится только файл с событиями за текущий период, полученный в результате первичной обработки, мы имеем дело с первичным обучением модели. Следовательно, после обработки данных ноутбук выполнит последовательность первичного обучения модели. Для ноутбука инференса реализована та же логика. Если это первые данные для инференса в бакете, то модель вычислит предсказания для всех профилей.

![Дообучение модели](https://habrastorage.org/r/w1560/getpro/habr/upload_files/43b/e0a/5e6/43be0a5e64ef6f4ec31d9cd7f0ff16e6.png "Дообучение модели")

_Дообучение модели_

В случае, если в бакете уже находились данные за предыдущие периоды,  произойдет объединение строк и построение новых цепочек. Далее - дообучение модели.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ad3/a0a/faa/ad3a0afaa01c36a2088dd9d38a5885f0.png)

Схема для повторного инференса выглядит схожим образом. Отличие заключается в том,  что в данных за предыдущие периоды будут оставлены только события клиентов текущего периода. За счёт этого цепочки будут перестроены только для новых клиентов - именно тех, по которым значения предиктивных атрибутов ещё не сформированы.

В процессе дообучения создается новая модель-кандидат. Для того, чтобы определить, лучше ли эта модель, чем предыдущая, сравнивается AUC-ROC score новой модели на тестовой выборке с аналогичным показателем предыдущей модели, который сохраняется в бакете S3. При отсутствии увеличения метрики несколько итераций дообучения подряд - процесс дообучения заканчивается. Если же после дообучения модель показала лучшее качество, чем предыдущая лучшая модель, то происходит обновление лучшей модели (а также токенизатора, значения порога предсказания, файла с AUC-ROC score).

## Ограничения в MVP версии проекта

В процессе реализации решения обнаружили некоторые ограничения, проработка которых может помочь расширить функциональность этого сервиса в будущем. 

Например, в настоящее время отсутствует метод для загрузки ноутбуков в проект DataSphere. Поэтому при реализации мы изменили логику и ввели концепцию сервисного ноутбука, который однократно создается в DataSphere вручную. Это -разовая операция, нет необходимости выполнять ее для каждой новой задачи предсказания.

При заведении новой задачи выполнения целевого действия Model Manager размещает рабочие ноутбуки в S3 бакете. Сервисный ноутбук скачивает их в файловую систему DataSphere и последовательно запускает, если для задачи, соответствующей ноутбуку, есть новые первично обработанные данные. 

Переход к данной концепции позволил ускорить и удешевить вторичную обработку. Теперь она сопровождается единоразовым поднятием кластера Data Proc и единой вторичной обработкой файлов всех задач клиента, под которые есть свежие данные. 

Подводя итог, важно отметить, что в совокупности с концепцией сервисного ноутбука добавление метода загрузки ноутбуков в API позволит добиться еще более эффективной автоматизации развертывания пайплайна в Yandex Cloud, исключив необходимость в использовании UI.

**Отключение режима запуска ноутбуков в Serverless в DataSphere** 

MVP изначально проектировалось в Serverless-парадигме - планировалось выделять ресурсы только в момент запуска вычислительных задач. Разумеется, мы с огромным удовольствием воспользовались Serverless-режимом запуска ноутбуков в DataSphere - это позволяло распределять рабочую нагрузку на CPU- и GPU-ресурсы в зависимости от характера нагрузки, хотя и приходилось взамен жертвовать временем на сериализацию, десериализацию переменных в процессе перехода между инстансами. Однако данный режим более не поддерживается, и, следовательно, нами был осуществлен переход к Dedicated. 

Cтоит подчеркнуть, что переход от режима Serverless к Dedicated не повлиял на увеличение расходов на ресурсы вследствие простаивания инстансов, поскольку выделенная ВМ на запуск ноутбука через API после завершения выполнения инструкций моментально удаляется. 

В рамках доработки текущего ограничения мы должны будем подобрать наиболее оптимальную конфигурацию ВМ при запуске в режиме Dedicated. 

**Ручная активация docker-образа в проекте** 

Пока что нет возможности взаимодействовать с ресурсом Docker-образ проекта DataSphere посредством API. Однако в ближайшем будущем этот функционал будет добавлен в API, что также обеспечит большую гибкость при развертывании пайплайна.  

Для повышения функциональности ML-сервиса в CDP все перечисленные ограничения будут проработаны при разработке его продакшн-версии, в том числе некоторые из них будут усовершенствованы в сотрудничестве с Yandex Cloud.

## Итоги MVP

Подводя итоги, можно с уверенностью сказать, что цели MVP были достигнуты.

- Мы проверили гипотезу о возможности практически полностью автоматизированного развертывания инфраструктуры сервиса в облаке.
    
- Построили эффективный конвейер предобработки данных.
    
- Научились запускать код обучения и инференса по внешним триггерам.
    
- Организовали процесс дообучения модели.
    

Использование ML-сервисов Yandex Cloud позволяет значительно упростить разработку решений по предиктивному анализу данных. Разработанный функционал является отличным дополнением к уже существующим широким возможностям платформы CleverData Join в части накопления знаний о едином профиле пользователя и построении его клиентского пути и позволяет обогащать данные о клиентской базе и открывает возможности использования предиктивных атрибутов для сегментации аудитории и организации адресных взаимодействий с клиентом