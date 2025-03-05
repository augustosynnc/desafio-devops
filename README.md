# Desafio DevOps - VExpenses
## Pré-requisitos

Antes de executar o código, certifique-se de que os seguintes requisitos estão atendidos:

### 1. **Terraform instalado**
O Terraform é a ferramenta usada para criar e gerenciar a infraestrutura como código. Para instalá-lo:

1. **Acesse o Site Oficial**:
   - Vá para o [site oficial do Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).

2. **Siga as Instruções de Instalação**:
   - Escolha o sistema operacional que você está usando (Windows, Linux ou Mac).
   - Siga os passos para baixar e instalar o Terraform.

3. **Verifique a Instalação**:
   - Abra o terminal ou prompt de comando e execute:
     ```bash
     terraform --version
     ```
   - Se a instalação estiver correta, você verá a versão do Terraform instalada.

---

### 2. **Credenciais da AWS configuradas**
Para que o Terraform possa criar recursos na AWS, ele precisa de credenciais de acesso. Aqui está como configurá-las:

#### **Opção 1: Usando o AWS CLI**

1. **Instale o AWS CLI**:
   - Siga as instruções de instalação no [site oficial da AWS](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

2. **Configure as Credenciais**:
   - Execute o comando abaixo e siga as instruções:
     ```bash
     aws configure
     ```
   - Insira suas credenciais (Access Key e Secret Key) quando solicitado.

3. **Verifique a Configuração**:
   - As credenciais serão salvas automaticamente no arquivo `~/.aws/credentials`.

#### **Opção 2: Configuração Manual**

1. **Crie um Arquivo de Credenciais**:
   - No diretório `~/.aws/`, crie um arquivo chamado `credentials` (se não existir).
   - Adicione o seguinte conteúdo:
     ```
     [default]
     aws_access_key_id = SUA_ACCESS_KEY
     aws_secret_access_key = SUA_SECRET_KEY
     ```
   - Substitua `SUA_ACCESS_KEY` e `SUA_SECRET_KEY` pelas suas credenciais da AWS.

2. **Verifique as Permissões**:
   - Certifique-se de que o usuário IAM associado às credenciais tem permissões para:
     - Criar e gerenciar instâncias EC2.
     - Criar e gerenciar VPCs, subnets, Security Groups, etc.

---

### 3. **Git instalado (opcional)**
O Git é necessário apenas se você for clonar o repositório do GitHub. Para instalá-lo:

1. **Acesse o Site Oficial**:
   - Vá para o [site oficial do Git](https://git-scm.com/).

2. **Siga as Instruções de Instalação**:
   - Escolha o sistema operacional que você está usando (Windows, Linux ou Mac).
   - Siga os passos para baixar e instalar o Git.

3. **Verifique a Instalação**:
   - Abra o terminal ou prompt de comando e execute:
     ```bash
     git --version
     ```
   - Se a instalação estiver correta, você verá a versão do Git instalada.

## Tarefa 1: Descrição Técnica do Código Terraform

### Objetivo Geral
Este projeto foi desenvolvido como parte de um desafio técnico para a área de **DevOps**, com o objetivo de **demonstrar conhecimentos em Infraestrutura como Código (IaC) utilizando Terraform**, além de **habilidades em segurança e automação de configuração de servidores**. A infraestrutura criada inclui uma VPC (Virtual Private Cloud), uma subnet pública, um Internet Gateway, um Security Group e uma instância EC2 com o servidor web Nginx instalado e configurado automaticamente. O código também gera um par de chaves SSH para acesso seguro à instância EC2.

O uso do Terraform garante que a infraestrutura seja **replicável, versionável e de fácil manutenção**, seguindo as melhores práticas de IaC. Além disso, a implementação de medidas de segurança e a automação da configuração do servidor demonstram habilidades essenciais para um profissional de DevOps.

### Recursos Criados e Suas Funções

1. **Provider AWS**:
   - Configura o provedor AWS para a região `us-east-1`, onde todos os recursos serão criados.

2. **Variáveis**:
   - `projeto`: Define o nome do projeto (padrão: "VExpenses").
   - `candidato`: Define o nome do candidato (padrão: "SeuNome").
   - `meu_ip`: Define o IP público do usuário para restringir o acesso SSH.

3. **Key Pair**:
   - **`tls_private_key.ec2_key`**: Gera uma chave privada RSA para acesso seguro à instância EC2.
   - **`aws_key_pair.ec2_key_pair`**: Cria um par de chaves na AWS usando a chave pública gerada.

4. **VPC (Virtual Private Cloud)**:
   - **`aws_vpc.main_vpc`**: Cria uma VPC com o bloco CIDR `10.0.0.0/16`, habilitando suporte a DNS e nomes de host. A VPC é o núcleo da infraestrutura, fornecendo uma rede isolada para os recursos.

5. **Subnet**:
   - **`aws_subnet.main_subnet`**: Cria uma subnet pública com o bloco CIDR `10.0.1.0/24` na zona de disponibilidade `us-east-1a`. A subnet permite organizar os recursos dentro da VPC.

6. **Internet Gateway**:
   - **`aws_internet_gateway.main_igw`**: Cria um Internet Gateway e o associa à VPC, permitindo que os recursos na subnet pública acessem a internet.

7. **Route Table e Association**:
   - **`aws_route_table.main_route_table`**: Cria uma tabela de rotas com uma rota padrão (`0.0.0.0/0`) apontando para o Internet Gateway.
   - **`aws_route_table_association.main_association`**: Associa a subnet à tabela de rotas, garantindo que o tráfego da subnet seja roteado corretamente.

8. **Security Group**:
   - **`aws_security_group.main_sg`**: Cria um Security Group que define as regras de tráfego de entrada e saída para a instância EC2. As regras incluem:
     - Acesso SSH (porta 22) restrito ao IP especificado na variável `meu_ip`, seguindo as melhores práticas de segurança.
     - Acesso HTTP (porta 80) e HTTPS (porta 443) a partir de qualquer lugar.
     - Permissão de tráfego de saída para as portas 80 (HTTP) e 443 (HTTPS).

9. **AMI (Amazon Machine Image)**:
   - **`data.aws_ami.debian12`**: Busca a AMI mais recente do Debian 12, que será usada para criar a instância EC2.

10. **Instância EC2**:
    - **`aws_instance.debian_ec2`**: Cria uma instância EC2 `t2.micro` usando a AMI do Debian 12. A instância é configurada com:
      - Um volume raiz de 20 GB do tipo `gp3`, escolhido por oferecer melhor desempenho e custo-benefício.
      - Um IP público associado.
      - Um script de `user_data` que instala e configura automaticamente o servidor web Nginx, além de criar uma página HTML simples. Essa automação garante que o servidor esteja pronto para uso imediatamente após a criação.

11. **Outputs**:
    - **`private_key`**: Exibe a chave privada gerada para acesso SSH à instância EC2 (marcada como sensível).
    - **`ec2_public_ip`**: Exibe o IP público da instância EC2, que pode ser usado para acessar o servidor web.

### Funcionamento Geral
Quando o código é executado com o comando `terraform apply`, ele segue uma sequência de passos para criar os recursos. Primeiro, o Terraform valida o código e gera um plano de execução. Em seguida, cria a VPC, a subnet, o Internet Gateway e a tabela de rotas. Por fim, provisiona a instância EC2 e configura o Nginx automaticamente usando o `user_data`.

O uso do `user_data` garante que a instância EC2 esteja pronta para uso imediatamente após a criação, sem a necessidade de intervenção manual. Além disso, a restrição de acesso SSH ao IP especificado e a configuração do Security Group seguem as melhores práticas de segurança.

### Habilidades Demonstradas
Este projeto demonstra as seguintes habilidades essenciais para um profissional de DevOps:
- **Infraestrutura como Código (IaC)**: Uso do Terraform para criar e gerenciar recursos na AWS de forma replicável e versionável.
- **Segurança**: Implementação de medidas de segurança, como restrição de acesso SSH e configuração de regras de tráfego no Security Group.
- **Automação**: Uso do `user_data` para automatizar a instalação e configuração do servidor web Nginx.

### Possíveis Expansões
No futuro, este projeto pode ser expandido para incluir:
- Um balanceador de carga (ELB) para distribuir o tráfego entre múltiplas instâncias EC2.
- A integração com um sistema de CI/CD para automatizar a implantação de aplicações.
- A criação de um banco de dados RDS para armazenamento de dados.

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
