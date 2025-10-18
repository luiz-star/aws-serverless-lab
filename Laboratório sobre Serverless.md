AWSTemplateFormatVersion: '2010-09-09'
# Versão do formato do template CloudFormation.

Description: LAB 1 - Arquitetura Serverless (API Gateway + Lambda + DynamoDB)
# Descrição do que será criado.

Resources:
  # 1) Tabela DynamoDB para armazenar as mensagens
  MessagesTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Messages  # Nome da tabela
      AttributeDefinitions:
        - AttributeName: id      # Definindo o atributo 'id'
          AttributeType: S       # Tipo String
      KeySchema:
        - AttributeName: id      # Chave primária vai ser o 'id'
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST  # Pagamento sob demanda (ideal para testes/labs)

  # 2) Role (permissão) para a função Lambda acessar DynamoDB e gerar logs
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: LambdaDynamoDBRole  # Nome visível da Role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com  # Lambda pode assumir esta role
            Action: sts:AssumeRole
      Policies:
        - PolicyName: LambdaDynamoDBPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:PutItem        # Permite inserir itens na tabela
                Resource: !GetAtt MessagesTable.Arn  # Só na tabela criada
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup     # Permite criar grupos de log
                  - logs:CreateLogStream    # Permite criar streams de log
                  - logs:PutLogEvents       # Permite gravar eventos de log
                Resource: "*"

  # 3) Função Lambda que insere a mensagem recebida no DynamoDB
  SaveMessageFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: SaveMessageFunction
      Runtime: python3.9                 # Linguagem usada: Python 3.9
      Handler: index.lambda_handler      # Nome do arquivo (index) e função (lambda_handler)
      Role: !GetAtt LambdaRole.Arn       # Referência à role criada acima
      Code:
        ZipFile: |                      # Código da função embutido no template
          import json
          import boto3
          import uuid

          dynamodb = boto3.resource('dynamodb')
          table = dynamodb.Table('Messages')

          def lambda_handler(event, context):
              body = json.loads(event['body'])            # Lê o corpo da requisição
              message = body.get('message', '')           # Pega o campo 'message'
              item = {
                  'id': str(uuid.uuid4()),                # Cria um ID único
                  'message': message                      # Salva a mensagem recebida
              }
              table.put_item(Item=item)                   # Insere na tabela DynamoDB
              return {
                  'statusCode': 200,                      # Retorna sucesso
                  'body': json.dumps({'message': 'Mensagem salva!', 'item': item})
              }

  # 4) API Gateway HTTP para expor um endpoint público
  HttpApi:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      Name: messages-api
      ProtocolType: HTTP           # Tipo HTTP (mais simples que REST)

  # 5) Integração do API Gateway com a função Lambda
  LambdaIntegration:
    Type: AWS::ApiGatewayV2::Integration
    Properties:
      ApiId: !Ref HttpApi
      IntegrationType: AWS_PROXY
      IntegrationUri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${SaveMessageFunction.Arn}/invocations
      PayloadFormatVersion: '2.0'

  # 6) Rota da API: define o caminho e método HTTP que chama a Lambda
  ApiRoute:
    Type: AWS::ApiGatewayV2::Route
    Properties:
      ApiId: !Ref HttpApi
      RouteKey: POST /messages   # Quando alguém faz POST em /messages...
      Target: !Sub integrations/${LambdaIntegration}  # ... chama a integração com a Lambda

  # 7) Stage da API: define o ambiente (prod) e ativa deploy automático
  ApiStage:
    Type: AWS::ApiGatewayV2::Stage
    Properties:
      ApiId: !Ref HttpApi
      StageName: prod
      AutoDeploy: true           # Deploy é feito automaticamente

  # 8) Permissão para a API Gateway poder invocar a função Lambda
  LambdaApiInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref SaveMessageFunction
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${HttpApi}/*/POST/messages

Outputs:
  # Mostra ao final da criação o endpoint da API para facilitar o teste
  ApiEndpoint:
    Description: Endpoint HTTP da API para testar o POST /messages
    Value: !Sub "https://${HttpApi}.execute-api.${AWS::Region}.amazonaws.com/prod/messages"
  # Mostra o nome da tabela DynamoDB criada
  TableName:
    Description: Nome da tabela DynamoDB criada
    Value: !Ref MessagesTable
