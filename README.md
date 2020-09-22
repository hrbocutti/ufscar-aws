# Trabalho de conclusão:

Alunos:

- Higor Bocutti <a href="google.com"><img alt="Github" title="hrbocutti" src="https://github.githubassets.com/images/icons/emoji/octocat.png" width="25"></a> 
- Geanderson
- Dalmo

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

#### Execute deploy to K8S

Navegar até a pasta kubernets: `../configs/kubernets`

```shell script
$ kubectl apply -f service.yaml
```


## RDS

## Troubleshooting

- No basic auth credentials: [LINK-FIX](https://docs.aws.amazon.com/AmazonECR/latest/userguide/common-errors-docker.html)


