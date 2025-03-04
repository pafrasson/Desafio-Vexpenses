
## Autor

- [@pafrasson](https://www.github.com/pafrasson)


# Descrição técnica do código inicial

Este projeto utiliza Terraform para criar uma infraestrutura básica na AWS, incluindo:

- VPC (Virtual Private Cloud)
- Subnet
- Grupo de segurança
- Key Pair (chave SSH)
- Instância EC2 rodando Debian 12


## Recursos criados

#### Provider AWS

Define a região do servidor aws: us-east-1 (Norte da Virgínia).

```json
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



#### Criação da Chave SSH

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

#### Rede - VPC e Subnet

Cria uma VPC com suporte a DNS.

```
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
}
```
Cria uma subnet dentro da VPC, na zona us-east-1a.

```
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}
```
#### Internet Gateway e Roteamento

Cria um Internet Gateway para acesso externo à instância EC2.
