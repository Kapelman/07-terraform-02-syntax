# Домашнее задание к занятию "7.2. Облачные провайдеры и синтаксис Terraform."

Зачастую разбираться в новых инструментах гораздо интересней понимая то, как они работают изнутри. 
Поэтому в рамках первого *необязательного* задания предлагается завести свою учетную запись в AWS (Amazon Web Services) или Yandex.Cloud.
Идеально будет познакомится с обоими облаками, потому что они отличаются. 

## Задача 1 (вариант с AWS). Регистрация в aws и знакомство с основами (необязательно, но крайне желательно).

Остальные задания можно будет выполнять и без этого аккаунта, но с ним можно будет увидеть полный цикл процессов. 

AWS предоставляет достаточно много бесплатных ресурсов в первый год после регистрации, подробно описано [здесь](https://aws.amazon.com/free/).
1. Создайте аккаут aws.
1. Установите c aws-cli https://aws.amazon.com/cli/.
1. Выполните первичную настройку aws-sli https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html.
1. Создайте IAM политику для терраформа c правами
    * AmazonEC2FullAccess
    * AmazonS3FullAccess
    * AmazonDynamoDBFullAccess
    * AmazonRDSFullAccess
    * CloudWatchFullAccess
    * IAMFullAccess
1. Добавьте переменные окружения 
    ```
    export AWS_ACCESS_KEY_ID=(your access key id)
    export AWS_SECRET_ACCESS_KEY=(your secret access key)
    ```
1. Создайте, остановите и удалите ec2 инстанс (любой с пометкой `free tier`) через веб интерфейс. 

В виде результата задания приложите вывод команды `aws configure list`.

## Задача 1 (Вариант с Yandex.Cloud). Регистрация в ЯО и знакомство с основами (необязательно, но крайне желательно).

1. Подробная инструкция на русском языке содержится [здесь](https://cloud.yandex.ru/docs/solutions/infrastructure-management/terraform-quickstart).
2. Обратите внимание на период бесплатного использования после регистрации аккаунта. 
3. Используйте раздел "Подготовьте облако к работе" для регистрации аккаунта. Далее раздел "Настройте провайдер" для подготовки
базового терраформ конфига.
4. Воспользуйтесь [инструкцией](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs) на сайте терраформа, что бы 
не указывать авторизационный токен в коде, а терраформ провайдер брал его из переменных окружений.


Решение: 

1. Зарегистрируемся в cloud.yandex.ru и проверим конфигурацию профиля:

```
vagrant@vagrant:~$ yc config list
token: ***********************************
cloud-id: b1g0k29qecug0jk3jt4i
folder-id: b1g5fh031uiok7frafp8
compute-default-zone: ru-central1-a
```

2. Создадим сервисного пользователя
```
vagrant@vagrant:~$ yc iam service-account create --name service-kapelman
id: ajeandk8p5i8tahq66g9
folder_id: b1g5fh031uiok7frafp8
created_at: "2022-09-25T19:05:18.810190572Z"
name: service-kapelman

vagrant@vagrant:~$ yc iam service-account list
+----------------------+------------------+
|          ID          |       NAME       |
+----------------------+------------------+
| aje2e5p4jmfabmosensm | my-robot2        |
| ajeandk8p5i8tahq66g9 | service-kapelman |
+----------------------+------------------+
```

- присвоим новому пользователю полномочия на создание compute ресурсов.
```
vagrant@vagrant:~$ yc resource-manager folder add-access-binding b1g5fh031uiok7frafp8 --role compute.admin --subject serviceAccount:ajeandk8p5i8tahq66g9
done (1s)
```

- создадим ключи для подключения:
```
vagrant@vagrant:~/terraform-02$ yc iam key create --service-account-id ajeandk8p5i8tahq66g9 --folder-name default --output key.json
id: aje020qckjmhu69r4l4u
service_account_id: ajeandk8p5i8tahq66g9
created_at: "2022-09-25T20:09:46.025059068Z"
key_algorithm: RSA_2048
```
- создадим отдельный профиль и присвоим ему необходимые параметры:
```
vagrant@vagrant:~/terraform-02$ yc config profile create 'sa-terraform'
Profile 'sa-terraform' created and activated
```

```
vagrant@vagrant:~$ cat key.json
{
   "id": "ajejgst30k941dc79ism",
   "service_account_id": "ajeandk8p5i8tahq66g9",
   "created_at": "2022-09-25T20:43:39.228304203Z",
   "key_algorithm": "RSA_2048",
   "public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyc1y+6Zpgs0aD2ozL5Ao\n8ukLFBGcDFA22Sh9siUG4Pv2wumAtLGgfeS+ENupT7U42BtkGbaDhwxNBpycSqtd\n2BU0SSvIsmvkqfBt5ooUkhFqMUN0WSmUv6LhtjgiabDWui2rq9KrACWFq4sbvRt/\ns1Ss+lLR7n2TV/y3CwyiJCIgxQmeH6IBxLXDdEwsdSOipjPBazBkk//vcn2Fr20r\n1WwAZ4ZcWLz9PoDcsOzsghotYf5hpUqQr9sK5eqFfYp2MLjnfyHkEE8vJRCANwMu\nk0Afk1ND8IcKPNkGJk2SH3eo+TNTizpdZ/WOe/Oh9bsxgiWlbDCZ2tiYSpPfaSrT\nKQIDAQAB\n-----END PUBLIC KEY-----\n",
   "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDJzXL7pmmCzRoP\najMvkCjy6QsUEZwMUDbZKH2yJQbg+/bC6YC0saB95L4Q26lPtTjYG2QZtoOHDE0G\nnJxKq13YFTRJK8iya+Sp8G3mihSSEWoxQ3RZKZS/ouG2OCJpsNa6Laur0qsAJYWr\nixu9G3+zVKz6UtHufZNX/LcLDKIkIiDFCZ4fogHEtcN0TCx1I6KmM8FrMGST/+9y\nfYWvbSvVbABnhlxYvP0+gNyw7OyCGi1h/mGlSpCv2wrl6oV9inYwuOd/IeQQTy8l\nEIA3Ay6TQB+TU0Pwhwo82QYmTZIfd6j5M1OLOl1n9Y5786H1uzGCJaVsMJna2JhK\nk99pKtMpAgMBAAECggEAH2pq9I8vDMxWOsEbL9Pe9BXggiLNqsMQDtV1X/bQr9S5\n0RUd0sN2SzMBfclcfcqmC0qUVkZqCmuZUCawVBWCegGvDpcQ2uneArCpw0KKukSY\nxguMwNauz/iQ79ekT9TWUMyMVabptQ+iVBbHXjS6OBY+CYg8I0cMWZ2/dypj0YDx\nnYvFx5nfzok3tXzg7KLNoldWrGnlplufN+TLsuKBns+mq1P+6FVnD5QkBQSphW4C\n63tTxx9i88ji0geT+FTv3gfkeDbzcDGxxUs7sT9xQMSzkceTl9/odwxHTKlLpvrK\nbILszMHmTTY5tLJHhlq/uQA48SCv+MOEz3v38w6NaQKBgQD2hKyuNeSwZl+B1Y/j\naOK3GQ32Co0e0G/OnCJLH/SntlDCHi6q7cUw1QqE2k5TkmLqRwqPvPagfZFEpxko\nyQt8VN1mt4B/hsvfVbvCyP1/QBxqdzEiTZyxhDrID7G7pnLTkM38/s9ZbslXw+5q\n+yKJYNGjl/5rgO2Zv4iUFUxhXQKBgQDRkHv3uERaJyWPMvN6Jv7vKTUTNZDXRyaN\nXzUJXP5H5JxEK8rvJe6xWc/7PsvmWuhYe9arUUVFKnN2QSqdLlHH0FimGe8Pkoe3\nXUnM3Jx/6Imo7aJngYfW/0iIT8c58U+SfiEUkJdARC8qAwICjv7UOREDY2rS0Tey\nxzHmWDwgPQKBgG9DBFkAc/31xodn5zBhZ2nyQe3ZZ0YQF1Zt+8BiZN7JF3v1eWSm\nOgjHLp81lIJ9oG1SsP6c78cRxV3x+RYCX0+3UdIJYlKseRmMrVjFtDwZqHmY4DE2\nTFGGd61SAArMnijEw2O7ccRQj0kwYkwgmr7cVuH6ONc2coag/rivQDD1AoGBAIJ/\n0eLaGZ52YDpDRUFdBUYTSBzVL4QPp59DmXhiM2q7nuAI0U+JNJG2VwCjA0BIfgWT\n4INAkb1XiR0ryYil7oFaacnNvoPZALCb5DgxbTdtrEPI72g7TkcBI77Wxz562c1k\nw97Vh4qaqzAjPV4wg9nOS5zrjPsJFAE9cAJ8Eb0VAoGBAJp2tta6h5Z5Hz++sGa7\nXv5Qpu9A094n2Rpgqp7yGB85N4Rvaz7cmC6tPZ37RsIS33SjEB+Epl0ocUz407Et\nwuxJpbhCMP/tGI3FV/wybuRO6fgaIVUQxLxiZRE9Ev6wXowDwRMKFb4Pe0HhI3Gd\nN245PJTVr1Vov4Mv1mPqhv1Z\n-----END PRIVATE KEY-----\n"
}vagrant@vagrant:~$ yc config set service-account-key key.json
vagrant@vagrant:~$ yc config set cloud-id b1g0k29qecug0jk3jt4i
vagrant@vagrant:~$ yc config set folder-id b1g5fh031uiok7frafp8
```
- Создадим необходимые переменные окружения
```
vagrant@vagrant:~$ export YC_TOKEN=$(yc iam create-token)
vagrant@vagrant:~$ export YC_CLOUD_ID=$(yc config get cloud-id)
vagrant@vagrant:~$ export YC_FOLDER_ID=$(yc config get folder-id)
vagrant@vagrant:~$ nano ~/.terraformrc
```



- дополним файл main.tf следующей командой:
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  zone = "ru-central1-a"
}
```
- запустим terraform init
```
vagrant@vagrant:~/terraform-02$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of yandex-cloud/yandex...
- Installing yandex-cloud/yandex v0.80.0...
- Installed yandex-cloud/yandex v0.80.0 (unauthenticated)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Incomplete lock file information for providers
│
│ Due to your customized provider installation methods, Terraform was forced to calculate lock file checksums locally for the following providers:
│   - yandex-cloud/yandex
│
│ The current .terraform.lock.hcl file only includes checksums for linux_amd64, so Terraform running on another platform will fail to install these providers.
│
│ To calculate additional checksums for another platform, run:
│   terraform providers lock -platform=linux_amd64
│ (where linux_amd64 is the platform to generate)
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```

## Задача 2. Создание aws ec2 или yandex_compute_instance через терраформ. 

1. В каталоге `terraform` вашего основного репозитория, который был создан в начале курсе, создайте файл `main.tf` и `versions.tf`.
2. Зарегистрируйте провайдер 
   1. для [aws](https://registry.terraform.io/providers/hashicorp/aws/latest/docs). В файл `main.tf` добавьте
   блок `provider`, а в `versions.tf` блок `terraform` с вложенным блоком `required_providers`. Укажите любой выбранный вами регион 
   внутри блока `provider`.
   2. либо для [yandex.cloud](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs). Подробную инструкцию можно найти 
   [здесь](https://cloud.yandex.ru/docs/solutions/infrastructure-management/terraform-quickstart).
3. Внимание! В гит репозиторий нельзя пушить ваши личные ключи доступа к аккаунту. Поэтому в предыдущем задании мы указывали
их в виде переменных окружения. 
4. В файле `main.tf` воспользуйтесь блоком `data "aws_ami` для поиска ami образа последнего Ubuntu.  
5. В файле `main.tf` создайте рессурс 
   1. либо [ec2 instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance).
   Постарайтесь указать как можно больше параметров для его определения. Минимальный набор параметров указан в первом блоке 
   `Example Usage`, но желательно, указать большее количество параметров.
   2. либо [yandex_compute_image](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/compute_image).
6. Также в случае использования aws:
   1. Добавьте data-блоки `aws_caller_identity` и `aws_region`.
   2. В файл `outputs.tf` поместить блоки `output` с данными об используемых в данный момент: 
       * AWS account ID,
       * AWS user ID,
       * AWS регион, который используется в данный момент, 
       * Приватный IP ec2 инстансы,
       * Идентификатор подсети в которой создан инстанс.  
7. Если вы выполнили первый пункт, то добейтесь того, что бы команда `terraform plan` выполнялась без ошибок. 


В качестве результата задания предоставьте:
1. Ответ на вопрос: при помощи какого инструмента (из разобранных на прошлом занятии) можно создать свой образ ami?
1. Ссылку на репозиторий с исходной конфигурацией терраформа.  
 
Решение:

- заполним файл
```
vagrant@vagrant:~/terraform-02$ cat main.tf
provider "yandex" {
  token                    = "$YC_TOKEN"
  cloud_id                 = "$YC_CLOUD_ID"
  folder_id                = "$YC_FOLDER_ID"
  zone                     = "ru-central1-a"
}

resource "yandex_compute_image" "foo-image" {
  name       = "my-custom-image"
  source_url = "https://storage.yandexcloud.net/lucky-images/kube-it.img"
}


resource "yandex_compute_instance" "vm" {
  name        = "test"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"

  resources {
    cores  = 2
    memory = 4
  }

  boot_disk {
    initialize_params {
      image_id = "${yandex_compute_image.foo-image.id}"
    }
  }

  network_interface {
    subnet_id = "${yandex_vpc_subnet.foo.id}"
  }

  metadata = {
    foo      = "bar"
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

resource "yandex_vpc_network" "foo" {}

resource "yandex_vpc_subnet" "foo" {
  v4_cidr_blocks = ["10.2.0.0/16"]
  zone       = "ru-central1-a"
  network_id = "${yandex_vpc_network.foo.id}"
}

vagrant@vagrant:~/terraform-02$ ls -al
total 28
drwxrwxr-x  3 vagrant vagrant 4096 Sep 26 21:37 .
drwxr-xr-x 37 vagrant vagrant 4096 Sep 26 21:24 ..
-rw-------  1 vagrant vagrant 2402 Sep 26 21:10 key.json
-rw-rw-r--  1 vagrant vagrant 1012 Sep 26 21:37 main.tf
drwxr-xr-x  3 vagrant vagrant 4096 Sep 25 20:49 .terraform
-rw-r--r--  1 vagrant vagrant  258 Sep 25 20:49 .terraform.lock.hcl
-rw-rw-r--  1 vagrant vagrant  130 Sep 26 21:01 versions.tf
```
- выполниим команду plan

```
vagrant@vagrant:~/terraform-02$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_image.foo-image will be created
  + resource "yandex_compute_image" "foo-image" {
      + created_at      = (known after apply)
      + folder_id       = (known after apply)
      + id              = (known after apply)
      + min_disk_size   = (known after apply)
      + name            = "my-custom-image"
      + os_type         = (known after apply)
      + pooled          = (known after apply)
      + product_ids     = (known after apply)
      + size            = (known after apply)
      + source_disk     = (known after apply)
      + source_family   = (known after apply)
      + source_image    = (known after apply)
      + source_snapshot = (known after apply)
      + source_url      = "https://storage.yandexcloud.net/lucky-images/kube-it.img"
      + status          = (known after apply)
    }

  # yandex_compute_instance.vm will be created
  + resource "yandex_compute_instance" "vm" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + metadata                  = {
          + "foo"      = "bar"
          + "ssh-keys" = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtnMd57eG81JsenyqtQJAL54D1++KL15Fy+OmgbQdJoAKCnA5TognKsmw/Xy9xCYyOmF8Ydi41YUNZY61iUq4qR3htkwCSu7br/9a/tMAyOJPVib5IIjzQX5tO970LpHOWaClTOvhmm63qEVbSbmGUpDNhPxaAOR2JGmapB6+L+EDRvY9YgPizIB1RhAMS5ryaTaY5/DJQq5rgB7dd7GpZ7Gkn8H1ELHJaMYarD9K/K/Avu7R780a8FuUgjvDdH+qncyVMkUpS+U8b4dXCsAUWj0BpPimXxVGnNup0oN+Q+yb+qC2IV8+pdWCexgjyadKB35M5UC7Wee2mqnaXMcep vagrant@vagrant
            EOT
        }
      + name                      = "test"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = "ru-central1-a"

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = (known after apply)
              + name        = (known after apply)
              + size        = (known after apply)
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = (known after apply)
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.foo will be created
  + resource "yandex_vpc_network" "foo" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = (known after apply)
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.foo will be created
  + resource "yandex_vpc_subnet" "foo" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = (known after apply)
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.2.0.0/16",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
---
```
### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
