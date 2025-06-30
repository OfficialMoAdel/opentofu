---
source: https://github.com/abdelghafarabdelmordy/create-vm-terraform-vsphere
---
```yml
tofu init
tofu plan
tofu apply || tofu apply -auto-approve
tofu show

```

local.tf
```js
resource "local_file" "pet"{
 filename = "/root/pets.txt"
 content = "we love pets"
 file_permation= "0700"
}
```
tofu plan
tofu apply
tofu destroy   || tofu destroy -auto-approve    مسح 

main.tf
varibles.tf
outputs.tf
provider.tf
opentofu.tf




varibles.tf
```
provider "aws" {
region = "us-east-1"
}
provider "aws" {
region = "ca-central-1"
alias = "central"
}
```

main.tf
```
resouce "aws_key_pair" "alpha" {
ky_name = "alpha"
public_key = "ssh-rsa aaaakjlkhlh@a-server"
}
resouce "aws_key_pair" "beta" {
ky_name = "beta"
public_key = "ssh-rsa aaaakjlkhlh@a-server"
provider = aws.central
}
```



main.tf
```
resouce "aws_instance" "webserver" {
ami = var.ami
instance_type = var.instance_type
}
```
varibles.tf
```
provider "instance_type" {
 default = "12.micro"
}
provider "ami" {
 default = "ami-01211123231yt211kj1k3"
}



or

provider "instance_type" {

}
provider "ami" {
 
}

tofu apply
var.ami 
  enter a value: jkl
var.instance_type 
  enter a value: t3.micro


or
tofu apply -var "ami=878idod" -var "instance_type=t2.micro"


or
export TF_VAR_instance_type="t3.micro"
export TF_VAR_ami="878idod"
tofu apply



```


varibles.tf
```
provider "instance_type" {
 default = "12.micro"
 description = "size of EC2"
 type = string
}
provider "ami" {
 default = "ami-01211123231yt211kj1k3"
 description = "Type of AMI to use"
 type = string
}


variable "count"{
 default = "2"
 type = number
 description = "Count of VM's"

}

variable "monitoring" {
 default = "true"
 type = bool
description = "Enable detailed monitoring"
}

#list

variable "servers" {
 default = ["web1", "web2", "web3"]
 type = list
}

variable "servers" {
 default = ["web1", "web2", "web3"]
 type = list(string)
}
variable "prefix" {
 default = [1, 2, 3]
 type = list(number)
}



variable "servers" {
 default = {
   "prodaction" = "m5.large"
   "development" = "t2.micro"
 }
 type = map
}

variable "instance_type" {
 default = {
   "prodaction" = "m5.large"
   "development" = "t2.micro"
 }
 type = map(string)
}

variable "servers_count" {
 default = {
   "web" = 3
   "db" = 1
   "agent" = 2
 }
 type = map(number)
}



#object

variable "bella" {
 type = object({
  name =string
  color = string
  age = number
  food= list(string)
  favorite_pet = bool
 })
 default = {
   name ="bella"
  color = "brown"
  age = 7
  food= ["fish", "chicken", "turkey"]
  favorite_pet = true
 }
}


#tuples
variable "web" {
 type = tuple([string,number, bool])
 default = {"web1", 7, true}
 }



```

```
#list

resouce "aws_instance" "web" {
 ami = var.ami
 instance_type = var.instance_type
 tage = {
  name = var.servers[0]

  }
}



#map
resouce "aws_instance" "production" {
 ami = var.ami
 instance_type = var.instance_type["development"]
 tage = {
  name = var.servers[0]

  }
}


```

### Resource Attributes And Dependencies

```
resouce "aws_key_pair" "alpha" {
 key_name = alpha
 public_key = "ssh-rsa AAAAB3Njvlkdjskdnajdhajs;faf.....alpha@a-server"
}
```

```
show tofu

```

```
resouce "aws_key_pair" "alpha" {
 key_name = alpha
 public_key = "ssh-rsa...."
}

resouce "aws instance" "cerberus "{
 ami = var.ami
 instance_type = var.instance_type
 key_name = aws_key_pair.alpha.key_name
}

```


```
resouce "aws_instance" "db" {
 ami = var.db_ami
 instance_type = var.web_instance_type
}



resouce "aws_instance" "web" {
 ami = var.web_ami
 instance_type = var.db_instance_type
 depends_on = {
  aws_instance.db

  }
}
```

```
Let's now recap resource attributes and
dependencies in OpenTofu. When we provision a
resource, OpenTofu records several values
related to that resource.
Earlier, we used an example configuration file to
create an AWS keypair resource. This resource
relies on an existing user-supplied keypair and
requires the public key as a mandatory
argument. Upon creation of this resource,
OpenTofu exports several attributes. This can be
seen by running the tofushow command like this.
Here, we can see that the AWS keypair resource
called alpha
has exported several attributes, such as ARN,
fingerprint, ID, keyname, public key, and tags all.
And, we can look up each attribute exported by a
resource within the documentation.
Attributes exported by a resource can be used as
input for another resource block. For example,
this AWS keypair resource that we just created
can be supplied to an EC2 instance. To do this,
we use a reference expression like this. Here, the
value for the key_name argument within the AWS
instance type block is a reference expression
pointing to the key_name resource attribute

that is exported by the AWS keypair resource
called alpha. The syntax used for this expression
is the resource type.resourceName.attribute,
which in our case is aws-
keypair.alpha.key_name.
Running OpenTofu apply will create both the
keypair resource and the EC2 instance.
However, keep a note of the order in which the
resources are created. By default, OpenTofu tries
to create
the resources in parallel. However, in this case,
the EC2 instance depends on the keypair
resource. The keypair resource is created first
and the AWS instance next. This is known as
implicit
dependency. When the resources are deleted,
they are removed in a reverse order. The EC2
instance first and then the keypair. There is
another way to make sure that a specific
resource is created
before another in the configuration. For example,
let's assume that we have two EC2 instance
types, one for a database and the other one for
web. Now, there is no implicit dependency
between these two
resources, no attributes of one being used by the
other. In this case, if we want to create the DB

instance before the web instance, for example,
we can make use of a meta argument called
depends on, like this. This is called an explicit
dependency. We have specified that the web
instance depends
on the DB instance and hence the DB instance
will be created first and only then will the web
instance be provisioned. Well, that's it for this
lecture. I will see you in the next.


Resource Attributes


Hello and welcome to the demo of resource
attributes in OpenTofu. So, our first question is to
navigate to the configuration directory
slash root slash OpenTofu projects slash project
chronos and inspect the files created inside.
Once we inspect domain.tf, we can see that there
is a resource
for time static with the name of time_update. So,
we have inspected that. Let's move on to the
next question. What is the resource type of the
resource that's currently defined in the domain.tf
file?
If unsure, refer to the documentation, which
again is available in the top right corner. But once
we opened up domain.tf, we can see that the
resource is time static.
So, we can go ahead and select Stefan Matié.
The next question says, as you can see, the
resource block is empty.
This is because time static does not need any
arguments to be supplied to work. When applied
as it is, OpenTofu creates a logical resource
locally
similar to the random pet with the current time.
Now, moving on to the next question. Which of
the following attributes are exported by the time
static resource?

If unsure, refer to the documentation. The
documentation tab is available on the top right
panel as always. So, if you open up the
OpenTofu reference Terraform docs,
you will be brought to the documentation page.
And if you take a look at the documentation, you
can see the attributes that are going to be
exported,
which are day, hour, id, minute, month, second,
unix, and year. And going back into the given
answers, there was no answer for decade,
century, file name, or content,
but there was one for ID. So, id is the correct
answer. How do we refer to the attribute called id
using a reference expression?
So, the right way to do a reference expression is
to use the resource type followed by the resource
name.
So, in our case, it would be time static dot
time_update dot id with the attribute at the end.
Now, update the main.tf file and add a new local
file resource
called time with the following requirements. So,
let's go and open up main.tf and create a new
resource called local file called time.

So, let's call it time. And then file name needs to
be, and we're just going to copy this over.
Don't make any mistakes. And then content
needs to be, again, copied over here. And then
this timestamp of this file is,
and then the ID from the time update resource.
So, we have to reference somehow the
time static resource. And we do that by using a
dollar curly brace here
to reference the time static resource, then the
time_update, and then the id attribute. And we
save that up. Now, once we're ready, we need to
go into the project,
pronouns, and do an OpenTofu init, followed by a
OpenTofu apply. We check the execution plan.
We make sure that the time static time update
will be created,
as well as the local file .time, which we confirm
with yes, and then check if the appropriate
resources were created.
Good. Moving on to the next question. All right.
So, our next question says, what is the attribute
id that is created for the local file resource called
"time"?
Make use of the OpenTofu show command and
identify the attribute values. So, what we do is we
just run the OpenTofu show command.
And what we look for is for the local file resource,
the id attribute, which in this case is F095 ending
in FF48.
So, this is our file. We select it. And that's it. The
last question says, what is the attribute called
RFC 339 that is created
for the time static resource called time update?
So, make use of the OpenTofu show command
again and identify the attribute values.
So, we can reuse the same command. What we
look for is the RFC 3339 attribute. And we notice
that this is the value that we're looking for.
Let's select it. And that's it. That will be all for this
lab. Thank you for watching. And I'll see you in
the next one.



Resource Targeting

In this lecture, we will learn how to use resource
targeting with OpenTofu plan or apply
commands. In the multiple providers lecture, we
used an example with two resource blocks.
One with the random string resource and another
with an AWS EC2 instance. In this example, you
can see these two resource blocks with a slight
modification.
The AWS instance resource called web has a tag
with the key called name that uses the ID
generated by the random string resource as a
suffix.
To do this, we have used what is known as an
interpolation sequence, wrapping the resource
attribute random_string.server_suffix.id inside $
curly braces, like this.
Now let's say that we change the length of the
random string resource from 6 to 5. If we run
OpenTofu apply now, as expected, the random
string resource is recreated with a
length of 5. However, that is not all, because the
tag for the AWS instance resource depends on
the random string resource and it too is updated,
but what if we want to only update the random
string resource and leave the AWS instance
called web as it is. Let's rewind back the
resources to how they were before we ran the
last Terraform apply.
To the point where the random string resource as
well as the tag had the 6 character long random
string.
Now let's use the target flag along with the
Terraform apply command, like this. And here we
are making use of the resource address of the
syntax resourceType.resourceName
to target the changes to be only applied on the
random string resource called serversuffix. When
we do this, in the execution plan, OpenTofu will
warn us that the resource targeting is
in effect. During this apply, only the random string
resource is recreated, the AWS EC2 instance is
untouched. Now keep in mind that resource
targeting should only be used in rare occasions,
such as to
correct a mistake or to process an urgent change
without having to execute the entire plan or
modify the whole configuration.
When used, OpenTofu will warn that the applied
changes could be incomplete, so use it only
when the situation warrants it.
Well that's it for this lecture, I'll see you in the
next one.


Resource Dependencies


resource dependencies. So our first question for
this lab is which arguments should be used
to explicitly set dependencies for a resource? So
explicitly means that you need to define it exactly
in your configuration.
And the key word to do that is depends on.
Resource A relies on another resource B, but
doesn't access any of its attributes
in its own arguments. What is this type of
dependency called? So if we watch the previous
lectures, we know that this type of dependency
is called the explicit dependency. The next
question says, how do we make use of implicit
dependency? So we can use the reference
expressions.
This would be calling the resource A and
resource B by using the resource type and then
the resource name.
The next question is in the configuration directory
slash root slash OpenTofu project slash key
generator, create a file called key.tf
with the following specifications. Resource type is
TLS private key, resource name is private key,
then algorithm is RSA, RSA bits 4096.

When ready, run OpenTofu init, plan and apply.
Okay, so let's go into the key generator and
create our key.tf.
Then we need to do a resource block with TLS
private key. And we're gonna call that resource
PVT key, PVT key.
And then we need the algorithm to be RSA and
the RSA bits to be 4096. When ready, we need to
run OpenTofu init.
Go into our project, key generator, run OpenTofu
init to download the provider, then OpenTofu plan
to see if we set everything properly.
The execution plan seems to be okay, so
OpenTofu apply next and confirm yes in the
prompt. And it has been completed.
So let's now check to see if our configuration is
okay. and it seems to be. Let's go to the next
question.
Resource TLS private key generates a secure
private key and encodes it as a PEM. It is a
logical resource that contributes
only to the current OpenTofu state. You can see
the details of the resource, including the private
key by running the OpenTofu show command.

And yes, we can see the details of that private
key. Let's move on to the next question. Now let's
use the private key created by this resource
in another resource of type local file. Update the
key.tf with the requirements. So we have another
resource called resource name,
which is key details. So let's do our resource,
which is local file as instructed. And the name is
key details.
Now the file name should be [slash dash
/root/key.txt. And the content, use a reference
expression to the attribute called private key "M"
of the "PVT" key resource.
So we need to do content equals, and then our
reference expression to the private key, and then
private key PEM.
So to the private key, which is this resource, the
name PVT key, which is this, and the attribute
called private key PEM.
And once we do that, we can save it up, run
OpenTofu in it again, to download the local file,
then OpenTofu plan to make sure that the
execution plan goes through. And now OpenTofu
apply, confirming yes in the prompt.

And that has been finished. Let's now check to
see if everything is okay. And it seems to be.
Let's move on to the next question.
Now destroy these two resources using
OpenTofu destroy. So easy enough. Type in
OpenTofu destroy and confirm with yes. After
that, check if everything completed.
Good. For the next question, navigate to the
directory [root, open OpenTofu projects, explicit
dependency. Okay. And here, within this
directory,
create two local file type resources in main.tf file.
Okay. So main.tf, close these out, and do our
resource local file, resource name,
should be whale in the first case, and file name
should be /root/whale. Okay. The content needs
to be well.
And now we need another resource, but we're
just gonna copy this and replace a couple of
values like this.
Resource called "whale" should depend on "krill",
but do not use reference expressions. When
ready, run OpenTofu init, plan, and apply.
So we cannot use reference expressions, and
that on its own gives us a hint to use the depends
on the whole resource.

So here we're gonna do depends on, and here
we're gonna point it to the local file krill so that
we know that whale depends on the krill,
so the krill needs to be created first, and then the
whale. So once we save that up and do
OpenTofu init,
oh, actually, let's make sure we're in the right
directory. Always be careful about this. OpenTofu
init. OpenTofu plan. We can see that whale and
krill will be created,
so OpenTofu apply. Value yes, and that's it. Let's
check our configuration, and everything seems to
be okay. That's it for this lab.
Thank you for watching, and I'll see you in the
next one.


# Output Variables

Along with input variables, OpenTofu also
supports output variables. These variables can
be used to store the value of an expression in
OpenTofu.
For example, to store the public IP address of an
EC2 instance created by the resource block
called web, we can make use of an output
variable like this.
The syntax used to create this block is the
keyword called output, followed by the name that
we want to call the variable.
Inside the block, the mandatory argument for
value is the reference expression that we saw in
the previous lecture. Optionally, we can also add
a description for this variable.
The output variable that we just added will be
displayed after we run the OpenTofu apply
command. And it will be displayed every time we
run the apply command, even if there are no
changes
to be applied. We can print out all the output
variables within the configuration by using the
OpenTofu output command like this.
Optionally, to fetch the value of just one of the
output variables, we can pass in the name output
variable with the OpenTofu output command.

We already saw that dependent resources can
make use of a reference expression to get the
output from one resource block as an input to
another block.
As such, output variables are not required here.
The best use of OpenTofu output variables is
when you want to quickly display details
about a provisioned resource on screen or to
feed the output variables to other Infrastructure
as Code tools, such as ad hoc scripts or Ansible
playbooks or configuration management and
testing.

```


#### Resource Attributes
https://registry.terraform.io/providers/hashicorp/time/latest/docs/resources/static
```
 resource "time_static" "time_update" {

}

resource "local_file" "time" {

  filename = "/root/time.txt"

  content = "Time stamp of this file is ${time_static.time_update.id}"

}
```

```
tofu show
# local_file.time:
resource "local_file" "time" {
    content              = "Time stamp of this file is 2025-06-21T12:25:57Z"
    content_base64sha256 = "GV47AeZvTl6VYBf0B7izwp49tZ2pOGqx6QpENJ2hqDU="
    content_base64sha512 = "iLa8pGiFlQO3q63Enpe/B30Tf1YQdS144su3By3dclrTRyo5fjIs1Pl809CqTzaOWTxUuJMq//OjX6ANUFRgaA=="
    content_md5          = "793e6e615d50b6f876cfcc8e8ca2b128"
    content_sha1         = "2858dbedde2438b96d8c1b11586392f38770280b"
    content_sha256       = "195e3b01e66f4e5e956017f407b8b3c29e3db59da9386ab1e90a44349da1a835"
    content_sha512       = "88b6bca468859503b7abadc49e97bf077d137f5610752d78e2cbb7072ddd725ad3472a397e322cd4f97cd3d0aa4f368e593c54b8932afff3a35fa00d50546068"
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/root/time.txt"
    id                   = "2858dbedde2438b96d8c1b11586392f38770280b"
}

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

#### Resource Targeting

```
resource "random_string" "server-suffix" {
 length = 6
 upper = false
 special = false
}

resource "aws_instance" "web" {
  ami = "ami-013356knkd354"
  instance_type = "web ${random_string.server-suffix.id}"

}
```

```
 tofu apply —target random_string.server-suffix
```

main.tf
```
resource "local_file" "whale" {
  filename = "/root/whale"
  content = "whale"
  depends_on = [ local_file.krill ]
}

resource "local_file" "krill" {
  filename = "/root/krill"
  content = "krill"
}
```

key.tf
```
resource "tls_private_key" "pvtkey" {
  algorithm = "RSA"
  rsa_bits = 4096
}

resource "local_file" "key_details" {
  filename = "/root/key.txt"
  content = tls_private_key.pvtkey.private_key_pem
}
```


####  Output Variables

```
resource "aws_insatance" "cerberus"{
 ami = var. ami
 instance_type = var.instance_type
}

output "pub_ip"{
 value = aws_instance.cerberus.public_ip
 description = "print the public IPv4 address"

}



output "<variable_name>"{
 value = "<variable value>"
 <arguments>
}



```
variable.tf
```
variable "amil'{
 default = "ami-06178cf087598769c"
}

variable "instance_type"{
 default = "m5.large"
}

variable "region"{
 default = "eu-west-2"
}
```

```
$ tofu apply
pub_ip = 54.214.145.69

$ tofu output pub_ip
54.214.145.69
```


lab
main.tf
```
resource "random_pet" "my-pet" {
  length    = var.length
}

output "pet-name" {
    value = random_pet.my-pet.id
    description = "Record the value of pet ID generated by the random_pet resource"

}

resource "local_file" "welcome" {
    filename = "/root/message.txt"
    content = "Welcome to Kodekloud."
}

output "welcome_message" {
  value = local_file.welcome.content
}
```
var
``` 

variable "prefix" {
  default = "Mrs"  
}


variable "separator" {
    default = "."
}

variable "length" {
    default = "1"
}
```


lab2

```
resource "random_uuid" "id1" {

}

resource "random_uuid" "id2" {

}

resource "random_uuid" "id3" {

}

resource "random_uuid" "id4" {

}

resource "random_uuid" "id5" {

}

resource "random_uuid" "id6" {

}

resource "random_uuid" "id7" {

}

resource "random_integer" "order1" {

  min     = 1

  max     = 99999

}

resource "random_integer" "order2" {

  min     = 1

  max     = 222222

}
```

```
output "id1" {

   value = random_uuid.id1.result

}

output "id2" {

    value = random_uuid.id2.result

}

output "id3" {

    value = random_uuid.id3.result

}

  

output "id4" {

    value = random_uuid.id4.result

}

output "id5" {

    value = random_uuid.id5.result

}

output "id6" {

    value = random_uuid.id6.result

}

output "id7" {

    value = random_uuid.id7.result

}

output "order1" {

 value = random_integer.order1.result

}

output "order2" {

 value = random_integer.order1.result

}
```


## OpenTofu State

```
tofu apply -refresh=false
```


```

Introduction to OpenTofu State


state concepts. What it is, why it's used, and
some of the best practices to be followed when
working
with the state. When a resource is created for the
first time after running the OpenTofu apply
command, a state
file is created by default within the same directory
containing the configuration files. The file called
terraform.tfstate is the state file.
And note that by default, OpenTofu also creates
a backup of this file called
terraform.tfstate.backup. The state file is a JSON
data structure that contains the complete record
of the infrastructure
created by OpenTofu. If we inspect the contents
of this state file, we can see details related to the
infrastructure
that was created with the current configuration.
The details such as the resource ID, the provider
info, and all the resource attributes are
stored within this file. The state file contains
every detail pertaining to the infrastructure that
was created by OpenTofu and it uses it as a
source of truth when using commands such as
OpenTofu plan and

apply. If we run OpenTofu plan without making
any changes to the configuration file, the very
first thing that OpenTofu does is to check if the
state file exists and then refresh it before
generating the execution plan. When OpenTofu
refreshes the state file, it checks the current state
of the resource in the real world and determines
if there are any changes to be made with respect
to what
is defined within the configuration files. In this
example, it finds that there are no changes to be
made. And the same thing happens when we run
OpenTofu apply.
The state file is refreshed again and since there
are no changes between the state and the
configuration file, the resource is not updated.
We can however force OpenTofu to not refresh
state by passing in the -refresh=false option.
false option. When we do this, OpenTofu will no
longer update the state file to keep it in sync with
the
actual resource. This of course is not
recommended as it can introduce
inconsistencies, especially if changes are
somehow introduced to the actual resource by
other means such as a manual update.

However, in large environments containing
hundreds and thousands of resources, the
OpenTofu refresh can take a lot of time to
complete.
In such cases, the dash refresh equals false
option may come in use when you have to add or
update resources quickly.
An important point to note here is that while we
can disable state refresh, the state file itself
cannot be disabled and is mandatory for
OpenTofu to work.
Let's now see how the state works to keep
resources in sync with the configuration. Let's
update the variable for instance type from
m5.large to p3.micro and run OpenTofu plan.
As expected, OpenTofu refreshes the state file
and understands that there is a change in the
instance type. And the execution plan generated
shows that the resource will be recreated
because of
this change. To sum up, OpenTofu uses a state
file as a blueprint of the resources in the real
world.
Irrespective of the size of the infrastructure, the
type of the provider used and the number of
resources defined, OpenTofu will always create a
state file and use it to store information

about the state of the infrastructure in the real
world. The state file also keeps track of
dependencies between the resources.
If we take the example of resources created in
this example configuration, the state file created
will record the dependency of the web resource
on the DB resource.
And this is what allows OpenTofu to decide in
which order the resources should be provisioned
or destroyed. During the provisioning process,
the DB instance is created first, followed by the
web instance.
And during the deletion, the web instance is
destroyed first, followed by the DB instance.
There are a few things to be wary of when
working with the state.
First one is that the state file contains sensitive
information. It contains every little detail of the
infrastructure. Now, these can be variables that
we have used for the configuration or attributes
such
as SSH keys and initial passwords. And since the
state file is stored locally as plain text files,
anyone can see it.
And as a result, we need to make sure that the
state file is stored in a secure remote backend.

And it is for this exact reason why we should
never commit state files to version control
systems like GitHub, GitLab, or Bitbucket.
Instead, we should make use of the remote state
backends, such as Amazon S3, Google Cloud
Storage, or Azure Blob Storage to save the state
file.
Version control repositories should only be used
to store configuration files that do not contain any
sensitive data. And finally, OpenTofu state is a
JSON data structure that is meant for internal
use with
OpenTofu. We should never manually attempt to
edit the state files ourselves. However, we can
use several commands to manipulate the state.
We will revisit them in the upcoming lectures.


lab

Hi, and welcome to the lab about OpenTofu state.
So, right out of the box, our first question is,
which location is the OpenTofu state file stored in
by default?
So, if we noticed in the previous lectures,
whenever by default we initialize OpenTofu, we
know that the state file is created inside the
Configuration Directory.
Next question says, which option should we use
to disable state? So, we cannot disable state.
That is not something that OpenTofu uses as one
of its options,
because it uses the state file to have a reference
of the applied and existing resources on the
infrastructure. So, the answer is, we cannot
disable state.
The next question says, which format is the state
file stored in by default? And if you've ever
opened up the terraform.tfstate,
you would have noticed that the format it is
stored in is JSON. Which of the following
commands does not refresh the state?
So, OpenTofu plan does and OpenTofu apply
does as well, leaving OpenTofu init, which does
not have a need to refresh the state.

What is the name of the state file that is created
by default? So, when created by default, as seen
before,
the default naming convention for the state file is
Terraform.tfstate. Okay, navigate to the
configuration directory slash root slash OpenTofu
project slash project slash.
We have created a few configuration files. The
directory has been initialized and the provider
plugins downloaded inside the .terraform
directory.
However, there is no terraform.tfstate file created.
Why is that? So, we know that the OpenTofu init
does not refresh the state file or
do anything with the state file. There is definitely
no syntax error. The plan does refresh the state
file, but it does not create it, but apply does.
So, judging by our situation here, the OpenTofu
apply was not run. Okay, so, run the OpenTofu
show command and identify the ID created for
the resource called speedforce. So, let's change
to our OpenTofu project, project flush, run
OpenTofu show command. And we see that there
is no state.
So, what do we do here? We cannot print the
state. Because there is no state to be printed. So,
the answer is no details printed.
There is no state. Now, run OpenTofu apply in
this directory. So, let's go ahead and do an
OpenTofu apply, confirm yes and prompt.
And we see that four resources were created.
We can go ahead and do a check now to see if
everything is okay.
So, moving on to the next question. Now, check
OpenTofu show again. What is the value of the
ID for the resource called speedforce?
So, let's type in OpenTofu show and extend this a
little bit so it's easier to see. And let's look for the
speed force.
So, let's resource called speedforce. And we see
that there is no state. So, let's resource called
speed force. Here it is.
And we're looking for the ID, which is this. It
starts with EBEB and ends with 2981. So, EBEB
2981.
This is the resource we're looking for. Okay, let's
see about the next question. Okay, we have just
added a new configuration file called AWS infra

into the configuration directory and provisioned
the resources. And an AWS S3 bucket was
created. So, okay, we have noted that.
Inspect the Terraform team. Inspect the
Terraform TF state file or run the OpenTofu show
command. So, we can do both.
We can open up the state file here. Or we can do
another OpenTofu show and take a look at it that
way.
You will notice that all the attribute details for all
the resources created by this configuration are
now printed on the screen.
Among them is an EC2 instance, which is
created by the resource called dev server. See if
you can find the private IP for the instance that
was created.
So, I think the easiest to do right here is to just
search for private IP by using the hotkey control
F and we can see that the private IP of the
instance is
10.40.132.251. And there is no secondary private
IP. So, this is the one that we're looking for. So,
10.40.132.251. We will now soon be working with
AWS services and deploying resources on AWS
with OpenTofu.
And that concludes this lab. Thank you for
watching and have a nice day.
```

#### lab

aws-infra.tf
```
resource "aws_instance" "dev-server" {
    instance_type = "t2.micro"
    ami         = "ami-02cff456777cd"
}

resource "aws_s3_bucket" "flashpoint"  {
    bucket = "project-flashpoint-paradox"
}
```
provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.13.1"
    }
  }
}


provider "aws" {
  region = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  s3_use_path_style = true
  endpoints {
    ec2 = "http://aws:4566"
    iam = "http://aws:4566"
    s3 = "http://aws:4566"
    dynamodb = "http://aws:4566"
  }
}
```


## Working with OpenTofu
### commands

```
tofu validate
tofu ftm
tofu show -json
tofu providers
tofu refresh
tofu graph
```

```
Let's now revisit all the commonly used
OpenTofu commands. First, let's look at the
OpenTofu validate command. When we run this
command, if everything is correct with the
configuration, we should
see a successful validation message. But if there
is an error in the configuration file, the validate
command will show you the
line in the file that is causing the error with hints
on how to fix it. In this example, we have an
incorrect argument used for the local file
resource, it should
be file permission and not file permission. The
OpenTofu format command scans the
configuration files in the current working directory
and
formats the code into a canonical format. This is
a useful command to improve the readability of
the OpenTofu configuration files.
When we run this command, the files that were
changed in the Configuration Directory are
displayed on the screen. The OpenTofu show
command prints out the current state of the
infrastructure as seen by OpenTofu.
We have already seen this command in action so
far in the course. We can also make use of the —
json flag to print the contents in adjacent formats.

The OpenTofu providers command prints out all
the providers used by the configuration. In this
example, there is only one provider shown as the
configuration only uses AWS resources.
Let's revisit the OpenTofu output command. If
you want to print out all output variables in the
configuration directory, use the command
OpenTofu output. You can also print the value of
a specific output variable by adding the name of
the variable to the end.
In the previous lecture, we went over OpenTofu
State and we learned that commands like
OpenTofu plan and OpenTofu apply refresh state
before generating the execution plan.
This can also be done by running the OpenTofu
refresh command. The OpenTofu refresh
command is used to sync OpenTofu with the real-
world infrastructure.
For example, if there are any changes made to a
resource created by OpenTofu outside of its
control, such as a manual update, the OpenTofu
refresh command will pick it up and update
the state file. This reconciliation is useful to
determine what action to take next during a
subsequent apply. An important point to note
here is that this command will not modify any
infrastructure

the state file. This reconciliation is useful to
determine what action to take next during a
subsequent apply. An important point to note
here is that this command will not modify any
infrastructure
resources but updates the understanding of
infrastructure by means of a state file. The
OpenTofu graph command is used to create a
visual representation of the dependencies
in an OpenTofu configuration or an execution
plan. This command can be ran as soon as you
have a configuration file ready, even before you
have initialized the configuration directory with
the OpenTofu init command. The output is
generated in a dot format and can be visualized
using the graph visualization
tools such as GraphViz. Here we have an
example of our AWS EC2 Instance and how all of
the resources, Providers and
Modules are tied together. This output can be
really useful to get an idea of how all of your
configuration ties
together.

lab

Hello and welcome to the lab about OpenTofu
commands. So, let's look at our first question.
Which command can be used to create a visual
representation of our OpenTofu resources?
So, we know that the command to do a visual
representation is the tofu-graph command. The
second question says we have created a
configuration directory at
slash root slash OpenTofu project slash project
Shazam. So, let's open that up. The configuration
file inside will be used to create an RSA-type
private key
and then a certificate signing request or a CSR
using this key. However, there is an error with the
configuration.
Use the OpenTofu validate command to
troubleshoot and fix the issue. You don't have to
create the resources yet. You only need to fix
errors reported by the OpenTofu validate.
So, first in your terminal open up the project
Shazam. And here we're going to run the
OpenTofu validate. So, it can show us what
exactly is wrong with the configuration.
On main.tf line 8 the resource TLS private key
DSA bits equals 2048. An argument named DSA
bits is not expected here.

Did you mean RSA bits? And that's exactly what
we meant. It was a typo here. We're going to
save that up and check our configuration now.
And it seems to be good. Moving on to the next
question. If you completed the previous question
correctly, tofu-validate should have passed.
Now run tofu-plan and generate a configuration
plan. So, let's try to do OpenTofu plan. And did it
generate a configuration plan?
It did. So, the answer is yes. Now try creating
resources with a OpenTofu apply. Did it work?
So, let's try.
OpenTofu apply. Value yes. And it actually
produced an error. So, the answer is no. The
OpenTofu apply failed in spite of our validation
working.
This is because the validate command only
carries out a general verification of the
configuration. It validated the resource block and
the argument syntax,
but not the values that the arguments expect for
a specific resource. Okay. So, let's see what we
can do next.

The error in the configuration is inside the
resource block for the TLS private key type
resource. It contains the configuration that we
needed for generating RSA type key.
Inspect the resource block and fix the issue.
Okay. So, let's do the main.tf. Let's scroll this
down a little bit and let's take a look at our
configuration.
So, looking through our main.tf file, we see that
there is a resource with a local file called key
data,
which has a filename, a content, and a
permission. Then we have a TLS private key,
which has an algorithm of RSA, a number of RSA
bits,
and an ECDSA curve, which actually belongs to
the ECDSA algorithm. And since we're using
RSA, we do not need to use this argument.
So, let's go ahead and delete it and check the
rest of our configuration. So, TLS cert request
CSR private key file seems to be okay,
which depends on the local file resource with the
subject common name and organization. Good.
So, let's go ahead and try to do an OpenTofu init
plan and apply now and see what happens.

So, the plan does go through. Let's do the apply
and yes. And actually, we got produced with an
inconsistent final plan.
But let's check and see if everything is actually
okay. So, OpenTofu plan shows changes to be
applied. Make sure the configuration is up to date
and apply.
So, let's verify that again one more time. Do
OpenTofu apply and run that apply one more
time. So, it seems that we needed to first update
the TLS private key resource
and then run another apply for the new TLS cert
request to generate properly. Now, if we run
OpenTofu apply again and check our
configuration,
now everything seems to be okay. Now, format
the main TF file into a canonical format. We know
that the command to format the code to
canonical format is OpenTofu fmt.
So, if we run it, we see that everything is sorted
into a nice and readable, pleasing format. So,
let's go and check our configuration now and
move over to the next question.
So, navigate to the directory of
/root/OpenTofu/project/projectA. Okay, and let's
immediately change our terminal as well so we
don't have a problem later on.

Project A. Good. We have already created the
resource specified in this configuration. Fetch the
details from the state file and identify the value of
the filename argument.
Okay, so let's take a look at the terraform.tfstate
and search for the filename which seems to be
/root/codes. So, filename equals /root/codes.
In these OpenTofu labs, we have used multiple
Providers so far. But what are actually providers?
So, providers are essentially plugins that you
download to be able to work with the specified
providers
for specific providers like the local file provider or
the AWS or Azure or GCP provider. So, in a
nutshell, they are plugins.
Which one is a valid subcommand of the
OpenTofu providers command? So, list is not,
status is not, and update is not, but mirror
actually is.
So, we're going to select mirror and to actually
take a look at which are supported, you can do
the OpenTofu providers help.
And that will give you the list of available
subcommands. Now, our next question is a new
configuration directory
/root/openTofu/project/provider has been created.

So, let's open that up and switch our terminal
right away as a good practice. And let's say, let's
see.
We have already run the OpenTofu init
command. Now, check the provider plugins that
have been downloaded from the command line
utility
instead of inspecting the configuration directory.
After that, choose the correct option. So, similarly
to our last question, we instead of using the help
flag,
we'll just use OpenTofu providers and we see
that we have two providers, AWS and local. So,
we select local and AWS.
And that's it for this lab. Thank you for watching
and I'll see you in the next one.
```

###  LifeCycle Rules


```
Let's now go over different lifecycle rules that we
can make use of in OpenTofu. Using lifecycle
rules, we can control how OpenTofu creates and
destroys resources.
In this example, we have an EC2 instance
created with the instance type of m5.large. If we
change the AMI for this specific resource, the
default behavior of OpenTofu is to first destroy
the current resource and then create the new
instance with the new instance AMI.
To make sure that the resource to be replaced is
created first, we can make use of the create-
before-destroy rule.
Lifecycle rules can be added by using the
lifecycle block that goes inside the resource
block. And as we can see right now on the
OpenTofu apply, the new instance is first created
and then the old instance is deleted after that.
Now, if we do not want a resource to be
destroyed for any reason, even if changes in the
configuration files would warrant the said
resource to be deleted, then we can use a
lifecycle rule like this.
When the argument prevent-destroy is set to
true, OpenTofu will reject any changes that will
result in the resource getting destroyed and
display an error message.

This is an especially useful safeguard to prevent
your resources from getting accidentally deleted,
especially resources such as databases, which
may be something you do not want to change
once they're provisioned.
Please remember that this property does not
make the resource immune to the OpenTofu
destroy command. It will only prevent resource
deletion from changes made to its configuration
and a subsequent OpenTofu apply.
When the ignore-changes rule is applied, it will
prevent a resource from being updated based on
a list of attributes we define in the lifecycle block.
Using the same example as before, we have an
EC2 instance with the tag name
CerberusWebServer. If the tag is changed either
within the configuration or if the change is
introduced on the actual EC2 instance by some
other means outside of our OpenTofu
configuration,
a subsequent OpenTofu apply will always try to
update the resource to the value specified in the
configuration file. In some cases, we may not
want OpenTofu to do this.
In this case, we do not want OpenTofu to update
the resource as the change is just the tag,
something that we are happy to ignore.

To do this, we can add an ignore_changes
lifecycle, just like this. With this rule, any changes
to tags will be ignored during the apply.
Observe that the argument ignore changes can
accept a list as identified by the square brackets
enclosing the tags. As such, we can add multiple
attributes here that we may want OpenTofu to
ignore.
You can also replace the list with the all keyword
if you do not want the resource to be modified for
changes in any resource attribute.
Now, let's head over to the Hands on labs and
practice updating and deleting infrastructure
resources.

lab

Hello and welcome to the lab about OpenTofu
lifecycle rules. The first question that we have is
we have a directory called root OpenTofu
projects
project mysterio. The main.tf file has already a
couple of resource blocks, which resource types
they use. So let's open up our main.tf and we can
see that there is a local file and a random string.
So we may have a local file and a random string,
which is the correct answer. The next question
says now create these two resources that have
been defined in this
configuration file. So what do we do? We change
the directory to project mysterio, run OpenTofu
init, then for good measure, we run OpenTofu
plan. To verify the execution plan,
two resources will get created, we run OpenTofu
apply, we get the same execution plan, confirm
with Yes, and create the resources. Next
question, which resource is created first
in this case. So we have the local file and the
random string. So since the content of the local
file resource is an attribute reference to the
random string's ID, this resource
has to be created first in order for this to be
created after, so string is the correct answer. We
have modified the resource configuration again,
run an OpenTofu plan now, what would happen?

So let's see. If we run the OpenTofu plan and
extend this a little bit, we see here that the
content
changed and resource length changed here. And
it, the resources must be replaced. So both
resources will be replaced. Why is the string
resource being recreated? So there are a couple
of suggestions here,
the resource name was changed, which is not
the case. The variable name was changed, which
is also not the case. But the value for the
argument called keepers has changed.
And if we take a look at here, this value has been
changed. So this is the correct answer. All the
resources for the random provider can be
recreated by using a map type argument called
keepers. A change in the value will force the
resource to be recreated. This argument accepts
arbitrary key value pairs. And in our example, it is
set to the key called length, whose value
was updated from 10 to 12 in the variables.tf.
Running an OpenTofu apply now will destroy the
current random string resource and then create a
new one with the length that is 12 characters
long.

Let's change the order in which the resource
called string is recreated. Update the
configuration so that when applied, a new
random string is created first before the old one
is destroyed.
When ready, apply the changes with OpenTofu
apply. So here we have to make use of the
lifecycle the lifecycle rule of create before
destroy. So let's minimize that a little bit. So we
have room
here. And what we want to do is our string
resource needs to have a lifecycle rule. And the
lifecycle rule that we're going to use is called
create before destroy,
and we will set it to true. Now, once we do
OpenTofu apply, we can see that the resources
will be replaced. And then we will confirm with
Yes.
Now, as you can see here, it first created the
string, then created the file, and then destroyed
the old string. So this is the order that we wanted
when we set up the create before
destroy rule. Let's check our syntax. And
everything seems to be in order. Let's move on.
Let's move on. The resource block for the file
resource has been updated.

This will force the resource to be recreated
during the next apply. But before that, let's also
add a lifecycle rule of create before destroy to
this resource block. When ready,
apply the changes with tofu apply. Okay, so we
need to update the main.tf file by adding the
lifecycle rule under the resource of local file,
similar to what we did for the random string.
So we can go ahead and just copy this whole
section here and place it just below the content,
save that up. And once the guide says, once the
lifecycle rule has been added,
only run the apply command once, and we will
learn why soon. Let's follow that and do an apply
and then check our configuration. Okay.
Great, we have now added the lifecycle rule and
forced the resources to be created first and then
destroyed. What is the ID of the file resource we
just created,
run the OpenTofu show or OpenTofu state show
local file file to find out. And out of the two, this
command will give us exactly what we want. But
if the terminal, like in my case,
does not allow you to copy and paste, you can do
OpenTofu show and look for the local file
resource, and then look for the file.

And what we're looking for is the ID, which is
F09E and ends with 5376. And that would be our
answer. Read the contents of the file
/root/random_text. Okay, manually, try opening it
with Visual Studio Code or cat command and a
text that what is the content of this file. So let's
see. /root/random text. If we take a look at the
file tree here, we do not see any file like that. But
just to be sure, open up the terminal and go to
[root and just do an Is
just to be sure. And same result, we can only see
the README and OpenTofu projects, providing
the -LAH flag will give us like the all of the files,
including the hidden files.
But again, we do not see that file. So we pick
there is no such file or directory. Where did the
file go? If you observe the output of the previous
supply and scroll up, so, yep,
somewhere around here, local file, and it was
definitely created. You will see that the lifecycle
rule we applied caused the local file to be created
first and the same file to be
destroyed during the recreate operation. This
goes to show that it is not always advisable to
use this. In this example, the filename argument
for the local file resource has to be unique,

which means that we cannot have two instances
of the same file created at the same time. The
random string resource, on the other hand, is a
logical resource
that is only recorded in the state and does not
have such a restriction. If you run OpenTofu
apply again, the file resource will be created as it
does not exist currently.
So if we run OpenTofu apply now, and we need
to be in our directory and run that OpenTofu
apply now,
we should see the new random text file created.
We have now wiped out the resources that were
created in this configuration directory and
updated the main.tf file. Now it only contains
a single random pet resource called super pet.
So if a change is made in neither or both, the pet
ID will be created. Now update the configuration
so that the resource super pet
is not destroyed under any circumstances with an
OpenTofu destroy or OpenTofu apply. Use the
OpenTofu apply command after making the
changes. So what we need to do here is add
another lifecycle rule called prevent destroy and
set it to true. Now if we change any values, either
the length or the prefix and run the apply, the
resource won't be destroyed at any point. So

if we run a subsequent OpenTofu apply, nothing
will happen. If we change the length to 12 and
Mr. to Mrs. and run an OpenTofu apply now,
it will wam us that the lifecycle prevent destroy is
set, but the plan calls for this resource to be
destroyed, therefore warning us that we cannot
destroy the resource while that
lifecycle rule is in place. And that's it for this lab.
Thank you for watching and I'll see you in the
next one.

```

```
resource "aws_instance" "cerberus" {
    ami = "ami-02cff456777cd"
    instance_type = "m5.large"
    tags ={
     Name = "Cerberus-Webserver"
	lifecycle {
	 create_before_destroy = true
	 }
    }
}




لو مش عايز احزف اى شى 
lifecycle {
	 prevent_destroy = true
	 }

لو عايز اتجاهل اى تغير ممكن يحصل

resource "aws_instance" "cerberus" {
    ami = "ami-02cff456777cd"
    instance_type = "m5.large"
    tags ={
     Name = "Cerberus-Webserver-1"
	lifecycle {
	 ignore_changes = [
	 tags
	 ]
	 }
    }
}



```

### Data Sources
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/s3_bucket
https://opentofu.org/docs/language/data-sources/
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ebs_volume

```
data "aws_key_pair" "cerbrtus-key"{
 key_name = "alpha"
}

or
data "aws_key_pair" "cerbrtus-key"{
 filter{
  name = "tage:project"
  values = ["cerberus"]
 }
}

resource "aws_instance" "cerberus"{
 ami = var.ami
 instance_type = var.instance_type
 key_name = data.aws_key_pair.cerbrtus-key.key_name
}
```

```
In this section, we will take a look at data sources
in OpenTofu. By now, we have seen how to
provision resources using OpenTofu,
and we have used the reference expressions
whenever we need to make use of the attributes
of one resource as an argument for another.
Earlier, we used the example where we created
an AWS Key Pair resource, and then used the
Key Name attribute of this resource
as an argument for creating an AWS EC2
instance. In this case, both the KeyPair and the
AWS instance are part of the same
OpenTofu configuration, which allows us to make
use of the reference attributes. However, there
would be several cases where a resource
already exists
or was created by other means. For example, an
existing Key Pair that was perhaps created
manually, or perhaps using some other tools
such as CloudFormation, Ansible, etc.,
or it may have been created already as part of a
different OpenTofu configuration. To make use of
such a resource within our OpenTofu
configuration,
we can make use of data sources. Let's assume
that the KeyPair called alpha already exists in our
AWS account.

Now, to make use of this KeyPair within our
OpenTofu configuration, we can use a data block,
just like this.
The data block is quite similar to the resource
block. It is identified by the keyword called data,
followed by the instance type,
which in this case is AWS Key Pair. Next, we
have the logical name for the datasource. Just
like a resource block, this can be named
anything.
This KeyPair resource will be used to create an
EC2 instance called Cerberus, so let's name it
CerberusKey. Inside the block, we have to
specify some arguments that will uniquely identify
the key.
We already know that the key name is alpha, so
we can use the key name as the argument to
identify this resource
and specify alpha as the name. And that's it. We
now create a new resource block for creating an
EC2 instance called Cerberus.
Note that for this argument called key name,
we're making use of a reference expression to
the data source called Cerberus Key.

The syntax to refer to this data source is the
keyword called data, followed by the instance
type, the data source name, and the key_name
attribute.
The details about the arguments accepted by the
datasource and the attributes exported can be
found in the provider documentation.
In this example, we use the key_name with the
value alpha to uniquely identify the Key Pair. As
you can see from the documentation, this is
optional.
We can use any of the other ways to identify the
resource, such as the key ID or filters. With
filters, we can use valid filters such as tags to
identify a resource.
For example, this Key Pair has a tag project with
value equal Cerberus. So instead of using the
Key Pair as an argument,
we can also use filters to identify the resource,
like this. Finally, to distinguish between resources
and data sources, let's do a quick comparison.
So the resources are created with the resource
block and data sources are created with the data
source. Resources in OpenTofu are used to
create, update, and destroy infrastructure,

whereas data sources are used to read
information from a specific resource. Regular
resources are also called managed resources, as
it is by extension managed by OpenTofu.
Data sources are also called data sources.

data sources. The first question that we have on
our list is a data source once created can be
used to create,
update and destroy infrastructure. True or false?
The answer is false because a data source can
only read resource data and use that information
within OpenTofu.
For any more information, refer to the
documentation link on the top right. So the
answer is false. The next question says a data
source can be created using the data block. This
is
true because we're using the data block the
same way we're using the resource block. A new
configuration directory has been created at
/root/OpenTofu/project/lexcorp.
Let's check that out and let's switch our terminal.
There's OpenTofu and project lexcorp. A data
source block is defined in the main.tf file to read
the contents of an existing file.
There is also an output variable that uses
reference expressions to print the file content
using this data source. However, something is
wrong. Troubleshoot and fix the issue.
When ready, run OpenTofu init plan and apply to
create the data source. The configuration should
print out the output variable correctly.

Okay, so what do we need to do here? We can
see that the output os version is a value to the
data
local file that dot os dot content. Okay, so
checking out our main.tf file, we can see an
output block and a data source block. The output
block has an output called os version and it is a
value to data dot local file dot content. And we
have a data source called local file called os. And
we have a data source of type local file called os
with a file name argument equals slash
etc slash os dash release. Right off the bat, we
notice that the value equals data, then local file,
then content. So first of all, the thing that we have
to correct is that we have
to reference the name of the local file data
source. Now, the second problem that we have
here is that we don't have a block called data
source, the block is just called data. So now we
have
data "local file" "os" with a file name of that of
slash etc slash os dash release. Once we save
that up and run OpenTofu init, it will download the
required providers. OpenTofu plan will show us
the

execution plan. And then OpenTofu apply will
apply these changes once we confirm yes in the
prompt. Good. Now let's check our configuration.
And it seems to be okay. Now let's practice how
to work
with data sources from other Providers. The next
few questions will be based on the AWS
Provider. Although we have only predominantly
worked with local and a random provider. This
exercise will
help you learn how to work with different data
sources using the documentation. Don't worry if
the configuration blocks and the arguments are
unfamiliar, we will cover them in detail in the
upcoming sections. Let's check out what's
installed for us. We have now created a new
configuration file called ebs.tf within the same
configuration directory we have been working
on. What is the resource type that we are
working with here. So let's take a look at our
ebs.tf.
And we can see that we are working with a data
of AWS EBS volume called GPT volume. so
actually, the data type that we're working in is the
first value which is AWS EBS volume.

Once this data source is created, how do we
fetch the volume ID for the resource that is
created in AWS, you may have to look up
documentation for this one, the documentation
tab is available
in the top right. Okay, so let's check out the
documentation and see what's in store for us.
Okay, so we were supposed to fetch the volume
ID. And if we take a look at the attribute
reference,
we actually see that there is a volume ID attribute
that we can fetch for our use case. So going back
into our configuration, the correct answer is
volume id.
Another file called s3.tf has now been created. It
too has a data source that will be used to read
data off an existing S3 bucket. However, there is
a mistake in the argument used,
what is wrong here, you may have to look up the
documentation for this one. And the
documentation is again available in the top right.
Okay, so let's open up the documentation and
see what's
exactly happening here. So let's go first back to
our configuration. And notice that we use the
bucket name as an attribute. So let's check out
what we have in the S3. So attributes reference

says that we have an ID, ARN, bucket domain
name, bucket regional domain name, hosted
zone ID, region, website endpoint and website
domain. There is however, no bucket name. So
it's not
a valid argument. So let's go ahead and choose
bucket name is not a valid argument. We will get
more practice with data sources and AWS
resources in the upcoming lectures.
That is it for this lab. Thank you for watching, and
I'll see you in the next one.
```


### Meta Arguments &amp; Count and for-each

```
and forEach metaarguments. To create multiple
instances using the same resource block,
we can make use of metaarguments such as
count and forEach. So let's do the count first.
Because we have set the value of count to 3,
a OpenTofu apply command will create 3 EC2
instances with the resource block called web.
The OpenTofu state list command confirms that 3
EC2 instances were created and recorded in the
state
file. However, you can see from the output the
instances are created as a list. The resources are
identified as aws instance.web0 where the index
0 is the first instance in the
list. The next two resources are webl and web2.
Instead of hardcoding the value of count as 3, we
can make use of a list type variable. For
example, let's consider a new variable
called web servers that contains the names of all
web servers to be created. Webl , web2, web3,
like this. To get the count of items in the list,
we can make use of the built-in length function,
like this. In this example, this will return a value of
3 which is then assigned to the count argument.

Now, if we want to loop through the list type
variable and assign a tag for each instance with
the corresponding values at each index, we can
use an expression like this.
Here, for the first iteration, the count index would
be 0 and the value of bar.webservers[0] would be
webl. And for the next iteration, it would be web2
and web3 for the final iteration.
So count is pretty handy to loop over a list and
provision instances. However, there is one
significant drawback of using count. To see this in
action, let's remove webl from the
list and run tofu apply. The expectation is that
OpenTofu would just remove the EC2 instance at
index 0 which has a tag called webl. However, in
reality, when we run tofu plan or apply,
the execution plan would show two resources to
change and one to destroy. This happens
because while using count, the resources are
created as a list of resources. The first element in
a list
is always 0. As a result, when we remove the first
element webl from our webservers list, the web2
became the new element at index 0. This means
that the tag with name equals webl
will no longer match what is defined and is
marked to be updated. The first element in a list
is

always 0. As a result, when we remove the first
element webl from our webservers list, the web2
became the new element at index 0. This means
that the tag with name equals webl no longer
matches what is defined and is marked to be
updated. Similarly, web3 became the new
element at index 1 and here again the tag for this
instance is marked to be updated. There is no
longer an element at index 3 and as a result the
current instance at this index is marked to be
destroyed. ForEach is another way to create
multiple instances using the same resource
block. Let's use the same example as before but
this time with for each. To do this, we can update
the
resource block like this. We use the meta
argument called for each to loop through the
variable called webservers and each value
expression is used to assign the corresponding
value to the key called
name. Another thing to note here is that the
variable type of webservers cannot be a list when
working with for each. Instead, we need to either
use a set or a map. In this example,
we're going to change it to a set. When we run
the OpenTofu apply now, the three resources will
be created as before.

Like before, if we run an OpenTofu state list, we
can see that the resources are now created as a
map of resources rather than a list.
And because the instances are created as a
map, if we remove webl from the webservers
variable like we did before, a subsequent
OpenTofu plan or apply will show that only the
corresponding instance
webl will be removed. The other two instances
corresponding to the elements web2 and web3
will remain unaffected.


lab


for each. The first question that we have in this
lab is the following.
A new configuration directory has been created
as /root/OpenTofu project / project shade. Inspect
it. How many files will be created by this
configuration?
So in the main.tf, we only see one resource of
type local sensitive file, so only one resource
will be created. Next question says now add a
count argument to create three instances of this
resource.
When ready, run OpenTofu init plan and apply.
So what we need to do here is add another
argument called count with the value of three.
Now in our terminal, open up the Configuration
Directory, run OpenTofu init to initialize the
Providers to apply our configuration.
Let's check out the execution plan. And it seems
three files will be created. So we will confirm with
yes and then check our task.
And it seems to be completed successfully. Let's
go to the next one. The resource local sensitive
file name is now created as what?

So we noticed that it actually has three iterations
of this file. And it has actually been created as a
list.
Every time that we use count, the resources are
always created as lists. Our next question says
what is the ID for the resource element at index
one?
So by running the OpenTofu show, we can take a
look at the name of element one. And the ID
begins with 6B32, 6B32 and ends with 64CE.
This is our correct answer. How many files were
actually created when the apply was run? So if
we take a look at the root user data, we only
have one of these files.
So even though we used three counts, it actually
created only one of those files. We have now
created a variables.tf file in the same
configuration directory.
We can see it here. Update the main.tf file to
make use of the list type variable defined for the
file
name argument. Use count to loop through all
the elements of this list and do not use
hardcoded values. Use the variable called
content for the argument called content.

So let's go to our main.tf file. And what we need
to do here is set the file name to use the variable
users var.users.
And then count index to iterate through all the
indices. The content needs to be our content like
here. And the count needs to be of length
var.users.
Good. Now if we check this, our syntax seems to
be correct. We have reverted back to the old
configuration file and cleaned up the resources
created
so far. A variable called users now has a default
value added to it. So let's check out the users.
Okay. So it has a couple of values in total. What
type of variable is it? So we can check the type
here, which is a list of strings.
Can the same elements in this list be used as it is
for a set instead? So to recall our memory, what
is the difference between a list and a set?
A set cannot have any duplicate values, but a list
can. So if we check the users, we can see that
/root/user10 is actually put in
two times into this variable, meaning that this
cannot be a set because of the duplicate
elements. Let's do the same exercise as before,
but this time we will make use of the foreach

metaarguments to create the files in this
configuration. Just like before, don't use any
hardcoded values. Use foreach to loop through
the list type variable called users.
Use the variable called content as the value of
the argument content within the main.tf. When
ready, run OpenTofu init plan and apply.
Okay. So opening up our main again and our
variables.tf, we can see that we have the same
list of strings
for users, the same default value for the variable
content, and we have the local sensitive file
resource. So now we have to iterate using the
foreach to loop through the list type variable
users.
So in here, we're going to go ahead and put in
actually above just to make it a little bit cleaner
for each function to set all var users because our
users have a duplicate
value, all / root / users. And a foreach needs a
set in order to iterate through the values. So we
have to convert it to a set first.
And now our users need to, the content needs to
be our content. And the file name, lastly, needs to
be each value.

So each value in the foreach loop. We set that
up, run OpenTofu init, OpenTofu plan for good
measure. You can see three files created, which
is good, and OpenTofu apply.
And confirm Yes. Now check our configuration.
And it seems to be working. And as you can see,
the toset function made sure that only one
/root/user 10
is created. So the next question says the
resource called name is now created as what we
have the list
of strings, set of strings, a set map, or a list. In
our case, it is created as a map.
The resource address with the file name
/root/userll is now represented as what So we
have a few suggestions here, local file.name,
and brackets [root/
userll. This would not be the correct naming
convention. We also have local file.name, and
brackets in square brackets zero, and
also a similar in square brackets one, which are
not the correct convention. We have the local file
name, and in square brackets /root/user 11 ,
which is

also not correct, because we're using local
sensitive value. And also the value needs to be
put in double quotes.
So the only only example that makes sense is
the local_sensitive_file.name in square brackets
under quotes slash root slash user 11.
And that is the correct answer. That's it for this
lab. Thank you for watching, and I'll see you in
the next one.
```

#### Count

```
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  count         = length(var.webservers)
  tags = {
    Name = var.webservers[count.index]
  }
}
```

```
variable "ami" {
  default = "ami-06178cf087598769c"
}

variable "instance_type" {
  default = "m5.large"
}

variable "webservers" {
  type    = list
  default = ["web1", "web2", "web3"]
}
```




#### for-each

```
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  for_each      = var.webservers
  tags = {
    Name = each.value
  }
}
```

```
variable "ami" {
  default = "ami-06178cf087598769c"
}

variable "instance_type" {
  default = "m5.large"
}

variable "webservers" {
  type    = set
  default = ["web1", "web2", "web3"]
}
```



```
resource "local_sensitive_file" "name" {
    filename = var.users[count.index]
    content = var.content
    count = length(var.users)
}
```

```
variable "users" {
    type = list
}

variable "content" {
    default = "password: S3cr3tP@ssw0rd"
}
```


#### Version Constraints

```
In this lecture, we will go over provider versions
and version constraints in OpenTofu. We now
know that the Providers use a plugin-based
infrastructure and that they are available in the
public registry.
Without additional configuration, the OpenTofu
init command downloads the latest version of the
provider plugins that are needed by the
configuration files.
However, this is not something that we may
desire every time. The functionality of a provider
plugin may vary drastically from one version to
another.
Our OpenTofu configuration may not work as
expected when using a version different from the
one it was written. Instead, we can make use of a
specific version of a provider.
The instructions on how to do this are available in
the provider documentation in the registry. So, for
example, if we look up the local provider within
the registry, the default version will be the latest
version, which is in this case 2.4.1.
Instead, let's say that we want to make use of a
specific older version, 2.3.0. We are now
introduced to a new block here, called the
Terraform block.

We saw earlier that the Terraform block is used
to configure settings related to Terraform, or in
our case, OpenTofu.
To make use of a specific version of the provider,
we need to make use of another block called the
required providers inside the Terraform block.
And inside the required providers block, we can
have multiple arguments for every provider that
we want to use. In our example here, we have
one argument with the key called local or the
local provider.
The value of this argument is an object with the
source address of the provider and the exact
version we want to install, which is 2.3.0 in this
case.
Now, with the Terraform block configured to use
the version 2.3.0 of the local provider, when we
run an OpenTofu init command, the version 2.3.0
of the local provider will be downloaded.
We can also make use of version constraints to
control the version of the provider plugin to be
downloaded. Using the not equal to symbol
instead like this, we'll make sure that a specific
version is not downloaded.
In this case, we have specifically asked
OpenTofu not to use the version 2.4.1, so it
downloads the previous version available, which
is 2.4.O.

And then we can also make use of comparison
operators to make sure only versions lesser than
or greater than a given version are used.
In this example, the version that we require to be
used has to be lesser than 2.4.1, resulting in
using the version 2.4.0.
In the second example, the version that we
require to be used has to be higher than 2.3.0,
resulting in the version 2.4.0 again.
We can also combine the comparison operators
like this to make use of a specific version within a
range. In this example, we want to make use of
any version that's greater than 2.2.3, but less
than 2.4.1 and not the version 2.4.0.
As a result, OpenTofu downloads the version
2.3.0, which is acceptable. And finally, we have
the pessimistic constraint operator that uses the
tilde greater than symbol like this.
This operator allows OpenTofu to download the
specific version of any available incremental
version of the provider based on the value we
provide it.
So, for example, here we have given the value of
2.2 following the tilde greater than symbol. This
means OpenTofu can either download 2.2 or
incremental versions such as 2.2.1, 2.2.3, 2.3, all
the way up to 2.4.1.

The maximum version that we can use in this
case is 2.4.1 , and this version is downloaded
when we run OpenTofu init.
Here's another example, but this time let's use
the tilde greater than 2.2.0. Now OpenTofu can
download the version 2.2.0 or the version 2.2.1 ,
2.2.3, but not the 2.3.O or 2.4.O.
We can only have a maximum version of 2.2.3 in
the registry that falls into this constraint, and
that's the version that is going to be downloaded.



lab

Hello and welcome to the demo about OpenTofu
versions. The first question that we have on our
list is to navigate to the directory
slash root slash OpenTofu project slash omega,
where we have added a configuration file.
Inspect the file and choose the correct version of
the provider from the below options.
So if we open up the main.tf, we can see that the
provider is local and a version is hardcoded
to 1.2.2. So that is our correct answer. Next
question on the list. Change to the directory
called /root/OpenTofu project/rotate.
We have already initialized the configuration
directory using the OpenTofu init command.
Inspect the rotation.tf file and find out the correct
version of the provider plugin that is
downloaded. Choose the correct version from the
below options and you don't have to create the
resources. So in here we have a couple of
version constraints. So let's break it down to
understand
it better. So the version needs to be higher than
3.45.0. It cannot be the version 3.46.0 and it has
to be less than 3.48.0. So what we could do here
is we could either open up the

provider documentation on the registry and check
out available versions. But since the repository
was already initialized, all that we have to do is
open up the .terraform folder and
look for the HashiCorp Google provider and
check out the version that has been downloaded,
which is 3.47.0, which is the correct answer.
So our next question is which one of the below is
not a valid version constraint operator? So let's
take a look at them. We have the exclamation
point equals or not equals,
which is definitely a valid operator. We've seen it
here. We have greater than or equals and less
than or equals and less than and equals two,
which are both valid operators. And we have the
pessimistic operator of tilde greater than, which
we have used briefly in the previous lectures.
The only one that is not valid is the equals equals
operator.
0K, the next question says we have been
working on a project called Nautilus under the
configuration directory / root / OpenTofu project /
Nautilus.
And due to a version mismatch, we do not want
to download the AWS provider version 3.17.0,
which version constraint can be used to achieve
this? You can try to add the options below in the

Nautilus.pf to verify the correct syntax. So we can
play around with it. But the correct version
constraint, if we do not want to use a specific
version, is the not equals operator or
exclamation point equals sign and then the
version. So this is the correct version constraint.
Sign and then the version. So this would be our
correct answer.
Now navigate to the directory
/root/OpenTofu_project/Lexicorp, where we have
added the configuration files. Inspect the file and
find out which version
of providers will be downloaded. If unsure, refer
to the documentation, which is available in the
top right corner. So we have a couple of version
constraints here for
Kubernetes. So greater than 1.12.0, not equal to
1.13.1 and lesser than 1.13.3. So judging by this,
the version that is going to be selected is the
1.13.2. Because we cannot use point one,
and it has to be lesser than dot three, logically
leaving out the 1.13.2. Now the Helm provider
uses a pessimistic operator. And we're going to
have to consult the documentation here,

we're using tilde greater than 1.2.0, meaning that
the highest incremental version of 1.2 will be
used. And if we check the Helm documentation,
and we go into versions,
and view all versions, 1.0 is here and the highest
is 1.2.4. So going back into our question, the
correct answer is Helm 1.2.4 and Kubernetes
1.13.2.
That's it for this lab. Thank you for watching and
I'll see you in the next one.
```

```
terraform {
  required_providers {
    k8s = {
      source  = "hashicorp/kubernetes"
      version = "> 1.12.0, != 1.13.1, < 1.13.3 "
    }
  

    helm = {
      source  = "hashicorp/helm"
      version = "~> 1.2.0"
    }
  }
}
```



## Remote State and State Locking?

```
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "We love pets!"
}
```

```
terraform {
  backend "s3" {
    bucket         = "kodekloud-opentofu-state-bucket01"
    key            = "finance/terraform.tfstate"
    region         = "us-west-1"
    dynamodb_table = "state-locking"
  }
}
```


```

$ tofu state show aws_s3_bucket.finance
# tofu state <subcommand> [options] [args]
			Sub-command
			list
			mv
			pull
			rm
			show
			push
```

```
remote state in OpenTofu. So in the previous
lecture, we went over what state files are in
OpenTofu.
And to be specific, we covered local state files,
where the file was created and stored in the
same directory
containing the configuration files, most likely in a
laptop or a development server of the user. We
covered the following points about state.
The state files map OpenTofu configurations to
the real-world infrastructure. They also track
metadata, such as dependencies, which allow
OpenTofu to create and delete resources in the
correct order.
In some cases, disabling state refreshes may
help in improving the performance of OpenTofu
operations. While working with large
configuration files, especially those which make
use of a number of different cloud Providers.
One additional use of state is that it allows
members of a team to collaborate and provision
resources as a team.
However, in examples we have seen so far, the
OpenTofu state file is created locally on the client
machine. And this does not provide an efficient or
secure way to work as a team.

Unlike the configuration files, it is not a good idea
to store the state file in a version control system
as it would most probably contain sensitive
information pertaining to the infrastructure.
Besides the obvious security implications, there
is another reason not to use the local state files,
especially as a team.
For example, let's assume that we are working
with a local state file and we try to run the
OpenTofu apply command simultaneously from
two terminals.
While the first operation is in progress, OpenTofu
locks the state file and as a consequence, we will
not be able to run any operation from the second
terminal, until the first operation finishes.
This is because OpenTofu implements a very
important feature called state locking. And it
ensures that the state file does not get corrupted
by multiple operations trying to update it at the
same time.
Version control systems such as GitHub do not
support locking and as a result, if it is used to
store the state file, multiple users can use the
same state file simultaneously,

which can result in issues like conflicts and data
loss within the state file. For example, let's
consider that both the configuration file and the
state file were checked into GitHub
to be used between two users in a collaboration.
If both userl and user2 pull the files from this
repository and then update the same AWS S3
bucket resource
and then commit the changes back to the remote
repository, it can lead to unintended
consequences such as the corruption of the state
file.
And because of all these reasons, it's a much
better option to store the state file in a secured
shared storage by making use of the remote
backends.
With this option, the state file no longer resides in
the configuration directory or version control
systems and is moved to a shared storage
solution such as AWS S3,
Azure Portal, Google Cloud Storage or any other
supported remote backend. When a remote
backend is configured, OpenTofu will
automatically load the state file from the shared
storage every time it is required by an OpenTofu
operation.

It will also upload the updated OpenTofu state
after every apply. Most importantly, many of
these remote backend options such as the AWS
S3 allow for state locking.
This ensures that the integrity of the state file is
maintained. Finally, the remote backends provide
different ways to secure the storage,
such as encryption at rest and in transit, to make
sure that the sensitive information stored inside is
secure. Let's take a quick look at how we can
configure a remote state using the AWS S3 as
backend.
For this, we need two things, an S3 bucket,
which will be used to store the OpenTofu state,
and optionally, if we want to enable state locking,
a DynamoDB table.
Creating these resources is outside of the scope
of this course, but check out the Terraform for
Beginners course where we go over these in
detail, including a Hands-on lab.
We need the bucket name, key, region, and a
DynamoDB table to configure S3 as the remote
backend. Now, let's configure a remote state for
our configuration for which the state has been
created locally.

To do this, we need to make use of the Terraform
block. Remember, we used the Terraform block
earlier when working with the provider version
constraints.
Next, we add another block called backend within
the Terraform block. Here, we provide the values
we recorded as part of the Course PreRequisites
step.
The backend name specifies the type of backend
we want to use. For making use of the AWS S3
as the backend, we will use the name S3.
The backend block with S3 expects three
arguments. First is the name of the existing
bucket. Next is the key, which is an S3 object
path where the state file should be stored.
In this case, we want to store the OpenTofu state
file within a folder called finance, within the
bucket called
KodeKloudOpenTofuStateBucket01.
The final argument that is required is the region
in which the bucket has been created. In this
example, it's the US West 1 region.
As mentioned earlier, DynamoDB table is
optional, but if used, will enable state locking on
this backend. We have everything in place to
start using the remote backend.

However, if we run the OpenTofu apply command
now, we will see an error that says backend
reinitialization required. This is because before
we start using the new remote state backend,
we need to run the OpenTofu init command that
will reinitialize the new backend to be used.
Since we already have a local state file in the
configuration directory,
the init process gives us an option to copy the
terraform.tfstate file to the remote backend.
Giving a value of yes will copy the state file to the
S3 bucket.
Once done, it is safe to delete the local state file
from the Configuration Directory. Subsequent
OpenTofu plan or apply commands will now lock
the state file
and pull it down from the AWS S3 bucket into the
memory. Any subsequent changes to the state
will be uploaded to the backend instantaneously
and once the operation is complete, the lock will
be released. The state file will not be stored in
the local configuration directory anymore.
Let's now look at commands specifically related
to OpenTofu State. OpenTofu provides a set of
commands that allow us to list, pull, and
manipulate state

and these should be used instead of manually
editing the file. The syntax of these commands
are OpenTofu state followed by a subcommand
such as list, move, pull, rm, show, etc.
The first subcommand we will take a look at is
the list command. This will list all the resources
recorded within the OpenTofu state file.
As you can see from the output, it will only print
the resource address, which is the resource type
dot resource name.
We can also pass an additional argument to this
command for matching resource address. For
example, we can run this command to list a
matching AWS S3 bucket type resource called
Cerberus Finance.


lab


Remote State. In this lab we will work with
remote state files.
We will first start with the local state and then
migrate it to remote state with an S3 backend.
The configuration directory we will work with is
the /root/OpenTofu/projects/remote_state.
So you can open it up on the VS Code file tree
on the right. Okay, so let's see what our first task
is.
Create a simple configuration to create a local file
resource within the directory called Remote
State. The resource block should be created
inside the main.tf file.
Follow the below specifications for provisioning
this resource. Alright, so let's create the main.tf
file and we need to create a resource of local file
with the name of state.
The file name is going to be
/root/variable/local.state. So we need to use a
variable called local.state from the variables.tf.
So that's going to be ${var.local_state}. And then
let's copy this over because we're going to need
it. And now let's do content. Content equals and
let's go ahead and paste this here.

And then copy this whole section and put in state
here. So this configuration uses the variable local
state. So use the variable called local state in the
variables.tf file which is already created for you
and make use of the variable interpolation syntax
of dollar curly braces.
So the remote state has the default value of
remote and the local state has the default value
of local.
So let's go in and check our configuration one
more time. And this seems to be appropriate.
Now we need to run the init command in the
directory, followed by the plan for good measure.
Just know that the resources will be created. And
as we can see here, the variable is working and
it's working here as well.
Then we can run OpenTofu apply and confirm
yes in the prompt. Now if we check our
configuration, everything seems to be in order.
Has a state file been created after you have run
OpenTofu apply? As we can see here, the
terraform.tfstate has been created.
What is the name of the state file created for this
configuration? So the name is terraform.tfstate.
So this one is correct.

Be careful about this one. This one is missing the
appropriate extension. So terraform.tf state. Let's
now move the state to a remote AWS S3
backend.
For this, we will make use of an S3 compatible
storage called MinlO. MinlO provides an S3
compatible API and allows us to configure the S3
backend in the same way as the actual S3
service in AWS Cloud.
To explore MinlO and the S3 bucket that has
been created, click on the MinlO browser tab on
the top of the terminal window.
That one right here. This will show us the
browser. Use the following credentials to log in.
foo foo and bar bar bar.
Let's go ahead and try foo foo and bar bar bar.
And we have a bucket here called Remote State.
Okay, let's move on. We have already created an
S3 bucket that we will use to store the remote
state.
From the MinlO browser, identify the name of this
bucket. There is only one bucket created as of
now. So we identified it already.

It is called remote state. So remote state. Before
we add configuration for the S3 bucket, let's first
change the local file resource.
Change the variable used to remote state instead
of local state. Once done, OpenTofu plan and
apply. Okay, so remote and remote.
We save that up. OpenTofu plan just to see the
execution plan. It will replace with remote here
and remote state here.
So OpenTofu apply. Confirm with yes. And then
check our resources. That seems to be in order.
Great. Now let's configure the remote backend
with S3.
Add a block in the new file called terraform.tf with
the following arguments. Bucket equals remote
state. Key equals terraform.tfstate.
And region equals US East 1. Do not run
OpenTofu init yet. Since we are making use of
MinlO, we also have to add a couple of additional
arguments for this to work.
We will do that in the next step. When using the
regular S3 service from AWS, the above
arguments should be sufficient to configure the
remote state.

All right. So let's see. We have to create our
backend configuration. And let's do that in a file
called terraform.tf.
Inside of that file, we will open up a Terraform
block. And we're going to call backend for AWS
Now we have a couple of arguments. The bucket
is going to be Remote State. Then the key is
going to be terraform.tfstate.
And the region is going to be US East 1. And we
save that up. And then we do a check.
So we move on to the next section. Now this is
the part where MinlO needs a couple of more
arguments that were added automatically by the
lab.
To make sure the S3 backend with MinlO works,
we will have to add a few additional arguments.
The terraform.tf file has been updated.
So we've checked it out. Please note that these
arguments are optional and not needed when
working with the regular S3 service in AWS.
Okay. So try running OpenTofu apply. Are you
able to do it? If not, why? So let's do an
OpenTofu apply.

And it says backend initialization required.
Please run OpenTofu init. So what we need to do
is we need to reinitialize our backend.
Run OpenTofu init in our configuration directory
now. Once done, you can proceed to delete the
terraform.tfstate file from the local directory.
From the MinlO browser, you should now be able
to see the state file uploaded to the bucket called
remote state.
The logger and credentials are the same as we
used before. Okay. So here, when we run
OpenTofu init, we have the ability to copy the
existing state to the new backend.
We can actually do that since we already applied
a couple of resources. Now that we initialized the
backend and copied over the existing state file to
the new backend,
we can go into the MinlO browser and the remote
state bucket, and we can check that there is one
new object in the remote state bucket,
meaning that our remote backend configuration
for OpenTofu is working correctly. So let's go
ahead and check. And we can see that it is all
working as it should.
That's it for this lab. Thank you for watching.


State Commands

Let's now look at commands specifically related
to the OpenTofu state. OpenTofu provides a set
of commands that allow us to list, pull, and
manipulate state,
and these should be used instead of manually
editing the file. The syntax of this command is
OpenTofu state, followed by a subcommand such
as list, mv, pull, rm, show, etc.
The first subcommand that we will take a look at
is the list command. This will list all the resources
recorded within the OpenTofu state file.
As you can see from the output, it will only print
the resource address, which is the resource type
dot resource name.
We can also pass in an additional argument, so
the first subcommand we will take a look at is the
list command.
This will list all the resources recorded within the
OpenTofu state file. As you can see from the
output, it will only print the resource address,
which is the resource type dot resource name.
We can also pass in an additional argument to
this command for a matching resource address.
For example, we can run this command to list a
matching AWS S3 bucket type resource called
Cerberus Finance.

Let's revisit the OpenTofu state show command.
This command will show the attributes of a single
resource in the state file that matches the given
address.
For example, to print all the attributes for an S3
bucket with the resource name Cerberus
Finance, use the OpenTofu state show command
like this.
Next, let's take a look at the OpenTofu state
move command. This command is used to move
the items in an OpenTofu state file or to move
items from one state file to another in a different
configuration directory.
The syntax of this command has a source and a
destination following the move subcommand.
Moving the resources within the same state file is
in fact moving it from its current resource address
to another, which basically implies renaming a
resource.
Here, we have the configuration file and the state
file output for a DynamoDB table resource. To
rename the DynamoDB table resource name
from state locking to state locking DB, use the
OpenTofu state move command like this.
Specify the source name as state locking and
target as state locking DB. Once this is done, the
name of the resource in the state file changes to
a new name.

Next, manually rename the resource name in the
configuration file. A subsequent OpenTofu apply
command should show no changes to be applied
for the resource as the new resource name is
updated on both the state file as well as the
configuration file.
In the previous lecture, we saw that when using
the remote state, the local state file is no longer
required to be stored in the Configuration
Directory.
To download the state file locally, we can make
use of the OpenTofu State pull command. The
output of this command can also be passed to a
JSON query tool like jq to filter the required data.
The OpenTofu state rm command is used to
delete items from the OpenTofu state file. This
command is used when you no longer wish to
manage one or more resources via the current
OpenTofu configuration and state.
The syntax of this command is OpenTofu state
rm followed by the resource address. Once it is
removed from the state file, make sure to remove
the associated resource block from the
configuration as well.
Please note that the resources removed from a
state file are not actually destroyed. It is only
removed from the OpenTofu state file
management.

The OpenTofu state push command can be used
to override the remote state with a local state file.
This command should be run with extreme
caution as this can break your configuration if not
used properly.
OpenTofu does try to prevent accidental pushes
to the remote state. For example, if the local
state that you are trying to push is completely
different from the remote state, then OpenTofu
will throw an error like this.
You can use the —force option if you want to
override this behavior, but it is not recommended
and should be used with caution.

lab

Hello, and welcome to the lab of the OpenTofu
state commands. The first question on the list is
we have created a few resources in the
configuration
directory at /root/OpenTofu/project/project_anime.
So let's open up that folder and inspect it. Which
of the following resource names are not part of
the OpenTofu State?
So what we need to do here is run the OpenTofu
state list in the appropriate directory. Always be
careful when running commands like this that
you're in the right directory.
And now we can see that there is a local file
classics, local file hall of fame, local file new
shows and top 10.
So in here, we see top 10, we see a hall of fame,
we see new shows, and we see classics.
And here on the list of questions, we do not see
Super Pets. So let's select the super pets as our
answer.
Which command would you use to show the
attributes of the resource called classics stored in
the OpenTofu State? So the command that you
should use is OpenTofu state show, followed by
the address of the

resource. So in our case, it's classic, local file
that classic. And now we can see the state file for
the local file classic.
So that would be the OpenTofu state show local
file resource. What is the value of the attribute
called ID that is created for the local file resource
called top ten? So the same way we can use the
OpenTofu state show local file top 10. And we will
see that the ID starts with a 961, a 961 and ends
with 21f, which would
be this one. We no longer wish to manage the file
located at /root/anime/hall of fame .txt by
OpenTofu. Remove the resource responsible for
this file completely from the management of
OpenTofu.
Okay, so what we need to do here is we need to
remove the resource block called hall of fame
from the main.tf file, and also remove it from the
state by using the OpenTofu state
RM. So let's do it one by one. First, the hall of
fame needs to be deleted. And then we need to
run the OpenTofu state rm local file dot hall of
fame.
And it notified us that it successfully removed one
resource of the instance. And let's now check.
And yep, that seems to be all right.

So let's go to our next question. Now navigate to
the directory at /root/OpenTofu project/super
pets. Okay, we're here and we're going to change
the directory in our terminal as well.
So we're ready. Just like the previous
Configuration Directory, we have already created
the resource, inspect the configuration and
identify the only Resource type used.
Okay, so let's inspect the configuration. We have
the random pet, we have the S3 backend and
variables. So the only resource type used is the
random pet.
Within this configuration, the OpenTofu State
commands are working. Try it. But there is no
terraform.tfstate file present. What is the reason
for this behavior?
Okay, so first of all, let's go ahead and try some
OpenTofu state commands. So we can do
OpenTofu state show, followed by random pet as
the resource type, and then
one of the resource names for good measure.
And we can see that indeed, the OpenTofu state
commands do work.
Now, the reason why this works is because there
is a Terraform block in the tofu.tf containing the
backend S3.

So the answer to our question is we are using a
remote state. Next question is, what is the ID of
the random pet resource called superpet2 in the
state
file? So now we can see that the ID is wonder-
super-butt. So that is the answer to our question.
And lastly, rename the resource from superpetl
to ultrapet.
Change the name in the main.tf file as well as the
state. Okay, so we need to go ahead and open
up the main.tf and change the superpetl to
ultrapet.
And now to change the name in the state, we
need to use the state move command. So
OpenTofu state move random_pet.superpetl
needs to be random_pet.ultrapet.
And once we do that, we have successfully
renamed the resource. And that's it. That would
be all for this lab.
Thank you for watching, and I'll see you in the
next one.

lab

Hello, and welcome to the lab of the OpenTofu
state commands. The first question on the list is
we have created a few resources in the
configuration
directory at /root/OpenTofu/project/project_anime.
So let's open up that folder and inspect it. Which
of the following resource names are not part of
the OpenTofu State?
So what we need to do here is run the OpenTofu
state list in the appropriate directory. Always be
careful when running commands like this that
you're in the right directory.
And now we can see that there is a local file
classics, local file hall of fame, local file new
shows and top 10.
So in here, we see top 10, we see a hall of fame,
we see new shows, and we see classics.
And here on the list of questions, we do not see
Super Pets. So let's select the super pets as our
answer.
Which command would you use to show the
attributes of the resource called classics stored in
the OpenTofu State? So the command that you
should use is OpenTofu state show, followed by
the address of the

resource. So in our case, it's classic, local file
that classic. And now we can see the state file for
the local file classic.
So that would be the OpenTofu state show local
file resource. What is the value of the attribute
called ID that is created for the local file resource
called top ten? So the same way we can use the
OpenTofu state show local file top 10. And we will
see that the ID starts with a 961, a 961 and ends
with 2 If, which would
be this one. We no longer wish to manage the file
located at /root/anime/hall of fame .txt by
OpenTofu. Remove the resource responsible for
this file completely from the management of
OpenTofu.
Okay, so what we need to do here is we need to
remove the resource block called hall of fame
from the main.tf file, and also remove it from the
state by using the OpenTofu state
RM. So let's do it one by one. First, the hall of
fame needs to be deleted. And then we need to
run the OpenTofu state rm local file dot hall of
fame.
And it notified us that it successfully removed one
resource of the instance. And let's now check.
And yep, that seems to be all right.

So let's go to our next question. Now navigate to
the directory at /root/OpenTofu project/super
pets. Okay, we're here and we're going to change
the directory in our terminal as well.
So we're ready. Just like the previous
Configuration Directory, we have already created
the resource, inspect the configuration and
identify the only Resource type used.
Okay, so let's inspect the configuration. We have
the random pet, we have the S3 backend and
variables. So the only resource type used is the
random pet.
Within this configuration, the OpenTofu State
commands are working. Try it. But there is no
terraform.tfstate file present. What is the reason
for this behavior?
Okay, so first of all, let's go ahead and try some
OpenTofu state commands. So we can do
OpenTofu state show, followed by random pet as
the resource type, and then
one of the resource names for good measure.
And we can see that indeed, the OpenTofu state
commands do work.
Now, the reason why this works is because there
is a Terraform block in the tofu.tf containing the
backend S3.

So the answer to our question is we are using a
remote state. Next question is, what is the ID of
the random pet resource called superpet2 in the
state
file? So now we can see that the ID is wonder-
super-butt. So that is the answer to our question.
And lastly, rename the resource from superpetl
to ultrapet.
Change the name in the main.tf file as well as the I
state. Okay, so we need to go ahead and open
up the main.tf and change the superpetl to
ultrapet.
And now to change the name in the state, we
need to use the state move command. So
OpenTofu state move random_pet.superpetl
needs to be random_pet.ultrapet.
And once we do that, we have successfully
renamed the resource. And that's it. That would
be all for this lab.
Thank you for watching, and I'll see you in the
next one.
```


main.tf
```
resource "local_file" "state" {
  filename = "/root/${var.remote-state}"
  content = "This configuration uses ${var.remote-state} state"
}
```

variables.tf
```
variable remote-state {
    type = string
    default = "remote"
}

variable local-state {
    type = string
    default = "local"
}
```

terraform.tf
```
terraform {
  backend "s3" {
    key = "terraform.tfstate"
    region = "us-east-1"
    bucket = "remote-state"
    endpoint = "http://172.16.238.105:9000"
    force_path_style = true
    skip_credentials_validation = true
    skip_metadata_api_check = true
    skip_region_validation = true
  }
}
```

### State Commands
```
tofu state show aws_s3_bucket.cerberus-finance


tofu state mv random_pet.super_pet_1 random_pet.ultra_pet
```


```
resource "random_pet" "suultra_pet" {
    length = var.length1
    prefix = var.prefix1
}

resource "random_pet" "super_pet_2" {
    length = var.length2
    prefix = var.prefix2
}
```

```
variable "length1" {
    default = "1"
}

variable "length2" {
    default = "2"
}

variable "prefix1" {
    default = "Super"
}

variable "prefix2" {
    default = "Wonder"
}
```

```
terraform {
  backend "s3" {
    key = "terraform.tfstate"
    region = "us-east-1"
    bucket = "remote-state"
    endpoint = "http://172.16.238.105:9000"
    force_path_style = true

    skip_credentials_validation = true

    skip_metadata_api_check = true
    skip_region_validation = true
  }
}
```

## OpenTofu Provisioners


```
However, keep a note that just merely specifying
the provisioner block will not guarantee that the
commands will run. For it to work, there should
be network connectivity between the local
machine and the remote instance.
For Linux instances, this means an SSH
connection and, in case of Windows, WinRM
connectivity. This can be achieved by making use
of an appropriate security group while creating
the instance.
In this case, we are creating the security group
resource, but it could also be an existing security
group that is already created in the VPC where
we are provisioning the instance.
There should also be an authentication
mechanism that can be used to connect to these
instances. Most commonly, an SSH key pair can
be used to access the instance.
In this example, we are using an SSH key pair in
the resource definition. This key is created by the
AWS key pair resource in this example, but could
also be an existing key pair within your account.
To facilitate authentication, we can make use of
the connection block like this. This is an example
of a Linux-based AMI, as the connection type is
set to SSH.

The host argument is where we specify the public
IP of the instance that was created. Since we are
operating directly inside the resource block, we
can make use of the expression self.public ip
that will return the provisioned instance.
This will effectively translate into the public IP
address of the instance that we have
provisioned. The user in this case is Ubuntu, and
that's the default user created within this AMI.
The key we use to connect to the instance is the
private key. Remember that earlier we used the
public key pair as an argument for this resource.
Now, when we run OpenTofu apply, we should
see the remote exec provisioner connecting to
the instance using the details in the connection
block,
and completing the task specified in the inline list.
Next, let's look at the local-exec provisioner.
Local-exec provisioner is used to run tasks on
the local machine where we are running the
OpenTofu binary, and not on the resources
created by OpenTofu.
This is especially handy when we want to gather
some data and write to a file locally. For example,
if we want to store the public IP address of an
EC2 instance that we just provisioned to a file
called ip.txt in the tmp filesystem,

we can run the local-exec provisioner like this.
Just like we saw with the remote exec
provisioner, the provisioner block for local exec
also goes inside the resource block.
Once the resource is created after OpenTofu
apply, we can see the public IP recorded file. By
default, provisioners are run after resources are
created.
This is the default behavior and is called a
create-time provisioner. We can also make the
provisioner run before a resource is destroyed.
This is called a destroy-time provisioner. This can
be specifically set by using the when argument
and specifying its value as destroy within the
provisioner block.
Another default behavior of the provisioner is that
if the command or script specified within the
provisioner fails, the OpenTofu apply operations
error out.
In this example, an incorrect path used in the
command causes the script to fail and results in
an error.
This is the default provisioner behavior but can
also be set by using on_failure arguments inside
the provisioner block, like this.

Any resource created while the provisioner fails
is marked as tainted. For OpenTofu apply
operations to not fail and the resources to be
created successfully,
even if the provisioner command or script fails,
we can set the value of the on_failure argument
to continue. As best practice, OpenTofu
recommends to use provisioners as a last resort.
Whenever possible, make use of options natively
to the resource types for the provider used. For
example, use the user data while creating an
EC2 instance in AWS, or custom data for Azure
virtual machines, or metadata for GCP, etc.


Let's now recap provisioners in OpenTofu.
Provisioners provide a way for us to carry out
tasks such as running commands or scripts on
remote resources or locally on the machine
where OpenTofu is installed.
For example, to run a bash script after a resource
is created, we can make use of the RemoteExec
provisioner.
Using the same web server example we have
used so far, we can pass in an inline script like
this.
As you can see from this example, the
provisioner block goes directly inside a resource
block. These commands in the inline list will now
be run on the remote instance after it is
deployed.
In this example, we're running an app update,
installing, enabling and starting Nginx on an EC2
instance once it's deployed.
However, keep a note that just merely specifying
the provisioner block will not guarantee that the
commands will run. For it to work, there should
be network connectivity between the local
machine and the remote instance.

For Linux instances, this means an SSH
connection and, in case of Windows, WinRM
connectivity. This can be achieved by making use
of an appropriate security group while creating
the instance.
In this case, we are creating the security group
resource, but it could also be an existing security
group that is already created in the VPC where
we are provisioning the instance.
There should also be an authentication
mechanism that can be used to connect to these
instances. Most commonly, an SSH key pair can
be used to access the instance.
In this example, we are using an SSH key pair in
the resource definition. This key is created by the
AWS key pair resource in this example, but could
also be an existing key pair within your account.
To facilitate authentication, we can make use of
the connection block like this. This is an example
of a Linux-based AMI, as the connection type is
set to SSH.
The host argument is where we specify the public
IP of the instance that was created. Since we are
operating directly inside the resource block, we
can make use of the expression self.public_ip
that will return the provisioned instance.

This will effectively translate into the public IP
address of the instance that we have
provisioned. The user in this case is Ubuntu, and
that's the default user created within this AMI.
The key we use to connect to the instance is the
private key. Remember that earlier we used the
public key pair as an argument for this resource.
Now, when we run OpenTofu apply, we should
see the remote exec provisioner connecting to
the instance using the details in the connection
block,
and completing the task specified in the inline list.
Next, let's look at the local-exec provisioner.
Local-exec provisioner is used to run tasks on
the local machine where we are running the
OpenTofu binary, and not on the resources
created by OpenTofu.
This is especially handy when we want to gather
some data and write to a file locally. For example,
if we want to store the public IP address of an
EC2 instance that we just provisioned to a file
called ip.txt in the tmp filesystem,
we can run the local-exec provisioner like this.
Just like we saw with the remote exec
provisioner, the provisioner block for local exec
also goes inside the resource block.

Once the resource is created after OpenTofu
apply, we can see the public IP recorded file. By
default, provisioners are run after resources are
created.
This is the default behavior and is called a
create-time provisioner. We can also make the
provisioner run before a resource is destroyed.
This is called a destroy-time provisioner. This can
be specifically set by using the when argument
and specifying its value as destroy within the
provisioner block.
Another default behavior of the provisioner is that
if the command or script specified within the
provisioner fails, the OpenTofu apply operations
error out.
In this example, an incorrect path used in the
command causes the script to fail and results in
an error.
This is the default provisioner behavior but can
also be set by using on_failure arguments inside
the provisioner block, like this.
Any resource created while the provisioner fails
is marked as tainted. For OpenTofu apply
operations to not fail and the resources to be
created successfully,

Any resource created while the provisioner fails
is marked as tainted. For OpenTofu apply
operations to not fail and the resources to be
created successfully,
even if the provisioner command or script fails,
we can set the value of the on_failure argument
to continue. As best practice, OpenTofu
recommends to use provisioners as a last resort.
Whenever possible, make use of options natively
to the resource types for the provider used. For
example, use the user data while creating an
EC2 instance in AWS, or custom data for Azure
virtual machines, or metadata for GCP, etc.


lab

Hello and welcome to the demo of OpenTofu
provisioners. Our first question says navigate to
the directory /root/OpenTofu/projects [Project
Cerberus. We have an empty main.tf file in this
directory.
And we seem to have that. Using the
configuration file, write a resource block to
provision a simple EC2 instance with the
following specifications.
So the resource name needs to be Cerberus, the
AMI needs to be this, and use the variable
named AMI. Region needs to be us-east-2, use
the variable named region.
Instance type is M5 large, but again, use the
variable named instance type. And once ready,
OpenTofu init plan apply to provision this EC2
instance.
Okay, so let's do an AWS instance resource with
the name of Cerberus. And then our AMI needs
to be var.ami. And our instance type
needs to be var.instance_type. Good. Now we
need to create the variables here. So let's go
ahead and create a variable AMI. And for the
default value,
we're going to use the AMI that was provided to
us. Then we need to create another variable
called region.

And it is going to have a default value of EU-
West-2. Good. And the last variable that we need
is going to be the instance type.
And that instance type is going to have a default
value of M5.large. Save that up in our terminal,
navigate to the directory project Cerberus,
run OpenTofu init to initialize, followed by
OpenTofu apply. Let's check our execution plan
that gets generated. And we can see here that
the resource AWS EC2 Instance Cerberus will be
created
with our AMI with the region we selected and the
instance type is going to be m5.large. So let's
confirm with yes and prompt. Wait for that to
apply.
Good. And let's check if everything is okay. And
everything seems to be in order. Let's move on.
Perfect. The instance has been created by
OpenTofu.
To inspect the details of this instance, you can
run OpenTofu show command from the
configuration directory. This will print out the
resource attributes from the state file in a human-
readable format.

So here we see the output of the OpenTofu show
command. Let's move on. The AMI ID we have
used is a Red Hat Enterprise Linux 8 image in
the London region that only
accepts SSH key-based authentication. However,l
when we created the instance, we did not make
use of a key. Let's create a new key pair.
So go ahead and move on. So a new SSH key
pair has been created in the directory
/root/OpenTofu_project/project_Cerberus.SSH.
And we can see it here. The private key is called
Cerberus
and the public key is called Cerberus.pub. Using
the public key, create a new key pair in AWS with
the following specification.
Okay, so let's go ahead and begin creating our
resource, which is going to be AWS key pair with
the name of Cerberus-key. And be careful of the
autocomplete.
Now we need the key name to be Cerberus. And
we need the file function to read the public key of
Cerberus. pub.
So our public key is going to be an argument to
the file function to the SSH - Cerberus dot pub
key. Let's save that up and run OpenTofu

and run OpenTofu init, followed by OpenTofu
apply to apply our AWS key pair. Confirm yes in
the prompt and check our configuration.
Everything seems to be okay. So let's move on to
the next question. Okay, the next question says,
let us configure the Cerberus resource to make
use of this key.
Update the resource block to make use of the
key called Cerberus. Once the configuration is
updated, run an OpenTofu plan and OpenTofu
apply.
This will trigger the replacement of the instance
with the new one, having the key pair created in
our previous set.
Okay, good. So let's go ahead and edit our
instance. So that it has a key name called
Cerberus. And let's save that up, run an
OpenTofu apply.
And let's verify that the instance indeed does get
recreated. And now the Cerberus key will get
applied to the instance.
Confirm yes in the prompt and wait for the
instance to be recreated. Great. Now let's check
our question to see if our syntax is correct.

And it seems that everything's okay. Let's move
on. Let us now install Nginx with an EC2
instance. To do this, let's make use of the user
data argument.
Using the file function again, or by making use of
the here doc syntax, use the script called Nginx
install.sh as the value for the user data argument.
Do not run OpenTofu apply. Okay, so let's go
ahead and go into our instance resource and
supply the user data argument to a file called
install-nginx.sh, which is located here. Save that
up and let's check if our arguments are correctly
set up. Okay, seems to be good. Let's move on.
What will happen if we run OpenTofu apply now?
Easiest way to check, run OpenTofu apply, see
what happens. Okay, so we can see that the
AWS EC2 instance.serverless will be updated
with the user data script.
But if we do that, will the changes actually apply?
So the answer is that it will not be applied
because the user data only runs once at the
first creation time of the instance. So the current
server will be modified, but Nginx will not be
installed. In this case, an AWS EC2 Instance will
be modified, but Nginx will not be installed.

It is due to the fact that user data scripts only run
at first boot, whereas the instance modification
just causes a reboot.
Let's apply the updated configuration in the next
step. Run OpenTofu apply and let the EC2
instance be modified. Now run the OpenTofu
apply, confirm yes and the prompt,
and check our syntax. Seems to be okay. Let's
move on. Where should you add a provisioner
block? So if you watched the previous videos, we
noted that the provisioner block needs to be
nested
inside the resource block. So it would be a
nested block inside the resource block. Which of
the following provisioners does not need a
connection block defined?
Again, if you watch the previous videos, we know
that only the local exec provisioner does not
need a connection block, whereas the
remote exec and file provisioners do. What is the
public IPv4 address that has been allocated to
this EC2 instance?
So the easiest way that we can just check is run
OpenTofu show, and then we need to look for the
public IPv4.

Which is 54.214.169.15. So this is what we
select. We use the public IPv4 address to access
the server. However, when the server is rebooted
or recreated, this IP address would change.
To fix this, let's create an Elastic IP address. An
elastic IP address is a static IPv4 address which
does not change over time.
Create an elastic IP resource with the following
specifications. So let's go ahead and create a
new resource. AWS EIP with the name of EIP.
Be careful of the autocomplete again. And let's
go ahead and set the values as instructed. VPC
needs to be set to true.
Then instance needs to be the ID of the instance
created for the resource servers. And we need to
use a reference expression.
So let's go ahead and type in
aws instance.servers.id, which is the ID of our
instance. And now we need to create a local-exec I
provisioner for the EIP resource and use it to
print the
attribute called public DNS to a file, Okay, so let's
go ahead and create a provisioner. Be careful of
the autocomplete provisioner, which is called
local-exec.

And for the command, we're going to echo AWS
EIP with the name of AWS EIP, which is a static
IP address.
For the command, we're going to echo the AWS
ElP.eip.publicDNS into / root / serverless
publicDNS.txt. And let's go ahead and save that.
And then let's run a OpenTofu apply. Confirm yes
in the prompt. And our serverless public DNS.txt
got created and it indeed does have the public
DNS in its
value. So let's go ahead and check our question
to see if everything is okay. And it seems to be all
right.
Let's move on. What was the public IP address
that was created for this Elastic IP? Okay, so
again, OpenTofu show is our friend here.
Let's get the output and look for the AWS Elastic
IP resource. And the public IP seems to be
127.86.195.235.
Let's select that as our answer and move on to
the last question. In the current configuration,
which dependency is not true?
Okay, so we have a couple. Resource serverless
depends on the serverless key. Let's see. So our
resource serverless does indeed depend on the
key called serverless.

So that is true. Resource called serverless
depends on the resource called EIP. So we do
not see any dependency here.
But if we check the EIP, we can see that the EIP
does indeed depend on the AWS EC2 instance
called serverless to be created. On the AWS
instance called serverless to be created first. So
the last option is a resource called EIP that
depends on the resource called serverless.
So that would be true. And even though we have
a key name added here as serverless, it does not
have an implicit
dependency or explicit dependency. Okay, so our
Elastic IP has a dependency to AWS instance dot
serverless dot ID. So now we know that our EIP
depends on the resource called serverless.
The only dependency that is not true is the
resource called serverless depends on the
resource called EIP, because in the resource
serverless, we do not see any dependency
towards the EIP resource. It's the other way
around. So let's select that as our answer. Great.
That was it for this lab.
Thank you for watching and I'll see you in the
next one.


provisioners. Our first question says navigate to
the directory /root/OpenTofu/projects /Project
Cerberus. We have an empty main.tf file in this
directory.
And we seem to have that. Using the
configuration file, write a resource block to
provision a simple EC2 instance with the
following specifications.
So the resource name needs to be Cerberus, the
AMI needs to be this, and use the variable
named AMI. Region needs to be us-east-2, use
the variable named region.
Instance type is M5 large, but again, use the
variable named instance type. And once ready,
OpenTofu init plan apply to provision this EC2
instance.
Okay, so let's do an AWS instance resource with
the name of Cerberus. And then our AMI needs
to be var.ami. And our instance type
needs to be var.instance_type. Good. Now we
need to create the variables here. So let's go
ahead and create a variable AMI. And for the
default value,
we're going to use the AMI that was provided to
us. Then we need to create another variable
called region.

And it is going to have a default value of EU-
West-2. Good. And the last variable that we need
is going to be the instance type.
And that instance type is going to have a default
value of M5.large. Save that up in our terminal,
navigate to the directory project Cerberus,
run OpenTofu init to initialize, followed by
OpenTofu apply. Let's check our execution plan
that gets generated. And we can see here that
the resource AWS EC2 Instance Cerberus will be
created
with our AMI with the region we selected and the
instance type is going to be m5.large. So let's
confirm with yes and prompt. Wait for that to
apply.
Good. And let's check if everything is okay. And
everything seems to be in order. Let's move on.
Perfect. The instance has been created by
OpenTofu.
To inspect the details of this instance, you can
run OpenTofu show command from the
configuration directory. This will print out the
resource attributes from the state file in a human-
readable format.

So here we see the output of the OpenTofu show
command. Let's move on. The AMI ID we have
used is a Red Hat Enterprise Linux 8 image in
the London region that only
accepts SSH key-based authentication. However,
when we created the instance, we did not make
use of a key. Let's create a new key pair.
So go ahead and move on. So a new SSH key
pair has been created in the directory
/root/OpenTofu_project/project_Cerberus.SSH.
And we can see it here. The private key is called
Cerberus
and the public key is called Cerberus.pub. Using
the public key, create a new key pair in AWS with
the following specification.
Okay, so let's go ahead and begin creating our
resource, which is going to be AWS key pair with
the name of Cerberus-key. And be careful of the
autocomplete.
Now we need the key name to be Cerberus. And
we need the file function to read the public key of
Cerberus.pub.
So our public key is going to be an argument to
the file function to the SSH - Cerberus dot pub
key. Let's save that up and run OpenTofu

and run OpenTofu init, followed by OpenTofu
apply to apply our AWS key pair. Confirm yes in
the prompt and check our configuration.
Everything seems to be okay. So let's move on to
the next question. Okay, the next question says,
let us configure the Cerberus resource to make
use of this key.
Update the resource block to make use of the
key called Cerberus. Once the configuration is
updated, run an OpenTofu plan and OpenTofu
apply.
This will trigger the replacement of the instance
with the new one, having the key pair created in
our previous set.
Okay, good. So let's go ahead and edit our
instance. So that it has a key name called
Cerberus. And let's save that up, run an
OpenTofu apply.
And let's verify that the instance indeed does get
recreated. And now the Cerberus key will get
applied to the instance.
Confirm yes in the prompt and wait for the
instance to be recreated. Great. Now let's check
our question to see if our syntax is correct.

And it seems that everything's okay. Let's move
on. Let us now install Nginx with an EC2
instance. To do this, let's make use of the user
data argument.
Using the file function again, or by making use of
the here doc syntax, use the script called Nginx
install.sh as the value for the user data argument.
Do not run OpenTofu apply. Okay, so let's go
ahead and go into our instance resource and
supply the user data argument to a file called
install-nginx.sh, which is located here. Save that
up and let's check if our arguments are correctly
set up. Okay, seems to be good. Let's move on.
What will happen if we run OpenTofu apply now?
Easiest way to check, run OpenTofu apply, see
what happens. Okay, so we can see that the
AWS EC2 instance.serverless will be updated
with the user data script.
But if we do that, will the changes actually apply?
So the answer is that it will not be applied
because the user data only runs once at the
first creation time of the instance. So the current
server will be modified, but Nginx will not be
installed. In this case, an AWS EC2 Instance will
be modified, but Nginx will not be installed.

It is due to the fact that user data scripts only run
at first boot, whereas the instance modification
just causes a reboot.
Let's apply the updated configuration in the next
step. Run OpenTofu apply and let the EC2
instance be modified. Now run the OpenTofu
apply, confirm yes and the prompt,
and check our syntax. Seems to be okay. Let's
move on. Where should you add a provisioner
block? So if you watched the previous videos, we
noted that the provisioner block needs to be
nested
inside the resource block. So it would be a
nested block inside the resource block. Which of
the following provisioners does not need a
connection block defined?
Again, if you watch the previous videos, we know
that only the local exec provisioner does not
need a connection block, whereas the
remote exec and file provisioners do. What is the
public IPv4 address that has been allocated to
this EC2 instance?
So the easiest way that we can just check is run
OpenTofu show, and then we need to look for the
public IPv4.

Which is 54.214.169.15. So this is what we
select. We use the public IPv4 address to access
the server. However, when the server is rebooted
or recreated, this IP address would change.
To fix this, let's create an Elastic IP address. An
elastic IP address is a static IPv4 address which
does not change over time.
Create an elastic IP resource with the following
specifications. So let's go ahead and create a
new resource. AWS EIP with the name of EIP.
Be careful of the autocomplete again. And let's
go ahead and set the values as instructed. VPC
needs to be set to true.
Then instance needs to be the ID of the instance
created for the resource servers. And we need to
use a reference expression.
So let's go ahead and type in
aws instance.servers.id, which is the ID of our
instance. And now we need to create a local-exe
provisioner for the EIP resource and use it to
print the
attribute called public DNS to a file, Okay, so let's
go ahead and create a provisioner. Be careful of
the autocomplete provisioner, which is called
local-exec.

And for the command, we're going to echo AWS
EIP with the name of AWS EIP, which is a static
IP address.
For the command, we're going to echo the AWS
ElP.eip.publicDNS into / root / serverless
publicDNS.txt. And let's go ahead and save that.
And then let's run a OpenTofu apply. Confirm yes
in the prompt. And our serverless public DNS.txt
got created and it indeed does have the public
DNS in its
value. So let's go ahead and check our question
to see if everything is okay. And it seems to be all
right.
Let's move on. What was the public IP address
that was created for this Elastic IP? Okay, so
again, OpenTofu show is our friend here.
Let's get the output and look for the AWS Elastic
IP resource. And the public IP seems to be
127.86.195.235.
Let's select that as our answer and move on to
the last question. In the current configuration,
which dependency is not true?
Okay, so we have a couple. Resource serverless
depends on the serverless key. Let's see. So our
resource serverless does indeed depend on the
key called serverless.

So that is true. Resource called serverless
depends on the resource called EIP. So we do
not see any dependency here.
But if we check the EIP, we can see that the EIP
does indeed depend on the AWS EC2 instance
called serverless to be created. On the AWS
instance called serverless to be created first. So
the last option is a resource called EIP that
depends on the resource called serverless.
So that would be true. And even though we have
a key name added here as serverless, it does not
have an implicit
dependency or explicit dependency. Okay, so our
Elastic IP has a dependency to AWS instance dot
serverless dot ID. So now we know that our EIP
depends on the resource called serverless.
The only dependency that is not true is the
resource called serverless depends on the
resource called EIP, because in the resource
serverless, we do not see any dependency
towards the EIP resource. It's the other way
around. So let's select that as our answer. Great.
That was it for this lab.
Thank you for watching and I'll see you in the
next one.
```

```
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"

  provisioner "remote-exec" {
    inline = [ "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx",
    ]
  }

  connection {
    type        = "ssh"
    host        = self.public_ip
    user        = "ubuntu"
    private_key = file("/root/.ssh/web")
  }

  key_name = aws_key_pair.web.id

  vpc_security_group_ids = [ aws_security_group.ssh-access.id ]
}

resource "aws_key_pair" "web" {
  << code hidden >>
}

resource "aws_security_group" "ssh-access" {
  << code hidden >>
}
```

```
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.webserver2.public_ip} >> /tmp/ips.txt"
  }
}
```

##### Destroy Time Provisioner
```
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo Instance ${aws_instance.webserver.public_ip} Created! > /tmp/instance_state.txt"
  }

  provisioner "local-exec" {
    when    = destroy
    command = "echo Instance ${aws_instance.webserver.public_ip} Destroyed! > /tmp/instance_state.txt"
  }
}
```

##### Failure Behavior
```
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    on_failure = fail
    command    = "echo Instance ${aws_instance.webserver.public_ip} Created! > /tmp/instance_state.txt"
  }

  provisioner "local-exec" {
    when    = destroy
    command = "echo Instance ${aws_instance.webserver.public_ip} Destroyed! > /tmp/instance_state.txt"
  }
}
```


```
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


| <font color="#4bacc6">Provider</font>       | <font color="#4bacc6">Resource | <font color="#4bacc6">Option</font> |
| ------------------------------------------- | ------------------------------ | ----------------------------------- |
| <font color="#4bacc6">AWS</font>            | aws instance                   | user data                           |
| <font color="#4bacc6">Azure</font>          | azurrerm virtual machine       | custom data                         |
| <font color="#4bacc6">GCP</font>            | google_compute_instance        | meta data                           |
| <font color="#4bacc6">Vmware vSphere</font> | vsphere_virtual_machine        | user data.txt                       |
main.tf
```
resource "aws_instance" "cerberus" {
  ami = var.ami
  instance_type = var.instance_type
  key_name = "cerberus"
  user_data = file("./install-nginx.sh")

}

variable "ami" {
  default = "ami-06178cf087598769c"
}

variable "region" {
  default = "eu-west-2"
}

variable "instance_type" {
  default = "m5.large"
}

resource "aws_key_pair" "cerberus-key" {
  key_name = "cerberus"
  public_key = file(".ssh/cerberus.pub")
}

resource "aws_eip" "eip" {
  vpc = true
  instance = aws_instance.cerberus.id
  provisioner "local-exec" {
    command = "echo ${aws_eip.eip.public_dns} >> /root/cerberus_public_dns.txt "
  }
}
```

provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.38.0"
    }
  }
}

provider "aws" {
  region                      = var.region
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  endpoints {
    ec2 = "http://aws:4566"
  }
}
```

```bash
#!/bin/bash
sudo yum update -y
sudo yum install nginx -y
sudo systemctl start nginx
```

```
tofu show
# aws_instance.cerberus:
resource "aws_instance" "cerberus" {
    ami                                  = "ami-06178cf087598769c"
    arn                                  = "arn:aws:ec2:eu-west-2::instance/i-8c1c139f9264d9759"
    associate_public_ip_address          = true
    availability_zone                    = "eu-west-2a"
    disable_api_stop                     = false
    disable_api_termination              = false
    ebs_optimized                        = false
    get_password_data                    = false
    hibernation                          = false
    id                                   = "i-8c1c139f9264d9759"
    instance_initiated_shutdown_behavior = "stop"
    instance_state                       = "running"
    instance_type                        = "m5.large"
    ipv6_address_count                   = 0
    ipv6_addresses                       = []
    monitoring                           = false
    placement_partition_number           = 0
    primary_network_interface_id         = "eni-f48a1e5f"
    private_dns                          = "ip-10-216-186-149.eu-west-2.compute.internal"
    private_ip                           = "10.216.186.149"
    public_dns                           = "ec2-54-214-162-160.eu-west-2.compute.amazonaws.com"
    public_ip                            = "54.214.162.160"
    secondary_private_ips                = []
    security_groups                      = []
    source_dest_check                    = true
    subnet_id                            = "subnet-b9a5f2e8"
    tags_all                             = {}
    tenancy                              = "default"
    user_data_replace_on_change          = false
    vpc_security_group_ids               = []

    root_block_device {
        delete_on_termination = true
        device_name           = "/dev/sda1"
        encrypted             = false
        iops                  = 0
        tags                  = {}
        throughput            = 0
        volume_id             = "vol-f2126a20"
        volume_size           = 8
        volume_type           = "gp2"
    }
}
```


## Import, Tainting Resources and Deubugging
### Tofu Taint
https://opentofu.org/docs/internals/debugging/

```
In this lecture, we will take a look at the
OpenTofu taint, or now known as the dash-
replace flag. A note here about the taint
command.
It is now deprecated in favor of the OpenTofu
apply-replace command. The new command still
uses the same taint logic, so it is important to
understand how
the taint works, even if the same command is not
used anymore. Okay, let's get started. There
would be cases when a resource creation fails
when we run an OpenTofu apply.
Let's take the same example as before where we
used a local provisioner to store the public IP
address of the EC2 instance in a file.
However, when we tried to apply the changes,
the resource creation failed because the path
specified in the command of the local-exec
provisioner is invalid.
When this happens, and a resource creation fails
for any reason, OpenTofu marks the resource as
tainted. And this can be seen when we run the
OpenTofu plan again.
The resource called webserver is tainted, and as
a result, OpenTofu will attempt to recreate it
during the subsequent apply.

In this example, the step that failed was the
command executed by the local exec provisioner.
The EC2 instance itself was successfully
provisioned.
In spite of this, OpenTofu will attempt to recreate
the entire resource because it was marked as
tainted. There would be cases where we want to
force a particular resource to be recreated.
For example, consider a case where manual
changes were made to an AWS EC2 instance
which is managed by OpenTofu, such as for
example changing the version of Nginx by
manual methods.
To revert this, we can either destroy the resource
using the OpenTofu destroy command and then
run the apply again.
Or a better way would be to taint the resource
using the OpenTofu apply-replace command
followed by the address of the resource we're
replacing.
When we run OpenTofu plan next, we will see
that the resource is set to be recreated. To undo
a taint that was applied to a resource, use the
OpenTofu untaint command also followed
by the address of the resource we are trying to
untaint. The resource will not be recreated now
during a subsequent apply.

and Debugging. Our first question on the list is
which environment variable should be used to
export the logs to a specific path?
If unsure, refer to the documentation, which as
always is available in the top right corner. So if
we followed along the course, we know that the
variable tf_log_path
is the right variable to use in this case. And that is
the correct answer. The next question says, can
you export the debug logs from OpenTofu just by
setting TF LOG PATH
environment variable and providing a path as the
value to this variable? The correct answer is no,
because you also need to export the variable
TF LOG and set it to one of the levels.
Okay, so the next question, we have a
configuration directory called [root
/OpenTofu/projects/project_a enable logging with
the log level set to error and then
export the logs to the path of /temp/project_a.log.
Once the environment variables are set, run
OpenTofu init and apply. It's okay if this results in
an error, do not change any
configuration files before you export the logs.
Okay, so what we need to do here is we need to
first export the TF LOG with the value of error.
So first of all, let's as always, change our

terminal location, and then do an export of
TF LOG=error. And then export TF LOG PATH
to the empty project_a.log. Now that we have
these two variables exported,
we can run OpenTofu init and then OpenTofu
apply. After confirming yes in the prompt, even
though we have an authentication failure,
we've been wamed that this is okay, and we
need to check our configuration. And sure
enough, the log file has been created in the TMP
location,
which log level provides the most details when
you run OpenTofu commands. So the trace
would provide the most detailed logging.
Now navigate to
/root/OpenTofu/projects/projectB, we have
already created a main.tf file for provisioning an
AWS EC2 instance with the tag name,
projectB web server, run a tofu init and apply to
provision the instance. So as always, switch our
location in the terminal, make sure we're in the
right directory.
And we can go ahead and run a OpenTofu init.
And OpenTofu apply. Maximize this a little bit and
confirm with yes.
Let's wait a couple of moments for the instance
to be created. And then check if it's working
correctly. Good. Next question.
Now try running OpenTofu plan again. What is
the effect? Let's see. We run OpenTofu plan. And
we see that the AWS EC2 instance project B is
tainted, so it must be replaced.
So the answer to our question is the resource will
be replaced. Why is the resource called project B
being replaced?
So if we run the OpenTofu plan again and inspect
the output, taint was set on the resource, as we
can see right here.
So the answer to our question is project B
resource is tainted. Apply the OpenTofu
configuration so that the resource called project
B is replaced.
The resource is currently tainted. So what we
need to do is we need to use the tofu apply with
the —replace command. So do that tofu apply
with the —replace=aws_instance.project_b. End
quote. What's going to happen is OpenTofu will
replace the following instance, destroying the first
one and creating the new one. Now that this is
complete,
we can go ahead and check our configuration.
And sure enough, our configuration looks solid.
Well, that's it for this lab. Thank you for watching,
and I'll see you in the next one.
```

###### ProjectB
main.tf
```
resource "aws_instance" "ProjectB" {
    ami = "ami-0c9bfc21ac5bf10eb"
    instance_type = "t2.large"
    tags = {
        Name = "projectb_webserver"
        Description = "Oversized Webserver"
    }
}
```

provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "4.15.0"
    }
  }
}

provider "aws" {
  region                      = "ca-central-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  endpoints {
    ec2                       = "http://aws:4566"
  }
}
```

###### ProjectA
main.tf
```
resource "aws_instance" "ProjectA" {
    ami = "ami-0c9bfc21ac5bf10eb"
    instance_type = "t2.large"
}
```
provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "4.15.0"
    }
  }
}

provider "aws" {
  region                      = "ca-central-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true

  endpoints {
    iam                       = "http://aws:4566"
  }
}
```


### Tofu Import

```
Let's now recap the OpenTofu import command.
OpenTofu import is used to bring resources
created by other means into the management of
OpenTofu.
For example, let's say we have a couple of
resources already created by OpenTofu, but
there may be resources that are provisioned
using another Infrastructure as Code tool such as
Ansible,
or resources created manually using the
management console. And all of these resources
can be imported into OpenTofu. Let's use an
existing EC2 instance as an example that we
want to import into OpenTofu.
As a prerequisite, we need a unique attribute for
this resource before we can import it. Let's copy
the instance ID for the EC2 instance from the
management console.
One thing to keep in mind is that the OpenTofu
import does not update the configuration files. It
only imports resources into the state file.
And as of this recording, the configuration files
have to be manually updated with the resources
that will be imported.
So, back into our terminal. Let's add an empty
resource block like this. Here, we have decided
to name the resource as WebServer2.

The resource can be named anything; however, it
should be unique within the current Terraform
configuration and state. Once the resource block
is defined, even though it is empty, we can now
run the OpenTofu import command.
The syntax to use in this command is OpenTofu
import followed by the resource address, which is
the resource type dot resource name.
After the resource address, we have to pass in
the unique resource attributes that we identified
from the management console.
Once we run the command, the WebServer2
resource will be imported into the state. Next step
is to complete the resource block and fill out all of
its arguments.
One way to identify the arguments to use is to
inspect the management console and identify the
resource attribute values such as instance type
and AMI ID.
However, this can be tedious as there are several
options that were used to create the systems.
Altematively, since it has already been imported,
we can also inspect the state file and look for the
attributes on our own.

Once we have all the details, fill in the arguments
in the resource block for WebServer2. Once all of
the arguments and correct values have been
provided, a OpenTofu plan will refresh the state
and understand that the instance already exists
and it will carry out no action.
The resource is now under the control of
OpenTofu. Going forward, any changes to the
resource can be carried out by updating the
configuration file and following the OpenTofu
workflow of init, plan, and apply.
Aside from using the OpenTofu import command,
there is a relatively new addition to OpenTofu's
import system, and it is called the import block.
Instead of running OpenTofu import commands
like we've seen in the previous slides, here we
make use of a special import block that makes it
easier to import resources.
The import block has the following arguments,
which is the instance address this resource will
have in its state file, and an ID attribute, which is
a string with the import ID of the resource.
Now, all that's left to do is to create the
configuration for the AWS IAM instance resource,
like this. You can see that the resource matches
the address argument from the import block.

OpenTofu processes the import block during the
plan stage. Once a plan is approved, OpenTofu
imports the resource into its state during the
subsequent apply stage.
You can remove the import blocks after
completing the import or safely leave them in
your configuration as a record of the resource
origin for future module maintainers.

lab

Hello and welcome to the lab of OpenTofu
Import. The first question on our list is to navigate
to the directory of
flashroot slash opentofu projects slash project
jade. We have a few resources created using the
configuration files. Inspect them first. So there is
an AWS EC2 Instance resource,
an output variable to instances, a required
provider, and the provider settings, and a couple
of variables. Okay, let's see what's in store for us.
Okay, so the next question says which of the
below resources is not part of this configuration.
So we need to run an OpenTofu show. But first,
let's go into the OpenTofu projects
project Jade, and then run the OpenTofu show
command. Just to have a feel of the resources
that are applied for this project.
For this project. Okay, so going back all the way
to the top, we can see that there is an AWS
instance running called Jade agent.
Then we have Jade app. Then we have Jade app
to then Jade LBR, then Jade web server. And
that's about it. The rest are just outputs.
So this one is non-existent in the state file. So go
ahead and select it. What is the name of the SSH
key which is used by all of these instances.

So a nice way to check is to take a look at the
state file again. And what we're looking for is the
key name.
So let's go ahead and check out key name,
which is Jade. And just for good measure, let's
take another just to be sure Jade and one more
Jade.
Jade. So Jade is the key that is being used. Is
the key pair resource created by this OpenTofu
configuration.
So if we take a look at variables, there's only
variables there, then the Terraform and provider
blocks and provider.tf and main.tf only has AWS,
AWS instance as the resource and output for
instances. So the answer is no, it is not created
by OpenTofu.
That's right, the key called Jade has been
created using the AWS CLI. The command used
to create this key is AWS EC2 create key pair
with the endpoint of AWS at 4566
key name jade_query_key_material output to text
and then output all of that to root OpenTofu
projects projects jade and jade.p and jade.pem.

The private key is created in the same
configuration directory we have been working on.
We have another EC2 instance created called
the Jade MV using the AWS CLI.
Using the AWS CLI, inspect the EC2 instance
and find the ID that is created by it. Okay, so in
order to do that, here are some specifications of
this EC2 instance.
This is the AMI ID, the instance type, the key
name, and the key type. The AMI ID, the instance
type, the key name.
Remember the syntax to use the AWS CLI, AWS
command and subcommand options. Also, make
sure to pass in the endpoint as we are using the
AWS test framework.
And usually this is not required when you interact
with actual EC2 instances. So we need to use an
AWS command to
inspect the EC2 instance and get the appropriate
instance ID. So the appropriate command to use
this can be found in the AWS documentation.
But here we will type it in as follows. So AWS
EC2 as our command, and then describe
instances as our subcommand.

Now let's just put in the endpoint to get that out of
the way, because we're using an AWS local test
framework.
Okay. And let's see, describe instances is not
correct. So it needs to be plural. And now we're
getting somewhere.
Good. So let's see what we have here. So I'm
pressing space right now to load up the entire
configuration from the CLI, which will give us all
of the instances.
So let's see, we have instances with what we're
looking for is the AMI ID of A, B, 2. So this
matches, the instance type matches, and the key
name matches, which is good.
So let's now take a look at the EC2 instance.
Let's now take a look at the instance ID. So here
it is at the very top.
Right, so let's find that one here. So it starts with
IBD and ends in C14. So this is our instance.
Let's manage this instance called Jade MV with
OpenTofu. First, let's create an empty resource
block for this instance in the main.tf.
In the configuration directory, /root/OpenTofu
project/project Jade. Okay, so let's minimize this.
And let's go into our main. And let's go ahead
and create the resource AWS EC2 instance

called Jade-MV.
And we will fill in the argument for this block later.
So let's go and check it out. And that seems to be
good.
Let's move on. All right, now we need to run
OpenTofu import to import the instance into the
OpenTofu State.
The resource name should be the same as the
one used in the previous question, which is the
JAD-MV. Okay, so let's clear up our terminal and
extend it a little bit so we can see better.
And now what we need to do is we need to run
the OpenTofu import. And the resource that we're
importing is the AWS instance dot Jade-MV.
And then we need the ID of the resource. So we
determined that already. But since we cleared
our terminal, let's open up another terminal.
And inside of that terminal, let's run the AWS.
Actually, we might have it in history. Now we got
AWS EC2 describe instances.
Don't forget the endpoint http://aws:4566. And
let's just go ahead and run that and find our
instance ID. And let's copy that over.

And let's use it here. For some reason, the
terminal does not allow me to paste. So we're
gonna have to type it out like this.
1084 Cl 4. Good. So an argument definition is
required. So what we're missing here is and I
believe I forgot to wrap it in double quotes.
No, that's not it. So let's see OpenTofu import
AWS instance dot Jade-MV. Am I missing
anything? Okay, that seems about right.
So let's go ahead and import our resource. And
the import was successful. Good. Let's check the
question. And that seems to be all right.
Let's go to the next one. Great. We're nearly
there. What would happen if we run OpenTofu
apply? Let's see.
Let's run the OpenTofu apply. But in the right
directory, and we see here that we're missing
required arguments. So there is an error that
resource arguments are not defined.
Select this as the answer. And let's fix that now.
Complete the resource block for Jade-MV.
Inspect the state file to make sure that all
arguments used to create this resource

are defined in the resource block. If unsure, run
the OpenTofu apply after filling the arguments
and correct them until a plan shows no changes
to apply.
Okay, good. So let's go ahead and do OpenTofu
apply. Just to see again, what type of arguments
we need.
Okay. On main.tf, resource launch template, one
of the AMI instance type launch templates must
be specified. Okay. Okay, so let's do an
OpenTofu show command.
And let's see what outputs we have for our
resources. So the first resource, actually, let's go
all the way to the top.
So we have the JAD-MV resource right here that
has been imported. And that's the instance that
we're looking for.
So now, from our apply, we know that we're
missing the launch template, instance type, and
AMI. So let's go ahead and populate the AMI,
which is the AMI right here. And now, we need
the instance type, which is T2 large, right here.
T2 dot large.
And lastly, we need the launch template. So
launch template is available.

Actually, by looking at the error right here, launch
template requests one of either AMI, instance
type, or launch template must be specified.
Since we specified AMI, that is resolved. So let's
go ahead and save this up and run a OpenTofu
apply now.
And let's see what we get. Okay, so there is
actually one more thing that starts to change
when we run the apply, and that is the tag.
So let's cancel that. And let's do another field
called tags. And let's do a map. And the name is
going to be Jade-MV.
Good. And OpenTofu plan, let's see what
happens in the plan. Good. The resource will still
be updated in place.
And I think there is one more thing that we have
here in the OpenTofu show for our instance that
we forgot to supply,
and that is the key name. So let's go ahead and
supply that. over here. And let's clean that up,
save that.
And let's run an OpenTofu plan now. Okay, so
aside from timeouts, nothing changes. So let's
test out and see if our configuration works.
And that seems to be okay. Great. That will be
the end of this lab. Thank you for watching, and
I'll see you in the next one.
```

```
aws ec2 create-key-pair --endpoint http://aws:4566 --key-name jade --query 'KeyMaterial' --output text > /root/opentofu-projects/project-jade/jade.pem


aws ec2 describe-instances --endpoint http://aws:4566


tofu import aws_instance.jade-mw i-47a8764efa9248c14
```

main.tf
```
resource "aws_instance" "ruby" {
  ami           = var.ami
  instance_type = var.instance_type
  for_each      = var.name
  key_name      = var.key_name
  tags = {
    Name = each.value
  }
}

output "instances" {
  value = aws_instance.ruby
}

resource "aws_instance" "jade-mw" {
  ami = "ami-082b3eca746b12a89"
  instance_type = "t2.large"
  key_name = "jade"
  tags                                 = {
        "Name" = "jade-mw"
    }
}
```
provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "4.15.0"
    }
  }
}

  

provider "aws" {
  region                      = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true

  endpoints {
    ec2 = "http://aws:4566"
  }
}
```
variables.tf
```
variable "name" {
  type    = set(string)
  default = ["jade-webserver", "jade-lbr", "jade-app1", "jade-agent", "jade-app2"]

}

variable "ami" {
  default = "ami-0c9bfc21ac5bf10eb"
}

variable "instance_type" {
  default = "t2.nano"
}

variable "key_name" {
  default = "jade"
}
```


## OpenTofu Modules

https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest/submodules/ssh

```
configuration directory containing OpenTofu
configuration files is a module. For example, let's
consider a configuration directory called aws-
instance in our local
machine. This directory is present under root-
OpenTofu-project. This directory has OpenTofu
configuration files that are used to create a
simple EC2
instance in AWS as shown here. And since we
can run OpenTofu commands from within this
directory directly, this configuration
is known as the root module. Or in other words,
every OpenTofu configuration is a root module.
Users can call other modules, which allows
resource configurations to be packaged and
reused. Let's consider a use case where we want
to make use of the code written in the AWS
instance
directory to create a dev web server instance.
For this, let's create a new directory called
development within the opentofu-projects
directory.
Now instead of writing the configuration for
creating the dev AWS instance from scratch or
copy and paste the code, we can use a module
block to use the code which is already located

in the aws-instance directory. To do this, inside
the development directory, create a configuration
file containing a module block like this.
In this module block, we can provide the path to
the local aws-instance directory which contains
the actual OpenTofu configuration for the aws-
instance.
Since our configuration directory where we will
run OpenTofu commands is now the
development directory, it is now the root module
and the aws-instance directory that we are calling
within our main.tf file is called the child module.
To call a child module, we use the module
keyword followed by a logical name for the
module. In our case, we have just called it dev
web server. Inside the module block, we have
one required argument which is the source.
This is the path where the child module is stored.
Take a note that in this example, the path for the
child module is specified as a relative
path with respect to the configuration directory or
the root module which is the development
directory. This could also be an absolute path to
the child module as well.

What we have here is an example of a local
module given the fact that it resides within the
local machine where OpenTofu is installed.
One of the advantages of using modules is that it
can be easily shared with others. We saw earlier
that the OpenTofu registry is used to store
provider plugins, such as
the one used for AWS. Besides the provider
plugins, the registry is also used as a public
repository that stores
modules. And instead of relying on local
modules, we can reuse modules that have
already been developed and stored under the
public registry.
In this lecture, let's create a new module in
OpenTofu and then put it to use to deploy
multiple environments of the same infrastructure.
Let's assume that an organization called FlexlT
Consulting has a blueprint of a prototype payroll
software that needs to be deployed in several
countries.
Each country will have its own instance of
software deployed on the AWS cloud using the
same architecture. It has a single EC2 instance
using a custom AMI that will host the application
server.

A DynamoDB NoSQL DB that will be used to
store the employee and payroll data. Then we
have an S3 bucket which will be used to store the
pay stubs, tax forms and other
important documents. Users access the
application hosted on the EC2 instance. This is
an architecture in its most simplified form that
uses the default VPC without any
considerations for VPC endpoints or IAM roles
for the services. The architecture diagram at a
very high level looks like this.
So the idea is to create a reusable OpenTofu
module and use it to deploy the same stack of
resources in different countries.
And based on this high-level architecture, we can
create our configuration in a directory. Since we
plan to create a module, we create it under a
directory called modules.
For instance, /root/OpenTofuProjects/modules.
This is where we plan to store all our reusable
modules. Inside this, we have the resource block
for all the required services like the EC2
instance,
the S3 bucket and the DynamoDB. To ensure
that we have consistent environments and to use
a specific instance type for all

the environments created by the module, the
value of instance type has been hardcoded in the
configuration, like this. We will see why this is
done soon.
The AMI, however, is set to a variable called AMI
and is user-configurable based on the region we
want to deploy the application.
Same goes for the tag called name. Here, we
make use of an interpolation expression to use
the value of the name dependent on
the region where we want to deploy this
application. An explicit dependency is defined in
the EC2 resource block to ensure it is created
only
after the DynamoDB and the S3 buckets are
created. And the same goes for the S3 bucket
name. To ensure we have a globally unique
name for the bucket, we are adding the region as
a
prefix to the bucket name. Finally, the
DynamoDB name and the primary key for the
table have been assigned fixed
values. Please note that to improve the
readability, the configuration files have been
truncated. And our variables look like this.

The variables called appRegion and AMI have
the variable type defined. The variable called
bucket, however, has a default value assigned to
it.
These variables are used in the configuration
files we just saw. Now let's deploy an instance of
this application stack first in the US East 1
region.
So we create another directory called
USPayrollApp and inside this directory we create
a main.tf file with the contents like this.
We use the module block and specify the path to
the PayrollApp module we created earlier. To
make sure that we deploy it to the US East 1
region and use the custom AMI ID in that
region, we can specify the app region and the
AMI in our root module like this. And that's it.
Since this is our working configuration directory,
the path /root/OpenTofuProject
/USPayrollApp is now the root module and the
module /PayrollApp, which is used as the source,
is the child module.
You would have noticed that the only
configurable value in the main.tf file is the app
region and the AMI.

We can also optionally provide a specific value
for the bucket argument, but if not, it will take the
default value that was set at the module level,
which is flexit-it-payroll-alpha-22001c.
We do not want the instance type, the
DynamoDB name, and the primary key to
change, which is why the values were hardcoded
in the module configuration.
With just the module block, our configuration file
is complete and ready to be applied. If we run the
OpenTofu init in the development directory, we
will see that OpenTofu downloads
the module from source along with the AWS
plugin which is used by the module. A plan
followed by an apply will show that the
DynamoDB table, the S3 bucket and the
AWS EC2 instance are created. Observe that the
DynamoDB table name, primary key and
instance type for the EC2 instance
take values that were defined in the module. The
bucket name, on the other hand, makes use of
the value we defined in the root.
To deploy the same stack in the UK region, let's
create another directory called uk-payroll-app.
We follow the same process of creating a main.tf
file and then use the module block.

And we use the value of eu-west-2 for the app
region. We have also changed the AMI to select
the one that's available in this region.
We also configure the provider to use the eu-
west-2 region. A OpenTofu apply now will create
the exact same stack but in the London region.
Notice the name of the AWS S3 bucket created
now, that has a prefix of eu-west-2. Also note the I
addressing syntax of the new resource that is
created.
Since we are making use of a module, the
DynamoDB table resource, the DynamoDB table
resource created is now identified by addressing
syntax that goes like this.
First comes the module address, which is the
keyword module, followed by the module name,
separated by a dot. Next comes the resource
address, which is the resource type, followed by
the resource
name, also separated by a dot. So in this
example, the DynamoDB table resource in the
root module can be addressed as module.us-
payroll.aws-dynamodb-table.payroll-db.
So now we're going to take a look at the public
registry for OpenTofu and Terraform. Since
OpenTofu sources the modules the same way
Terraform does, we're mostly going to

be looking at the Terraform Registry. The
modules can be categorized into two types, a
verified module and the community module.
For example, if we want to create an AWS
security group, instead of creating the whole
configuration from scratch, we can just search fo
a matching module for creating security groups i
the
registry. Notice that this search gave us quite a
few matching results. What we are looking for
here is a module for creating a security group for
the AWS IAM provider,
so let's go with the one that is appropriate. A
module that is available in the registry usually
contains details about the module,
such as the publisher information, the different
versions of the module available in the registry,
and instructions on how to use the module with
examples.
The module may also contain several sub-
modules that can be used in different cases. To
make use of this module in our root module, we
add the module block as before, but the
value for the source argument would change as
the module is no longer local. To create a
security group that allows inbound SSH, we can
create a configuration file with

a sub-module like this. It is a good idea to specify
the version of the module, as there can be
revisions made
to each module over the course of time. To do
this, we have used the version argument here.
Just like Providers, if the version is not specified,
OpenTofu will download the latest
version of the module from the registry, which
can break our configuration. We can also make
use of the version constraints with modules just
like we did with Providers.
This ensures that we always use a working
version of the module. From the documentation
for this sub-module, we can see that it requires 3
mandatory arguments.
The name of the security group, the VPC to
create it in, and the Ingress CIDR blocks which
accepts a list of IP ranges that should be allowed
SSH access.
In order to make use of this module, we can
either run OpenTofu init, or, if the provider plugins
have already been downloaded in the
configuration directory, we can simply run
OpenTofu get command that will download the
module from the registry. Finally, to create the
security group using the module, we run
OpenTofu plan and then apply.

This leaves us with a question of why should we
use modules? Modules bring in a lot of benefits,
such as simpler configuration.
Instead of having configuration files that are
hundreds and thousands of lines long, our root
module is very short, easy to understand and
manage.
Using a pre-configured module that has been
tested and validated also decreases the risk of
human errors resulting in inconsistent
environments.
As we just saw, we can reuse the same module
as many times as we want, thereby improving th
reusability of our code.
And finally, with modules, we can lock in certain
aspects of the configuration such as variables
that are not user-configurable from the root
module.
This allows us to make sure that the provisioning
process maintains a standard configuration. We
will see more of that in the upcoming lecture.

lab

modules. First question that we have is the
following. A configuration directory has been
created
at the path of /root/OpenTofu/projects [Project
Sapphire. Inspect the main.tf file created in this
directory and answer the following questions.
All right, so we can see that it's using a module
called IAM IAM Users. And which configuration
block is defined
in the main.tf at the moment? So we can already
answer that a module is defined. What is the
source of the module used in this configuration?
If we check the source, we can see that it's
actually using the public OpenTofu registry. What
is the version of the module used?
As we can see here, we have a second argument
that says that the version being used is 5.28.0.
How many required arguments does this module
expect?
You may need to look up the documentation to
figure this one out. Refer to the registry at
Terraform. So let's go ahead and open this up in
another tab.
And let's take a look at the documentation. So
how many required arguments? So if we switch
to the inputs

and then check the required inputs, we can see
that only the name is required. So the answer to
our question
is only one argument is required. Which
argument is to be specified just to create an IAM
user with this module?
So it is the name argument. Now update this
module block that will allow it to create an IAM
user called Max.
Adhere to the following requirements. Only use
the module block in the main TF file and do not
add a separate resource block.
Module name needs to be IAM IAM-user. Only
add the single required argument at this time. So
let's go ahead and add the name.
Name equals max. The provider TF and variable
TF files have been added to this configuration
directory now. When you're ready, run a
OpenTofu init and plan.
So let's go ahead and change our directory to
project Sapphire. Run OpenTofu init to download
the Providers and the module.
And after that finishes, we can run the OpenTofu
plan. As the guide says, we do not have to
provision the resources at this moment,

but we can try if we wish to. So let's go ahead
and just run OpenTofu plan at the moment.
And sure enough, we can see that a user with his
access keys and a login profile will be created.
So let's go and check our configuration. That
seems to be okay. Let's move on. Okay, so how
many resources are set to be created
in the execution plan? We saw that the three
resources will be created. So let's go ahead and
run the OpenTofu plan.
Three resources will be created. Which
resources are set to be created? So we inspect it
again. We can see that an IAM user is going to
be created
an IAM user, IAM access key, and a login profile.
So this would be the correct answer. Why is the
module creating additional resources
when only the name for creating an IAM user
was defined in the main.tf file? Inspect the inputs
for this module in the documentation.
Okay, so the answer to this question lies in the
documentation. So if we go to the documentation
and take a look at what exactly will be created,

we can see that the inputs say create IAM
access key, create IAM user login profile and
create user all have a Boolean that by default is
set to true.
So the default configuration of this module makes
sure that a user, a user login profile and an IAM
access key will be created.
So the answer to our question is the three
resources will be created by default as per the
module configuration.
We only want to create an IAM user. Update the
module block to only allow create user and
disable IAM access key and IAM user login
profile.
So to do this, we just need to add the arguments
to the module, create IAM access key. Set it to
false and create IAM user login profile.
and set it to false. And once we save that up, we
can go ahead and run OpenTofu init just to
reinitialize everything
and then OpenTofu apply. And now we can see
that only one resource will be added, which is the
IAM user.
So we continue by typing yes in the prompt,
waiting a couple of moments for the user to
initialize and then check our question and that's
it.

That would be all for this lab. Thanks for
watching and I'll see you in the next one.
```

```
mkdir /root/opentofu-projects/development
main.tf
``` 


main.tf
```
module "dev-webserver" {
	source = "../aws-instance"
}
```




**Output**
```bash
$ mkdir /root/opentofu-projects/modules/payroll-app
app_server.tf dynamodb_table.tf s3_bucket.tf variables.tf
```

**app_server.tf**
```hcl
resource "aws_instance" "app_server" {
  ami             = var.ami
  instance_type   = "t2.medium"
  tags = {
    Name = "${var.app_region}-app-server"
  }
  depends_on = [ aws_dynamodb_table.payroll_db,
    aws_s3_bucket.payroll_data
  ]
}
```

**s3_bucket.tf**
```hcl
resource "aws_s3_bucket" "payroll_data" {
  bucket = "${var.app_region}-${var.bucket}"
}
```

**dynamodb_table.tf**
```hcl
resource "aws_dynamodb_table" "payroll_db" {
  name           = "user_data"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "EmployeeID"

  attribute {
    name = "EmployeeID"
    type = "N"
  }
}
```

variable.tf
```
variable "app_region"{
 type = string
}

variable "bucket" {
default = "flexit-payr011-a1pha-22001c"
}

variable "ami" {
type = string
}
```

```
mkdir /root/opentofu-projects/us-payroll-app
main.tf  provider.tf
```
main.tf
```
module "" {
 source = "../modules/payroll-app"
 app_region = "us-east-1"
 ami = "ami-2357552336avm"
}
```

```
mkdir /root/opentofu-projects/uk-payroll-app
main.tf  provider.tf
```
main.tf
```
module "" {
 source = "../modules/payroll-app"
 app_region = "eu-west-2"
 ami = "ami-35e140119877avm"
}
```
provider.tf
```
provider "aws" {
 region = "eu-west-2"
}

```



```
module "ssh_security_group" {
  source  = "terraform-aws-modules/security-group/aws//modules/ssh"
  version = "~> 5.0"
  # insert the 2 required variables here
  vpc_id = "vpc-7d8d215"
  ingress_cidr_blocks = ["10.10.0.0/16"]
  name = "ssh-access" 

}
```

```
tofu get
```


https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest/submodules/iam-user
main.tf
```
module "iam_iam-user" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-user"
  version = "5.28.0"
  # insert the 1 required variable here
  name = "max"
  create_iam_access_key = false
  create_iam_user_login_profile = false
}
```

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.11.0"
    }
  }
}

provider "aws" {
  region                      = var.region
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  endpoints {
    iam = "http://aws:4566"
    ec2 = "http://aws:4566"
    s3 = "http://aws:4566"
  }
}
```

```
variable "region" {
    default = "us-east-1"
}
```


local values


```
they can be used within the OpenTofu
configuration. Let's use an example that we have
used several times throughout this course.
An AWS instance called web with a specific AMI
and instance type. Let's assume that this EC2
instance has been created for the project
Cerberus and for the
finance department. So accordingly, we can add
tags to this EC2 instance like this. Now, let's
consider another AWS EC2 instance called DB
that has been created for the same
department and the same project, but perhaps
with a different AMI ID and instance type. We can
add another resource block like this. The
arguments for the AMI and instance type
may be different as compared to the web
resource, but the tags are the same. And using
the same approach as we did in modules, we can
add these common tags to the EC2 instances
belonging to the same department and project.
Reusing the same code for the tags over and
over again can make our configuration
unnecessarily long. Instead, we can simplify the
configuration

by using local values. To create the local value in
this example, we can make use of the locals
block like this. Inside the locals block, we add a
name such as common tags like this. And to do
this, we move all the common tags from the EC2
instances that we want to reuse. In this case, this
would be the key value pairs department equals
finance and project equals servers.
Now that we have the locals block defined, we
can reuse it as many times as we want within our
resource blocks. To do this, we can use the
expression with the syntax local.name like this. In
our example, the name of the local value is
common tags, so our expression would look like
this. Running OpenTofu plan or apply now will
show that both the AWS instance resource
blocks are now using common tags as expected.
So far, we have used local values with literal
constraints
such as the tags with department equals finance.
However, they can also be used to refer to other
values such as variables, resource attributes, or
other local values. Here's a sample main.tf file
for creating an S3 bucket in AWS. S3 bucket
name argument has to have a value that is
globally unique. To do this, let's use a
combination of a random string generated by a
random string

resource and a variable called project with the
default value of servers. To do this, let's add the
respective resource and variable blocks like this.
Here, we have decided to go
with a random string resource with a length of six
characters without using special characters or
uppercase letters. Now, let's make use of a local
value called bucket prefix and use the
interpolation sequence to form our bucket name
like this. And finally, we can use the expression
local.bucket_prefix as the value for the bucket
argument in our AWS S3 bucket resource block
like this. And that's it! Once the resource is
created using OpenTofu apply, we can see that
the bucket is created with the naming convention
we wanted, that is the project name dash random
string dash bucket. Well, that's it for this lecture. I
will see you in the next one.
```
main.tf
```
resource "aws instance" "web" {
ami = "ami-06178cf087598769c"
instance_type = "t2.medium"
tags = local.common_tage
}

resource "aws instance" "db" {
 ami = "ami-0567cf08759818b"
 instance_type = "mS.large"
 tags = local.common_tage

}

locals {
 common_tags {
  Department = "finance"
  Project = "cerberus"
 }
}

```



main.tf
```
resource "aws_s3_bucket" "finance_bucket"{
 acl = "private"
 bucket =
}

resource "random_string" "random-suffix"{
 length = 6
 special = false
 upper = false
}

variable "project" {
 default = "cerberus"
}

locals {
  bucket-prefix = "${var.project}-${var.random_string.random-suffix.id}-bucket"
 }
```


## funcation

```
In this lecture, we will recap several built-in
functions, operators, and conditional expressions
available in OpenTofu. We had a look at the
toSet function, which is used to convert a list to a
set.
And although we have used functions before, we
have done so by using them in the configuration
files, without seeing what the functions do
specifically and how they transform and combine
values provided to them. Fortunately, OpenTofu
provides an interactive console that can be used
for testing functions as well as interpolations.
To get into the console, we use the OpenTofu
console command. For example, we can test the
file function in the console like this, and it should
return
the contents of the main.tf file. Using the length
function shows that the number of elements in
the list available called
regions is 3. And finally, we can see that the
toSet function converts the same variable from a
list to
a set and removes duplicates. Remember that a
set cannot have duplicate values, so during the
transformation, the toSet function removes the
duplicate value of us-east-I , which is present in
the list variable.

Besides the three functions, there are dozens of
other useful built-in functions available in
OpenTofu. We will take a look at some of the
commonly used ones next.
Let's review the numeric functions, which are
used to transform and manipulate numbers. We
can use the max function to return the greatest
number from a set of numbers.
The min function does the reverse and returns
the smallest number. We can also make use of
variables with functions.
For example, to find the greatest number in a set
type variable called num with the minimum or
maximum function, the values inside the set can
be expanded to separate arguments.
This is done by using the expansion symbol by
using 3 periods or full stops, like this. Some other
commonly used numeric functions are ceil and
floor functions.
The ceil function returns the closest whole
number that is greater than or equal to the
argument provided to the function.
For example, ceil 10.1 will return the value of 11.
Same as ceil 10.9, as 11 is the closest whole
number greater than any of these two floating

numbers. Floor does the opposite and returns the
closest whole number that is lesser than or equal
to the argument provided.
Let's now take a look at some string functions.
These functions are used to transform or
manipulate string-type data. The first one that we
will take a look at is the split function.
This function is used to transform a string to a list
of elements by making use of a separator. The
syntax to do this is the split function followed by a
separator and the string enclosed
in brackets. Again, this can be tested using the
OpenTofu console. Notice that the string we have
used here has a few dummy AMI IDs that are
separated
by a comma. To split the string to a list containing
three elements, we can use the split function with
the separator, which is a comma and the
complete string, like this. Notice that the output is
a list now.
Just like before, let's also make use of a variable
and test this function. Here we have a variable
called AMI with the same string.
Passing in the variable as the second argument
to this function, comma being the first, we get the
same result as before.

Did you notice that in the variable called AMI,
there is a text that is all in uppercase? To convert
the string to use only a lowercase format, we can
use the lower function, like
this. And to do the opposite and change it all to
uppercase letters, we would use the function
upper. To convert the first letter of each word in a
string, use the title function, like this.
The substring function is used to extract a
substring from a string, or in other words, cut a
string into a smaller string by providing an offset
and a length.
The offset defines the index of the character after
which we want to cut a string. For example, to cut
from the very first character of the string, we
provide the offset 0, and
to cut from the second character, we use the
offset of 1, and so on and so forth. The length
defines the number of characters from the offset
to cut and convert to a substring.
Excluding the commas, the length of an AMI ID in
this example variable is 7. As such, we can
create a substring using the length of 7.
The offset can also be 8 or 16. The join function
does the opposite of split. It converts a list to a
string by adding together all the values.

It can also be used with a variable. Let's have a
look at collection functions, which are used for
collection data types,
such as a set, list, and a map. We have already
seen one example in this category, which is the
length function, which is used
to determine the number of elements in a list, in
this case, three. For example, to find the index or
location of the element AMI ABC in the AMI
variable,
we use the index function like this. The function
takes in two arguments. The first is the list, and
the second is the value of the element for which
we want to
find the index. In this example, the element AMI
ABC is located at index 1. Remember that the
index of a list always starts with 0.
To find an element in a list located at a specific
index, we can use the element function like this.
Here, the element located at index 2 is AMI EFG.
To check if a specific element is present in an
array or not, use the contains function
like this. This function will return a value of true or
false. In the first example, the element AMI ABC,
all uppercase, is present in the list and

like this. This function will return a value of true or
false. In the first example, the element AMI ABC,
all uppercase, is present in the list and
hence retums a value of true. In the second
example, the element AMI XYZ, all uppercase
again, retums a value of false,
as such an element is not present on the list.
Although the same value is present in the
lowercase, the contains function is case
sensitive,
thus not able to find the all uppercase AMI XYZ.
To wrap up this lecture, let's have a look at some
functions used for map data types.
Here, we have an updated AMI variable which is
now a map. This map uses regions as a key and
an associated AMI ID as the value.
To convert this map to a list with just the keys, we
can use the keys function and supply the map as
an argument, like this.
This would return a list of all the regions in the
keys of the map. To convert it to a list with just
the values, which are the AMI IDs in this
example, we

use the values function. This would retum a list of
all the AMI IDs. Finally, to look up the value ofa
specific key in a map, we can use the lookup
function,
like this. This function takes two arguments, the
map and the key to look up the value for, which in
this case, if we supply CA Central 1, it would
return the AMI EFG.
If the key provided to this function is not available
in the map, this will result in an error. However,
we can optionally pass in a third argument to this
function.
This default value will be returned if the key
provided in the argument does not exist in a map,
like this.
In this case, the lookup function will return the
value of AMI PQR, even though the key US West
2 does not exist within the map, as this value has
been supplied to the function
as a default.



In this lecture, we will look at some common
operators and conditional expressions in
OpenTofu. In the previous section, we saw how
to test functions using the OpenTofu console
command.
That's not the only use case for it. You can also
make use of arithmetic and logical operations
with OpenTofu,
and test the same using the OpenTofu console.
For example, we can make use of arithmetic
operations like these by making use of arithmetic
operators.
Besides arithmetic, OpenTofu also supports
equality, logical, and comparison operators.
Equality operators take two values of any data
type and produce a Boolean result of either true
or false.
To check if two values are exactly equal, we can
use the double equals operator like this.
Conversely, the exclamation equals operator
prints a value of true if the two values are not
equal.
In this example, the comparison result produces
false, as the value on the left of the operator is a
number,

and the one on the right is a string, as it is
enclosed within double quotes. The comparison
operator of greater than, less than, greater than
or equal to, and less than or equal to
take two values, both of type number, and
produce a Boolean result. For example, here, 5
greater than 7 results in false, or 5 greater than 4
results in true,
5 greater than 5 results in false, and 5 greater
than or equal to results in true, 4 less than 5
results in true, and 3 less than or equal to 4
equals true.
And finally, let's take a look at logical operators. A
logical AND operation accepts two Boolean
values and returns true only if both values are
true.
For example, here, 8 being greater than 7 and 8
being less than 10 are both true, hence the result
is true as well.
If either of the values is false, the result would be
false as well. The logical OR operation retums a
value of true if either value provided to it is true.
For example, 8 being greater than 9 or less than
10 produces our result of true since the second
condition is true.

Finally, the NOT operator returns false if the
condition is true and true if the condition is false.
For example, we have a Boolean variable here
called special with the default value of true.
The value of var special returns true. But, when
we use the NOT operator in front of this variable,
then the result is false.
Another variable b has a value of 25. We can add
a NOT operator before the comparison b greater
than 30.
Although the comparison is false and 25 is not
greater than 30, we get the result as true as we
are making use of the NOT operator on the result
of this comparison.
And the same operations can be applied to
variables in OpenTofu. If we consider a
variables.tf file with two variables of type number
called a and b,
we can use the operators like this. var.a greater
than var.b would result in true. var.a less than
var.b would result in false.
var.a plus var.b would result in 75. Great, so now
we know a lot about these various operators, but
how do we use them within OpenTofu?

Say we want to assign a variable to an argument
based on a condition. Say if it is greater than
another variable.
Let's make use of a simple example. Let's say we
want to generate a new password. This can be
done by making use of the random password
resource type from the random provider.
The resource will generate a random password
with a user-specified length argument. To print
the password on the screen after OpenTofu
apply, we will make use of an output variable
called password.
We have also defined a variable called length of
type number and used it in the main.tf. Note that
there is no default value assigned to this variable
in variables.tf file.
We will pass in a value for this manually when
running the OpenTofu apply command. If we run
OpenTofu apply now, a password with the
specified requirements will be printed.
But wait, what if you want OpenTofu to accept a
value of length that is at least 8 characters long?
And in case it isn't, to fail the OpenTofu apply
during the password creation. With common
scripting methods such as this with a bash script,
this can be done using an if-else statement like
this.

And in case it isn't, to fail the OpenTofu apply
during the password creation. With common
scripting methods such as this with a bash script,
this can be done using an if-else statement like
this.
It has a condition and two statements, one if the
condition equals true and the other if it's false. If
the length variable has a value less than 8, then
we assign it a value of 8 and if not, we use the
length as it is.
Once done, we can use the variable to generate
a password. The same can be done in OpenTofu
by making use of a conditional expression.
The syntax of this is as follows. First, we have to
specify the condition. This is followed by a
question mark and a value.
Following this, we have a colon and another
value. If the condition is true, the value to the
right of the question mark is considered.
If false, the value to the right of the colon is
considered. Let's put this to use and update our
OpenTofu configuration file for generating a
random password of at least 8 characters.
The value of the length argument can be updated
like this. The condition used here is to check if
the variable is lesser than 8.

If true, use 8 as the value for the length
argument. Else, make use of the length variable
as the value of length.
If we run OpenTofu init plan and apply now with
the value of length as 5, you can see that
OpenTofu still makes use of the length as 8 and
generates the password.
For any value greater than 8, 12, as in this
example, OpenTofu accepts the value and
generates the password as expected.
Well, that's it for this lecture. Head over to the
practice tests and practice working with functions
and conditional expressions in OpenTofu.
See you next time. OpenTofu


lab


functions and conditional expressions. So let's
get started. In this lab we will work on functions.
Just like we did in the lecture,
we make use of the OpenTofu console command
through this lab to test different functions and see
how they transform and manipulate values.
What value does floor 10.9 produce? So we
know that the floor function rounds down to the
first round number. So let's go ahead and select
10. Or before we do that, we can even
test using the OpenTofu console and typing in
floor 10.9. And we see that it rounds down to 10.
What value does the user-generated password
file produce? So again, by making use of the
OpenTofu console, we can go ahead and type in
the contents and close that up. And we can see
that every first word starts with a capital. So it
would be user-generated password
file. Okay, so which type of variable does the
function lookup work with? So lookup retrieves
the value of a single element only from a map.
So the correct answer is map. Navigate to the
configuration directory slash root slash OpenTofu
projects project sonic. Here we have a number of
variables declared in the variables.tf file. What
type of variable is cloud_users? So it seems to
be

a string type variable. This variable contains the
developer names for project sonic with the
names separated by a colon. Using this variable
and the count meta argument, create IAM users
for all the developers, write the resource block
and name it as cloud in the main.tf file. Convert
this variable from a string to a list and do not
change the variable defined in variables.tf.
When ready, create the users by running tofu init,
plan, and apply. So the trick here is that we need
to make use of Terraform functions and not
actually change the way the default value for the
variable is set up. So the first thing that we have
to do is we have to create a resource for the
AWS IAM user. And we're going to call it cloud as
instructed. Okay, the first argument that we need
is the name. So we're going to have to use the
cloud users variable. And we see that the
and we can see that the separator for the user
names is the colon. So we're going to have to
use the split function
by using the colon as the delimiter. And then
var.cloud_users as the second argument. And
then we're gonna do count index. And the count
is going to be the length of the split.

With the column being the delimiter and the bar
dot cloud users. So we're going to count how
many users we have by using the length and
using the split to use the column as the delimiter.
So now if we save this up, and exit our OpenTofu
console, switch to the project Sonic, run the
OpenTofu init command to download the required
modules and providers.
And now run tofu plan. Let's see what happens.
So what is expected to happen is 1234567 users
should be created. So let's see if that is our
case. And we have seven users, starting from
array zero and going through all of the names for
each user. And that seems about right. So we
can go ahead with the fly. And once prompted,
we will type in yes in the prompt to confirm these
changes and apply all of these seven users. Let's
now check if our solution is good. And it seems to
be all okay. Okay, so let's move on to
the next question. What is the name of the IAM
user that is created at index six of the IAM user
What is the name of the IAM user that is created
at index six of the IAM user at address AWS IAM

user dot cloud, use the OpenTofu console
command to find out. So once again, we're going
into the console. And what we need to do now is
check out the AWS IAM user dot cloud.
And we're going to use brackets, and we're using
the index six. And then we need to get the name
of the user. And we can see that it is Bra. So we
select Bra. Next question is locate the index
of the element called Oni in the variable called
SF. So we have a variable called SF, and we
need to locate the index called Oni. So again, we
can use the OpenTofu console command
and make use of the index function, and then
provide the variable. And the value of the index
we're looking for was the bracket, and we get
that Oni is at index
seven. And if we were to count it, it would be
1234567. So the solution to this question is
number seven. What type is the variable called
media? So media is a set of strings, as we can
see
here. We have now updated the main.tf in this
configuration directory and added a new
resource block to create an S3 bucket called
sonic media. You can see it here. Create an
additional resource

called upload sonic media to upload the files
listed in the variable called media to this bucket.
Use the following specification. Use the foreach
meta argument to upload all of the elements of
the media variable. Bucket, use the reference
expression to the bucket sonic media. Source
each element in the variable called media. Key
should be the name of the files being uploaded
minus the slash root. For example, Eggman.jpg,
Shadow.jpg, etc. And do not alter the variables.
When ready, run OpenTofu apply to create the
bucket and upload the items.
Okay, so this one's a little bit tricky, but nothing
we cant handle. So we have to approach this
carefully. Okay, so the first thing that we need to
do is we need to update the main.tf file
to upload the AWS S3 objects. So let's create a
resource of AWS S3 object. And let's call it as
instructed upload sonic media. Good. The bucket
needs to be a reference to the bucket above. So
we're going to go ahead and do AWS S3 bucket
dot sonic media dot ID. And again, this is what
we
call a reference expression. Good. Now we need
the key. Okay, so we're going to have to use the
substring function here. So we're going to do
substring. And what we're going to do is we're

going to take each value. And we're going to start
from the offset seven. Because if we take a look
at our media, we must not have the slash media
or slash root in the name. So our offset needs to
start from 01234567. So from the seventh
character, so we provide the seven, and then 20
which is the length number. Okay, now we need
the source to be each value as well. And finally,
our
for each would be var media. So we're using the
variable media, we're sub using substring to only
get the naming conventions and not the prefix.
And we're using the for each. So let's save that
up,
exit our console and do a TOFU apply. And the
apply does seem to be fine. If we check out the
key, it only contains the name and extension. And
the source still has the slash media, which is
good. So let's confirm with Yes. And after that
applies, we need to check our tasks to see if
everything is good. Okay, everything seems to be
in order. Let's move on to the next question. For
the final question, navigate to the configuration
directory called

slash root slash OpenTofu projects slash project
Mario. We have an empty main.tf file here. The
provider and variables have already been
configured for you.
Okay, let's see what's in store for us in the
second question. What is the value of the
variable called small? So let's check the variable
and it is t2.nano. Okay.
What is the current value for the variable called
name? So for the variable called name, we do
not have any default value. So we're going to
select undefined.
Okay, create an EC2 instance with the resource
name Mario servers. Use the following
specifications. AMI, use the variable called AMI.
Tags, create a tag with key name and value set
to the variable
called name. Instance type, use the conditional
expression so that if the instance is created with
a tag name equal to tiny, it should use the
variable called small else the variable called
large. We will supply the variable called name
using the dash var command line flag. Okay, so
let's get started. We need to open up the main.tf
and slowly start building our

resource. So resource is going to be again, an
AWS EC2 Instance. And the name is as
instructed going to be Mario servers.
The AMI is going to be var AMI, which has been
supplied for us. And the tags will have to be
name equal var name, which has also been
supplied for us. Okay, now for the tricky part, we
need a conditional expression so that if the
instance is created with a tag name of tiny,
it should use the variable called small else the
variable called large. So our instance type needs
to be based on the var name. If it's equal to tiny,
then we need to use the var small else var large.
And that's it. That's a conditional expression in
OpenTofu. So our instance type equals to if the
var name is set to tiny, then it will use the
var small. But if it's set to anything else, it will use
the var large. So we can save this up
now. And let's go ahead and test this out. So let's
go and do change of directory to project Mario.
Let's do a tofu init. And now tofu plan providing
the dash var flag of
name equal tiny. And we can see here that the
instance type in this case is T2 nano. And if we
check our

variables, that is when we use the small variable.
So let's now run the plan again. But this time, this
time, let's have to open up a new terminal. Okay,
now let's try to do the same configuration.
But this time, we're going to supply something
else to the instance type. So we're going to use
the dash bar again, but the name is going to be
big. And let's wait for the plan. And if our
conditional expression is right, we should see
that the instance is t2.micro. So let's wait for the
execution plan. And our instance type is
t2.2xlarge,
which is great. Exactly how we want it. So let's
go ahead and check the configuration now. And it
seems that our configuration is correct.
That's it for this lab. Thank you for watching, and
I'll see you in the next one.
```

```
resource "aws_iam_policy" "adminUser" {
  name   = "AdminUsers"
  policy = file("admin-policy.json")
}

resource "local_file" "pet" {
  filename = var.filename
  count    = length(var.filename)
}
```

```
resource "local_file" "pet" {
  filename = var.filename
  for_each = toset(var.region)
}

variable "region" {
  type = list
  default = [ "us-east-1",
    "us-east-1",
    "ca-central-1"
  ]
  description = "A list of AWS Regions"
}
```

```
$ tofu console
>file("/root/opentofu-projects/main.tf")
<<EOT
resource "aws_instance" "development" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
}
EOT
> length(var.region)
3
> toset(var.region)
toset([
  "ca-central-1",
  "us-east-1",
])
```

###### Numeric Functions
```
variable "num" {
  type        = set(number)
  default     = [ 250, 10, 11, 5 ]
  description = "A set of numbers"
}
```

```
$ tofu console
> max (-1, 2, -10, 200, -250)
200
> min (-1, 2, -10, 200, -250)
-250
> max(var.num...)
250
> ceil(10.1)
11
> ceil(10.9)
11
> floor(10.1)
10
> floor(10.9)
10
```
###### String Functions

```
variable "ami" {
  type        = string
  default     = "ami-xyz,AMI-ABC,ami-efg"
  description = "A string containing ami ids"
}
```

```
$ tofu console
> split(",", "ami-xyz,AMI-ABC,ami-efg")
[ "ami-xyz", "AMI-ABC", "ami-efg" ]
> split(",", var.ami)
[ "ami-xyz", "AMI-ABC", "ami-efg" ]
> lower(var.ami)
ami-xyz,ami-abc,ami-efg
> upper(var.ami)
AMI-XYZ,AMI-ABC,AMI-EFG
> title(var.ami)
Ami-Xyz,AMI-ABC,Ami-Efg
> substr(var.ami, 0, 7)
ami-xyz
> substr(var.ami, 8, 7)
AMI-ABC
> substr(var.ami, 16, 7)
ami-efg

> join(",",["ami-xyz","AMI-ABC","ami-efg"])
ami-xyz,AMI-ABC,ami-efg

> join(",", var.ami)

```


###### Collection Functions

```
variable "ami" {
  type        = list
  default     = ["ami-xyz", "AMI-ABC", "ami-efg"]
  description = "A list of numbers"
}
```

```
$ tofu console
> length(var.ami)
3
> index(var.ami, "AMI-ABC")
1
> element(var.ami, 2)
ami-efg
> contains(var.ami, "AMI-ABC")
true
> contains(var.ami, "AMI-XYZ")
false
```

###### Map Functions

**variables.tf**
```hcl
variable "ami" {
  type = map
  default = { "us-east-1"    = "ami-xyz",
    "ca-central-1" = "ami-efg",
    "ap-south-1"   = "ami-ABC"
  }
  description = "A map of AMI ID's for specific regions"
}
```

**Output / Tofu Console**
```
$ tofu console
> keys(var.ami)
tolist([
  "ap-south-1",
  "ca-central-1",
  "us-east-1",
])

> values(var.ami)
tolist([
  "ami-ABC",
  "ami-efg",
  "ami-xyz",
])

> lookup(var.ami, "ca-central-1")
ami-efg

> lookup(var.ami, "us-west-2")
Error: Error in function call

on <console-input> line 1:
(source code not available)

Call to function "lookup" failed: lookup failed to find key 'us-west-2'.

> lookup (var.ami, "us-west-2", "ami-pqr")
ami-pqr
```

Numeric Operators
```
$ tofu console
> 8 + 8
16

> 8 - 8
0

> 2 * 2
4

> 8 / 2
4

```
Equality Operators
```
$ tofu console
> 8 == 8
true

> 8 - 7
false

> 8 != "8"
false
```

Comparison Operators
```
$ tofu console
> 8 > 9
false

> 8 - 7
true

> 8 > 8
false

> 8 >= 8
true

```

Logical Operators
```
$ tofu console

> 8 > 7 && 8 < 10
true
> 8 > 10 && 8 < 10
false
> 8 > 10 || 8<10
true

> var.special
true
> ! var.special
false

> ! (var.b > 30)
true
```
variable.tf
```
variable special {
 type = bool
 default = true
 description = "Set to true to use special characters"
 }
 
variable b {
 type = number
 default = 25
}
```

```
variable a {
 type = number
 default = 50
}
variable b {
 type = number 
 default = 25
}
```

```
$ tofu console

> var.a > var.b
true
> var.a < var.b
false
> var.a + var.b
75
```
main.tf
```hcl
resource "random_password" "password-generator" {
  length = var.length < 8 ? 8 : var.length
}

output "password" {
  value     = random_password.password-generator.result
  sensitive = true
}
```
variable.tf
```hcl
variable "length" {
  type        = number
  description = "The length of the password"
}
```
output
```text
$ tofu apply -var=length=5 -auto-approve
random_password.password-generator: Creating...
random_password.password-generator: Creation complete after 0s [id=none]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

password = <sensitive>
```

```bash
$ if [ $length -lt 8 ]; then
  length=8;
  echo $length;
else
  echo $length;
fi
# Generate Password
```



```
resource "aws_iam_user" "cloud" {
     name = split(":",var.cloud_users)[count.index]
     count = length(split(":",var.cloud_users))
}

resource "aws_s3_bucket" "sonic_media" {
     bucket = var.bucket
}

resource "aws_s3_object" "upload_sonic-media" {
    bucket = aws_s3_bucket.sonic_media.id
    key = substr(each.value, 7, 20)
    source = each.value
    for_each = var.media
}
```

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.38.0"
    }
  }
}

  

provider "aws" {
  region                      = var.region
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  s3_use_path_style = true
  endpoints {
    ec2 = "http://aws:4566"
    iam = "http://aws:4566"
    s3 = "http://aws:4566"
  }
}
```

```
variable "region" {
  default = "ca-central-1"
}

variable "cloud_users" {
     type = string
     default = "andrew:ken:faraz:mutsumi:peter:steve:braja"
}

variable "bucket" {
  default = "sonic-media"
}

variable "media" {
  type = set(string)
  default = [
    "/media/tails.jpg",
    "/media/eggman.jpg",
    "/media/ultrasonic.jpg",
    "/media/knuckles.jpg",
    "/media/shadow.jpg",
      ]
}

variable "sf" {
  type = list
  default = [
    "ryu",
    "ken",
    "akuma",
    "seth",
    "zangief",
    "poison",
    "gen",
    "oni",
    "thawk",
    "fang",
    "rashid",
    "birdie",
    "sagat",
    "bison",
    "cammy",
    "chun-li",
    "balrog",
    "cody",
    "rolento",
    "ibuki"
  ]

}
```


```
tofu console 
> floor(01.9)
1
> floor(10.9)
10
> title("user-generated password file")
"User-Generated Password File"
> 

> q
╷
│ Error: Invalid reference
│ 
│   on <console-input> line 1:
│   (source code not available)
│ 
│ A reference to a resource type
│ must be followed by at least one

root in ~/opentofu-projects on ☁️  (us-east-1) took 8m9s 
❯ cd project-sonic/

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) 
❯ tofu init 

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "5.38.0"...
- Installing hashicorp/aws v5.38.0...
- Installed hashicorp/aws v5.38.0 (signed, key ID 0C0AF313E5FD9F80)

Providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://opentofu.org/docs/cli/plugins/signing/

OpenTofu has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that OpenTofu can guarantee to make the same selections by default when
you run "tofu init" in the future.

OpenTofu has been successfully initialized!

You may now begin working with OpenTofu. Try running "tofu plan" to see
any changes that are required for your infrastructure. All OpenTofu commands
should now work.

If you ever set or change modules or backend configuration for OpenTofu,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 6s 
❯ tofu plan 

OpenTofu used the selected providers
to generate the following execution
plan. Resource actions are indicated
with the following symbols:
  + create

OpenTofu will perform the following actions:

  # aws_iam_user.cloud[0] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "andrew"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[1] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "ken"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[2] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "faraz"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[3] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "mutsumi"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[4] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "peter"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[5] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "steve"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[6] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "braja"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

Plan: 7 to add, 0 to change, 0 to destroy.
╷
│ Warning: AWS account ID not found for provider
│ 
│   with provider["registry.opentofu.org/hashicorp/aws"],
│   on provider.tf line 10, in provider "aws":
│   10: provider "aws" {
│ 
│ See
│ https://registry.terraform.io/providers/hashicorp/aws/latest/docs#skip_requesting_account_id
│ for implications.
╵

────────────────────────────────────

Note: You didn't use the -out option
to save this plan, so OpenTofu can't
guarantee to take exactly these
actions if you run "tofu apply" now.

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 13s 
❯ tofu apply 

OpenTofu used the selected providers
to generate the following execution
plan. Resource actions are indicated
with the following symbols:
  + create

OpenTofu will perform the following actions:

  # aws_iam_user.cloud[0] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "andrew"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[1] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "ken"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[2] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "faraz"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[3] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "mutsumi"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[4] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "peter"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[5] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "steve"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

  # aws_iam_user.cloud[6] will be created
  + resource "aws_iam_user" "cloud" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "braja"
      + path          = "/"
      + tags_all      = (known after apply)
      + unique_id     = (known after apply)
    }

Plan: 7 to add, 0 to change, 0 to destroy.
╷
│ Warning: AWS account ID not found for provider
│ 
│   with provider["registry.opentofu.org/hashicorp/aws"],
│   on provider.tf line 10, in provider "aws":
│   10: provider "aws" {
│ 
│ See
│ https://registry.terraform.io/providers/hashicorp/aws/latest/docs#skip_requesting_account_id
│ for implications.
╵

Do you want to perform these actions?
  OpenTofu will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_iam_user.cloud[3]: Creating...
aws_iam_user.cloud[5]: Creating...
aws_iam_user.cloud[4]: Creating...
aws_iam_user.cloud[6]: Creating...
aws_iam_user.cloud[1]: Creating...
aws_iam_user.cloud[2]: Creating...
aws_iam_user.cloud[0]: Creating...
aws_iam_user.cloud[0]: Creation complete after 1s [id=andrew]
aws_iam_user.cloud[4]: Creation complete after 1s [id=peter]
aws_iam_user.cloud[3]: Creation complete after 1s [id=mutsumi]
aws_iam_user.cloud[1]: Creation complete after 1s [id=ken]
aws_iam_user.cloud[5]: Creation complete after 1s [id=steve]
aws_iam_user.cloud[2]: Creation complete after 1s [id=faraz]
aws_iam_user.cloud[6]: Creation complete after 1s [id=braja]
╷
│ Warning: AWS account ID not found for provider
│ 
│   with provider["registry.opentofu.org/hashicorp/aws"],
│   on provider.tf line 10, in provider "aws":
│   10: provider "aws" {
│ 
│ See
│ https://registry.terraform.io/providers/hashicorp/aws/latest/docs#skip_requesting_account_id
│ for implications.
╵

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 21s 
❯ tofu console 
> aws_iam_user.cloud[6]
{
  "arn" = "arn:aws:iam::000000000000:user/braja"
  "force_destroy" = false
  "id" = "braja"
  "name" = "braja"
  "path" = "/"
  "permissions_boundary" = ""
  "tags" = tomap(null) /* of string */
  "tags_all" = tomap({})
  "unique_id" = "jnbg6uzpm2qadh1wqa0z"
}
> index(var.sf, "oni")
7
> exit

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 17m4s 
❯ tofu init 

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v5.38.0

OpenTofu has been successfully initialized!

You may now begin working with OpenTofu. Try running "tofu plan" to see
any changes that are required for your infrastructure. All OpenTofu commands
should now work.

If you ever set or change modules or backend configuration for OpenTofu,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 3s 
❯ tofu apply 
aws_iam_user.cloud[1]: Refreshing state... [id=ken]
aws_iam_user.cloud[3]: Refreshing state... [id=mutsumi]
aws_iam_user.cloud[0]: Refreshing state... [id=andrew]
aws_iam_user.cloud[5]: Refreshing state... [id=steve]
aws_iam_user.cloud[6]: Refreshing state... [id=braja]
aws_iam_user.cloud[4]: Refreshing state... [id=peter]
aws_iam_user.cloud[2]: Refreshing state... [id=faraz]

OpenTofu used the selected providers
to generate the following execution
plan. Resource actions are indicated
with the following symbols:
  + create

OpenTofu will perform the following actions:

  # aws_s3_bucket.sonic_media will be created
  + resource "aws_s3_bucket" "sonic_media" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "sonic-media"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)
    }

  # aws_s3_object.upload_sonic-media["/media/eggman.jpg"] will be created
  + resource "aws_s3_object" "upload_sonic-media" {
      + acl                    = (known after apply)
      + bucket                 = (known after apply)
      + bucket_key_enabled     = (known after apply)
      + checksum_crc32         = (known after apply)
      + checksum_crc32c        = (known after apply)
      + checksum_sha1          = (known after apply)
      + checksum_sha256        = (known after apply)
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + force_destroy          = false
      + id                     = (known after apply)
      + key                    = "eggman.jpg"
      + kms_key_id             = (known after apply)
      + server_side_encryption = (known after apply)
      + source                 = "/media/eggman.jpg"
      + storage_class          = (known after apply)
      + tags_all               = (known after apply)
      + version_id             = (known after apply)
    }

  # aws_s3_object.upload_sonic-media["/media/knuckles.jpg"] will be created
  + resource "aws_s3_object" "upload_sonic-media" {
      + acl                    = (known after apply)
      + bucket                 = (known after apply)
      + bucket_key_enabled     = (known after apply)
      + checksum_crc32         = (known after apply)
      + checksum_crc32c        = (known after apply)
      + checksum_sha1          = (known after apply)
      + checksum_sha256        = (known after apply)
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + force_destroy          = false
      + id                     = (known after apply)
      + key                    = "knuckles.jpg"
      + kms_key_id             = (known after apply)
      + server_side_encryption = (known after apply)
      + source                 = "/media/knuckles.jpg"
      + storage_class          = (known after apply)
      + tags_all               = (known after apply)
      + version_id             = (known after apply)
    }

  # aws_s3_object.upload_sonic-media["/media/shadow.jpg"] will be created
  + resource "aws_s3_object" "upload_sonic-media" {
      + acl                    = (known after apply)
      + bucket                 = (known after apply)
      + bucket_key_enabled     = (known after apply)
      + checksum_crc32         = (known after apply)
      + checksum_crc32c        = (known after apply)
      + checksum_sha1          = (known after apply)
      + checksum_sha256        = (known after apply)
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + force_destroy          = false
      + id                     = (known after apply)
      + key                    = "shadow.jpg"
      + kms_key_id             = (known after apply)
      + server_side_encryption = (known after apply)
      + source                 = "/media/shadow.jpg"
      + storage_class          = (known after apply)
      + tags_all               = (known after apply)
      + version_id             = (known after apply)
    }

  # aws_s3_object.upload_sonic-media["/media/tails.jpg"] will be created
  + resource "aws_s3_object" "upload_sonic-media" {
      + acl                    = (known after apply)
      + bucket                 = (known after apply)
      + bucket_key_enabled     = (known after apply)
      + checksum_crc32         = (known after apply)
      + checksum_crc32c        = (known after apply)
      + checksum_sha1          = (known after apply)
      + checksum_sha256        = (known after apply)
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + force_destroy          = false
      + id                     = (known after apply)
      + key                    = "tails.jpg"
      + kms_key_id             = (known after apply)
      + server_side_encryption = (known after apply)
      + source                 = "/media/tails.jpg"
      + storage_class          = (known after apply)
      + tags_all               = (known after apply)
      + version_id             = (known after apply)
    }

  # aws_s3_object.upload_sonic-media["/media/ultrasonic.jpg"] will be created
  + resource "aws_s3_object" "upload_sonic-media" {
      + acl                    = (known after apply)
      + bucket                 = (known after apply)
      + bucket_key_enabled     = (known after apply)
      + checksum_crc32         = (known after apply)
      + checksum_crc32c        = (known after apply)
      + checksum_sha1          = (known after apply)
      + checksum_sha256        = (known after apply)
      + content_type           = (known after apply)
      + etag                   = (known after apply)
      + force_destroy          = false
      + id                     = (known after apply)
      + key                    = "ultrasonic.jpg"
      + kms_key_id             = (known after apply)
      + server_side_encryption = (known after apply)
      + source                 = "/media/ultrasonic.jpg"
      + storage_class          = (known after apply)
      + tags_all               = (known after apply)
      + version_id             = (known after apply)
    }

Plan: 6 to add, 0 to change, 0 to destroy.
╷
│ Warning: AWS account ID not found for provider
│ 
│   with provider["registry.opentofu.org/hashicorp/aws"],
│   on provider.tf line 10, in provider "aws":
│   10: provider "aws" {
│ 
│ See
│ https://registry.terraform.io/providers/hashicorp/aws/latest/docs#skip_requesting_account_id
│ for implications.
╵

Do you want to perform these actions?
  OpenTofu will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.sonic_media: Creating...
aws_s3_bucket.sonic_media: Creation complete after 0s [id=sonic-media]
aws_s3_object.upload_sonic-media["/media/eggman.jpg"]: Creating...
aws_s3_object.upload_sonic-media["/media/shadow.jpg"]: Creating...
aws_s3_object.upload_sonic-media["/media/tails.jpg"]: Creating...
aws_s3_object.upload_sonic-media["/media/ultrasonic.jpg"]: Creating...
aws_s3_object.upload_sonic-media["/media/knuckles.jpg"]: Creating...
aws_s3_object.upload_sonic-media["/media/tails.jpg"]: Creation complete after 0s [id=tails.jpg]
aws_s3_object.upload_sonic-media["/media/ultrasonic.jpg"]: Creation complete after 0s [id=ultrasonic.jpg]
aws_s3_object.upload_sonic-media["/media/shadow.jpg"]: Creation complete after 1s [id=shadow.jpg]
aws_s3_object.upload_sonic-media["/media/knuckles.jpg"]: Creation complete after 0s [id=knuckles.jpg]
aws_s3_object.upload_sonic-media["/media/eggman.jpg"]: Creation complete after 1s [id=eggman.jpg]
╷
│ Warning: AWS account ID not found for provider
│ 
│   with provider["registry.opentofu.org/hashicorp/aws"],
│   on provider.tf line 10, in provider "aws":
│   10: provider "aws" {
│ 
│ See
│ https://registry.terraform.io/providers/hashicorp/aws/latest/docs#skip_requesting_account_id
│ for implications.
╵

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.

root in ~/opentofu-projects/project-sonic via 💠 default on ☁️  (us-east-1) took 22s 
❯ cd ~/opentofu-projects/project-mario/

root in ~/opentofu-projects/project-mario via 💠 default on ☁️  (us-east-1) 
❯ tofu init 

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "5.38.0"...
- Installing hashicorp/aws v5.38.0...
- Installed hashicorp/aws v5.38.0 (signed, key ID 0C0AF313E5FD9F80)

Providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://opentofu.org/docs/cli/plugins/signing/

OpenTofu has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that OpenTofu can guarantee to make the same selections by default when
you run "tofu init" in the future.

OpenTofu has been successfully initialized!

You may now begin working with OpenTofu. Try running "tofu plan" to see
any changes that are required for your infrastructure. All OpenTofu commands
should now work.

If you ever set or change modules or backend configuration for OpenTofu,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

root in ~/opentofu-projects/project-mario via 💠 default on ☁️  (us-east-1) took 5s 
❯  tofu paln -var name="tiny"
OpenTofu has no command named "paln". Did you mean "plan"?

To see all of OpenTofu's top-level commands, run:
  tofu -help


root in ~/opentofu-projects/project-mario via 💠 default on ☁️  (us-east-1) 
❯ tofu paln -var name="big"
OpenTofu has no command named "paln". Did you mean "plan"?

To see all of OpenTofu's top-level commands, run:
  tofu -help
```

### Dynamic Blocks and Splat Expressions
```
and splat expressions. So far, we have seen
ways to loop through a list or set or a map type
data structure
using arguments such as count and for each.
These are handy when we need to create
multiple instances using the same resource
block.
In this example, we have a variable called
backendServers which is a list and using count,
we can create two instances, serverl and
server2 using the resource called backend.
Now, let's consider another scenario. A new
configuration to create a new VPC in AWS. and a
security group that will allow inbound traffic to
specific ports in this VPC.
VPC or Virtual Private Cloud in AWS or any cloud
provider for that matter is just a virtual network in
which we can create resources such as EC2
instances.
We will use a CIDR of 10.0.0.0/16 for this VPC.
Inside the VPC, we can have additional logical
partition called subnets.
If the resource is public facing, we can use a
public subnet. In our example, let's use a private
subnet that does not have a public IP address

and no direct connectivity to the Intemet. A
security group acts as a virtual firewall for the
VPC and in our scenario, lets us allow inbound
access to port 22 and 8080.
Now, let's use OpenTofu to create these
resources. Let's start with the VPC. To do this,
we create a resource block like this.
We have specified the CIDR block with the IP
range and also added a tag with name equal to
backend VPC.
Next, let's add the resource block for the private
subnet. Here, for the VPC ID, we can make use
of the reference expression
aws_vpc.backend_vpc.id and we have also
provided a CIDR block IP range of 10.0.2.0/24 for I
the subnet and a tag of name equals private
subnet.
Let's set up the required ingress rules to the VPC
using the AWS security group resource. Let's call
it backend-sg.
Again, we have used the reference expression as
the value for the VPC ID argument. Next, it's time
to define the ingress rules.

For this example, we decided to allow port 22
and port 8080 into the VPC. To do this, we need
to add a block called ingress inside the resource
block
for the security group, like this. Here, we have
opted to allow incoming traffic from anywhere to
port 22 within this VPC.
Now, to allow port 8080, we need to add another
ingress block. The only difference between the
two ingress blocks is the port 22 for the first
and 8080 for the second one. And, since these
two ingress blocks are already inside the
resource block for the AWS security group
resource, they can be referred to as nested
blocks.
Now, if we want to add more ingress rules, then
we would have to add even more nested blocks,
which can make our configuration unnecessarily
long and complex to read.
To simplify this configuration, we can make use
of dynamic blocks. To do this, first, let's remove
the two ingress blocks we added
and create a new list type variable for all the
ports that we want to allow within the security
group.

Let's call this variable ingress_ports. Next, we
can create a dynamic block inside the resource
block. The dynamic block begins with the
keyword dynamic followed by the name of the
block.
Since we are creating this block for the nested
ingress block, we have to call it ingress. Now, to
loop through our variable called ingressPorts, we
use a forEach metaargument, like this.
Next, we add all the repeatable content to be
created for each ingress block inside a content
block, like this.
Observe that, for the fromPort and toPort
argument, we have used the value as
ingress.value. Here, ingress refers to the name ofl
the dynamic block we're using.
The values for the from and toPort will be fetched
from the ingressPorts variable during each
iteration of the forEach loop.
And that's it. If we run OpenTofu apply now, the
three resources, VPC, subnet, and the security
group, will be created.
Remember that we used ingress, which is the
name of the dynamic block, to loop through the
list of ingress ports to be created.

Instead of using the name of the dynamic block,
we can use another name by making use of an
iterator, like this.
Here, we have used the value of the iterator as
port. Once it is defined, we can replace the name
ingress with port to loop through the list.
Finally, let's take a look at splat expressions
using the same example. To do this, let's add an
output variable called toPorts that will display all
the ports for the ingress rules defined within our
security group.
To do this, let's add an output variable called
toPorts that will display all the toPorts for the
ingress rules defined within our security group.
Now, since the ingress rules are created as a list,
to iterate over all the elements in the list, we can
make use of a special symbol, like this.
The reference expression we have used here is
called a splat expression. In this example, using
the splat expression, we are fetching the toPort
attribute for every ingress rule within our security
group.
And since we have two ingress rules created by
the dynamic block, a Tofu output command will
show a result like this.
Well, that's it for this lecture. I'll see you in the
next one.
```

```
resource "aws_vpc" "backend-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "backend-vpc"
  }
}

resource "aws_subnet" "private-subnet" {
  vpc_id     = aws_vpc.backend-vpc.id
  cidr_block = "10.0.2.0/24"

  tags = {
    Name = "private-subnet"
  }
}

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


to 

```
varable "ingress_ports"{
 type = list
 default = [22,8080]
}

output "to_ports"{
 value = aws_secuirty_group.backend-sg.ingress[*].to_port
}
```

```
 dynamic "ingress" {
  for_each = var.ingress_ports
  content{
    from_port   = ingress.value
    to_port     = ingress.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    }
  }

or

 dynamic "ingress" {
  iterator = port
  for_each = var.ingress_ports
  content{
    from_port   = port.value
    to_port     = port.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    }
  }
```

```
tofu output
to_ports = [
 22, 
 8080,
]
```


  
### OpenTofu Workspaces


```
tofu workspace list
* defult
tofu workspace new production

tofu workspace new development

tofu workspace list

* defult
  production
  development



tofu workspace select production

$ Is
main.tf provider.tf terraform.tfstate.d variables.tf

$ tree terraform.tfstate.d/
terraform.tfstate.d/

 development
   terraform.tfstate
 production
    terraform. tfstate

```

Here is the content extracted from the images:

**Output (Tofu Console)**
```text
$ tofu console
> terraform.workspace
development
> lookup(var.instance_type, terraform.workspace)
t2.micro
```

**main.tf**
```hcl
resource "aws_instance" "webserver" {
  ami           = var.ami
  instance_type = lookup(var.instance_type, terraform.workspace)
  tags = {
    Environment = "Development"
  }
}
```

**Output (Tofu Console)**
```text
$ tofu workspace select production
$ tofu console
> terraform.workspace
production
> lookup(var.instance_type, terraform.workspace)
m5.large
```

**variables.tf**
```hcl
variable "ami" {
  default = "ami-24e140119877avm"
}

variable "region" {
  default = "ca-central-1"
}

variable "instance_type" {
  type = map
  default = {
    "development" = "t2.micro"
    "production"  = "m5.large"
  }
}
```



```
In this lecture, we will learn about workspaces in
OpenTofu and how to use them. We know by
now that every OpenTofu configuration uses a
state file to provision and manage infrastructure.
Whether the state file is stored locally or
remotely, all the examples we have seen so far
have a one-to-one mapping between the
configuration directory and the state file.
Here is an example of an AWS EC2 instance
resource block called webserver. Let's assume
that this instance is to be created for a
development project and add the tag of
environment equals development.
And we have used an AMI and an instance type
that suits the need for this development web
server. Now, if we want to make use of the same
configuration files to create a similar resource for,
say, a production environment, we can make use
of workspaces.
With workspaces, we can use the same
configuration directory to create multiple
infrastructure environments such as the
development and the production environments.
A workspace called default is automatically
created within every configuration directory. And
this can be inspected by running the OpenTofu
workspace list command like this.

To create a new workspace, we can make use of
the new subcommand followed by the name of
the workspace.
Here we have created two new workspaces, one
for production that will be used for the production
environment and another called development for
the development environment.
Keep a note that once a new workspace is
created by this command, OpenTofu immediately
switches to it as well.
The asterisk before development in the OpenTofu
workspace list command indicates that we are
currently in the development namespace. Let's
now see how to use the workspaces we just
created within our configuration to provision
resources.
The configuration directory we are using here is
//root//opentofu/projects//webserver, which
contains the development and the production
workspaces. Both the production and dev
environments are to have two EC2 instances in
the ca-central-l region with the same AMI.
However, let's assume that for production we
require a more powerful machine. So, let's use
the instance type of t2.micro for development and
an m5.large instance type for production.

The existing main.tf and variable.tf files were
written with the development environment in
mind so the current version looks like this.
To make use of two different instance types for
each environment, first let's change the variable
called instance_type to a map, like this.
The map has two key-value pairs. The first key is
called development for the dev environment with
a value of t2.micro.
The second one is production with a value of
m5.large. Now, to make use of the corresponding
AMI based on the workspace we are in, we can
make use of the lookup function, like this.
For the lookup function, we are using the
expression terraform.workspace to return a value
from the variable called AMI.
Terraform.workspace function retums the value
of the workspace we are currently in.
This can be tested using the OpenTofu.console
command. So, if we are in the development
workspace, this expression returns development
and the lookup function returns t2.micro.
We can switch the workspace to production using
the OpenTofu.workspace.select command, like
this. Now, the same lookup function will return
the value of m5.large.

And finally, let's update the value for the tag with
key called environment to also use the
expression of terraform.workspace.
And finally, let's update the value for the tag with
key called environment to also use the
expression terraform.workspace. This will make
sure that the value is set based on the
workspace we are in.
If we run OpenTofu apply now within the
development workspace, we can see that the
execution plan uses the type t2.micro and the
correct value for the tag.
And to create the resources for the production
environment, all we have to do is switch to the
production workspace.
And once we run OpenTofu apply, we can see
that the instance type of m5.large is used and for
the tags environment is set to the value of
production.
When using workspaces and local state, instead
of using the terraform.tfstate file in the
configuration directory, OpenTofu stores the state
files in a separate directory called
terraform.tfstate.d.

And inside this directory, we can see a directory
by the name of each workspace for which we
have completed at least one OpenTofu apply
command.
Within this directory, there would be a
terraform.tfstate.d directory. Within this directory,
there would be a terraform.tfstate file associated
with the resources created for that specific
workspace.
Well, that's it for this lecture. I will see you in the
next one.


lab

Hello, and welcome to the lab OpenTofu
workspaces. So first off, when we start off and
create a configuration in OpenTofu,
what is the workspace that is created to begin
with? So we already saw this in the previous
lectures, but let's revisit the topic once again.
So let's go into our project Sapphires here. And if
we go and type in OpenTofu workspace list, we
can see that it's using a default workspace, which
is always on by default.
So the correct answer is default. Navigate to the
Configuration Directory of project Sapphire. We
have a few configuration files already created
here.
How many workspaces are created for this
configuration currently? So as we can see, and
again, by running the workspace list,
we can see that only one workspace is created.
Create three new workspaces called US Payroll,
UK Payroll, and India Payroll.
So let's go ahead and do that. So OpenTofu
workspace new US Payroll. Then UK Payroll.
And lastly, India Payroll.

select, and then the name of the workspace,
which in this case is US Payroll. And if we check,
it seems to have selected the appropriate
workspace as seen here.
Where would the state file for the workspace
called India Payroll be stored? So when working
with the OpenTofu workspaces,
there is a dedicated folder called
terraform.tfstate.d. And each folder in that folder
will be named as per the workspace.
So in our case of India Payroll, it is going to be
stored in the terraform.tfstate.d slash India
Payroll. So this would be the correct answer.
Let's now write a main.tf file to make use of the
same module that we saw in the OpenTofu
modules lecture.
The project Sapphire configuration directory will
be used to deploy the same payroll application
stack in different regions. The module is located
at the path of /root/openTofu/projects/modules
[payload app. Okay, so let's get started. Inside
the configuration directory, we have already
added the variables.tf and the provider.tf files.
Inspect them. So in the variables, we have the
variable region and variable AMI with each
having their own default for workspace,

a provider, which has the AWS as the required
provider and the region based on the Terraform
workspace. So what type of variable is region?
So the region is a map variable as seen here.
What is the default value of the key called India
Payroll for the variable region?
So let's take a look at the variable region. Take a
look at the India Payroll. And notice that AP-
South-I is the default region for that workspace.
What is the default value of the key called India
Payroll for the variable AMI? So the AMI starts
with 551 and ends with AVM.
So 551 AVM is the right answer. Now update the
main.tf of the root module to call the child module
located at /root/OpenTofu project/modules/payroll
app. Adhere to the following specifications.
Module name needs to be payroll app. This
module expects two mandatory arguments,
the app region, which uses the values for a
variable called region. AMI, which uses the
values from the variable called AMI.
The values for these two arguments should be
selected based on the workspace you're in. For
example, if on US payroll workspace,

the app region should be US-East-l and the AMI-
24E or for UK payroll, the app region should be
EU-West-2 and the AMI-35E, etc.
Once ready, run Tofu init and you don't have to
create, that is, apply resources yet. Refer to the
README file located at /root/OpenTofu project
[modules/payroll app to see how to use the
module. Okay, so right off the bat, we can look at
the README
and we see how the module is supposed to be
used. So that gives us some guidance right off
the bat.
So let's go ahead and take this and copy it over
to our root module. And what we need to add
here is we need to have a lookup
function that's going to look up the var region,
which we have already set, and base it off of the
Terraform workspace.
Now, the same kind of config goes for the AMI,
but we're going to be looking at the var AMI and
again,
basing it off of the Terraform workspace we're in.
So we're going to set that up and we can run
OpenTofu init.

And that went through, so we can go ahead and
check our configuration to see if everything is set
up properly.
Okay, great. Let's move over to the last question.
Now, using the same configuration, create the
resource on all three workspaces that you
created earlier.
Okay, good. Since we're already on the US
payroll workspace, we can go ahead and do an
OpenTofu apply right away to this workspace.
And once we're prompted, we're going to type in
yes into the prompt to apply the resources in the
US payroll workspace.
Okay, now let's switch our workspace to the I-JK
payroll. Okay. And do the same command again,
OpenTofu apply. Type yes in the prompt.
And now we need to select our last workspace,
the India payroll, and do one last OpenTofu
apply, type yes in the prompt.
And now we have three environments, three
different workspaces created using one single
configuration. Let's check the validity of our lab tol
see if everything is set up properly.
Okay, great. The validity of our lab to see if
everything is set up properly. And it seems
everything is set up as it should be.
That's it for this lab. Thank you for watching and
I'll see you in the next one.
```

lab

```
  

module "payroll_app" {

  source = "/root/opentofu-projects/modules/payroll-app"

  # Enter the following required arguments

  app_region = lookup(var.region, terraform.workspace)

  ami        = lookup(var.ami, terraform.workspace)

}
```

```
terraform {

  required_providers {

    aws = {

      source = "hashicorp/aws"

      version = "5.38.0"

    }

  }

}

  
  

provider "aws" {

  region                      = lookup(var.region, terraform.workspace)

  skip_credentials_validation = true

  skip_requesting_account_id  = true

  s3_use_path_style = true

  endpoints {

    ec2 = "http://aws:4566"

    dynamodb = "http://aws:4566"

    s3 = "http://aws:4566"

  }

}
```

```
variable "region" {

    type = map

    default = {

        "us-payroll" = "us-east-1"

        "uk-payroll" = "eu-west-2"

        "india-payroll" = "ap-south-1"

    }

  

}

variable "ami" {

    type = map

    default = {

        "us-payroll" = "ami-24e140119877avm"

        "uk-payroll" = "ami-35e140119877avm"

        "india-payroll" = "ami-55140119877avm"

    }

}
```

module/payroll-app
app-server.tf
```
resource "aws_instance" "app_server" {

  ami = var.ami

  instance_type =  "t2.medium"

  tags = {

    Name = "${var.app_region}-app-server"

  }

  depends_on = [ aws_dynamodb_table.payroll_db,

                 aws_s3_bucket.payroll_data

               ]

  

}
```

dynamodb.tf
```
resource "aws_dynamodb_table" "payroll_db" {

  name           = "${var.app_region}_user_data"

  billing_mode   = "PAY_PER_REQUEST"

  hash_key       = "EmployeeID"

  

  attribute {

    name = "EmployeeID"

    type = "N"

  }

  attribute {

    name = "Department"

    type = "S"

  }

  attribute {

    name = "age"

    type = "N"

  }

  attribute {

    name = "Address"

    type = "S"

  }

  global_secondary_index {

    name             = "EmployeeID"

    hash_key         = "EmployeeID"

    projection_type    = "ALL"

  }

  global_secondary_index {

    name             = "age"

    hash_key         = "age"

    projection_type    = "ALL"

  }

  global_secondary_index {

    name             = "Department"

    hash_key         = "Department"

    projection_type    = "ALL"

  }

   global_secondary_index {

    name             = "Address"

    hash_key         = "Address"

    projection_type    = "ALL"

  }

}
```

3-bucket.tf
```
resource "aws_s3_bucket" "payroll_data" {

  bucket = "${var.app_region}-${var.bucket}"

}
```

variables.tf
```
variable "app_region" {

    default = "us-east-1"

}

variable "ami" {

    type = string

    default = "ami-24e140119877avm"

}

variable "bucket" {

    default = "flexit-payroll-alpha-22001c"

}
```


```
tofu workspace select india-payroll
 tofu apply -auto-approve

tofu workspace select us-payroll
 tofu apply -auto-approve

tofu workspace select uk-payroll
 tofu apply -auto-approve
```



### Migrating existing Terraform configuration to OpenTofu
```
compatible is 1.6 OpenTofu ,before mentioned As
workload any potentially SO ,1.6 Terraform with
1.6 version Terraform with running have you that
.ersion. OpenTofu to migrated be can
several in approached be should migration The
disaster a prepare to need you ,First .teps.
.afe. be to just ,plan recovery
changes all apply to is do to thing next the Then
then and OpenTofu installing After .erraform. with
.ile. state your up backing
run and OpenTofu initialize to need you Next
the Then .lan. the inspect then and Tofu-apply
.hange. small a out test to is do to thing last
any include not does change this that sure Be
that something ,infrastructure the of parts crucial
.ith. test to okay you're
migration the during issues any encounter you If
you help can that places are there ,OpenTofu to
.ISSLES. these with out
and Slack OpenTofu the join to is them of One
the on ask or ,chat via problems the discuss
discussion new a opening by discussions GitHub
.roup. appropriate the in topic

existing migrating of example our of case the ln
group appropriate the ,configuration Terraform
about talk Let's .ection. Q&A the be would
.penTofu. from Terraform to back migrating
bugs encounter you where cases be might There
to how seen we've and ,OpenTofu with issues or
let's now but ,lecture previous the in these cover
roll to needed steps the about talk
in and steps the at look Let's .erraform. to back
to back roll to them take to have we order which
Terraform
of backup another create Let's .penTofu. from
init Terraform run to need we Then .le. state your
directory working the in Terraform initialize to
.gain.
no that verify and plan Terraform run then We
we Then .lan. the in are changes unexpected
.hange.
to safe it's then ,succeeded rollback the If
.roject. your for again Terraform using continue
while bug a encounter you if happens what So
process the during even or OpenTofu using
might these While Terraform? to back rolling of
development the ,moment the at cases rare be
significant become might divergence

later down the road. So let's take a look at how to
handle these scenarios. If you encounter a bug
or any problem of any kind related to OpenTofu, it
should be
reported on the OpenTofu GitHub in the Issues
section. When opening a new issue, there are
guidelines to be followed, like if the issue is a bug
report or a feature request, an RFC standard, or
even a security vulnerability. It can be specific to
a version of OpenTofu that's under heavy
development at the moment,
like currently the version 1.7.0. Each of these
issue types is going to have a guideline on how
to file the issue properly.
In order to provide a detailed and proper issue,
be sure to follow the guidelines and be
descriptive about your problem.



lab


Hello and welcome to lab.OpenTofu migration
and rollback to Terraform. So our first question is
to navigate to the folder
slash root slash OpenTofu projects slash
migration. Inspect the main.tf file and initialize the
configuration directory and apply the local file
resource via Terraform.
So our main.tf just contains a local file resource
to create the filename terraform.txt with the
contents this file has been created with
Terraform.
So in order to apply that, we need to change our
directory to the migration folder, run Terraform init
to initialize the configuration,
and then Terraform apply to create resources.
Confirm yes in the prompt, and we see that our
resource was created.
So let's go ahead and check that, and everything
seems to be in order. Let's move on. What file
has been created by the local file resource?
We can see that it was the terraform.txt, so that's
what we select. Let's now proceed with migrating
our Terraform configuration to OpenTofu.
You can refer to the migration link in this page to
view a detailed description. As a first step, install
OpenTofu in your system.

Follow the documentation link at the top right of
this page, titled OpenTofu installation guide. So
let's do that first.
Let's open up the OpenTofu installation guide,
and in order to know what installation we need to
pick, we first need to figure out what distro we
are running.
So let's go ahead and run cat /etc/os-release,
and we can see here that we're running Ubuntu
22. What we can do now is we can go into our
installation,
and you have the option of using Snap for Linux,
but here we can just use the Debian installation
script.
So we can go ahead and copy that over, and we
can go back to our VSCode and create a new
file,
install.sh, and let's add a shebang here, for bash,
and paste in our script, save that up, run chmod
plus x install.sh, and then run the installation.
And after a while, it should download and install
the OpenTofu for us, and we can verify with the
tofu CLI.
Now that is complete. Okay, so the next part
says, after the OpenTofu installation is complete,
So the next part says,

after the OpenTofu installation is complete,
archive and compress the Terraform state file
created inside the slash OpenTofu project slash
migration folder using tar.
So we need to backup our Terraform TF state
before the migration to OpenTofu just to be safe.
So what we need to do is we need to run the tar
command
the parameter is create, zip, and file, and let's
just copy over the name here so we don't make a
typo,
and in case the copy and paste is not working for
you in the terminal, just make sure that you type
in
the right naming convention, so tfstate.tar.gz. And
we select our file, which is terraform.tfstate. And
let's make sure that it got created.
It did, so let's check our question. And that
seems to be in order. Let's move on. Okay, next
question says,
which command will you use to determine the
version of OpenTofu that has been installed? So
if you know the Terraform command,

it is the same for OpenTofu, just using the Tofu
CLI. So it would be the OpenTofu version
command, which would output the version of
OpenTofu,
which you currently have installed. Okay, next
question says, navigate to /root/OpenTofu project
[migration and initialize the configuration
directory with the OpenTofu workflow.
So we're already in that directory. So what we
can do is we can just go ahead and run
OpenTofu init and wait for OpenTofu to initialize
and then check if our configuration is okay. All
right, let's move on. Inspect the OpenTofu plan
output. What resources will be created?
So let's run the OpenTofu plan. And we can see
that the local file resource will be created. And
with the file name terraform.txt.
So let's select terraform.txt because this is our
current configuration. Let's now amend our
main.tf file. Change the file name and contents
of the local file resource. The file name should be
OpenTofu.txt and the content should be this file
has been created with OpenTofu.
So let's go ahead and copy that over right away
and paste it here and then rename our file name
argument

to OpenTofu.txt. And let's save that up. And let's
check if our configuration is okay. Good, let's
move on. Inspect the output of the OpenTofu plan
command now.
So let's go ahead and run the OpenTofu plan. We
can see already that the file name is opentofu.txt
and the content is changed as well.
What resource will be created? So now we know
that OpenTofu.txt will be created. Apply the
changes made to the main.tf file
using the OpenTofu configuration. So now we
know that we need to run the OpenTofu apply,
confirm yes in the prompt and apply.
And we can see that the OpenTofu.txt got
created and let's check and everything seems to
be okay. Moving on.
Now let's look at the reverse scenario of
migrating back from OpenTofu to Terraform.
Terraform is already installed for this lab
so we won't be installing it again. However, the
following documentation can serve as a
reference when installation is actually needed.
Good. Okay, so let's look at what we need to do
next. As a next step, we will back up the state file
again.

Archive and compress the OpenTofu state file
created inside the /OpenTofu project /migration
folder using tar. Name the resultant file as
terraform.tfstate.tar.gz.
Okay. So let's go ahead and open up our terminal
again. And what we need to do is again to
compress the current terraform.tfstate.
So let's go ahead and run the tar command with
create zip file flags and then the name of the
archive is terraform.tfstate.tar.gz.
And that is the same name that we're archiving.
So let's go ahead and create that. You can see
that indeed got created here and here.
So let's check. And everything seems to be in
order. Now initialize the configuration directory
using the Terraform initialization command.
So now instead of typing OpenTofu in it, you type
in Terraform in it again. To have Terraform
initialize the directory,
check everything. Everything seems to be okay.
Now revert the changes made earlier to the local
file resource. Change the file name to
terraform.txt.
Change that for right now. And the content of this
file has been created with Terraform. And save
that up.

And let's check if everything is okay. Good. Now
let's run Terraform apply to revert our changes
and start using Terraform again.
We can see that the file indeed got created and
the old OpenTofu.txt got deleted. So let's check.
Everything seems to be okay.
Okay. That was it for the migration to OpenTofu
and reverting back to Terraform. Thank you for
watching and I'll see you in the next one.
```