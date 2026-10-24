---
title: OpenTofu Course Notes
source: https://github.com/abdelghafarabdelmordy/create-vm-terraform-vsphere
tags:
  - opentofu
  - terraform
  - iac
  - devops
  - aws
aliases:
  - Tofu Notes
  - OpenTofu Cheatsheet
created: 2026-10-19
---
https://github.com/Azure/aztfexport


# OpenTofu Course Notes


## Basic Commands

```bash
tofu init                          # initialize (download providers)
tofu plan                          # preview the execution plan
tofu apply || tofu apply -auto-approve
tofu show                          # print current state in human-readable format
tofu destroy || tofu destroy -auto-approve    # مسح
```

- أول `apply` ينشئ ملف الحالة `terraform.tfstate` (+ نسخة احتياطية `terraform.tfstate.backup`) في نفس مجلد الـ configuration.
- سير العمل الأساسي دائمًا: `init → plan → apply`.

**بنية الملفات المعتادة:**

```
main.tf
variables.tf
outputs.tf
provider.tf
```

## First Resource — local_file

```hcl
resource "local_file" "pet" {
  filename        = "/root/pets.txt"
  content         = "We love pets!"
  file_permission = "0700"
}
```

## Multiple Providers (alias)

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  region = "ca-central-1"
  alias  = "central"
}
```

```hcl
resource "aws_key_pair" "alpha" {
  key_name   = "alpha"
  public_key = "ssh-rsa AAAA...alpha@a-server"
}

resource "aws_key_pair" "beta" {
  key_name   = "beta"
  public_key = "ssh-rsa AAAA...alpha@a-server"
  provider   = aws.central    # استخدام الـ provider المُعرّف بـ alias
}
```

## Input Variables

**طرق تمرير القيم:**

1. `default` داخل بلوك الـ variable.
2. Interactive prompt عند `apply` (`enter a value:`).
3. سطر الأوامر: `tofu apply -var "ami=ami-123" -var "instance_type=t2.micro"`.
4. متغيرات البيئة: `export TF_VAR_instance_type="t3.micro"`.

```hcl
variable "instance_type" {
  default     = "t2.micro"
  description = "Size of EC2"
  type        = string
}

variable "ami" {
  default     = "ami-0123"
  description = "Type of AMI to use"
  type        = string
}
```

### Variable Types

```hcl
variable "count" {
  default     = 2
  type        = number
  description = "Count of VMs"
}

variable "monitoring" {
  default     = true
  type        = bool
  description = "Enable detailed monitoring"
}

# list
variable "servers" {
  default = ["web1", "web2", "web3"]
  type    = list(string)
}

variable "prefix" {
  default = [1, 2, 3]
  type    = list(number)
}

# map
variable "instance_type" {
  default = {
    "production"  = "m5.large"
    "development" = "t2.micro"
  }
  type = map(string)
}

variable "servers_count" {
  default = {
    "web"   = 3
    "db"    = 1
    "agent" = 2
  }
  type = map(number)
}

# object
variable "bella" {
  type = object({
    name         = string
    color        = string
    age          = number
    food         = list(string)
    favorite_pet = bool
  })
  default = {
    name         = "bella"
    color        = "brown"
    age          = 7
    food         = ["fish", "chicken", "turkey"]
    favorite_pet = true
  }
}

# tuple
variable "web" {
  type    = tuple([string, number, bool])
  default = ["web1", 7, true]
}
```

> [!tip] list vs map في الوصول
> - list: `var.servers[0]` (بالـ index)
> - map: `var.instance_type["development"]` (بالمفتاح)

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  tags = {
    Name = var.servers[0]
  }
}

resource "aws_instance" "production" {
  ami           = var.ami
  instance_type = var.instance_type["development"]
}
```

## Resource Attributes & Dependencies

- عند إنشاء أي resource، يقوم OpenTofu بتسجيل مجموعة من الـ attributes المُصدَّرة (مثل `id`, `arn`, `public_ip`...) — راجعها بـ `tofu show`.
- **Reference Expression**: `resource_type.resource_name.attribute` — مثال: `aws_key_pair.alpha.key_name`.

```hcl
resource "aws_key_pair" "alpha" {
  key_name   = "alpha"
  public_key = "ssh-rsa AAAA..."
}

resource "aws_instance" "cerberus" {
  ami           = var.ami
  instance_type = var.instance_type
  key_name      = aws_key_pair.alpha.key_name   # implicit dependency
}
```

- **Implicit dependency**: تُستنتج تلقائيًا من استخدام attribute لمورد آخر → يُنشأ keypair أولاً ثم الـ instance (والحذف بالترتيب العكسي).
- **Explicit dependency**: بلا reference — استخدم `depends_on`:

```hcl
resource "aws_instance" "db" {
  ami           = var.db_ami
  instance_type = var.db_instance_type
}

resource "aws_instance" "web" {
  ami           = var.web_ami
  instance_type = var.web_instance_type
  depends_on    = [ aws_instance.db ]   # list وليس map
}
```

## Resource Targeting

```bash
tofu apply -target=random_string.server-suffix
```

- يطبق التغييرات على مورد محدد فقط (بصيغة `resourceType.resourceName`) دون باقي الخطة.

> [!warning]
> استخدم `-target` في حالات نادرة فقط (تصحيح خطأ أو تغيير عاجل) — OpenTofu يحذرك من أن التغييرات المطبقة قد تكون غير مكتملة.

```hcl
resource "random_string" "server-suffix" {
  length  = 6
  upper   = false
  special = false
}

resource "aws_instance" "web" {
  ami = "ami-0133"
  tags = {
    Name = "web-${random_string.server-suffix.id}"   # interpolation
  }
}
```

## Output Variables

```hcl
resource "aws_instance" "cerberus" {
  ami           = var.ami
  instance_type = var.instance_type
}

output "pub_ip" {
  value       = aws_instance.cerberus.public_ip
  description = "Print the public IPv4 address"
}

# القالب العام
output "<name>" {
  value = <reference expression>
  <optional arguments>   # description, sensitive...
}
```

```bash
$ tofu apply
pub_ip = 54.214.145.69          # يظهر بعد كل apply

$ tofu output pub_ip
54.214.145.69                   # قيمة output واحد فقط
```

- أفضل استخدام للـ outputs: عرض سريع لتفاصيل الموارد، أو تمريرها لأدوات IaC أخرى (Ansible, scripts...).

## Example — random_pet + outputs (Lab)

```hcl
resource "random_pet" "my-pet" {
  length = var.length
}

output "pet-name" {
  value       = random_pet.my-pet.id
  description = "Record the value of pet ID generated by the random_pet resource"
}

resource "local_file" "welcome" {
  filename = "/root/message.txt"
  content  = "Welcome to Kodekloud."
}

output "welcome_message" {
  value = local_file.welcome.content
}
```

```hcl
variable "prefix"    { default = "Mrs" }
variable "separator" { default = "." }
variable "length"    { default = "1" }
```

---

## Resource Attributes (time_static example)

[Time provider docs](https://registry.terraform.io/providers/hashicorp/time/latest/docs/resources/static)

```hcl
resource "time_static" "time_update" {}

resource "local_file" "time" {
  filename = "/root/time.txt"
  content  = "Time stamp of this file is ${time_static.time_update.id}"
}
```

```text
$ tofu show
# time_static.time_update:
resource "time_static" "time_update" {
    day     = 21
    hour    = 12
    id      = "2025-06-21T12:25:57Z"
    minute  = 25
    month   = 6
    rfc3339 = "2025-06-21T12:25:57Z"
    second  = 57
    unix    = 1750508757
    year    = 2025
}
```

- الـ logical resources (مثل `time_static`, `random_pet`, `tls_private_key`) تُسجَّل في الـ state فقط دون إنشاء شيء حقيقي في السحابة.

## OpenTofu State

### المفاهيم الأساسية

- **state file** = سجل كامل بالبنية التحتية المنشأة (JSON) — مصدر الحقيقة عند `plan` و`apply`.
- عند `plan`/`apply`: يُحدَّث الـ state أولاً (refresh) بمقارنة الواقع بالإعدادات، ثم تُبنى خطة التنفيذ.
- `tofu apply -refresh=false` — تخطي الـ refresh (مفيد مع آلاف الموارد؛ غير مستحسن لأنه قد يسبب عدم اتساق).
- الـ state **لا يمكن تعطيله** — إلزامي لعمل OpenTofu.
- يتتبع أيضًا **dependencies** بين الموارد → يحدد ترتيب الإنشاء/الحذف.

### Best Practices

> [!warning] حساسية البيانات
> - الـ state يحتوي معلومات حساسة (SSH keys, passwords) كنص صريح (plain text).
> - **لا ترفعه أبدًا** على Git/GitLab/Bitbucket — استخدم remote backend (S3, GCS, Azure Blob).
> - **لا تعدّله يدويًا** — استخدم أوامر `tofu state`.

## Remote State & State Locking

```hcl
terraform {
  backend "s3" {
    bucket         = "kodekloud-opentofu-state-bucket01"
    key            = "finance/terraform.tfstate"
    region         = "us-west-1"
    dynamodb_table = "state-locking"   # optional → enables state locking
  }
}
```

**لماذا remote state؟**

- التعاون الجماعي: state محلي لا يصلح لفريق عمل.
- **State locking**: أثناء العملية، يُقفل الملف فلا يمكن لعملية ثانية تعديله في نفس الوقت (Git لا يدعم locking → تعارضات وتلف في الملف).
- تشفير at rest/in transit في التخزين.
- بعد تغيير الـ backend: شغّل `tofu init` لإعادة التهيئة — سيقترح نسخ الـ state المحلي إلى الـ backend، ثم يمكن حذف الملف المحلي.
- مع S3-compatible محلي مثل MinIO نضيف وسائط إضافية (`endpoint`, `force_path_style`, `skip_*`):

```hcl
terraform {
  backend "s3" {
    key                        = "terraform.tfstate"
    region                     = "us-east-1"
    bucket                     = "remote-state"
    endpoint                   = "http://172.16.238.105:9000"
    force_path_style           = true
    skip_credentials_validation = true
    skip_metadata_api_check    = true
    skip_region_validation     = true
  }
}
```

## State Commands

```bash
tofu state list                                  # كل الـ resource addresses
tofu state list aws_s3_bucket.cerberus-finance   # بالفلترة
tofu state show aws_s3_bucket.cerberus-finance   # attributes مورد واحد
tofu state mv random_pet.super_pet_1 random_pet.ultra_pet   # rename
tofu state pull > backup.tfstate                 # تنزيل الـ state (يصلح للـ jq)
tofu state rm local_file.hall_of_fame            # إخراج مورد من الإدارة
tofu state push backup.tfstate                   # استبدال الـ remote (خطير!)
```

- `state rm` **لا يدمّر** المورد — فقط يخرجه من إدارة الـ state (احذف بلوكه من الإعدادات أيضًا).
- `state push` بحذر شديد — قد يفسد الإعدادات؛ `-force` غير مستحسن.

## Common Commands

```bash
tofu validate     # فحص صياغة الإعدادات (syntax فقط — لا يضمن صحة القيم)
tofu fmt          # تنسيق الكود بصيغة قياسية
tofu show -json   # الحالة بصيغة JSON
tofu providers    # قائمة الـ providers المستخدمة
tofu refresh      # مزامنة الـ state مع الواقع (لا يعدّل أي بنية تحتية)
tofu graph        # رسم dependencies بصيغة DOT (يُعرض بـ GraphViz)
```

> [!note]
> `validate` قد ينجح بينما `apply` يفشل — الفحص عام ولا يتحقق من صحة قيم الوسائط لمورد معين.

- sub-commands صالحة لـ `tofu providers`: مثل `mirror` (وليس list/status/update).

## LifeCycle Rules

```hcl
resource "aws_instance" "cerberus" {
  ami           = "ami-02cff456777cd"
  instance_type = "m5.large"
  tags = {
    Name = "Cerberus-Webserver"
  }

  # أثناء الاستبدال: أنشئ الجديد قبل حذف القديم
  lifecycle {
    create_before_destroy = true
  }
}
```

```hcl
# لو مش عايز احزف اى شى
lifecycle {
  prevent_destroy = true
}
```

```hcl
# لو عايز اتجاهل اى تغير ممكن يحصل
resource "aws_instance" "cerberus" {
  ami           = "ami-02cff456777cd"
  instance_type = "m5.large"
  tags = {
    Name = "Cerberus-Webserver-1"
  }
  lifecycle {
    ignore_changes = [ tags ]   # أو ignore_changes = all
  }
}
```

| Rule | الوظيفة |
|------|---------|
| `create_before_destroy = true` | عند الاستبدال: الإنشاء قبل الحذف |
| `prevent_destroy = true` | يرفض أي تغيير يؤدي لحذف المورد (خطأ) — **لا يحمي من `tofu destroy`** |
| `ignore_changes = [attr]` | تجاهل التغييرات على وسائط محددة (أو `all`) |

> [!tip] random resources
> موارد الـ random provider تُعاد إنشاؤها عند تغيير الـ **keepers** (map من أزواج key/value — تغيير أي قيمة يجبر إعادة الإنشاء).

## Data Sources

[aws_s3_bucket](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/s3_bucket) · [opentofu docs](https://opentofu.org/docs/language/data-sources/) · [aws_ebs_volume](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ebs_volume)

- **resource** = إنشاء/تحديث/حذف (managed) — **data source** = قراءة معلومات مورد موجود مسبقًا فقط.
- للوصول لموارد أُنشئت يدويًا أو بأدوات أخرى (CloudFormation, Ansible) أو في configuration آخر.

```hcl
data "aws_key_pair" "cerberus-key" {
  key_name = "alpha"
}

# أو بالفلترة
data "aws_key_pair" "cerberus-key" {
  filter {
    name   = "tag:project"
    values = ["cerberus"]
  }
}

resource "aws_instance" "cerberus" {
  ami           = var.ami
  instance_type = var.instance_type
  key_name      = data.aws_key_pair.cerberus-key.key_name
}
```

```hcl
# قراءة ملف موجود محليًا
data "local_file" "os" {
  filename = "/etc/os-release"
}

output "os_version" {
  value = data.local_file.os.content
}
```

> [!note]
> Data source **لا ينشئ ولا يعدل ولا يحذف** — قراءة فقط.

---

## Meta-Arguments: Count & for_each

### Count

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  count         = length(var.webservers)
  tags = {
    Name = var.webservers[count.index]
  }
}

variable "webservers" {
  type    = list
  default = ["web1", "web2", "web3"]
}
```

- الموارد تُنشأ كـ **list**: `aws_instance.web[0]`, `[1]`, `[2]`.

> [!warning] عيب count
> عند إزالة عنصر من منتصف القائمة، كل العناصر بعده تنزاح indices → يُحدَّث معظمها ويُحذف الأخير. استخدم `for_each` لتفادي ذلك.

```hcl
# مثال count مع local_sensitive_file
resource "local_sensitive_file" "name" {
  filename = var.users[count.index]
  content  = var.content
  count    = length(var.users)
}

variable "users" {
  type = list
}

variable "content" {
  default = "password: S3cr3tP@ssw0rd"
}
```

### for_each

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  for_each      = var.webservers    # set أو map (ليس list)
  tags = {
    Name = each.value
  }
}

variable "webservers" {
  type    = set
  default = ["web1", "web2", "web3"]
}
```

- الموارد تُنشأ كـ **map** → إزالة عنصر تحذف مورد ذلك العنصر فقط.
- `each.value` / `each.key`.
- لو الـ variable هو list فيه تكرارات → حوّله بـ `toset()` (يزيل التكرار).

## Version Constraints

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.3.0"      # إصدار محدد
    }
  }
}
```

**معاملات القيود:**

| Operator | المعنى | مثال النتيجة |
|----------|--------|--------------|
| `=` | إصدار محدد بالضبط | 2.3.0 |
| `!=` | استثناء إصدار | أي إصدار ما عدا 2.4.1 |
| `>`, `<`, `>=`, `<=` | مقارنة | `< 2.4.1` → 2.4.0 |
| `> x, != y, < z` | نطاق مركب | `> 2.2.3, != 2.4.0, < 2.4.1` → 2.3.0 |
| `~>` | pessimistic | `~> 2.2` → حتى `< 2.5` · `~> 2.2.0` → حتى `< 2.3` |

```hcl
terraform {
  required_providers {
    k8s = {
      source  = "hashicorp/kubernetes"
      version = "> 1.12.0, != 1.13.1, < 1.13.3"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 1.2.0"
    }
  }
}
```

> [!note]
> `==` ليس معاملًا صالحًا. بدون تحديد version ينزل أحدث إصدار — قد يكسر الإعدادات.

## OpenTofu Provisioners

- تنفيذ أوامر/سكربتات بعد إنشاء المورد (remote) أو على الجهاز المحلي (local) — البلوك يوضع **داخل resource block**.
- **remote-exec**: يحتاج اتصال شبكة (SSH لـ Linux / WinRM لـ Windows) + authentication → **connection block**.
- **local-exec**: لا يحتاج connection — يعمل على جهاز OpenTofu نفسه.

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  key_name               = aws_key_pair.web.id
  vpc_security_group_ids = [ aws_security_group.ssh-access.id ]

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx",
    ]
  }

  connection {
    type        = "ssh"
    host        = self.public_ip     # الـ IP للـ instance المنشأة
    user        = "ubuntu"
    private_key = file("/root/.ssh/web")
  }
}
```

```hcl
# local-exec — مثال: تسجيل الـ IP محليًا
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.webserver.public_ip} >> /tmp/ips.txt"
  }
}
```

### Destroy-Time Provisioner

```hcl
provisioner "local-exec" {
  when    = destroy    # افتراضيًا: create-time
  command = "echo Instance ${aws_instance.webserver.public_ip} Destroyed! > /tmp/instance_state.txt"
}
```

### Failure Behavior

```hcl
provisioner "local-exec" {
  on_failure = continue   # الافتراضي fail: يفشل الـ apply والمورد يُعلَّم tainted
  command    = "..."
}
```

> [!warning] Provisioners كملاذ أخير
> يوصي OpenTofu باستخدام provisioners فقط عند عدم وجود بديل — استخدم خيارات المورد الأصلية أولاً:

| Provider | Resource | البديل |
|----------|----------|--------|
| AWS | aws_instance | `user_data` |
| Azure | azurerm_virtual_machine | `custom_data` |
| GCP | google_compute_instance | `metadata` |
| VMware vSphere | vsphere_virtual_machine | `user_data.txt` |

```hcl
# user_data بالـ heredoc — بديل أنظف من remote-exec
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  tags = {
    Name        = "webserver"
    Description = "An NGINX WebServer on Ubuntu"
  }
  user_data = <<-EOF
              #!/bin/bash
              sudo apt update
              sudo apt install nginx -y
              systemctl enable nginx
              systemctl start nginx
              EOF
}
```

> [!note]
> `user_data` يعمل مرة واحدة عند أول إنشاء فقط — تعديله على instance قائمة لا يثبّت شيئًا.

## Taint / Replace & Debugging

### Taint & Replace

- عند فشل إنشاء مورد (مثل فشل provisioner) → يُعلَّم **tainted** → يُعاد إنشاؤه بالكامل في الـ apply التالي.
- لإجبار إعادة إنشاء مورد (مثلاً بعد تعديل يدوي):

```bash
tofu apply -replace="aws_instance.project_b"   # الطريقة الحديثة
tofu untaint aws_instance.project_b            # التراجع عن الـ taint
```

- أمر `tofu taint` قديم (deprecated) — استخدم `apply -replace`.

### Debugging

```bash
export TF_LOG=trace              # levels: TRACE (أكثر تفصيلاً) DEBUG INFO WARN ERROR
export TF_LOG_PATH=/tmp/tofu.log
```

> [!note]
> `TF_LOG_PATH` وحده لا يكفي — يجب تعيين `TF_LOG` أيضًا.

## Tofu Import

- إدخال موارد موجودة (أُنشئت يدويًا أو بأدوات أخرى) تحت إدارة OpenTofu.
- **لا يكتب configuration** — يستورد إلى الـ state فقط؛ يجب إكمال resource block يدويًا بعدها.

```bash
# 1) بلوك resource فارغ بنفس الاسم أولاً
# 2) الاستيراد بمعرّف فريد من الـ console
tofu import aws_instance.jade-mw i-47a8764efa9248c14
# 3) أكمل الوسائط (استعن بـ tofu show) حتى يعرض plan "no changes"
```

### Import Block (الطريقة الحديثة)

```hcl
import {
  to = aws_instance.webserver2
  id = "i-47a8764efa9248c14"
}
```

- يُعالج أثناء `plan` ويستورد عند `apply` — يمكن إبقاؤه كسجل لأصل المورد أو حذفه بعد الانتهاء.

```bash
# أدوات مفيدة للفحص قبل الاستيراد
aws ec2 describe-instances --endpoint http://aws:4566
aws ec2 create-key-pair --endpoint http://aws:4566 --key-name jade --query 'KeyMaterial' --output text > /root/opentofu-projects/project-jade/jade.pem
```

---

## OpenTofu Modules

[security-group module](https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest/submodules/ssh)

- أي مجلد configuration هو **root module**. البلlock `module` يستدعي **child module** لإعادة استخدام الكود.

```hcl
# /root/opentofu-projects/development/main.tf
module "dev-webserver" {
  source = "../aws-instance"   # local path (نسبي لمجلد العمل)
}
```

### مثال كامل — payroll-app module

```hcl
# modules/payroll-app/app_server.tf
resource "aws_instance" "app_server" {
  ami           = var.ami
  instance_type = "t2.medium"        # hardcoded → قيمة موحدة لكل البيئات
  tags = {
    Name = "${var.app_region}-app-server"
  }
  depends_on = [ aws_dynamodb_table.payroll_db,
                 aws_s3_bucket.payroll_data ]
}

# modules/payroll-app/s3_bucket.tf
resource "aws_s3_bucket" "payroll_data" {
  bucket = "${var.app_region}-${var.bucket}"
}

# modules/payroll-app/dynamodb_table.tf
resource "aws_dynamodb_table" "payroll_db" {
  name         = "user_data"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "EmployeeID"
  attribute {
    name = "EmployeeID"
    type = "N"
  }
}

# modules/payroll-app/variables.tf
variable "app_region" { type = string }
variable "ami"        { type = string }
variable "bucket"     { default = "flexit-payroll-alpha-22001c" }
```

```hcl
# us-payroll-app/main.tf — نفس الموديول لبيئة أخرى
module "us-payroll" {
  source     = "../modules/payroll-app"
  app_region = "us-east-1"
  ami        = "ami-2357"
}
```

- عنوان مورد داخل module: `module.us-payroll.aws_dynamodb_table.payroll_db`.

### Modules من الـ Registry

```hcl
module "ssh_security_group" {
  source              = "terraform-aws-modules/security-group/aws//modules/ssh"
  version             = "~> 5.0"      # حدد دائمًا إصدارًا
  name                = "ssh-access"
  vpc_id              = "vpc-7d8d215"
  ingress_cidr_blocks = ["10.10.0.0/16"]
}

module "iam_iam-user" {
  source                        = "terraform-aws-modules/iam/aws//modules/iam-user"
  version                       = "5.28.0"
  name                          = "max"
  create_iam_access_key         = false   # تعطيل ما لا تحتاجه
  create_iam_user_login_profile = false
}
```

```bash
tofu get     # تنزيل الموديولات فقط (بدون providers)
```

> [!tip] لماذا modules؟
> - إعدادات أقصر وأسهل إدارة.
> - كود مجرب → أخطاء بشرية أقل واتساق بين البيئات.
> - إعادة استخدام بنفس الإعدادات مرات لا نهائية.
> - تثبيت قيم معينة (hardcoded) لا يمكن تغييرها من الـ root module.

## Local Values

- تسمية قيم/تعبيرات مكررة واستخدامها بـ `local.<name>`.

```hcl
locals {
  common_tags = {
    Department = "finance"
    Project    = "cerberus"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0617"
  instance_type = "t2.medium"
  tags          = local.common_tags
}

resource "aws_instance" "db" {
  ami           = "ami-0567"
  instance_type = "m5.large"
  tags          = local.common_tags
}
```

```hcl
# locals يمكنها دمج variables وresource attributes
locals {
  bucket_prefix = "${var.project}-${random_string.random-suffix.id}-bucket"
}

resource "aws_s3_bucket" "finance_bucket" {
  bucket = local.bucket_prefix
}

resource "random_string" "random-suffix" {
  length  = 6
  special = false
  upper   = false
}

variable "project" {
  default = "cerberus"
}
```

## Functions & Operators

اختبر كل شيء تفاعليًا عبر **`tofu console`** (اخرج بـ `exit`).

```text
$ tofu console
> file("/root/opentofu-projects/main.tf")   # محتوى ملف
> length(var.region)                         # عدد عناصر list
> toset(var.region)                          # list → set (يزيل التكرار)
```

### Numeric Functions

```text
> max(-1, 2, -10, 200, -250)   → 200
> min(-1, 2, -10, 200, -250)   → -250
> max(var.num...)              → 250      # توسيع set إلى وسائط (...)
> ceil(10.1)  → 11    > ceil(10.9)  → 11
> floor(10.1) → 10    > floor(10.9) → 10
```

### String Functions

```text
> split(",", "ami-xyz,AMI-ABC,ami-efg")   → [ "ami-xyz", "AMI-ABC", "ami-efg" ]
> lower(var.ami)     → ami-xyz,ami-abc,ami-efg
> upper(var.ami)     → AMI-XYZ,AMI-ABC,AMI-EFG
> title(var.ami)     → Ami-Xyz,AMI-ABC,Ami-Efg
> substr(var.ami, 0, 7)   → ami-xyz     # (string, offset, length)
> substr(var.ami, 8, 7)   → AMI-ABC
> join(",", ["a","b","c"]) → a,b,c      # عكس split
```

### Collection Functions

```text
> length(var.ami)               → 3
> index(var.ami, "AMI-ABC")     → 1     # موقع العنصر (يبدأ من 0)
> element(var.ami, 2)           → ami-efg
> contains(var.ami, "AMI-ABC")  → true
> contains(var.ami, "AMI-XYZ")  → false   # حساس لحالة الأحرف
```

### Map Functions

```text
> keys(var.ami)                      → [ "ap-south-1", "ca-central-1", "us-east-1" ]
> values(var.ami)                    → [ "ami-ABC", "ami-efg", "ami-xyz" ]
> lookup(var.ami, "ca-central-1")    → ami-efg
> lookup(var.ami, "us-west-2")       → Error: key not found
> lookup(var.ami, "us-west-2", "ami-pqr")   → ami-pqr   # قيمة افتراضية
```

### Operators

```text
# حسابية:  +  -  *  /
# مساواة:  ==   !=          (8 == "8" → false: number ≠ string)
# مقارنة:  >  <  >=  <=      (تقبل numbers فقط)
# منطقية:  &&   ||   !
> 8 > 7 && 8 < 10   → true
> 8 > 10 || 8 < 10  → true
> !var.special      → false
```

### Conditional Expression

```text
condition ? value_if_true : value_if_false
```

```hcl
# كأنها if/else — كلمة مرور بحد أدنى 8 أحرف
resource "random_password" "password-generator" {
  length = var.length < 8 ? 8 : var.length
}

output "password" {
  value     = random_password.password-generator.result
  sensitive = true
}
```

```bash
tofu apply -var=length=5 -auto-approve   # سيستخدم 8
```

```hcl
# مثال: instance_type حسب قيمة tag
instance_type = var.name == "tiny" ? var.small : var.large
```

```hcl
# أمثلة تطبيقية — functions مع meta-arguments
resource "aws_iam_user" "cloud" {
  name  = split(":", var.cloud_users)[count.index]
  count = length(split(":", var.cloud_users))
}

resource "aws_s3_object" "upload_sonic_media" {
  bucket = aws_s3_bucket.sonic_media.id
  key    = substr(each.value, 7, 20)   # إزالة بادئة المسار
  source = each.value
  for_each = var.media
}
```

## Dynamic Blocks & Splat Expressions

**المشكلة:** كرر بلوكات `ingress` المتشابهة يدويًا → إعدادات طويلة.

```hcl
# قبل: بلوك لكل منفذ
resource "aws_security_group" "backend-sg" {
  name   = "backend-sg"
  vpc_id = aws_vpc.backend-vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

```hcl
# بعد: dynamic block + variable
variable "ingress_ports" {
  type    = list
  default = [22, 8080]
}

resource "aws_security_group" "backend-sg" {
  name   = "backend-sg"
  vpc_id = aws_vpc.backend-vpc.id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

```hcl
# باسم مخصص عبر iterator
dynamic "ingress" {
  iterator = port
  for_each = var.ingress_ports
  content {
    from_port   = port.value
    to_port     = port.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Splat Expression

- الوصول لـ attribute في **كل** عناصر list من الموارد: `[*]`

```hcl
output "to_ports" {
  value = aws_security_group.backend-sg.ingress[*].to_port
}

# tofu output →
# to_ports = [ 22, 8080 ]
```

## OpenTofu Workspaces

- نفس configuration → عدة بيئات (state منفصل لكل workspace).

```bash
tofu workspace list              # * default (تُنشأ تلقائيًا)
tofu workspace new production
tofu workspace new development
tofu workspace select production
```

- `terraform.workspace` ترجع اسم الـ workspace الحالي — تُستخدم مع `lookup` لاختيار قيم حسب البيئة:

```hcl
resource "aws_instance" "webserver" {
  ami           = var.ami
  instance_type = lookup(var.instance_type, terraform.workspace)
  tags = {
    Environment = terraform.workspace
  }
}

variable "instance_type" {
  type = map
  default = {
    "development" = "t2.micro"
    "production"  = "m5.large"
  }
}
```

- الـ state المحلي يُخزن في `terraform.tfstate.d/<workspace>/terraform.tfstate` بدل ملف واحد.

```hcl
# modules + workspaces معًا
variable "region" {
  type = map
  default = {
    "us-payroll"    = "us-east-1"
    "uk-payroll"    = "eu-west-2"
    "india-payroll" = "ap-south-1"
  }
}

module "payroll_app" {
  source     = "/root/opentofu-projects/modules/payroll-app"
  app_region = lookup(var.region, terraform.workspace)
  ami        = lookup(var.ami, terraform.workspace)
}
```

```bash
tofu workspace select india-payroll && tofu apply -auto-approve
tofu workspace select us-payroll    && tofu apply -auto-approve
tofu workspace select uk-payroll    && tofu apply -auto-approve
```

## Migrating Terraform → OpenTofu

- الإصدارات المتوافقة: OpenTofu 1.6 ↔ Terraform 1.6 (وما فوق قابلة للترحيل غالبًا).

**خطوات الترحيل:**

1. خطة disaster recovery — نسخة احتياطية من الـ state.
2. تطبيق كل التغييرات المعلقة بـ Terraform أولاً.
3. تثبيت OpenTofu + نسخ احتياطي للـ state:
   ```bash
   tar -czf tfstate.tar.gz terraform.tfstate
   ```
4. `tofu init` ثم `tofu plan` (تأكد لا تغييرات غير متوقعة) ثم `tofu apply`.
5. اختبر بتغيير صغير غير حرج أولاً.

**التراجع (rollback) إلى Terraform:**

```bash
tar -czf terraform.tfstate.tar.gz terraform.tfstate   # نسخة احتياطية من state المقابل
terraform init
terraform plan    # تأكد لا تغييرات غير متوقعة
terraform apply
```

> [!note] عند مواجهة مشاكل
> أبلغ عن bugs في [OpenTofu GitHub Issues](https://github.com/opentofu/opentofu/issues) أو اسأل في OpenTofu Slack / GitHub Discussions. اتبع إرشادات فتح الـ issue (bug / feature / security).

**تسلسل أوامر شامل — من الملف الأصلي:**

```bash
tofu init
tofu validate
tofu fmt
tofu plan
tofu apply -var "key=value" -auto-approve
tofu show -json
tofu output <name>
tofu state list | show | mv | rm | pull
tofu import <addr> <id>
tofu apply -replace=<addr>
tofu workspace new | select | list
tofu destroy -auto-approve
```



## Multiple Providers — Azure Multi-Subscription


**الحالات اللي نحتاج فيها أكثر من provider:**
- النشر على أكثر من Azure subscription بنفس الكود.
- النشر بهويات (identities) متعددة على بيئات متعددة.
- النشر على أكثر من Azure AD tenant.
- معمارية Hub & Spoke.

- Enterprise Landing Zone.
**الفكرة:** نفس أسلوب الـ alias — لكن مع `azurerm`، كل provider block له `subscription_id` و `tenant_id` مختلفين، وكل resource يحدد الـ provider المطلوب صراحة.

```hcl
terraform {
  required_version = ">= 1.2.8"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.26.0"
    }
  }
}

# provider الاشتراك الأساسي (hub)

provider "azurerm" {
  alias           = "subscription_hub"
  subscription_id = "4b72ed90-7ca3-4e76-8d0f-xxxxxxxx"
  tenant_id       = "72f988bf-86f1-41af-91ab-xxxxxxxx"
  # client_id / client_secret اختيارية — الأفضل env vars أو Workload Identity
  # auxiliary_tenant_ids = ["558506eb-9459-4ef3-b920-xxxxxxxx"]   # tenants إضافية
  features {}
}

# provider الاشتراك الثاني (spoke) — ممكن يكون tenant مختلف

provider "azurerm" {
  alias           = "subscription_spoke"
  subscription_id = "59d574d4-1c03-4092-ab22-xxxxxxxx"
  tenant_id       = "558506eb-9459-4ef3-b920-xxxxxxxx"
  features {}
}

```

**استخدام الـ providers في الموارد:**

```hcl

resource "azurerm_resource_group" "rg_hub" {
  provider = azurerm.subscription_hub
  name     = "rg_hub"
  location = "westeurope"
}

resource "azurerm_resource_group" "rg_spoke" {
  provider = azurerm.subscription_spoke
  name     = "rg_spoke"
  location = "northeurope"
}

resource "azurerm_virtual_network" "vnet_hub" {
  provider            = azurerm.subscription_hub
  name                = "vnet_hub"
  resource_group_name = azurerm_resource_group.rg_hub.name
  location            = "westeurope"
  address_space       = ["10.1.0.0/16"]
}

resource "azurerm_virtual_network" "vnet_spoke" {
  provider            = azurerm.subscription_spoke
  name                = "vnet_spoke"
  resource_group_name = azurerm_resource_group.rg_spoke.name
  location            = "northeurope"
  address_space       = ["10.2.0.0/16"]
}

```

  
**النتيجة:** `apply` واحد ينشئ 4 موارد موزعة على الاشتراكين — `rg_hub` + `vnet_hub` في West Europe و `rg_spoke` + `vnet_spoke` في North Europe.


> [!tip] نقاط مهمة
> - `features {}` إلزامية داخل كل provider block للـ azurerm حتى لو فاضية.
> - الـ provider بدون alias هو الافتراضي؛ اللي بـ alias لازم تشير له صراحة بـ `provider = azurerm.<alias>`.
> - `tenant_id` مختلف لكل provider = نشر عبر أكثر من Azure AD tenant.
> - متحطش `client_secret` في الكود — استخدم environment variables أو Workload Identity (OIDC).

**مصادر:**
  
[Lab: 25_multi_provider](https://github.com/HoussemDellai/terraform-course/tree/main/25_multi_provider) · [Workload Identity Overview](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview) · [MS Learn Tutorial](https://learn.microsoft.com/en-us/azure/aks/learn/tutorial-kubernetes-workload-identity) · [AKS Labs](https://github.com/HoussemDellai/docker-kubernetes-course)


## Data Sources — http & jsondecode

  

- `data "http"` يجلب بيانات من أي REST API — النتيجة نص، ولو كان الرد JSON نحوّله object بـ `jsondecode()`.

- `request_headers` لتحديد الـ Accept header عند الحاجة.

  

```hcl

# الـ IP العام لجهازك — الرد نص عادي
data "http" "my_ip" {
  url = "http://ifconfig.me"
}


output "myIp" {
  value = data.http.my_ip.body
}
  

# آخر إصدار Terraform — الرد JSON
data "http" "tf_latest" {
  url = "https://checkpoint-api.hashicorp.com/v1/check/terraform"
  request_headers = {
    Accept = "application/json"
  }
}
  

output "terraform_latest_version" {
  value = jsondecode(data.http.tf_latest.body).current_version
}

  

# آخر إصدار azurerm provider — نفس الفكرة مع GitHub API
data "http" "azurerm_latest" {
  url = "https://api.github.com/repos/terraform-providers/terraform-provider-azurerm/releases/latest"
  request_headers = {
    Accept = "application/json"
  }
}

  

output "azurerm_latest_version" {
  value = jsondecode(data.http.azurerm_latest.body).name
}

```

  

```hcl

# data source خاص بـ provider — الإصدارات المتاحة لـ AKS في region معين
data "azurerm_kubernetes_service_versions" "current" {
  location        = "West Europe"
  include_preview = true
}

output "aks_latest_version" {
  value = data.azurerm_kubernetes_service_versions.current.latest_version
}

```

  
---
## Random Resources

  

```hcl

# كلمة مرور بسياسة صارمة

resource "random_string" "password" {
  length           = 20
  special          = true
  override_special = "!@#$%&*()-_=+[]{}<>:?"   # استبدال الرموز الافتراضية
  min_special      = 2
  min_upper        = 2
  min_lower        = 5
  min_numeric      = 3
}

  

# رقم عشوائي في نطاق محدد

resource "random_integer" "number" {
  min = 5
  max = 100
}


# UUID — مفيد للأسماء الفريدة
resource "random_uuid" "id" {}

output "random_values" {
  value = {
    password = random_string.password.result
    number   = random_integer.number.result
    uuid     = random_uuid.id.result
  }
}

```
  

> [!note]
> - كلها logical resources تُسجَّل في الـ state فقط (مثل `random_pet` و `time_static`).
> - `keepers` تتحكم في إعادة الإنشاء — راجع [[#LifeCycle Rules]].


---

## Variable Validation & Sensitive

- `sensitive = true` على الـ variable: لا تُطبع قيمته في plan/apply.
- `validation`: فحص القيمة عند الـ plan — يفشل مبكرًا قبل إنشاء أي شيء.


```hcl

variable "rg_name" {
  type        = string
  description = "The name of the Azure Resource Group."
  default     = "rg-frontend_app"
  sensitive   = true

  

  validation {
    condition     = length(var.rg_name) > 3 && substr(var.rg_name, 0, 3) == "rg-"
    # بديل بالـ regex: can(regex("^rg-", var.rg_name))
    error_message = "The rg_name value must be valid, starting with \"rg-\"."
  }
}

  

variable "location" {
  type    = string
  default = "westeurope"

  # anytrue + for expression: القيمة تطابق أي عنصر من القائمة

  validation {
    condition     = anytrue([for l in ["eastus", "westeurope"] : l == var.location])
    # بديل: contains(["eastus", "westeurope"], lower(var.location))
    error_message = "The region should be eastus or westeurope."
  }
}

```


> [!tip]
> `contains()` حساس لحالة الأحرف — استخدم `lower(var.location)` لتوحيد المقارنة.


---
## Feature Flags — Enable/Disable Resources
  
- لا يوجد attribute اسمه `enabled` — التحكم في إنشاء/تجاهل المورد كله يتم عبر `count` أو `for_each` مع bool.


```hcl

variable "resource_enabled" {
  type        = bool
  default     = true
  description = "Enable or disable creation of Resource Group"
}

  

# الطريقة 1: count

resource "azurerm_resource_group" "rg_count" {
  count    = var.resource_enabled ? 1 : 0
  name     = "rg_enabled"
  location = "westeurope"
}

  

# الطريقة 2: for_each مع toset — قائمة فيها عنصر واحد أو فاضية

resource "azurerm_resource_group" "rg_for_each" {
  for_each = var.resource_enabled ? toset(["any_value"]) : toset([])
  name     = "rg_enabled"
  location = "westeurope"
}

  

# الـ outputs تحتاج حماية من الـ null لما المورد معطّل

output "rg_id_count" {
  value = var.resource_enabled ? azurerm_resource_group.rg_count.0.id : null
}

  

output "rg_id_for_each" {
  value = var.resource_enabled ? azurerm_resource_group.rg_for_each["any_value"].id : null
}

```
  

**تعطيل nested block (وليس مورد كامل):**


```hcl

resource "azurerm_storage_account" "storage" {
  name                     = "storage091"
  resource_group_name      = azurerm_resource_group.rg_for_each["any_value"].name
  location                 = "westeurope"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # nested block يُتحكم فيه بـ dynamic + شرط
  dynamic "network_rules" {
    for_each = var.resource_enabled ? ["any_value"] : []
    content {
      default_action = "Deny"
      ip_rules       = ["100.0.0.1"]
    }
  }
}

```

  

> [!warning]
> `count` لا يُقبل داخل الـ nested blocks — استخدم `dynamic` + `for_each` بقائمة فاضية/مفردة.


---

## Provisioners — null_resource, interpreter & WinRM

- `null_resource`: مورد منطقي لا ينشئ شيئًا في السحابة — حاوية للـ provisioners فقط.
- `interpreter`: تغيير المفسّر الافتراضي لأمر الـ local-exec (مفيد على Windows).


```hcl
resource "null_resource" "time_now" {
  provisioner "local-exec" {
    command     = "Get-Date > date.txt"
    interpreter = ["PowerShell", "-Command"]
  }
}

```

```hcl

# remote-exec على Windows عبر WinRM بدل SSH

resource "null_resource" "remote" {
  provisioner "remote-exec" {
    connection {
      host     = azurerm_public_ip.windows.ip_address
      type     = "winrm"
      port     = 5985
      https    = false
      timeout  = "5m"
      user     = var.admin_username
      password = var.admin_password
    }
    inline = [
      "powershell.exe -ExecutionPolicy Unrestricted -Command {Install-WindowsFeature -name Web-Server -IncludeManagementTools}",
    ]
  }
}

```


> [!note]
> امتداد لقسم [[#OpenTofu Provisioners]] — فيه SSH على Linux؛ هنا WinRM على Windows بنفس الـ connection block.

