> 多云基础设施编排工具，不像 CloudFormation 那样绑定 AWS 平台，Terraform 可以同时编排各种云平台或者其他基础设施的资源。Terraform 实现多云编排的方法是 Provied 插件机制。

## 基础概念
### Provied 机制
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1700546959103-423bb1dd-a1bb-44d1-87fa-bd619ff5db57.png#averageHue=%23f1f0f0&clientId=uaea78b46-c740-4&from=paste&height=403&id=u38e2dd2e&originHeight=806&originWidth=1638&originalType=binary&ratio=2&rotation=0&showTitle=false&size=141374&status=done&style=none&taskId=ua19aa9ec-4748-4f69-b8ab-81dcebb54a2&title=&width=819)
Terraform使用的是HashiCorp自研的go-plugin库(https://github.com/hashicorp/go-plugin)，本质上各个Provider插件都是独立的进程，与Terraform进程之间通过rpc进行调用。Terraform引擎首先读取并分析用户编写的Terraform代码，形成一个由data与resource组成的图(Graph)，再通过rpc调用这些data与resource所对应的Provider插件；Provider插件的编写者根据Terraform所制定的插件框架来定义各种data和resource，并实现相应的CRUD方法；在实现这些CRUD方法时，可以调用目标平台提供的SDK，或是直接通过调用Http(s) API来操作目标平台。
#### 下载 provider 
` terraform init`的时候会分析代码中使用到的 Provider ，并尝试下载 Provider 插件到本地。会下载到 .terraform 文件夹中。
可以使用两种方式启用插件缓存：

- 配置 ` TF_PLUGIN_CACHE_DIR `环境变量
```shell
export TF_PLUGIN_CACHE_DIR="$HOME/.terraform.d/plugin-cache"
```

- 使用 CLI 配置文件。
```shell
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
```
弃用插件缓存之后，每次执行 `terraform init`的时候会优先从设置的缓存文件夹中找，没有的话优先下载到缓存目录，系统允许的话使用符号连接符，地址链接过去，否则copy 过来。 缓存文件夹的插件不会主动删除。
#### provider 搜索
[https://registry.terraform.io/](https://registry.terraform.io/)
或者直接对应的云文档中所有 terraform

#### provider 的声明
一组 Terraform 代码要被执行，相关的 Provider 必须在代码中被声明。不少的 Provider 在声明时需要传入一些关键信息才能被使用。如 region （区域）
> 下述华为云 配置 

```shell
# 华为云  ak sk region 资源配置 安全起见 可以 如环境变量 即以 $
provider "huaweicloudbj" {
  region     = "cn-north-1"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}

provider "huaweicloudxa" {
   region     = "cn-north-2"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}

# source 申明插件的原地址。 直接provider 搜索会有，唯一值。
terraform {
  required_providers {
    huaweicloudbj = { # 通过对应这个参数，可以实现单个云 多地区，多用户申明 也可以用 alias 别名
      source = "huaweicloud/huaweicloud"
      version = ">= 1.20.0"  #约束该版本 terraform 的插件版本要求 如果找不到，不执行下述操作
    }
  }
}
```

#### 内建 provider
绝大多数Provider是以插件形式单独分发的，但是目前有一个Provider是内建于Terraform主进程中的，那就是terraform_remote_state data source。该Provider由于是内建的，所以使用时不需要在terraform中声明required_providers。这个内建Provider的源地址是terraform.io/builtin/terraform

### 状态管理
>  `terraform apply`执行成功了一次，创建了期望的基础设施之后，再次执行 `terraform apply`，生成的新的执行计划将不会包含任何变更。`Terraform`会记住当前基础设施的状态，并将之与代码所描述的期望的状态进行对比。第二次 apply 时，因为当前状态已经与代码描述的状态一直了，所以会生成一个空的执行计划（重复执行问题的解决，创建 香港服务器的时候，可以多执行，避免创建失败）

#### 状态文件 
> 状态管理 是Terraform 中引入的一个独特的概念。简单说，Terraform 将每次执行基础设施变更操作的状态信息保存在一个状态文件中，默认情况下会保存在当前工作目录下的 `terraform.tfstate`文件里。

声明的 date 和一个 resource 
```shell
# Create a VPC
resource "huaweicloud_vpc" "example" {
  name = "terraform_vpc"
  cidr = "192.168.0.0/16"
}
```
   使用 `terraform apply`后，生成的 `terraform.tfstate`文件内容
```shell
{
  "version": 4,
  "terraform_version": "1.6.4",
  "serial": 1,
  "lineage": "4e8840ba-c94b-3f6e-3a2c-3ff022e4de49",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "huaweicloud_vpc",
      "name": "example",
      "provider": "provider[\"registry.terraform.io/huaweicloud/huaweicloud\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "cidr": "192.168.0.0/16",
            "description": "",
            "enterprise_project_id": "0",
            "id": "7194529f-0062-4949-8392-2a0dd4a987e0",
            "name": "terraform_vpc",
            "region": "cn-north-4",
            "routes": [],
            "secondary_cidr": null,
            "status": "OK",
            "tags": null,
            "timeouts": null
          },
          "sensitive_attributes": [],
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjo2MDAwMDAwMDAwMDAsImRlbGV0ZSI6MTgwMDAwMDAwMDAwfX0="
        }
      ]
    }
  ],
  "check_results": null
}

```
查询的 date 以及创建的 resource 信息都以 json 格式保存在 tfstate 文件中.
如果手动删除则会新建，并且 `terraform destory`的时候无法删除。
#### 生产环境的tfstate管理方案——Backend
`**terraform.tfstate**`**内容是明文的 **
> 一种是使用Vault或是AWS Secret Manager这样的动态机密管理工具生成临时有效的动态机密(比如有效期只有5分钟，即使被他人读取到，机密也早已失效)；另一种就是我们下面将要介绍的——Terraform Backend。

Backend 远程状态存储机制。Backend 是一种抽象的远程存储接口，如同Provider 一样，Backend 支持多种不同的存储服务。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1700552517359-58da4c9a-d484-4f9a-b77f-9e2a33ab04d8.png#averageHue=%23fefefe&clientId=uaea78b46-c740-4&from=paste&height=435&id=u125c4999&originHeight=870&originWidth=1217&originalType=binary&ratio=2&rotation=0&showTitle=false&size=110831&status=done&style=none&taskId=ud49824a6-0a25-4e64-9601-87983565eaf&title=&width=608.5)
支持的 Backend 列表（左侧）
Terraform Remote Backend 分为两种

- 标准 ： 支持远程状态存储与状态锁
- 增强：在标准的基础上支持远程操作（在远程服务器上执行 plan、apply 等操作）

目前增强型Backend只有Terraform Cloud云服务一种
状态锁是指，当针对一个tfstate 进行变更操作时，可以针对该状态文件加一把全局锁，确保同一时间只能有一个变更被执行。不同的 `Backend`对状态锁的支持不相同，实现状态锁的机制也不相同，例如 consule backend 就通过一个 .lock 节点来描述锁对应的会话信息，tfstate 文件被保存在 backend 定义的路径节点内；
##### 使用nacos 测试  增加如下 consul 配置
```shell
backend "consul" {
    address = "101.43.130.188:8500"
    scheme  = "http"
    path    = "my-ucloud-project"
  }
```
随后在consul 可见 backend 配置 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1700561258514-a7df45ef-a96d-4200-a0e8-71501b348893.png#averageHue=%23ecc355&clientId=u6023b330-f565-4&from=paste&height=885&id=u22e727c3&originHeight=1770&originWidth=2326&originalType=binary&ratio=2&rotation=0&showTitle=false&size=371418&status=done&style=none&taskId=ufea6a6d5-9859-427c-b562-adf3373e8a7&title=&width=1163)
##### 新建workspace 分区
```shell
yanzheng@MacBook-Pro-3 terraform % terraform workspace list #查询当前 backend 下的所有 workspace 
* default

yanzheng@MacBook-Pro-3 terraform %
yanzheng@MacBook-Pro-3 terraform %
yanzheng@MacBook-Pro-3 terraform %
yanzheng@MacBook-Pro-3 terraform % terraform workspace list
* default

yanzheng@MacBook-Pro-3 terraform % terraform workspace new feature1 #新增workspace 在当前的backend中
Created and switched to workspace "feature1"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
yanzheng@MacBook-Pro-3 terraform % terraform workspace list
  default
* feature1

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1700561731565-c84108ff-1582-4969-90c9-2440cc123945.png#averageHue=%23050505&clientId=u6023b330-f565-4&from=paste&height=508&id=uc7575b34&originHeight=1016&originWidth=1574&originalType=binary&ratio=2&rotation=0&showTitle=false&size=131339&status=done&style=none&taskId=uc3c9ead6-fe18-4914-889d-2e8dfb3a06f&title=&width=787)多出的 environment 文件内容就是 feature1  同步 Consul存储会发现多了一个文件：my-ucloud-project-env:feature1。这就是Terraform为feature1这个Workspace创建的独立的状态文件。

```shell
yanzheng@MacBook-Pro-3 terraform % terraform workspace select default #可用于切换workspace
Switched to workspace "default".
```
```shell
yanzheng@MacBook-Pro-3 terraform % terraform workspace delete feature1
Deleted workspace "feature1"!
```
##### 目前支持多工作区的Backend有：

- AzureRM
- Consul
- COS
- GCS
- Kubernetes
- Local
- Manta
- Postgres
- Remote
- S3
##### 该使用哪种隔离
相比起多文件夹隔离的方式来说，基于Workspace的隔离更加简单，只需要保存一份代码，在代码中不需要为Workspace编写额外代码，用命令行就可以在不同工作区之间来回切换。
但是Workspace的缺点也同样明显，由于所有工作区的Backend配置是一样的，所以有权读写某一个Workspace的人可以读取同一个Backend路径下所有其他Workspace；另外Workspace是隐式配置的(调用命令行)，所以有时人们会忘记自己工作在哪个Workspace下。
Terraform官方为Workspace设计的场景是：有时开发人员想要对既有的基础设施做一些变更，并进行一些测试，但又不想直接冒险修改既有的环境。这时他可以利用Workspace复制出一个与既有环境完全一致的平行环境，在这个平行环境里做一些变更，并进行测试和实验工作。
mock 一个相同的环境，进行仿真测试。
## 模块结构/语法
### 模块结构
#### 最小化模块结构
```shell
$ tree minimal-module/
.
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```
#### 推荐模块结构
```shell
$ tree complete-module/
.
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
├── ...
├── modules/
│   ├── nestedA/
│   │   ├── README.md
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   ├── outputs.tf
│   ├── nestedB/
│   ├── .../
├── examples/
│   ├── exampleA/
│   │   ├── main.tf
│   ├── exampleB/
│   ├── .../

```
文件介绍

- 一个README文件，用来描述模块的用途。文件名可以是README或者README.md，后者应采用Markdown语法编写。可以考虑在README中用可视化的图形来描绘创建的基础设施资源以及它们之间的关系。README中不需要描述模块的输入输出，因为工具会自动收集相关信息。如果在README中引用了外部文件或图片，请确保使用的是带有特定版本号的绝对URL路径以防止未来指向错误的版本
- 一个LICENSE描述模块使用的许可协议。如果你想要公开发布一个模块，最好考虑包含一个明确的许可证协议文件，许多组织不会使用没有明确许可证协议的模块
- 一个[examples文件夹](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples)用来给出一个调用样例(可选)
- 一个variables.tf文件，包含模块所有的输入变量。输入变量应该有明确的描述说明用途
- 一个outputs.tf文件，包含模块所有的输出值。输出值应该有明确的描述说明用途
- 嵌入模块文件夹，出于封装复杂性或是复用代码的目的，我们可以在modules子目录下建立一些嵌入模块。所有包含README文件的嵌入模块都可以被外部用户使用；不含README文件的模块被认为是仅在当前模块内使用的(可选)
- 一个main.tf，它是模块主要的入口点。对于一个简单的模块来说，可以把所有资源都定义在里面；如果是一个比较复杂的模块，我们可以把创建的资源分布到不同的代码文件中，但引用嵌入模块的代码还是应保留在main.tf里
- 其他定义了各种基础设施对象的代码文件(可选)
### 语法
[https://lonegunmanb.github.io/introduction-terraform/3.1.%E7%B1%BB%E5%9E%8B.html](https://lonegunmanb.github.io/introduction-terraform/3.1.%E7%B1%BB%E5%9E%8B.html)
## 命令行
[https://lonegunmanb.github.io/introduction-terraform/5.Terraform%E5%91%BD%E4%BB%A4%E8%A1%8C.html](https://lonegunmanb.github.io/introduction-terraform/5.Terraform%E5%91%BD%E4%BB%A4%E8%A1%8C.html)
