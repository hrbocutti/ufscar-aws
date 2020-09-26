# Trabalho de conclusão:

Alunos:

- Higor Bocutti <a href="https://github.com/hrbocutti"><img alt="Github" title="hrbocutti" src="https://github.githubassets.com/images/icons/emoji/octocat.png" width="25"></a> 
- Geanderson Braga Lopes <a href="https://github.com/GeandersonLopes"><img alt="Github" title="GeandersonLopes" src="https://github.githubassets.com/images/icons/emoji/octocat.png" width="25"></a>
- Dalmo Tavares <a href="https://github.com/dalmost"><img alt="Github" title="dalmost" src="https://github.githubassets.com/images/icons/emoji/octocat.png" width="25"></a>

---
### Arquitetura Cloud

![Arquitetura cloud AWS](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/arquitetura.png)

### Configuração Aws:

#### Pré requisitos:
 - AWS cli
 - kubectl
 - terraform
 
### Preparação do ambiente
<p>
Para começar é necessário preparar o ambiente da aws com as credencias.
<br>Forneça as credênciais para acesso ao CLI da AWS.
</p>  

```shell
aws configure
AWS Access Key ID [None]: YOUR_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_AWS_SECRET_ACCESS_KEY
Default region name [None]: YOUR_AWS_REGION
Default output format [None]: json
```

### Setup e inicialização do terraform

Em seu terminal, clone o seguinte repositório. Ele contém a configuração de exemplo usada neste guia.

```shell script
$ git clone https://github.com/hrbocutti/ufscar-aws.git
```

Você pode explorar este repositório alterando os diretórios ou navegando em sua IU.

```shell script
$ cd configs/terraform/
```

Initialize Terraform workspace

```shell script
$ terraform init
Initializing modules...
Downloading terraform-aws-modules/eks/aws 12.2.0 for eks...
- eks in .terraform/modules/eks
- eks.node_groups in .terraform/modules/eks/modules/node_groups
Downloading terraform-aws-modules/vpc/aws 2.6.0 for vpc...
- vpc in .terraform/modules/vpc

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "random" (hashicorp/random) 2.3.0...
- Downloading plugin for provider "kubernetes" (hashicorp/kubernetes) 1.13.2...
- Downloading plugin for provider "template" (hashicorp/template) 2.1.2...
- Downloading plugin for provider "local" (hashicorp/local) 1.4.0...
- Downloading plugin for provider "null" (hashicorp/null) 2.1.2...
- Downloading plugin for provider "aws" (hashicorp/aws) 3.7.0...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

#### Provisione o cluster EKS

```shell script
$ terraform apply
module.eks.data.aws_iam_policy_document.cluster_assume_role_policy: Refreshing state...
module.eks.data.aws_ami.eks_worker: Refreshing state...
module.eks.data.aws_partition.current: Refreshing state...
module.eks.data.aws_caller_identity.current: Refreshing state...
module.eks.data.aws_iam_policy_document.cluster_elb_sl_role_creation[0]: Refreshing state...
data.aws_availability_zones.available: Refreshing state...
module.eks.data.aws_ami.eks_worker_windows: Refreshing state...
module.eks.data.aws_iam_policy_document.workers_assume_role_policy: Refreshing state...

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  ##Truncated ...

Plan: 52 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Você pode ver que este modelo de terreno se aplica e provisionará um total de 51 recursos.
Se você estiver confortável com isso, confirme a execução com um `yes`.

Este processo deve levar aproximadamente 10 minutos. 
Após a aplicação bem-sucedida, seu terminal imprime as saídas definidas em `output.tf`.

#### Prepare o repositório ECR

Você pode criar um repositório para armazenar a imagem do docker.

```shell script
$ aws ecr create-repository --repository-name wp-ufscar --region us-east-1
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:049938869197:repository/wp-ufscar",
        "registryId": "049938869197",
        "repositoryName": "wp-ufscar",
        "repositoryUri": "049938869197.dkr.ecr.us-east-1.amazonaws.com/wp-ufscar",
        "createdAt": 1600752383.0,
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        }
    }
}
```

##### Tag docker
Exemplo: `$ docker tag ${IMAGE ID} ${REPOSITORY URL}`
```shell script
$ docker tag d4105c7a3a46 049938869197.dkr.ecr.us-east-1.amazonaws.com/wp-ufscar
```

##### Push image

Exemplo: `$ docker push ${REPOSITORY}`

```shell script
$ docker push 049938869197.dkr.ecr.us-east-1.amazonaws.com/wp-ufscar
```

#### Configurar o kubectl para acessar o cluster

Exemplo: `$ aws eks --region us-east-1 update-kubeconfig --name ufscar-qTKumoLe`

```shell script
$ aws eks --region ${region} update-kubeconfig --name ${cluster_name}
```

#### Execute deploy to K8S

Navegar até a pasta kubernets: `../configs/kubernets`

```shell script
$ kubectl apply -f service.yaml
```
##### Adicionar o loadbalancer para acesso externo da applicação

Exemplo: `$ kubectl expose deployment wp-ufscar --type=LoadBalancer --name=lb-service`

```shell script
$ kubectl expose deployment ${deployment} --type=LoadBalancer --name=lb-service
```


### Criação de Bucket para o S3:
<p>
Criação do Bucket bibliotecareservas:
<br>Inicialmente deverá ser selecionado o item Criar bucket.
</p> 

![Criação Bucket](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20bucket%201.png)

<p>
Posteriormente deverá ser registrado o nome do bucket.
</p> 

![Nome do Bucket](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20bucket%202.png)

<p>
Informado o bloqueio ao acesso público.
</p>

![Bloqueio](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20bucket%203.png)

<p>
E para finalizar concluir a criação do Bucket.
</p>

![Finalização](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20bucket%204.png)

<p>
Com isso o Bucket foi criado com sucesso.
</p>

![Bucket criado](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20bucket%205.png)


### Criação de Tabela Reserva - DynamoDB:

<p>
<br>Criação da Tabela Reservas.
<br>Criação do indice ID Livro.
</p> 

![Criar tabela](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20dynamodb%201.png)

<p>
Será necessário informar o nome da tabela junto com o indice.
</p>

![Nome da tabela](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20dynamodb%202.png)

<p>
Posteriormente deverá ser analisado a capacidade de leitura e de gravação.
</p>

![Capacidade](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20dynamodb%203.png)

<p>
Para finalizar deverá ser realizado a conclusão da criação da tabela.
</p>

![Conclusão](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20dynamodb%204.png)

<p>
Tabela Reservas criada com sucesso.
</p>

![Tabelas criadas](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/criacao%20dynamodb%205.png)


### Implantação do WordPress com o Amazon RDS:

#### Criação do Banco de Dados MySQL com o RDS:

<p>
No Amazon RDS console AWS clicar no botão em destaque.
</p>

![Criar RDS1](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS1.png)

<p>
Posteriormente escolher a opção MySQL.
</p>

![Criar RDS2](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS2.png)

<p>
E escolher o Template Dev/Test.
</p>

![Criar RDS3](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS3.png)

<p>
Criar usuário e senha para acesso ao Banco de Dados.
</p>

![Criar RDS4](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS4.png)

<p>
Posteriormente atribuir a instância como na imagem.
</p>

![Criar RDS5](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS5.png)

<p>
Como último passo realizar os procedimentos abaixo nas configurações adicionais.
</p>

![Criar RDS6](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS6.png)

<p>
E concluir a criação do Banco de Dados.
</p>

![Criar RDS7](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS7.png)

<p>
Banco de Dados criado com sucesso.
</p>

![Criar RDS8](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS8.png)


#### Criação da Instância no EC2:

<p>
Inicialmente executar a instância.
</p>

![Criar RDS9](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS9.png)

<p>
Posteriormente escolher a Amazon Machine Image.
</p>

![Criar RDS10](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS10.png)

<p>
Logo após o tipo da instância.
</p>

![Criar RDS11](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS11.png)

<p>
Deverá ser configurado o Grupo de Segurança da Instância.
</p>

![Criar RDS12](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS12.png)

<p>
Posteriormente deverá ser concluído a configuração.
</p>

![Criar RDS13](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS13.png)

<p>
Neste momento será criado uma chave SSH.
</p>

![Criar RDS14](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS14.png)

<p>
Deverá ser realizado o Download do Key Pair.
</p>

![Criar RDS15](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS15.png)

<p>
E por último executar a Instância.
</p>

![Criar RDS16](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS16.png)

#### Configuração do Banco de Dados RDS:

<p>
Deverá ser permitido que a Instância do EC2 acesse ao Banco de Dados.
</p>

![Criar RDS17](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS17.png)

<p>
Vá até a Guia Conectividade e Segurança e clique no Grupo de Segurança listado em VPC.
</p>

![Criar RDS18](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS18.png)

<p>
Posteriormente deverá ser configurado as Regras de Entrada.
</p>

![Criar RDS19](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS19.png)

<p>
E para concluir as Regras são salvas.
</p>

![Criar RDS20](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS20.png)

<p>
Com a chave SSH deverá ser acessado o Putty, e deverá ser realizado as seguintes codificações,
para criação do usuário de Banco de Dados. Anotar o endpoint da imagem abaixo, pois será utilizado.
</p>

![Criar RDS21](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS21.png)

```shell script
sudo yum install -y mysql
export MYSQL_HOST=wordpress.c6shsuwztqdl.us-east-1.rds.amazonaws.com
mysql --user=admin --password=Reservas wordpress
```

<p>
Posteriormente os seguintes comandos deverão ser executados.
</p>

```shell script
CREATE USER 'wordpress' IDENTIFIED BY 'reservas';
GRANT ALL PRIVILEGES ON wordpress.* TO wordpress;
FLUSH PRIVILEGES;
Exit
```

#### Configuração do Wordpress no EC2:

<p>
Para instalação do servidor Web Apache, deverá ser executado o comando abaixo. 
</p>

```shell script
sudo yum install -y httpd
```
<p>
Posteriormente o seguinte endereço Public DNS poderá ser acessado para verificar uma página
de Teste do Apache.
</p>

![Criar RDS22](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS22.png)

![Criar RDS23](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS23.png)

<p>
Para realizar o download e configuração do Wordpress as seguintes codificações deverão ser
realizadas.
</p>

```shell script
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```
<p>
Posteriormente abrir o arquivo nano e realizar as configurações abaixo.
</p>

```shell script
nano wp-config.php
```

```shell script
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpress' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

![Criar RDS24](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS24.png)

<p>
Para finalizar os seguintes comandos deverão ser executados para instalar as 
dependências para o aplicativo do WordPress.
</p>

```shell script
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
cd /home/ec2-user
sudo cp -r wordpress/* /var/www/html/
sudo service httpd restart
```

<p>
Posteriormente será possível ver a página inicial do Wordpress e o processo
de instalação.
</p>

![Criar RDS25](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS25.png)

![Criar RDS26](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS26.png)

<p>
Após estes procedimentos a instalação do WordPress se encontra ativa e acessível ao público com o 
objetivo para a Reserva de Livros.
</p>

![Criar RDS27](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS27.png)

![Criar RDS28](https://raw.githubusercontent.com/hrbocutti/ufscar-aws/master/assets/RDS28.png)


### Cognito:

##### Acessar a AWS procurando o serviço Cognito

### Primeiro, é necessario a criação do User Pool

1.) `No campo Pool name é necessario colocar o nome do pool que vai ser criado, no caso`
`criaremos com o nome de CognitoWord, após selecionar a opção "Step through settings".`

2.)`Na proxima tela selecionar Email address or phone number, após a opção "Alow email addresses",`
`para esse caso o acesso e o login é feito por email.`

3.) `Na opção "Which standard attributes do you want to require?", selecionar "name" e "email", no` `momento do cadastro serão exigidos do usuario apenas o nome e o email.`

4.) `Clicando no botão "Next step", ele vai para a proxima tela que seria "Policies", onde estão os` `opções de configuração da senha se a mesma vai ter letra, simbolos, números, quantidade de` `caracteres na senha e a quantidade de dias para expirar a senha. Todas essas opções serão padrão` `nessa pagina.`

5.) `Clicando no botão "Next step", ele vai para a proxima tela que seria "MFA and verifications", ` `onde estão os opções de configuração de multiplus fatores (que não iremos ativar), essa opção é` `utilizada para login e verificação em dois pontos, devemos selecionar a opção "Optional". Após vem a opção de selecionar o SMS text message, para que a verificação seja feita em duas frentes a senha no site com o email e o envio de um numero por SMS.`
`A proxima opção é "How will a user be able to recover their account?", que seria de recuperação de senha, nessa opção selecionaremos "Email only" para que a senha seja recuperada por email.`
`A proxima opção é "Which attributes do you want to verify?", que é selecionado o que será verificado para a recuperação da senha, nessa devemos selecionar "Email".`
`A proxima opção "You must provide a role to allow Amazon Cognito to send SMS messages", onde devemos marcar a regra de envio de SMS, a regra ja vem por padrão a regra é a "CognitoWord-SMS-Role", após a criação é gerado o nome da regra "arn:aws:iam::326554917983:role/service-role/CognitoWord-SMS-Role".`

6.) `Clicando no botão "Next step", ele vai para a proxima tela que seria "Message customizations", onde é customizado o email e o SMS que serão enviados para o usuario que terá que fazer o acesso.`
`Onde devem ser preenchido o SES Region (região Us East Virginia, onde é está sendo montado a estrutura por trás dessa parte).`
`No endereço de email, colocar o endereço do email que vai receber a senha e será utilizado para o acesso.`
`Na opção "Do you want to send emails through your Amazon SES Configuration?", utilizar a opção "No - Use Cognito (default)", onde será enviado um email padrão pelo Cognito, os demais campos continuam com os dados padrão.`

7.) `Clicando no botão "Next step", ele vai para a proxima tela que seria "Tag", onde você deve adicionar uma tag para poder idenficar qual user pool é de qual processo.`
`Clicando em "Add tag", preencher no campo "Tag Key", "Identify" e no campo "Tag Value" colocar  "teste1".`

8.) `Clicando no botão "Next step", ele vai para a proxima tela que seria a de "Devices", essa opção é marcada para que o computador seja reconhecido quando conectar no login, selecionar "No".`

9.) `Nas proximas opçãos teremos, não iremos selecionar nada mantendo os dados como os padrões ja pre-estabelecidos.`

10.) `Por ultimo, teremos a tela de "Review", onde podemos checar os dados inputados para a criação do``"User Pool", clicando no botão "Create Pool", ele cria o pool com os dados colocados acima.`

11.) `Após a criação é gerado os dados para os campos "Pool Id" que nesse caso é "us-east-1_DHHTFd2GJ",``e o campo "Pool ARN" que tem o valor "arn:aws:cognito-idp:us-east-1:326554917983:userpool/` `us-east-1_DHHTFd2GJ", guardar esses valores para a utilização quando for feita a integração com os demais serviços a serem utilizados.`


---
## Troubleshooting

- No basic auth credentials: [LINK-FIX](https://docs.aws.amazon.com/AmazonECR/latest/userguide/common-errors-docker.html)


