# Политика доступа (bucket policy)

Политики доступа устанавливают права на действия с бакетами, объектами и группами объектов.

Политика срабатывает, когда пользователь делает запрос к какому-либо ресурсу. В результате срабатывания политики запрос либо выполняется, либо отклоняется.

Проверка доступа происходит на трех уровнях: проверяется, разрешено ли действие [ролью пользователя](../security/index.md) (проверка сервиса {{ iam-name }} — {{ iam-short-name }}), политикой доступа и [списком разрешений ACL](acl.md).

1. Если запрос прошел проверку {{ iam-short-name }}, к нему применяется проверка политики доступа.
1. Проверка правил политики доступа происходит в следующем порядке:
   1. Если запрос подошел хотя бы под одно из правил `Deny`, то доступ будет запрещен.
   1. Если запрос подошел хотя бы под одно из правил `Allow`, то доступ будет разрешен.
   1. Если запрос не подошел ни под одно из правил, то доступ будет запрещен.
1. Если запрос не прошел проверку {{ iam-short-name }} или политики доступа, то применяется проверка доступа через ACL объекта.

{% include [storage-note-empty-policy](../_includes_service/storage-note-empty-policy.md) %}

Политику доступа можно настроить в консоли управления или описать в формате JSON по [специальной схеме](../s3/api-ref/policy/scheme.md), чтобы передать через один из программных инструментов — {{ yandex-cloud }} CLI, AWS CLI, {{ TF }} или API. Подробнее об управлении политикой см. в [инструкции](../operations/buckets/policy.md).

## Из чего состоит политика {#elements}

Политика доступа состоит из правил, а правило — из следующих базовых элементов:

Ресурс 

: Бакет, объект в бакете (`<имя бакета>/some/key`) или префикс (`<имя бакета>/some/path/*`), в том числе пустой префикс для обозначения всех объектов в бакете (`<имя бакета>/*`). В правиле можно указать несколько ресурсов.

  {% note info %}
  
  {% include [policy-bucket-objects](../../_includes/storage/policy-bucket-objects.md) %} 

  {% endnote %}

  Если вы описываете политику в формате JSON, у ресурса должен быть префикс `arn:aws:s3:::`, например `arn:aws:s3:::<имя бакета>`.
  
Действие 

: Набор операций над ресурсом, который будет запрещен или разрешен правилом. Подробнее читайте в разделе [Действия](../s3/api-ref/policy/actions.md).

Результат 

: Запрет или разрешение запрошенного действия. Сначала проверяется попадание запроса в фильтр с действием `Deny`, при совпадении запрос отклоняется и дальнейшие проверки не проводятся. При попадании в фильтр с действием `Allow` запрос разрешается. Если запрос не попал ни в один из фильтров, то запрос отклоняется.

Принципал 

: Получатель запрошенного разрешения. Это может быть пользователь {{ iam-short-name }}, федеративный пользователь, сервисный аккаунт или анонимный пользователь.

Условие 

: Определение случаев, когда действует правило. Подробнее читайте в разделе [Условия](../s3/api-ref/policy/conditions.md).

## Доступ к бакету через консоль управления {#console-access}

Если для бакета настроена политика доступа, то по умолчанию доступ к бакету через консоль управления {{ yandex-cloud }} запрещен. Чтобы разрешить доступ к бакету, нужно добавить в секцию `Statement` политики доступа правило, разрешающее любые запросы к ресурсам `<имя бакета>/*` и `<имя бакета>` через консоль управления.

Пример правила для конкретного пользователя {{ yandex-cloud }}:


```json
{
  "Effect": "Allow",
  "Principal": {
    "CanonicalUser": "<идентификатор пользователя>"
  },
  "Action": "*",
  "Resource": [
    "arn:aws:s3:::<имя бакета>/*",
    "arn:aws:s3:::<имя бакета>"
  ],
  "Condition": {
    "StringLike": {
      "aws:referer": "https://console.cloud.yandex.*/folders/*/storage/buckets/your-bucket-name*"
    }
  }
}
```




Идентификатор пользователя можно получить по [инструкции](../../iam/operations/users/get.md) в документации {{ iam-full-name }}.


## Примеры конфигурации {#config-examples}

* Правило, которое разрешает анонимному пользователю чтение объектов бакета по зашифрованному подключению:

  ```json
  {
    "Id": "epd4limdp3dg********",
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "f1qqoehl1q53********",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::<имя бакета>/*",
        "Condition": {
          "Bool": {
            "aws:SecureTransport": "true"
          }
        }
      }
    ]
  }
  ```

* Правило, которое разрешает скачивать объекты только из указанного диапазона IP-адресов:

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::<имя бакета>/*",
        "Condition": {
          "IpAddress": {
            "aws:SourceIp": "100.101.102.128/30"
          }
        }
      }
    ]
  }
  ```

* Правило, которое запрещает скачивать объекты с указанного IP-адреса:

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": "*",
        "Action": "*",
        "Resource": "arn:aws:s3:::<имя бакета>/*"
      },
      {
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::<имя бакета>/*",
        "Condition": {
          "IpAddress": {
            "aws:SourceIp": "100.101.102.103"
          }
        }
      }
    ]
  }
  ```

* Правило, которое дает разным пользователям полный доступ только к определенным папкам, каждому пользователю — к своей:

  ```json
  {
    "Version":"2012-10-17",
    "Statement":[
      {
        "Sid":"User1PermissionsResource",
        "Effect":"Allow",
        "Principal": {
          "CanonicalUser": "<идентификатор пользователя>"
        },
        "Action": "*",
        "Resource":["arn:aws:s3:::<имя бакета>/user1path/*"]
      },
      {
        "Sid":"User1PermissionsPrefix",
        "Effect":"Allow",
        "Principal": {
            "CanonicalUser": "<идентификатор пользователя>"
        },
        "Action": "s3:ListBucket",
        "Resource":["arn:aws:s3:::<имя бакета>"],
        "Condition": {
          "StringLike": {
            "s3:prefix": "user1path/*"
          }
        }
      },
      {
        "Sid":"User2PermissionsResource",
        "Effect":"Allow",
        "Principal": {
          "CanonicalUser": "<идентификатор пользователя>"
        },
        "Action": "*",
        "Resource":["arn:aws:s3:::<имя бакета>/user2path/*"]
      },
      {
        "Sid":"User2PermissionsPrefix",
        "Effect":"Allow",
        "Principal": {
          "CanonicalUser": "<идентификатор пользователя>"
        },
        "Action": "s3:ListBucket",
        "Resource":["arn:aws:s3:::<имя бакета>"],
        "Condition": {
          "StringLike": {
            "s3:prefix": "user2path/*"
          }
        }
      }
    ]
  }
  ```

* Правило, которое дает каждому пользователю и сервисному аккаунту полный доступ к папке с названием, равным [идентификатору пользователя](../../iam/operations/users/get.md) или [сервисного аккаунта](../../iam/operations/sa/get-id.md):

  ```json
  {
    "Version":"2012-10-17",
    "Statement":[
      {
        "Sid": "OwnDirPermissions",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "*",
        "Resource": ["arn:aws:s3:::<имя бакета>/${aws:userid}/*"]
      }
    ]
  }
  ```
