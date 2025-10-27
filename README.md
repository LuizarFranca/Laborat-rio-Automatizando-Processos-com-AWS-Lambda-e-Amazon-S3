# Laborat-rio-Automatizando-Processos-com-AWS-Lambda-e-Amazon-S3

Integração entre AWS Lambda e S3 — este tema é essencial para dominar automações serverless.

#Objetivo
Consolidar o entendimento sobre automação de tarefas usando AWS Lambda e Amazon S3, implementando uma função que é acionada automaticamente quando um arquivo é carregado em um bucket S3.
O laboratório mostra como criar uma arquitetura serverless, eliminando a necessidade de servidores, e demonstrando eventos e gatilhos nativos da AWS.

Pré-requisitos

Conta AWS ativa.
-Conhecimento básico sobre:
-AWS Lambda (funções, runtime, permissões IAM).
  -Amazon S3 (buckets e eventos).
  -IAM Roles e Policies.
  -AWS CLI configurada (opcional).
-Editor de código (VS Code ou Cloud9).

#Cenário do Laboratório
Você criará:
1.Um bucket S3 chamado lab-lambda-automacao-<seu-nome>.
2.Uma função Lambda em Python que:
 -É acionada automaticamente ao fazer upload de arquivos .txt no bucket.
 -Lê o conteúdo do arquivo. 
 -Gera um novo arquivo resultado.txt no mesmo bucket com um resumo do conteúdo (contagem de linhas e palavras).

 #Arquitetura do Laboratório

         +--------------------+
        |     Usuário        |
        | Uploads arquivo.txt|
        +---------+----------+
                  |
                  v
        +--------------------+
        |    Amazon S3       |
        |  (Evento: PUT *.txt)|
        +---------+----------+
                  |
                  v
        +--------------------+
        |  AWS Lambda        |
        |  (Python Runtime)  |
        |  Lê + processa     |
        |  Cria resultado.txt|
        +--------------------+
                  |
                  v
        +--------------------+
        |    Amazon S3       |
        |  (Arquivo gerado)  |
        +--------------------+



Passo a Passo
1️⃣ Criar o Bucket S3
No console AWS → S3 → Create bucket
 -Nome: lab-lambda-automacao-seunome
 -Região: escolha a mesma da Lambda (ex: us-east-1)
 -Desmarque "Block all public access" (somente se quiser testar acesso público).
 -Clique em Create bucket.


 2️⃣ Criar a Função Lambda
No console AWS → Lambda → Create function
 -Escolha Author from scratch.
 -Nome: processa-arquivos-s3
 -Runtime: Python 3.12
 -Função de execução: Create a new role with basic Lambda permissions.
 -Clique em Create function.

 3️⃣ Código da Função Lambda
Substitua o conteúdo padrão pelo código abaixo:

import boto3
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Captura informações do evento S3
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']

    print(f"Arquivo recebido: {file_key} no bucket {bucket_name}")

    # Baixa o arquivo temporariamente
    tmp_path = f"/tmp/{os.path.basename(file_key)}"
    s3.download_file(bucket_name, file_key, tmp_path)

    # Processa o conteúdo
    with open(tmp_path, 'r') as f:
        lines = f.readlines()
        word_count = sum(len(line.split()) for line in lines)

    # Gera resultado
    resultado = f"Arquivo: {file_key}\nLinhas: {len(lines)}\nPalavras: {word_count}\n"
    result_file = "/tmp/resultado.txt"
    with open(result_file, 'w') as f:
        f.write(resultado)

    # Upload do resultado para o bucket
    result_key = f"resultados/resultado-{os.path.basename(file_key)}"
    s3.upload_file(result_file, bucket_name, result_key)

    print(f"Resultado gerado: {result_key}")
    return {"status": "Processado com sucesso!", "arquivo": result_key}


Clique em Deploy.

4️⃣ Conceder Permissão ao S3 para Invocar a Lambda
No console → Aba Configuration → Permissions → Resource-based policy
Adicione a permissão automática:

aws lambda add-permission \
  --function-name processa-arquivos-s3 \
  --principal s3.amazonaws.com \
  --statement-id s3invoke \
  --action "lambda:InvokeFunction" \
  --source-arn arn:aws:s3:::lab-lambda-automacao-seunome \
  --source-account <ID_DA_SUA_CONTA>

(ou configure direto via interface no próximo passo)

5️⃣ Criar o Trigger (Evento S3 → Lambda)
1.Vá no bucket S3 criado.
2.Acesse Properties → Event notifications → Create event notification.
3.Nome: lambda-trigger-txt.
4.Tipo de evento: All object create events.
5.Filtro:
 -Prefix (opcional): uploads/
 -Suffix: .txt
6.Destino: Lambda function → processa-arquivos-s3.
7.Clique em Save changes.

6️⃣ Testar o Laboratório
1.Crie um arquivo teste.txt com o conteúdo:

mathematica
Este é um teste.
O laboratório de Lambda e S3 está funcionando!

2.Faça upload no bucket S3 → pasta uploads/.
3.Acesse o bucket → verifique a pasta resultados/.
4.Baixe o arquivo resultado-teste.txt → verifique o resumo gerado.

7️⃣ (Opcional) Monitorar Logs no CloudWatch
Acesse CloudWatch → Logs → Log groups → /aws/lambda/processa-arquivos-s3
Veja os logs de execução e depuração.

🧩 Desafio Extra

Expanda o laboratório:
Adicione um S3 Bucket secundário para armazenar os resultados.
Inclua uma notificação por e-mail (SNS) quando o arquivo for processado.
Crie um CloudFormation template para automatizar toda a criação.

