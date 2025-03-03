# Creating {{ MG }} clusters

A {{ MG }} cluster consists of one or more database hosts you can configure replication between. Replication is enabled by default in any cluster consisting of more than one host (the primary host accepts write requests and asynchronously duplicates changes in the secondary hosts).


{% note info %}

* The number of hosts you can create together with a {{ MG }} cluster depends on the selected [disk type](../concepts/storage.md#storage-type-selection) and [host class](../concepts/instance-types.md#available-flavors).
* Available disk types [depend](../concepts/storage.md) on the selected [host class](../concepts/instance-types.md).

{% endnote %}



{% list tabs %}

- Management console

   To create a cluster:

   1. In the [management console]({{ link-console-main }}), select the folder where you want to create a DB cluster.

   1. Select **{{ mmg-name }}**.

   1. Click **Create cluster**.

   1. Under **Basic parameters**:

      * Name the cluster in the **Cluster name** field. It must be unique within the folder.
      * (optional) Enter a cluster **description**.
      * Select the environment where you want to create the cluster (you cannot change the environment once the cluster is created):

         * `PRODUCTION`: For stable versions of your apps.
         * `PRESTABLE`: For testing, including the {{ mmg-short-name }} service itself. The prestable environment is updated first with new features, improvements, and bug fixes. However, not every update ensures backward compatibility.

      * Specify the DBMS version.

   1. {% include [mmg-settings-host-class](../../_includes/mdb/mmg/settings-host-class.md) %}

   1. Under **Storage size**:

      * Select the [disk type](../concepts/storage.md).

         {% include [storages-step-settings](../../_includes/mdb/settings-storages.md) %}

      * Select the size of storage to be used for data and backups. For more information about how backups take up storage space, see [{#T}](../concepts/backup.md).

   1. Under **Database**, specify the DB attributes:

      * DB name.
      * Username.
      * User password. Minimum of 8 characters.

   
   1. Under **Network settings**, select:

      * Cloud network for the cluster.
      * Security groups for the cluster's network traffic. You may also need to [set up security groups](connect/index.md#configuring-security-groups) to connect to the cluster.

         {% note info %}

         {% include [security-groups-note](../../_includes/vpc/security-groups-note-services.md) %}

         {% endnote %}


   1. Under **Hosts**, add the DB hosts created with the cluster:

      
      * Click **Add host**.
      * Select an [availability zone](../../overview/concepts/geo-scope.md).
      * Select the [subnet](../../vpc/concepts/network.md#subnet) in the specified availability zone. If there is no subnet, create one.
      * If the host must be available outside {{ yandex-cloud }}, enable **Public access**.


      To ensure fault tolerance, you need at least 3 hosts for `local-ssd` and `network-ssd-nonreplicated` disk types. For more information, see [Storage](../concepts/storage.md).

      By default, hosts are created in different availability zones. For details, see about [host management](hosts.md).

   1. Configure additional cluster settings, if required:

      {% include [mmg-extra-settings](../../_includes/mdb/mmg-extra-settings.md) %}

   1. Configure the [DBMS settings](../concepts/settings-list.md#dbms-cluster-settings), if required.

      {% include [mmg-settings-dependence](../../_includes/mdb/mmg/note-info-settings-dependence.md) %}

   1. Click **Create cluster**.

- CLI

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To create a cluster:

   
   1. Check whether the folder has any subnets for the cluster hosts:

      ```
      yc vpc subnet list
      ```

      If there are no subnets in the folder, [create the required subnets](../../vpc/operations/subnet-create.md) in {{ vpc-short-name }}.


   1. View a description of the create cluster CLI command:

      ```
      {{ yc-mdb-mg }} cluster create --help
      ```

   1. Specify the cluster parameters in the create command (only some of the supported parameters are given in the example):

      
      ```bash
      {{ yc-mdb-mg }} cluster create \
         --name <cluster name> \
         --environment=<environment, prestable or production> \
         --network-name <network name> \
         --host zone-id=<availability zone>,subnet-id=<subnet ID> \
         --mongod-resource-preset <host class> \
         --user name=<username>,password=<user password> \
         --database name=<database name> \
         --mongod-disk-type <network-hdd | network-ssd | local-ssd | network-ssd-nonreplicated> \
         --mongod-disk-size <storage size in GB> \
         --deletion-protection=<deletion protection for the cluster: true or false>
      ```

      You need to specify `subnet-id` if the selected availability zone has two or more subnets.


      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-db.md) %}

- {{ TF }}

   {% include [terraform-definition](../../_tutorials/terraform-definition.md) %}

   
   If you do not have {{ TF }} yet, [install it and configure the provider](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).


   To create a cluster:

   1. In the configuration file, describe the parameters of the resources you want to create:

      * Database cluster: Description of the cluster and its hosts.

      * {% include [Terraform network description](../../_includes/mdb/terraform/network.md) %}

      * {% include [Terraform subnet description](../../_includes/mdb/terraform/subnet.md) %}

      Example of the configuration file structure:

      
      
      ```hcl
      resource "yandex_mdb_mongodb_cluster" "<cluster name>" {
        name                = "<cluster name>"
        environment         = "<environment: PRESTABLE or PRODUCTION>"
        network_id          = "<network ID>"
        security_group_ids  = [ "<list of security groups>" ]
        deletion_protection = <cluster deletion protection: true or false>

        cluster_config {
          version = "<{{ MG }} version: {{ versions.tf.str }}>"
        }

        database {
          name = "<database name>"
        }

        user {
          name     = "<username>"
          password = "<user password>"
          permission {
            database_name = "<database name>"
            roles         = [ "<list of user roles>" ]
          }
        }

        resources_mongod {
          resource_preset_id = "<host class>"
          disk_type_id       = "<disk type>"
          disk_size          = <storage size, GB>
        }

        host {
          zone_id   = "<availability zone>"
          subnet_id = "<subnet ID>"
        }
      }

      resource "yandex_vpc_network" "<network name>" { name = "<network name>" }

      resource "yandex_vpc_subnet" "<subnet name>" {
        name           = "<subnet name>"
        zone           = "<availability zone>"
        network_id     = "<network ID>"
        v4_cidr_blocks = ["<range>"]
      }
      ```




      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-db.md) %}

      {% include [Maintenance window](../../_includes/mdb/mmg/terraform/maintenance-window.md) %}

      For more information on resources that you can create with {{ TF }}, see the [provider documentation]({{ tf-provider-mmg }}).

   2. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   3. Create a cluster.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      After this, all required resources will be created in the specified folder and the IP addresses of the VMs will be displayed in the terminal. You can check that the resources are there and their settings are correct using the [management console]({{ link-console-main }}).

      {% include [Terraform timeouts](../../_includes/mdb/mmg/terraform/timeouts.md) %}

- API

   To create a cluster, use the [create](../api-ref/Cluster/create.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Create](../api-ref/grpc/cluster_service.md#Create) gRPC API call and provide the following in the request:

   * ID of the folder where the cluster should be placed, in the `folderId` parameter.
   * Cluster name in the `name` parameter.
   * Cluster environment in the `environment` parameter.
   * Network ID in the `networkId` parameter.
   * Cluster configuration in the `configSpec` parameter.
   * Configuration of the cluster hosts in one or more `hostSpecs` parameters.

   
   * [Security group](../concepts/network.md#security-groups) identifiers in the `securityGroupIds` parameter.


   * Database configuration in one or more `databaseSpecs` parameters.
   * User settings in one or more `userSpecs` parameters.
   * Cluster deletion protection settings in the `deletionProtection` parameter.

      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-data.md) %}

{% endlist %}


{% note warning %}

If you specified security group IDs when creating a cluster, you may also need to [configure security groups](connect/index.md#configuring-security-groups) to connect to the cluster.

{% endnote %}


## Examples {#examples}

### Creating a single-host cluster {#creating-a-single-host-cluster}

{% list tabs %}

- CLI

   To create a cluster with a single host, provide a single `--host` parameter.

   Create a {{ mmg-name }} cluster with test characteristics:

   
   * Named `mymg`.
   * In the `production` environment.
   * In the `{{ network-name }}` network.
   * In the security group with the ID `{{ security-group }}`.
   * With one `{{ host-class }}` host in the `b0rcctk2rvtr8efcch64` subnet in the `{{ region-id }}-a` availability zone.
   * With 20 GB of network SSD storage (`{{ disk-type-example }}`).
   * With one user, `user1`, with the password `user1user1`.
   * With one database, `db1`.
   * With protection against accidental cluster deletion.


   Run the following command:

   
   ```bash
   {{ yc-mdb-mg }} cluster create \
     --name mymg \
     --environment production \
     --network-name {{ network-name }} \
     --security-group-ids {{ security-group }} \
     --mongod-resource-preset {{ host-class }} \
     --host zone-id={{ region-id }}-a,subnet-id=b0rcctk2rvtr8efcch64 \
     --mongod-disk-size 20 \
     --mongod-disk-type {{ disk-type-example }} \
     --user name=user1,password=user1user1 \
     --database name=db1 \
     --deletion-protection=true
   ```


- {{ TF }}

   Create a {{ mmg-name }} cluster and a network for it with test characteristics:

   * Name: `mymg`.
   * Version: `{{ versions.tf.latest }}`.
   * Environment `PRODUCTION`.
   * Cloud ID: `{{ tf-cloud-id }}`.
   * Folder ID: `{{ tf-folder-id }}`.
   * Network: `mynet`.
   * Host class: `{{ host-class }}`.
   * Number of `host` blocks: 1.
   * Subnet: `mysubnet`. Network settings:

      * Availability zone: `{{ region-id }}-a`.
      * Range: `10.5.0.0/24`.

      * Security group: `mymg-sg`. The group rules allow TCP connections to the cluster from the internet via port `{{ port-mmg }}`.
   * SSD network storage: `{{ disk-type-example }}`.
   * Storage size: 20 GB.
   * User: `user1`.
   * Password: `user1user1`.
   * Database: `db1`.
   * Protection against accidental cluster deletion: Enabled.

   Configuration file for a single-host cluster:

   
   
   ```hcl
   resource "yandex_mdb_mongodb_cluster" "mymg" {
     name                = "mymg"
     environment         = "PRODUCTION"
     network_id          = yandex_vpc_network.mynet.id
     security_group_ids  = [ yandex_vpc_security_group.mymg-sg.id ]
     deletion_protection = true

     cluster_config {
       version = "{{ versions.tf.latest }}"
     }

     database {
       name = "db1"
     }

     user {
       name     = "user1"
       password = "user1user1"
       permission {
         database_name = "db1"
       }
     }

     resources_mongod {
       resource_preset_id = "{{ host-class }}"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 20
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
     }
   }

   resource "yandex_vpc_network" "mynet" {
     name = "mynet"
   }

   resource "yandex_vpc_security_group" "mymg-sg" {
     name       = "mymg-sg"
     network_id = yandex_vpc_network.mynet.id

     ingress {
       description    = "MongoDB"
       port           = {{ port-mmg }}
       protocol       = "TCP"
       v4_cidr_blocks = [ "0.0.0.0/0" ]
     }
   }

   resource "yandex_vpc_subnet" "mysubnet" {
     name           = "mysubnet"
     zone           = "{{ region-id }}-a"
     network_id     = yandex_vpc_network.mynet.id
     v4_cidr_blocks = ["10.5.0.0/24"]
   }
   ```




{% endlist %}

### Creating sharded clusters {#creating-a-sharded-cluster}

You can create {{ mmg-name }} clusters with [standard](#std-sharding) or [advanced](#adv-sharding) sharding. For more information about sharding types, see [{#T}](../concepts/sharding.md#shard-management).

#### Standard sharding {#std-sharding}

Create a {{ mmg-name }} cluster and a network for it with multiple hosts:

* One `MONGOD` host.
* Three `MONGOINFRA` hosts.

Cluster test characteristics:

* Name: `mymg`.
* Environment `PRODUCTION`.
* Protection against accidental cluster deletion: Enabled.
* Version: `{{ versions.tf.latest }}`.
* Database: `db1`.
* User: `user1`.
* Password: `user1user1`.
* `MONGOD` host class: `{{ host-class }}`.
* `MONGOINFRA` host class: `c3-c2-m4`.
* SSD network storage: `{{ disk-type-example }}`.
* Storage size: 10 GB.
* Number of `host` blocks: 4. For each of them, set the host type: `mongod` or `mongoinfra`.

Network specifications:

* Network: `mynet`.
* Security group: `mymg-sg`. The group rules allow TCP connections to the cluster from the internet via port `{{ port-mmg }}`.
* Subnet: `mysubnet`.
* Availability zone: `{{ region-id }}-a`.
* Range: `10.5.0.0/24`.

{% list tabs %}

- {{ TF }}

   Configuration file for a cluster with standard sharding:

   ```hcl
   resource "yandex_mdb_mongodb_cluster" "mymg" {
     name                = "mymg"
     environment         = "PRODUCTION"
     network_id          = yandex_vpc_network.mynet.id
     security_group_ids  = [ yandex_vpc_security_group.mymg-sg.id ]
     deletion_protection = true

     cluster_config {
       version = "{{ versions.tf.latest }}"
     }

     database {
       name = "db1"
     }

     user {
       name     = "user1"
       password = "user1user1"
       permission {
         database_name = "db1"
       }
     }

     resources_mongod {
       resource_preset_id = "{{ host-class }}"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 10
     }

     resources_mongoinfra {
       resource_preset_id = "c3-c2-m4"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 10
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongod"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongoinfra"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongoinfra"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongoinfra"
     }

   resource "yandex_vpc_network" "mynet" {
     name = "mynet"
   }

   resource "yandex_vpc_security_group" "mymg-sg" {
     name       = "mymg-sg"
     network_id = yandex_vpc_network.mynet.id

     ingress {
       description    = "MongoDB"
       port           = {{ port-mmg }}
       protocol       = "TCP"
       v4_cidr_blocks = [ "0.0.0.0/0" ]
     }
   }

   resource "yandex_vpc_subnet" "mysubnet" {
     name           = "mysubnet"
     zone           = "{{ region-id }}-a"
     network_id     = yandex_vpc_network.mynet.id
     v4_cidr_blocks = ["10.5.0.0/24"]
   }
   ```

{% endlist %}

#### Advanced sharding {#adv-sharding}

Create a {{ mmg-name }} cluster and a network for it with multiple hosts:

* One `MONGOD` host.
* Two `MONGOS` hosts.
* Three `MONGOCFG` hosts.

Cluster test characteristics:

* Name: `mymg`.
* Environment `PRODUCTION`.
* Protection against accidental cluster deletion: Enabled.
* Version: `{{ versions.tf.latest }}`.
* Database: `db1`.
* User: `user1`.
* Password: `user1user1`.
* Host class: `{{ host-class }}`.
* SSD network storage: `{{ disk-type-example }}`.
* Storage size: 10 GB.
* Number of `host` blocks: 6. For each of them, set the host type: `mongod`, `mongos`, or `mongocfg`.

Network specifications:

* Network: `mynet`.
* Security group: `mymg-sg`. The group rules allow TCP connections to the cluster from the internet via port `{{ port-mmg }}`.
* Subnet: `mysubnet`.
* Availability zone: `{{ region-id }}-a`.
* Range: `10.5.0.0/24`.

{% list tabs %}

- {{ TF }}

   Configuration file for a cluster with advanced sharding:

   ```hcl
   resource "yandex_mdb_mongodb_cluster" "mymg" {
     name                = "mymg"
     environment         = "PRODUCTION"
     network_id          = yandex_vpc_network.mynet.id
     security_group_ids  = [ yandex_vpc_security_group.mymg-sg.id ]
     deletion_protection = true

     cluster_config {
       version = "{{ versions.tf.latest }}"
     }

     database {
       name = "db1"
     }

     user {
       name     = "user1"
       password = "user1user1"
       permission {
         database_name = "db1"
       }
     }

     resources_mongod {
       resource_preset_id = "{{ host-class }}"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 10
     }

     resources_mongos {
       resource_preset_id = "{{ host-class }}"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 10
     }

     resources_mongocfg {
       resource_preset_id = "{{ host-class }}"
       disk_type_id       = "{{ disk-type-example }}"
       disk_size          = 10
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongod"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongos"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongos"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongocfg"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongocfg"
     }

     host {
       zone_id   = "{{ region-id }}-a"
       subnet_id = yandex_vpc_subnet.mysubnet.id
       type      = "mongocfg"
     }
   }

   resource "yandex_vpc_network" "mynet" {
     name = "mynet"
   }

   resource "yandex_vpc_security_group" "mymg-sg" {
     name       = "mymg-sg"
     network_id = yandex_vpc_network.mynet.id

     ingress {
       description    = "MongoDB"
       port           = {{ port-mmg }}
       protocol       = "TCP"
       v4_cidr_blocks = [ "0.0.0.0/0" ]
     }
   }

   resource "yandex_vpc_subnet" "mysubnet" {
     name           = "mysubnet"
     zone           = "{{ region-id }}-a"
     network_id     = yandex_vpc_network.mynet.id
     v4_cidr_blocks = ["10.5.0.0/24"]
   }
   ```

{% endlist %}
