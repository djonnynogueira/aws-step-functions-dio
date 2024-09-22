# AWS Step Functions

O AWS Step Functions é um serviço de orquestração de fluxos de trabalho baseado em estados, utilizado para coordenar diferentes serviços da AWS e criar aplicativos distribuídos. Ele permite a criação e execução de fluxos de trabalho que coordenam múltiplas tarefas, como funções Lambda, operações de bancos de dados, ou chamadas para APIs de terceiros. Esses fluxos são definidos como máquinas de estados, que modelam a lógica do aplicativo em uma sequência de passos (steps), permitindo que cada estado seja responsável por uma tarefa específica.

O Step Functions utiliza um modelo visual onde você pode desenhar e configurar os diferentes estados (steps) por meio de um arquivo de definição no formato Amazon States Language (ASL), que é baseado em JSON. Com ele, você pode executar tarefas de forma sequencial ou paralela, gerenciar erros e exceções, e definir políticas de retry.

Conceitos Principais
- State (Estado): Representa uma unidade de trabalho. Cada estado pode executar uma função, fazer uma escolha (branching), aguardar um tempo, ou até mesmo encerrar o fluxo de trabalho.

- Tasks (Tarefas): São estados que realizam algum tipo de processamento. As tarefas podem ser desde a execução de uma função Lambda até interações com outros serviços da AWS como S3, DynamoDB, SNS, etc.

- Transitions (Transições): Conectam os diferentes estados dentro de um fluxo de trabalho. Após a conclusão de um estado, o fluxo de trabalho transita para o próximo estado com base nas regras definidas.

- Catch e Retry: Permitem capturar e lidar com erros, além de definir quantas vezes uma tarefa deve ser reexecutada antes de falhar.

- Parallel (Execuções paralelas): Executa múltiplas tarefas em paralelo.

- Wait: Um estado que adiciona um tempo de espera no fluxo de trabalho.

Arquitetura de um Fluxo de Trabalho
Cada máquina de estados é definida em um arquivo JSON, onde você especifica:

1. StartAt: Define o estado inicial.
2. States: Lista de estados e suas transições, ações e tratamentos de erro.

Exemplo básico de um fluxo de trabalho:
```{
  "Comment": "Exemplo de fluxo simples",
  "StartAt": "Início",
  "States": {
    "Início": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-west-2:123456789012:function:meuLambda",
      "Next": "Processamento"
    },
    "Processamento": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.status",
          "StringEquals": "sucesso",
          "Next": "Concluir"
        }
      ],
      "Default": "Falha"
    },
    "Falha": {
      "Type": "Fail",
      "Cause": "Erro",
      "Error": "Processo Falhou"
    },
    "Concluir": {
      "Type": "Succeed"
    }
  }
}
```
