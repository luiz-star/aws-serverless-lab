AWS Serverless: API Gateway + Lambda + DynamoDB
Objetivo do Lab
Você vai:
●	Criar uma arquitetura serverless na AWS usando CloudFormation.
●	Fazer um endpoint HTTP gravar mensagens em uma tabela DynamoDB via função Lambda.
________________________________________
Pré-requisitos
●	Conta AWS Free Tier.
●	Acesso ao AWS CloudShell.

Passo 1 – Baixe o template CloudFormation
1.	Clique em CloudShell no topo direito do Console AWS.
2.	No terminal do CloudShell, crie um arquivo chamado serverless-lab.yaml com o comando abaixo:
nano serverless-lab.yaml
3.	Copie e cole o código abaixo no editor (aperte Ctrl+Shift+V para colar):
Os comentários estão precedidos de # (YAML não aceita comentários dentro de blocos, então os comentários estão antes ou ao lado das linhas).
Observação: o código explicado linha a linha e bloco a bloco está em anexo a este documento em um bloco de notas chamado:  Código_Lab Serverless, você só precisa baixar > abrir com Bloco de notas >copiar e colocar de acordo com a aula.


4.	Salve e saia do editor: 
●	Pressione Ctrl+O (para salvar), depois Enter.
●	Pressione Ctrl+X para sair do nano.

Passo 2 – Crie a stack CloudFormation
Execute o comando abaixo no console CloudShell:
aws cloudformation create-stack \
  --stack-name serverless-lab \
  --template-body file://serverless-lab.yaml \
  --capabilities CAPABILITY_NAMED_IAM
●	Aguarde alguns minutos até a stack ser criada.
●	Você pode acompanhar o progresso no Console → CloudFormation.

Passo 3 – Pegue o endpoint da API
Quando a stack estiver criada, execute:

aws cloudformation describe-stacks \
  --stack-name serverless-lab \
  --query "Stacks[0].Outputs[?OutputKey=='ApiEndpoint'].OutputValue" \
  --output text
●	Guarde o link que aparecer (exemplo: https://xxxxxx.execute-api.sa-east-1.amazonaws.com/prod/messages).
________________________________________
Passo 4 – Teste a API
Envie uma mensagem usando o comando abaixo (troque <SEU_ENDPOINT> pelo link que você pegou acima):




curl -X POST "<SEU_ENDPOINT>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Mensagem de teste Serverless!"}'

Se tudo deu certo, você verá uma resposta parecida com:
{"message": "Mensagem salva!", "item": {"id": "...", "message": "Mensagem de teste Serverless!"}}
Passo 5 – Veja o resultado no DynamoDB
1.	No Console AWS, procure por DynamoDB.
2.	Clique em Tabelas > Messages > Explorar tabela.
3.	Veja a mensagem que você enviou!
Pronto!

4.	✅ Você acabou de construir e testar uma solução serverless real na AWS usando apenas um template e alguns comandos!
________________________________________
5.	DICA: Sempre apague a stack ao final dos testes para não gerar custo fora do Free Tier.



 
