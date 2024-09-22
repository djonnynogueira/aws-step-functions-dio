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

## Benefícios
- Visualização Gráfica: A AWS fornece uma interface gráfica para que você possa visualizar a execução do fluxo de trabalho, ajudando a entender falhas ou gargalos.
- Escalabilidade: Como os serviços da AWS são projetados para escalarem automaticamente, o Step Functions permite que você crie fluxos altamente escaláveis.
- Resiliência: Os mecanismos de retry e tratamento de erros garantem que o sistema seja mais robusto.

#### Exemplo Prático: Criando um Assistente de Delivery com AWS Step Functions
Imagine que você esteja criando um assistente para um serviço de delivery que coordena os seguintes passos:

- Receber o pedido.
- Verificar a disponibilidade do item.
- Solicitar o pagamento.
- Confirmar o pagamento.
- Enviar o pedido para entrega.
- Confirmar a entrega.

Vamos usar o AWS Step Functions para coordenar essas etapas, onde cada estado chamará uma função Lambda para executar uma tarefa específica.

Passo 1: Criar as Funções Lambda
Primeiro, você precisa de várias funções Lambda que executarão as tarefas individuais, como verificar o item, processar pagamento, etc. Vou definir um exemplo básico em Python para cada função:
```
# Verificar disponibilidade do item
def lambda_handler(event, context):
    # Simulação de verificação de estoque
    item_disponivel = True
    if item_disponivel:
        return {"status": "item_disponivel"}
    else:
        return {"status": "item_indisponivel"}

```
```
# Solicitar pagamento
def lambda_handler(event, context):
    # Simulação de solicitação de pagamento
    pagamento_sucesso = True
    if pagamento_sucesso:
        return {"status": "pagamento_confirmado"}
    else:
        return {"status": "pagamento_falhou"}

```

Passo 2: Definir o Fluxo de Trabalho
Agora vamos criar a definição da máquina de estados no AWS Step Functions. Esse fluxo tem várias etapas: verificar a disponibilidade do item, processar o pagamento e confirmar a entrega.

```
{
  "StartAt": "ReceberPedido",
  "States": {
    "ReceberPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-west-2:123456789012:function:receberPedido",
      "Next": "VerificarDisponibilidade"
    },
    "VerificarDisponibilidade": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-west-2:123456789012:function:verificarDisponibilidade",
      "Next": "ChecarDisponibilidade"
    },
    "ChecarDisponibilidade": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.status",
          "StringEquals": "item_disponivel",
          "Next": "ProcessarPagamento"
        }
      ],
      "Default": "EncerrarPorIndisponibilidade"
    },
    "ProcessarPagamento": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-west-2:123456789012:function:processarPagamento",
      "Next": "ChecarPagamento"
    },
    "ChecarPagamento": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.status",
          "StringEquals": "pagamento_confirmado",
          "Next": "EnviarParaEntrega"
        }
      ],
      "Default": "EncerrarPorFalhaNoPagamento"
    },
    "EnviarParaEntrega": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-west-2:123456789012:function:enviarParaEntrega",
      "Next": "ConfirmarEntrega"
    },
    "ConfirmarEntrega": {
      "Type": "Succeed"
    },
    "EncerrarPorIndisponibilidade": {
      "Type": "Fail",
      "Cause": "Item Indisponível",
      "Error": "Item não está disponível"
    },
    "EncerrarPorFalhaNoPagamento": {
      "Type": "Fail",
      "Cause": "Pagamento Falhou",
      "Error": "Erro de pagamento"
    }
  }
}
```

Passo 3: Implantação e Execução
- Criação do fluxo: No console AWS Step Functions, crie uma nova máquina de estados com a definição acima.
- Integração com Lambda: As funções Lambda devem estar ativas e associadas aos seus respectivos ARNs (identificadores) nos estados do fluxo.
- Execução: Após a implantação, você pode iniciar a execução do fluxo de trabalho manualmente ou acoplá-lo a um serviço externo, como uma API Gateway, que acionará o fluxo de trabalho automaticamente.

### Vantagens do AWS Step Functions
- Coordenação Simples: Reduz a complexidade de lidar com a integração entre múltiplos serviços da AWS e facilita o gerenciamento de fluxos de trabalho distribuídos.
- Manutenção Reduzida: Com Step Functions, você gerencia o fluxo e tratamento de erros de forma centralizada, sem precisar escrever código para cada detalhe de controle de fluxo.
- Auditabilidade: Cada execução do fluxo de trabalho é rastreada e registrada, permitindo fácil diagnóstico em caso de falhas.
