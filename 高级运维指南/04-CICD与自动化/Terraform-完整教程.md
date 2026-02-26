# Terraform 完整教程

> 基础设施即代码（IaC）实战指南
>
> @author erik.zhou

## 📚 目录

- [1. Terraform 简介](#1-terraform-简介)
- [2. 安装与配置](#2-安装与配置)
- [3. 核心概念](#3-核心概念)
- [4. HCL 语法](#4-hcl-语法)
- [5. Provider 配置](#5-provider-配置)
- [6. 资源管理](#6-资源管理)
- [7. 状态管理](#7-状态管理)
- [8. 模块化](#8-模块化)
- [9. 实战案例](#9-实战案例)
- [10. 最佳实践](#10-最佳实践)
- [11. 故障排查](#11-故障排查)
- [12. 学习检查清单](#12-学习检查清单)

## 🎯 学习目标

- 理解基础设施即代码（IaC）的概念和优势
- 掌握 Terraform 的核心概念和工作流程
- 能够编写 Terraform 配置文件
- 掌握多云环境的资源管理
- 了解状态管理和远程后端
- 能够设计和使用 Terraform 模块
- 掌握 Terraform 的最佳实践

## 1. Terraform 简介

### 1.1 什么是 Terraform

Terraform 是 HashiCorp 开发的开源基础设施即代码（IaC）工具，用于安全高效地构建、变更和版本化基础设施。

**核心特性**：
- 声明式配置语言（HCL）
- 执行计划预览
- 资源依赖图
- 多云支持
- 状态管理
- 模块化设计

### 1.2 IaC 的优势

```
传统方式 vs IaC
┌─────────────────┐     ┌─────────────────┐
│   手动操作      │     │   代码定义      │
│   - 易出错      │     │   - 可重复      │
│   - 难追踪      │     │     - 可版本化  │
│   - 不一致      │     │     - 可审计    │
│   - 难回滚      │     │     - 易回滚    │
└─────────────────┘     └─────────────────┘
```

### 1.3 工作流程

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Write   │ -> │   Plan   │ -> │  Apply   │ -> │ Destroy  │
│  编写配置 │    │  预览变更 │    │  应用变更 │    │  销毁资源 │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

## 2. 安装与配置

### 2.1 安装 Terraform

```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (Ubuntu/Debian)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Linux (CentOS/RHEL)
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform

# Windows (Chocolatey)
choco install terraform

# 验证安装
terraform version
```

### 2.2 配置自动补全

```bash
# Bash
terraform -install-autocomplete

# Zsh
echo 'autoload -U +X bashcompinit && bashcompinit' >> ~/.zshrc
echo 'complete -o nospace -C /usr/local/bin/terraform terraform' >> ~/.zshrc
source ~/.zshrc
```

### 2.3 配置凭证

```bash
# AWS
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"

# 或使用 AWS CLI 配置
aws configure

# Azure
az login

# Google Cloud
gcloud auth application-default login

# 阿里云
export ALICLOUD_ACCESS_KEY="your-access-key"
export ALICLOUD_SECRET_KEY="your-secret-key"
export ALICLOUD_REGION="cn-hangzhou"
```

## 3. 核心概念

### 3.1 Provider

Provider 是 Terraform 与云平台或服务交互的插件。

```hcl
# AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Azure Provider
provider "azurerm" {
  features {}
}

# Google Cloud Provider
provider "google" {
  project = "my-project"
  region  = "us-central1"
}
```

### 3.2 Resource

Resource 是基础设施的组件，如虚拟机、网络、存储等。

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

### 3.3 Data Source

Data Source 用于查询现有资源的信息。

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

### 3.4 Variable

Variable 用于参数化配置。

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}
```

### 3.5 Output

Output 用于输出资源的属性。

```hcl
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}
```

### 3.6 State

State 是 Terraform 管理的资源状态文件。

```bash
# 查看状态
terraform show

# 列出资源
terraform state list

# 查看特定资源
terraform state show aws_instance.web
```

## 4. HCL 语法

### 4.1 基本语法

```hcl
# 注释
// 单行注释
/* 多行
   注释 */

# 块（Block）
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# 参数（Argument）
ami = "ami-0c55b159cbfafe1f0"

# 表达式（Expression）
instance_type = var.instance_type
```

### 4.2 数据类型

```hcl
# 字符串
variable "name" {
  type    = string
  default = "example"
}

# 数字
variable "count" {
  type    = number
  default = 3
}

# 布尔值
variable "enabled" {
  type    = bool
  default = true
}

# 列表
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

# 映射
variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Project     = "myapp"
  }
}

# 对象
variable "instance_config" {
  type = object({
    instance_type = string
    ami           = string
    tags          = map(string)
  })
}

# 元组
variable "mixed_list" {
  type    = tuple([string, number, bool])
  default = ["example", 42, true]
}
```

### 4.3 表达式和函数

```hcl
# 字符串插值
name = "server-${var.environment}"

# 条件表达式
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# for 表达式
subnet_ids = [for s in var.subnets : s.id]

# 函数
# 字符串函数
upper_name = upper(var.name)
lower_name = lower(var.name)
trimmed    = trim(var.text)

# 数值函数
max_value = max(5, 12, 9)
min_value = min(5, 12, 9)

# 集合函数
merged_map = merge(var.map1, var.map2)
list_length = length(var.list)
contains_item = contains(var.list, "item")

# 编码函数
json_data = jsonencode(var.data)
yaml_data = yamlencode(var.data)

# 文件系统函数
file_content = file("${path.module}/config.txt")
template_content = templatefile("${path.module}/template.tpl", {
  name = var.name
})

# 日期和时间函数
timestamp = timestamp()
formatted_time = formatdate("YYYY-MM-DD", timestamp())

# 哈希和加密函数
md5_hash = md5("hello")
sha256_hash = sha256("hello")
base64_encoded = base64encode("hello")
```

### 4.4 动态块

```hcl
resource "aws_security_group" "example" {
  name = "example"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```



## 5. Provider 配置

### 5.1 AWS Provider

```hcl
# provider.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# 多区域配置
provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

provider "aws" {
  alias  = "eu_west"
  region = "eu-west-1"
}

# 使用别名
resource "aws_instance" "west" {
  provider = aws.us_west
  # ...
}
```

### 5.2 Azure Provider

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
    
    virtual_machine {
      delete_os_disk_on_deletion     = true
      graceful_shutdown              = false
      skip_shutdown_and_force_delete = false
    }
  }
  
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}
```

### 5.3 阿里云 Provider

```hcl
terraform {
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = "~> 1.200"
    }
  }
}

provider "alicloud" {
  access_key = var.access_key
  secret_key = var.secret_key
  region     = var.region
}
```

## 6. 资源管理

### 6.1 创建资源

```hcl
# main.tf
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# 子网
resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
    Type = "public"
  }
}

# 安全组
resource "aws_security_group" "web" {
  name        = "${var.project_name}-web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.project_name}-web-sg"
  }
}

# EC2 实例
resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[count.index % length(aws_subnet.public)].id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = templatefile("${path.module}/user_data.sh", {
    hostname = "web-${count.index + 1}"
  })
  
  root_block_device {
    volume_size = 20
    volume_type = "gp3"
    encrypted   = true
  }
  
  tags = {
    Name = "${var.project_name}-web-${count.index + 1}"
  }
}
```

### 6.2 资源依赖

```hcl
# 显式依赖
resource "aws_eip" "example" {
  vpc = true
  
  depends_on = [aws_internet_gateway.example]
}

# 隐式依赖（通过引用）
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  subnet_id     = aws_subnet.public.id  # 隐式依赖
  instance_type = "t2.micro"
}
```

### 6.3 资源生命周期

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  lifecycle {
    # 创建新资源后再删除旧资源
    create_before_destroy = true
    
    # 防止资源被删除
    prevent_destroy = true
    
    # 忽略特定属性的变更
    ignore_changes = [
      tags,
      user_data,
    ]
  }
}
```

### 6.4 条件资源创建

```hcl
# 使用 count
resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# 使用 for_each
resource "aws_instance" "servers" {
  for_each = var.servers
  
  ami           = each.value.ami
  instance_type = each.value.instance_type
  
  tags = {
    Name = each.key
  }
}
```

## 7. 状态管理

### 7.1 本地状态

```bash
# 默认状态文件
terraform.tfstate

# 备份文件
terraform.tfstate.backup

# 查看状态
terraform show

# 列出资源
terraform state list

# 查看特定资源
terraform state show aws_instance.web

# 移动资源
terraform state mv aws_instance.old aws_instance.new

# 删除资源（仅从状态中删除）
terraform state rm aws_instance.example

# 导入现有资源
terraform import aws_instance.example i-1234567890abcdef0
```

### 7.2 远程状态（S3 后端）

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

# 初始化后端
# terraform init -backend-config="bucket=my-terraform-state"
```

### 7.3 状态锁定

```hcl
# 创建 DynamoDB 表用于状态锁定
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name = "Terraform State Lock Table"
  }
}
```

### 7.4 远程状态数据源

```hcl
# 读取其他项目的状态
data "terraform_remote_state" "vpc" {
  backend = "s3"
  
  config = {
    bucket = "my-terraform-state"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# 使用远程状态的输出
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
  # ...
}
```

## 8. 模块化

### 8.1 创建模块

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support
  
  tags = merge(
    var.tags,
    {
      Name = var.name
    }
  )
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-public-${count.index + 1}"
      Type = "public"
    }
  )
}

# modules/vpc/variables.tf
variable "name" {
  description = "Name of the VPC"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "enable_dns_hostnames" {
  description = "Enable DNS hostnames"
  type        = bool
  default     = true
}

variable "enable_dns_support" {
  description = "Enable DNS support"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.this.id
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}
```

### 8.2 使用模块

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  name               = "my-vpc"
  cidr_block         = "10.0.0.0/16"
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones = ["us-east-1a", "us-east-1b"]
  
  tags = {
    Environment = "production"
    Project     = "myapp"
  }
}

# 使用模块输出
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
  # ...
}
```

### 8.3 公共模块

```hcl
# 使用 Terraform Registry 的模块
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
  
  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

## 9. 实战案例

### 9.1 完整的 AWS 三层架构

```hcl
# variables.tf
variable "project_name" {
  description = "Project name"
  type        = string
  default     = "myapp"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# main.tf
# VPC 模块
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "${var.project_name}-${var.environment}"
  cidr = var.vpc_cidr
  
  azs              = var.availability_zones
  private_subnets  = [for i, az in var.availability_zones : cidrsubnet(var.vpc_cidr, 8, i)]
  public_subnets   = [for i, az in var.availability_zones : cidrsubnet(var.vpc_cidr, 8, i + 100)]
  database_subnets = [for i, az in var.availability_zones : cidrsubnet(var.vpc_cidr, 8, i + 200)]
  
  enable_nat_gateway   = true
  single_nat_gateway   = false
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# ALB 安全组
resource "aws_security_group" "alb" {
  name        = "${var.project_name}-alb-sg"
  description = "Security group for ALB"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 应用层安全组
resource "aws_security_group" "app" {
  name        = "${var.project_name}-app-sg"
  description = "Security group for application servers"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 数据库安全组
resource "aws_security_group" "db" {
  name        = "${var.project_name}-db-sg"
  description = "Security group for database"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
}

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets
  
  enable_deletion_protection = false
  
  tags = {
    Name = "${var.project_name}-alb"
  }
}

# Target Group
resource "aws_lb_target_group" "app" {
  name     = "${var.project_name}-tg"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = module.vpc.vpc_id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
}

# ALB Listener
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# Launch Template
resource "aws_launch_template" "app" {
  name_prefix   = "${var.project_name}-"
  image_id      = data.aws_ami.amazon_linux_2.id
  instance_type = "t3.micro"
  
  vpc_security_group_ids = [aws_security_group.app.id]
  
  user_data = base64encode(templatefile("${path.module}/user_data.sh", {
    db_endpoint = aws_db_instance.main.endpoint
  }))
  
  tag_specifications {
    resource_type = "instance"
    
    tags = {
      Name = "${var.project_name}-app"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "app" {
  name                = "${var.project_name}-asg"
  vpc_zone_identifier = module.vpc.private_subnets
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"
  health_check_grace_period = 300
  
  min_size         = 2
  max_size         = 10
  desired_capacity = 3
  
  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${var.project_name}-app"
    propagate_at_launch = true
  }
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier     = "${var.project_name}-db"
  engine         = "mysql"
  engine_version = "8.0"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp3"
  storage_encrypted     = true
  
  db_name  = "myapp"
  username = "admin"
  password = random_password.db_password.result
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.db.id]
  
  backup_retention_period = 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "mon:04:00-mon:05:00"
  
  skip_final_snapshot = true
  
  tags = {
    Name = "${var.project_name}-db"
  }
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet-group"
  subnet_ids = module.vpc.database_subnets
  
  tags = {
    Name = "${var.project_name}-db-subnet-group"
  }
}

# Random Password
resource "random_password" "db_password" {
  length  = 16
  special = true
}

# Data Source
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# outputs.tf
output "alb_dns_name" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "db_endpoint" {
  description = "Database endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}
```

### 9.2 Kubernetes 集群（EKS）

```hcl
# eks.tf
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.0.0"
  
  cluster_name    = "${var.project_name}-eks"
  cluster_version = "1.27"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  cluster_endpoint_public_access = true
  
  eks_managed_node_groups = {
    general = {
      desired_size = 2
      min_size     = 1
      max_size     = 5
      
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
      
      labels = {
        role = "general"
      }
      
      tags = {
        Environment = var.environment
      }
    }
  }
  
  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

## 10. 最佳实践

### 10.1 项目结构

```
terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   │   └── ...
│   └── database/
│       └── ...
├── .gitignore
└── README.md
```

### 10.2 命名规范

```hcl
# 资源命名
resource "aws_instance" "web_server" {  # 使用下划线
  tags = {
    Name = "${var.project}-web-server"  # 使用连字符
  }
}

# 变量命名
variable "instance_type" {  # 使用下划线
  description = "EC2 instance type"
  type        = string
}
```

### 10.3 版本控制

```hcl
# versions.tf
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}
```

### 10.4 安全实践

```hcl
# 1. 使用变量存储敏感信息
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

# 2. 加密状态文件
terraform {
  backend "s3" {
    encrypt = true
  }
}

# 3. 使用 IAM 角色而非访问密钥
provider "aws" {
  # 不要硬编码 access_key 和 secret_key
  # 使用 IAM 角色或环境变量
}

# 4. 启用资源加密
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
}

resource "aws_s3_bucket_server_side_encryption_configuration" "example" {
  bucket = aws_s3_bucket.example.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

## 11. 故障排查

### 11.1 常见问题

```bash
# 1. 状态锁定问题
# 强制解锁（谨慎使用）
terraform force-unlock <lock-id>

# 2. 状态不一致
# 刷新状态
terraform refresh

# 3. 资源已存在
# 导入现有资源
terraform import aws_instance.example i-1234567890abcdef0

# 4. 依赖问题
# 查看依赖图
terraform graph | dot -Tpng > graph.png

# 5. 调试模式
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log
terraform apply
```

### 11.2 验证和格式化

```bash
# 格式化代码
terraform fmt -recursive

# 验证配置
terraform validate

# 检查语法
terraform validate -json

# 安全扫描
tfsec .
checkov -d .
```

## 12. 学习检查清单

### 基础知识
- [ ] 理解 IaC 的概念和优势
- [ ] 掌握 Terraform 的核心概念
- [ ] 熟悉 HCL 语法
- [ ] 了解 Provider 的作用

### 资源管理
- [ ] 能够创建和管理资源
- [ ] 理解资源依赖关系
- [ ] 掌握资源生命周期
- [ ] 能够使用条件创建资源

### 状态管理
- [ ] 理解状态文件的作用
- [ ] 掌握远程状态配置
- [ ] 了解状态锁定机制
- [ ] 能够导入现有资源

### 模块化
- [ ] 能够创建自定义模块
- [ ] 掌握模块的使用方法
- [ ] 了解公共模块的使用
- [ ] 理解模块版本管理

### 实战能力
- [ ] 能够设计完整的基础设施
- [ ] 掌握多环境管理
- [ ] 能够排查常见问题
- [ ] 遵循最佳实践

## 📚 参考资源

### 官方文档
- [Terraform 官方文档](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [HCL 语法参考](https://www.terraform.io/language/syntax)

### 学习资源
- [Terraform 最佳实践](https://www.terraform-best-practices.com/)
- [Terraform Up & Running](https://www.terraformupandrunning.com/)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)

### 工具
- [tfsec](https://github.com/aquasecurity/tfsec) - 安全扫描
- [checkov](https://www.checkov.io/) - 策略即代码
- [terraform-docs](https://terraform-docs.io/) - 文档生成
- [terragrunt](https://terragrunt.gruntwork.io/) - Terraform 包装器

### 相关技术
- AWS/Azure/GCP
- Kubernetes
- Ansible
- CI/CD

---

> 💡 **提示**：Terraform 是 IaC 的事实标准，掌握 Terraform 可以大大提升基础设施管理效率。建议从简单的资源开始，逐步学习模块化和状态管理。使用远程状态和模块化是生产环境的必备技能。
>
> @author erik.zhou

