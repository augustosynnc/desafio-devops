# Desafio DevOps - VExpenses
## Pré-requisitos

Antes de executar o código, certifique-se de que os seguintes requisitos estão atendidos:

1. **Terraform instalado**:
   - Siga as instruções de instalação no [site oficial do Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).

2. **Credenciais da AWS configuradas**:
   - Crie um usuário IAM na AWS com permissões para gerenciar recursos EC2, VPC e outros.
   - Configure as credenciais da AWS usando o AWS CLI ou manualmente no arquivo `~/.aws/credentials`.

3. **Git instalado (opcional)**:
   - Necessário apenas se você for clonar o repositório. Siga as instruções de instalação no [site oficial do Git](https://git-scm.com/).

## Tarefa 1: Análise Técnica do Código Terraform

### Descrição Detalhada

O código Terraform fornecido cria uma infraestrutura básica na AWS, composta pelos seguintes recursos:

#### Provider AWS
- Configurado para a região `us-east-1`.

#### Variáveis
- `projeto`: Define o nome do projeto (padrão: "Desafio VExpenses").
- `candidato`: Define o nome do candidato (padrão: "Augusto Cesar").
- Essas variáveis são usadas para padronizar os nomes dos recursos.

#### Key Pair
- `tls_private_key.ec2_key`: Gera uma chave privada RSA.
- `aws_key_pair.ec2_key_pair`: Cria um par de chaves na AWS usando a chave pública gerada.

#### VPC
- `aws_vpc.main_vpc`: Cria uma VPC com o bloco CIDR `10.0.0.0/16`, habilitando suporte a DNS.

#### Subnet
- `aws_subnet.main_subnet`: Cria uma subnet com o bloco CIDR `10.0.1.0/24` na zona de disponibilidade `us-east-1a`.

#### Internet Gateway
- `aws_internet_gateway.main_igw`: Cria um Internet Gateway para permitir acesso à internet.

#### Route Table e Association
- `aws_route_table.main_route_table`: Cria uma tabela de rotas com uma rota padrão para a internet.
- `aws_route_table_association.main_association`: Associa a subnet à tabela de rotas.

#### Security Group
- `aws_security_group.main_sg`: Cria um Security Group que permite:
  - Acesso SSH de qualquer lugar (`0.0.0.0/0` e `::/0`).
  - Todo o tráfego de saída.

#### AMI (Debian 12)
- `data.aws_ami.debian12`: Busca a AMI mais recente do Debian 12.

#### Instância EC2
- `aws_instance.debian_ec2`: Cria uma instância EC2 `t2.micro` usando a AMI do Debian 12.
  - A instância recebe um IP público e tem um volume raiz de 20 GB (tipo `gp2`).
  - O `user_data` executa um script básico para atualizar o sistema.

#### Outputs
- `private_key`: Exibe a chave privada (marcada como sensível).
- `ec2_public_ip`: Exibe o IP público da instância EC2.

### Observações
- **Segurança**: O Security Group permite acesso SSH de qualquer lugar, o que é inseguro. Recomenda-se restringir o acesso a IPs específicos.
- **Automação**: A instalação e configuração do Nginx não estão automatizadas. Isso pode ser feito usando o `user_data`.
- **Volume Root**: O volume raiz é do tipo `gp2`, que é uma geração antiga. Recomenda-se utilizar `gp3` para melhor desempenho e custo-benefício.

---

## Tarefa 2: Modificações e Melhorias do Código Terraform

### Melhorias Implementadas

1. **Restrição de Acesso SSH**:
   - O acesso SSH foi restrito ao IP especificado na variável `meu_ip`.
   - **Justificativa**: Aumenta a segurança ao permitir acesso apenas a IPs confiáveis.

2. **Automação da Instalação do Nginx**:
   - O `user_data` foi modificado para instalar e configurar o Nginx automaticamente.
   - **Justificativa**: Automatiza a configuração do servidor, reduzindo erros manuais.

3. **Uso do Volume Root `gp3`**:
   - O volume raiz foi atualizado para `gp3`.
   - **Justificativa**: Oferece melhor desempenho e custo-benefício.

4. **Restrição de Tráfego de Saída**:
   - O tráfego de saída foi limitado às portas 80 (HTTP) e 443 (HTTPS).
   - **Justificativa**: Aumenta a segurança ao evitar comunicação com serviços não autorizados.

5. **Adição de Tags**:
   - Tags consistentes foram adicionadas a todos os recursos.
   - **Justificativa**: Facilita a organização e identificação dos recursos.

6. **Bloqueio de Destruição**:
   - O recurso `lifecycle` foi adicionado para evitar a exclusão acidental da instância EC2.
   - **Justificativa**: Protege recursos críticos contra exclusões acidentais.

### Outras Melhorias
- **Geração Dinâmica de Chave SSH**:
  - A chave SSH é gerada automaticamente usando o recurso `tls_private_key`.
  - **Justificativa**: Elimina a necessidade de gerenciar manualmente as chaves SSH.

---

## Instruções de Uso

### Pré-requisitos
- **Terraform instalado**.
- **Credenciais da AWS configuradas**.

### Passos para Executar

1. **Inicialize o Terraform**:
   ```bash
   terraform init

2. **Planeje a Infraestrutura**:
   ```bash
   terraform plan
   
3. **Aplique a Infraestrutura**:
    ```bash
   terraform apply
    
4. **Acesse a Instância EC2**: Use o IP público exibido no output ec2_public_ip.
   
5. **Acesse via SSH com a chave privada gerada**:
    ```bash
   ssh -i chave_privada.pem admin@<ec2_public_ip>

Teste o Nginx: Abra o navegador e acesse http://<ec2_public_ip>. Você verá a mensagem "Hello from Terraform!".
