Laboratório AWS Serverless: API Gateway + Lambda + DynamoDB
Este repositório contém um laboratório prático para criar uma API serverless na AWS utilizando CloudFormation, API Gateway, AWS Lambda e Amazon DynamoDB.

Template IaC: serverless-lab.yaml
Guia detalhado: AWS Serverless API Gateway + Lambda + DynamoDB.md
Visão geral: Laboratório sobre Serverless.md
Versão em PDF: Laboratório sobre Serverless.pdf
Arquitetura
API Gateway recebe requisições HTTP.
Lambda processa a lógica de negócio.
DynamoDB armazena os dados.
CloudFormation orquestra a criação de todos os recursos.
Benefícios:

Escalabilidade automática
Baixo custo por uso
Infraestrutura como código (IaC)
Observabilidade integrada (CloudWatch)
Pré-requisitos
Conta AWS ativa
AWS CLI instalada e configurada (aws configure)
Permissão para CloudFormation, Lambda, API Gateway e DynamoDB
Git instalado
Opcional:

Node.js/Python (se quiser alterar a função Lambda)
cURL ou alguma ferramenta de API (HTTPie, Postman)
Deploy com CloudFormation
Clonar o repositório
git clone https://github.com/luiz-star/aws-serverless-lab.git
cd aws-serverless-lab
Validar o template
aws cloudformation validate-template --template-body file://serverless-lab.yaml
Fazer o deploy
aws cloudformation deploy --stack-name serverless-lab --template-file serverless-lab.yaml --capabilities CAPABILITY_IAM
Observações:

Se o template usar parâmetros, adicione: --parameter-overrides Param1=valor1 Param2=valor2
Se preferir outra região: --region us-east-1
Obter as saídas da stack
aws cloudformation describe-stacks --stack-name serverless-lab --query "Stacks[0].Outputs" --output table
Anote a URL do API Gateway (Output: ApiUrl ou semelhante).

Testes rápidos
Exemplos com cURL. Ajuste a URL e paths conforme as saídas do template.

Criar/POST
curl -X POST "$API_URL/items" -H "Content-Type: application/json" -d "{"id":"1","message":"hello"}"

Buscar/GET
curl "$API_URL/items/1"

Listar/GET
curl "$API_URL/items"

Atualizar/PUT
curl -X PUT "$API_URL/items/1" -H "Content-Type: application/json" -d "{"message":"updated"}"

Remover/DELETE
curl -X DELETE "$API_URL/items/1"

Se preferir Postman/Insomnia, importe a URL e crie as rotas correspondentes.

Observabilidade
Logs da Lambda: CloudWatch Logs (grupo /aws/lambda/NOME_DA_FUNCAO)
Métricas: CloudWatch Metrics (Invocations, Errors, Duration, Throttles)
Tracing distribuído (opcional): AWS X-Ray
Dicas:

Use logs estruturados em JSON.
Inclua requestId para correlação entre API Gateway e Lambda.
Custos
Recursos utilizam camadas gratuitas, mas podem gerar custo mínimo:

Lambda: invocações e duração
API Gateway: requests
DynamoDB: leitura/escrita e armazenamento
Apague a stack ao finalizar para evitar cobranças.

Limpeza
aws cloudformation delete-stack --stack-name serverless-lab
aws cloudformation wait stack-delete-complete --stack-name serverless-lab
Estrutura do repositório
serverless-lab.yaml — Template CloudFormation
AWS Serverless API Gateway + Lambda + DynamoDB.md — Passo a passo detalhado
Laboratório sobre Serverless.md — Resumo/guia rápido
Laboratório sobre Serverless.pdf — Documento em PDF
Próximos passos
Adicionar autenticação via Amazon Cognito
Criar estágio de desenvolvimento/produção no API Gateway
Habilitar métricas e alarmes no CloudWatch
Pipeline CI/CD (GitHub Actions) para deploy automático