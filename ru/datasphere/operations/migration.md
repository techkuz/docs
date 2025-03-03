# Как перенести работу в новую версию {{ ml-platform-name }}

Вместе с интерфейсом в {{ ml-platform-name }} изменилась и [ресурсная модель](../concepts/resource-model.md). Чтобы продолжить работу над проектами, созданными в старой версии {{ ml-platform-name }}, перенесите их в новую версию [автоматически](#auto) или [вручную](#data-transfer). 

## Перенести проекты и данные автоматически {#auto}

Вы можете автоматически перенести проекты, состояние интерпретатора, ресурсы проекта и все данные из старой версии {{ ml-platform-name }} в новую с помощью инструмента переноса проектов. Для этого в левом боковом меню выберите ![transfer](../../_assets/datasphere/transfer.svg) **{{ ui-key.yc-ui-datasphere.transferring.title }}** и следуйте подсказкам в интерфейсе.

{% note warning %}

После переноса проектов в новую версию {{ ml-platform-name }} проекты старой версии будут удалены.

{% endnote %}

{% note info %}

Автоматически перенести ноды невозможно. Если в проектах для переноса есть ноды, самостоятельно удалите их, перенесите проекты и создайте ноды заново, либо [обратитесь]({{ link-console-support }}) в техническую поддержку.

{% endnote %}

1. Выберите облако и проекты, которые вы хотите перенести. Для выбора вам доступны только облака, в которых есть проекты {{ ml-platform-name }}, и вы имеете роль `{{ roles-datasphere-admin }}` или `{{ roles-editor }}` на каталог с этим проектом.
1. Выберите [сообщество](../concepts/community.md), в которое вы будете переносить проекты. Сообщество должно иметь активный платежный аккаунт. Чтобы перенести проекты в сообщество, вам понадобится роль `{{ roles-datasphere-communities-admin }}` в этом сообществе или роль `{{ roles-editor }}` в организации. 
1. Нажмите **{{ ui-key.yc-ui-datasphere.transferring.transfer-selected-projects }}**.

После переноса проекты будут доступны в выбранном вами сообществе.

## Перенести ноутбуки и данные вручную {#data-transfer}

Чтобы продолжить работу над проектом в новой версии, [создайте](../operations/projects/create.md) проект заново и [загрузите](#data-transfer) в него нужные данные.

Если вы хотите сохранить не только код, но и [состояние интерпретатора](../concepts/save-state.md), создайте проект из [контрольной точки](#state-transfer).

Создайте [секреты](data/secrets.md#create), [подключения к хранилищам S3](data/connect-to-s3.md#s3-create), [датасеты](../concepts/dataset.md#init), [ноды](deploy/node-create.md) и [Docker-образы](user-images.md#docker-create) заново.

Если требуется, сделайте ресурсы [доступными для сообщества](../concepts/resource-model.md#sharing).

Чтобы перенести ноутбуки и данные, хранящиеся в старом проекте, в новую версию {{ ml-platform-name }}, выберите один из способов:
* [Используйте удаленный Git-репозиторий](#transfer-git).
* [Сохраните файлы старого проекта на локальный диск](#transfer-local).
* [Используйте объектное хранилище](#transfer-s3).

### Перенести данные с помощью удаленного Git-репозитория {#transfer-git}

1. В [консоли управления]({{ link-console-main }}) перейдите в сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_datasphere }}**.
1. Откройте старый проект, данные из которого нужно перенести.
1. [Создайте](./projects/work-with-git.md#init) в проекте локальный репозиторий.
1. [Отправьте](./projects/work-with-git.md#push) изменения в удаленный репозиторий.
1. В [новой версии]({{ link-datasphere-main }}) {{ ml-platform-name }} [склонируйте](./projects/work-with-git.md#clone) в проект удаленный репозиторий. 

### Сохранить файлы старого проекта на локальный диск {#transfer-local}

1. В [консоли управления]({{ link-console-main }}) перейдите в сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_datasphere }}**.
1. Откройте старый проект, данные из которого нужно перенести.
1. На панели слева выберите вкладку ![folder](../../_assets/datasphere/jupyterlab/folder.svg) **File Browser**.
1. Нажмите правой кнопкой мыши на файл, который нужно сохранить, и выберите пункт ![download](../../_assets/datasphere/jupyterlab/download.svg) **Download**. Последовательно сохраните все нужные файлы.
1. В [новой версии]({{ link-datasphere-main }}) {{ ml-platform-name }} откройте проект, в который нужно перенести файлы.
1. На вкладке ![folder](../../_assets/datasphere/jupyterlab/folder.svg) **File Browser** нажмите кнопку ![upload](../../_assets/datasphere/jupyterlab/upload.svg) **Upload Files** и выберите файлы на локальном диске.

Вы также можете сохранить все файлы старого проекта в виде ZIP-архива, для этого:
1. В старом проекте на верхней панели нажмите **File** и выберите пункт **Export project as zip**.
1. В открывшемся окне нажмите кнопку **Download** и дождитесь завершения загрузки ZIP-архива.
1. В [новой версии]({{ link-datasphere-main }}) {{ ml-platform-name }} откройте проект, в который нужно перенести файлы.
1. На вкладке ![folder](../../_assets/datasphere/jupyterlab/folder.svg) **File Browser** нажмите кнопку ![upload](../../_assets/datasphere/jupyterlab/upload.svg) **Upload Files** и выберите ZIP-архив на локальном диске.
1. Нажмите правой кнопкой мыши на ZIP-архив и выберите пункт ![extract](../../_assets/datasphere/jupyterlab/extract.svg) **Extract Archive**.

### Перенести данные с помощью объектного хранилища {#transfer-s3}

1. В [консоли управления]({{ link-console-main }}) перейдите в сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_datasphere }}**.
1. Откройте старый проект, данные из которого нужно перенести.
1. Если у вас еще нет подключения к S3, создайте его.

   {% cut "Как создать подключение к S3 в старой версии" %} 

   1. Получите ключ доступа у вашего S3-провайдера. Чтобы сделать это в {{ objstorage-full-name }}:
        1. [Создайте сервисный аккаунт](../../iam/operations/sa/create.md). 
        1. [Назначьте](../../iam/operations/sa/assign-role-for-sa.md) созданному аккаунту [роль](../../storage/security/index.md), которая разрешит чтение и запись данных.
        1. [Создайте ключ доступа](../../iam/operations/sa/create-access-key.md) для сервисного аккаунта.
   1. Создайте секрет:
        1. Перейдите на вкладку ![Secrets](../../_assets/datasphere/jupyterlab/secret.svg) и нажмите значок ![plus](../../_assets/datasphere/jupyterlab/add.svg).
        1. В диалоговом окне заполните поля:
            * **Name** — идентификатор ключа.
            * **Content** — секретный ключ в формате `<идентификатор статического ключа>:<секретная часть статического ключа>`.
            * **Scope** — выберите `project`.
        1. Нажмите кнопку **Save**.
        1. Чтобы секрет был доступен для использования, перезапустите ядро системы. Для этого выберите **Kernel ⟶ Restart kernel** в меню интерфейса {{ ml-platform-name }}.
   1. Перейдите на вкладку **S3 Mounts** ![S3 Mounts](../../_assets/datasphere/bucket.svg) и нажмите значок ![plus](../../_assets/datasphere/jupyterlab/add.svg).
   1. В диалоговом окне заполните поля:
        * **Endpoint** — хост хранилища. Для {{ objstorage-full-name }} это `https://{{ s3-storage-host }}/`.
        * **Bucket** — имя бакета в хранилище.
        * **Mount name** — имя бакета для использования внутри проекта {{ ml-platform-name }}. Автоматически генерируется из значения **Bucket**, но вы можете его поменять.
        * **Static access key** — секрет с ключом доступа. 
        * **Mode** — укажите режим доступа **Read / Write**.
   1. Нажмите кнопку **Test connection**. Если проверка прошла успешно, нажмите кнопку **Mount**.

        После подключения бакет будет доступен в списке на вкладке **S3 Mounts** ![S3 Mounts](../../_assets/datasphere/bucket.svg), и его можно будет просматривать как файловую систему.

   {% endcut %}

1. Перенесите необходимые данные старого проекта в объектное хранилище.
1. В [новой версии]({{ link-datasphere-main }}) {{ ml-platform-name }} [создайте](./data/connect-to-s3.md) коннектор S3 к этому же бакету.
1. Перенесите данные из хранилища в новый проект.

{% note info %}

Данные датасетов из старых проектов также можно сохранить в хранилище S3, а затем [инициализировать](../concepts/dataset.md#init) датасет в новой версии {{ ml-platform-name }} из этого хранилища.

{% endnote %}

## Перенести состояние интерпретатора {#state-transfer}

{% note warning %}

С помощью контрольной точки нельзя автоматически перенести Docker-образы, ноды, датасеты и другие ресурсы старого проекта в новую версию {{ ml-platform-name }}.

{% endnote %}

Чтобы перенести состояние интерпретатора старого проекта в новую версию с помощью контрольной точки:
1. В [консоли управления]({{ link-console-main }}) перейдите в сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_datasphere }}**.
1. Откройте старый проект и создайте контрольную точку проекта:
    * В [стандартном](../concepts/save-state.md#default-save) режиме сохранения состояний:
        1. Откройте панель ![checkpoints-panel](../../_assets/datasphere/jupyterlab/checkpoints-panel.svg) **Checkpoints**.

            Если вы уже выполняли код в открытом проекте, то в списке **Checkpoints** будут содержаться созданные ранее контрольные точки.
        1. Выполните код в ячейке или группе ячеек ноутбука и дождитесь выполнения кода.
    * В [автоматическом](../concepts/save-state.md#auto-save) режиме сохранения состояний: в правом верхнем углу вкладки ![checkpoints-panel](../../_assets/datasphere/jupyterlab/checkpoints-panel.svg) **Checkpoints** нажмите значок ![plus-sign](../../_assets/plus-sign.svg), либо используйте сочетание клавиш **Cmd** + **K** или **Ctrl** + **K**.

    Созданная контрольная точка появится в начале списка **Checkpoints**.
1. Сохраните контрольную точку:
    1. Откройте панель ![checkpoints-panel](../../_assets/datasphere/jupyterlab/checkpoints-panel.svg) **Checkpoints**.
    1. Выберите нужную контрольную точку в списке **Checkpoints** и нажмите кнопку **Pin**.
    1. В открывшемся окне введите имя сохраняемой контрольной точки.
    1. Нажмите кнопку **Save**.

    Сохраненная контрольная точка переместится в список **Pinned checkpoints**.
1. Экспортируйте контрольную точку:
    1. Откройте панель ![checkpoints-panel](../../_assets/datasphere/jupyterlab/checkpoints-panel.svg) **Checkpoints**.
    1. В списке **Pinned checkpoints** выберите контрольную точку.
    1. Нажмите кнопку **Share**.
    1. В открывшемся окне нажмите кнопку **Copy to clipboard**.

    Ссылка на контрольную точку будет скопирована в буфер обмена.
1. В новой версии {{ ml-platform-name }} создайте проект из контрольной точки:
    1. Перейдите на [страницу](https://datasphere.yandex.ru?create-project-from-previous-version=true) создания проекта из контрольной точки.
    1. Задайте параметры проекта:
        * Укажите ссылку на контрольную точку.
        * Введите имя проекта.
        * (Опционально) Введите описание проекта.
        * Выберите [сообщество](../concepts/community.md) для проекта.
    1. Нажмите кнопку **{{ ui-key.yc-ui-datasphere.common.create }}**.

#### См. также {#see-also}

* [Создать проект](./projects/create.md)
* [Создать секрет](./data/secrets.md#create)
* [Создать коннектор S3](./data/connect-to-s3.md)
* [Создать датасет](../concepts/dataset.md#init)
* [{#T}](./deploy/node-create.md)
* [Создать Docker-образ](user-images.md#create)
