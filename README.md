
## Autor

- [@pafrasson](https://www.github.com/pafrasson)

## Referência

 - [Terraform tutorials: Get Started - AWS](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)

# Descrição técnica do código inicial

Este projeto utiliza Terraform para criar uma infraestrutura básica na AWS, incluindo:

- VPC (Virtual Private Cloud)
- Subnet
- Grupo de segurança
- Key Pair (chave SSH)
- Instância EC2 rodando Debian 12


## Recursos criados

### Provider AWS

Define a região do servidor aws: us-east-1 (Norte da Virgínia).

```
  provider "aws" {
  region = "us-east-1"
}
```
#### Variáveis criadas
Variáveis para personalizar o nome do projeto e do candidato.

| variable   | type       | value       | description                           |
| :---------- | :--------- | :--------- | :---------------------------------- |
| `projeto` | `string` | VExpenses |Nome do projeto |

| variable   | type       | value       | description                           |
| :---------- | :--------- | :--------- | :---------------------------------- |
| `candidato` | `string` |Pedro Frasson |Nome do candidato |



### Criação da Chave SSH

Gera uma chave privada RSA de 2048 bits para acesso à instância EC2.

```
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}
```

Cria um Key Pair na AWS para permitir conexão SSH na instância EC2.

```
resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${var.projeto}-${var.candidato}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}
```

### Rede - VPC e Subnet

Cria uma VPC com suporte a DNS.

```
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.projeto}-${var.candidato}-vpc"
  }
}
```
Cria uma subnet dentro da VPC, na zona us-east-1a.

```
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "${var.projeto}-${var.candidato}-subnet"
  }
}
```
### Internet Gateway e Roteamento

Cria um **Internet Gateway** para acesso externo a instância EC2.

```
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "${var.projeto}-${var.candidato}-igw"
  }
}
```

Cria uma **Tabela de Rotas** permitindo tráfego para a Internet.

```
resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table"
  }
}
```

Associa a tabela de rotas para a **subnet** criada.

```
resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table_association"
  }
}
```

### Security Group

Cria um Security Group que permite conexões **SSH (porta 22) de qualquer lugar e tráfego de saída irrestrito.**

```
resource "aws_security_group" "main_sg" {
  name        = "${var.projeto}-${var.candidato}-sg"
  description = "Permitir SSH de qualquer lugar e todo o tráfego de saída"
  vpc_id      = aws_vpc.main_vpc.id

  # Regras de entrada
  ingress {
    description      = "Allow SSH from anywhere"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  # Regras de saída
  egress {
    description      = "Allow all outbound traffic"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-sg"
  }
}
```

### Instância EC2

Busca a **Versão da imagem de sistema mais recente do Debian 12** na AWS.

```
data "aws_ami" "debian12" {
  most_recent = true

  filter {
    name   = "name"
    values = ["debian-12-amd64-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["679593333241"]
}
```

Cria a instância EC2, utilizando a AMI encontrada e conectando à VPC.

```
resource "aws_instance" "debian_ec2" {
  ami             = data.aws_ami.debian12.id
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.main_subnet.id
  key_name        = aws_key_pair.ec2_key_pair.key_name
  security_groups = [aws_security_group.main_sg.name]

  associate_public_ip_address = true

  root_block_device {
    volume_size           = 20
    volume_type           = "gp2"
    delete_on_termination = true
  }

  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get upgrade -y
              EOF

  tags = {
    Name = "${var.projeto}-${var.candidato}-ec2"
  }
}

output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}
```

### Saídas (Outputs) configurados

Exibe a chave privada para acesso SSH e o IP público da instância.

```
output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}
```

## Como Executar o Código Terraform

### **Pré-requisitos**

- Conta AWS configurada
- AWS CLI instalado e autenticado
- Terraform instalado
