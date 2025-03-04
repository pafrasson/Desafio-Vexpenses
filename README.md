
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

```json
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}
```

| Parâmetro   | Tipo       | Descrição                                   |
| :---------- | :--------- | :------------------------------------------ |
| `id`      | `string` | **Obrigatório**. O ID do item que você quer |

#### add(num1, num2)

Recebe dois números e retorna a sua soma.

