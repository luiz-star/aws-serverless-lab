## Laboratório AWS Serverless: API Gateway + Lambda + DynamoDB
Este repositório contém um laboratório prático para criar uma API serverless na AWS utilizando CloudFormation, API Gateway, AWS Lambda e Amazon DynamoDB.

* Template IaC: serverless-lab.yaml

* Guia detalhado: AWS Serverless API Gateway + Lambda + DynamoDB.md

* Visão geral: Laboratório sobre Serverless.md

* Versão em PDF: Laboratório sobre Serverless.pdf


## Arquitetura

* API Gateway recebe requisições HTTP

* Lambda processa a lógica de negócio

* DynamoDB armazena os dados

* CloudFormation orquestra a criação de todos os recursos

# Benefícios:

* Escalabilidade automática
  
* Baixo custo por uso
  
* Infraestrutura como código (IaC)
  
* Observabilidade integrada (CloudWatch)

  
## Pré-requisitos

* Conta AWS ativa
  
* AWS CLI instalada e configurada: aws configure
  
* Permissões para CloudFormation, Lambda, API Gateway e DynamoDB
  
* Git instalado
  
# Opcional:

* Node.js/Python (se quiser alterar a função Lambda)
* cURL/HTTPie/Postman para testes de API

  
## Como fazer o deploy

1- Clonar o repositório

git clone https://github.com/luiz-star/aws-serverless-lab.git
cd aws-serverless-lab

2- Validar o template

aws cloudformation validate-template --template-body file://serverless-lab.yaml

3- Deploy

aws cloudformation deploy \
  --stack-name serverless-lab \
  --template-file serverless-lab.yaml \
  --capabilities CAPABILITY_IAM
  
Observações:

* Parâmetros: use --parameter-overrides Param1=valor1 Param2=valor2
* Região: adicione --region us-east-1 (ou a sua)
  
4- Obter as saídas (URL do API)

aws cloudformation describe-stacks \
  --stack-name serverless-lab \
  --query "Stacks[0].Outputs" \
  --output table

  
## Testes rápidos
Defina API_URL com a URL do API Gateway obtida nas saídas.

* Criar (POST)

curl -X POST "$API_URL/items" \
  -H "Content-Type: application/json" \
  -d "{\"id\":\"1\",\"message\":\"hello\"}"
  
* Buscar (GET)

curl "$API_URL/items/1"

* Listar (GET)

curl "$API_URL/items"

* Atualizar (PUT)

curl -X PUT "$API_URL/items/1" \
  -H "Content-Type: application/json" \
  -d "{\"message\":\"updated\"}"
  
* Remover (DELETE)

curl -X DELETE "$API_URL/items/1"


## Observabilidade

* Logs: CloudWatch Logs — /aws/lambda/NOME_DA_FUNCAO
* Métricas: CloudWatch Metrics (Invocations, Errors, Duration, Throttles)
* Tracing opcional: AWS X-Ray

  
## Limpeza

aws cloudformation delete-stack --stack-name serverless-lab
aws cloudformation wait stack-delete-complete --stack-name serverless-lab

## Estrutura

* serverless-lab.yaml — Template CloudFormation
  
* AWS Serverless API Gateway + Lambda + DynamoDB.md — Passo a passo
  
* Laboratório sobre Serverless.md — Resumo
  
* Laboratório sobre Serverless.pdf — PDF
  
## Próximos passos

* Autenticação via Cognito

* Estágios dev/prod no API Gateway

* Alarmes no CloudWatch

* CI/CD com GitHub Actions
