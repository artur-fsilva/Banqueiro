# Banqueiro - Algoritmo de Prevenção de Deadlock

Implementação em C# do Algoritmo do Banqueiro (Banker's Algorithm) com suporte a múltiplas threads para detecção e prevenção de deadlock em sistemas operacionais.

## Descrição

O Algoritmo do Banqueiro é um método de controle de recursos usado para prevenir deadlock. Ele funciona concedendo recursos apenas quando o sistema permanece em um estado seguro, onde existe uma sequência de processos que conseguem terminar sem esperar infinitamente por recursos.

Este projeto implementa o algoritmo em C# com suporte a múltiplas threads. Cada thread cliente requisita recursos, os utiliza e depois os libera de forma sincronizada com o algoritmo do banqueiro.

## Características Principais

- Thread-safe com uso de locks (Monitor)
- Implementa o algoritmo de verificação de segurança
- Simula múltiplos clientes concorrentes requisitando recursos
- Exibe estatísticas de requisições (aprovadas, negadas, totais)

## Requisitos

- .NET Framework 4.7.2 ou superior
- C# 7.1+
- Visual Studio (recomendado) ou qualquer IDE compatível com .NET

### Dependências

- System.Threading (padrão do .NET)
- System (padrão do .NET)

## Estrutura do Projeto

```
Banqueiro/
├── README.md           # Este arquivo
├── Banqueiro.cs        # Implementação do algoritmo
├── Logger.cs           # Classe para logging
├── Configuracao.cs     # Configurações
└── Program.cs          # Ponto de entrada
```

## Como Executar

### 1. Clonar o Repositório

```
git clone <url-do-repositorio>
cd Banqueiro
```

### 2. Compilar o Projeto

#### Usando Visual Studio

1. Abra Banqueiro.sln (ou crie um novo projeto)
2. Compile: Ctrl+Shift+B
3. Execute: Ctrl+F5 ou F5

#### Usando Linha de Comando (.NET CLI)

```
dotnet build
dotnet run
```

#### Usando csc (C# Compiler)

```
csc Banqueiro.cs Logger.cs Configuracao.cs Program.cs -out:Banqueiro.exe
Banqueiro.exe
```

### 3. Executar com Parâmetros

```
Banqueiro.exe 5 10 15
```

- 5: Número de clientes (threads)
- 10: Quantidade de recurso tipo 0
- 15: Quantidade de recurso tipo 1
- (Adicione mais parâmetros para cada tipo de recurso)

## Estruturas de Dados

### Matrizes do Algoritmo

| Matriz | Dimensão | Descrição |
|--------|----------|-----------|
| disponivel | [m] | Recursos livres no sistema |
| maximo | [n×m] | Demanda máxima declarada por cada cliente |
| alocacao | [n×m] | Recursos atualmente alocados a cada cliente |
| necessidade | [n×m] | Recursos ainda necessários (máximo - alocação) |

Onde:
- n = número de clientes
- m = número de tipos de recursos

### Exemplo de Saída

```
══════════════════════════════════════════════════════════════════════════════
  ALGORITMO DO BANQUEIRO — Simulação Multithreaded
══════════════════════════════════════════════════════════════════════════════

Disponível : [ 10,  15]

       R0  R1
C0  Max:[  8, 12]  Aloc:[  2,  3]  Nec:[  6,  9]
C1  Max:[  7, 10]  Aloc:[  1,  4]  Nec:[  6,  6]
C2  Max:[  9, 14]  Aloc:[  3,  2]  Nec:[  6, 12]
```

## Sincronização

### Lock (Monitor)

```
private readonly object _lock = new object();

lock (_lock)
{
    // Operações thread-safe aqui
    // Apenas uma thread por vez executa este bloco
}
```

### Operações Atômicas

Usa Interlocked.Increment() para incrementar contadores de forma segura sem deadlock.

## Algoritmo de Segurança

O algoritmo verifica se um estado é seguro simulando a execução de todos os clientes:

```
1. Cria cópia do vetor disponível (trabalho)
2. Para cada cliente i não finalizado:
   - Se necessidade[i] <= trabalho, simula conclusão
   - Adiciona alocação[i] a trabalho
3. Se todos os clientes conseguem terminar → ESTADO SEGURO
4. Caso contrário → ESTADO INSEGURO
```

## Fluxo de Execução

```
Inicializar Sistema (Gerar máximos e alocações)
              |
              ▼
Iniciar N Threads Clientes
              |
              ▼
Para Cada Iteração por Cliente:
  1. Gerar requisição aleatória
  2. Solicitar recursos
  3. Se aprovado: usar e liberar
  4. Se negado: aguardar e retry
              |
              ▼
Liberar Todos os Recursos (Limpeza Final)
              |
              ▼
Exibir Resumo de Estatísticas
```

## Métodos Principais

### SolicitarRecursos(int clienteId, int[] requisicao)

- Valida se requisição <= necessidade
- Verifica disponibilidade
- Realiza alocação tentativa
- Executa algoritmo de segurança
- Confirma ou reverte a alocação

Retorno:
- 0: Requisição aprovada
- -1: Requisição negada

### LiberarRecursos(int clienteId, int[] liberacao)

- Valida se liberação <= alocação
- Devolve recursos ao pool
- Atualiza matrizes

Retorno:
- 0: Liberação bem-sucedida
- -1: Liberação inválida

### EstadoSeguro()

- Simula execução de todos os clientes
- Detecta se existe sequência segura

Retorno:
- true: Estado seguro
- false: Estado inseguro (risco de deadlock)

## Estatísticas

A simulação exibe:
- Total de solicitações: Todas as requisições (aprovadas + negadas)
- Aprovadas: Requisições que resultaram em alocação segura
- Negadas: Requisições recusadas (insuficiência ou estado inseguro)
- Taxa de aprovação: Aprovadas / Total * 100%

## Configurações

O arquivo Configuracao.cs contém os parâmetros usados na simulação:

```
public const int ITERACOES_POR_CLIENTE = 20;     // Quantas vezes cada cliente tenta solicitar
public const int ESPERA_MIN_MS = 50;             // Tempo mínimo de espera em ms
public const int ESPERA_MAX_MS = 500;            // Tempo máximo de espera em ms
public const int ESPERA_NEGACAO_MS = 100;        // Espera quando requisição é negada
public const int IMPRIMIR_ESTADO_A_CADA = 5;     // Exibe estado a cada N operações
```

## Referências

- Silberschatz, A., Galvin, P. B., & Gagne, G. - Operating System Concepts (10ª ed.)
- Tanenbaum, A. S., & Bos, H. - Modern Operating Systems (4ª ed.)

## Troubleshooting

### Problema: Muitas requisições negadas

Solução: Aumente recursosIniciais ou reduza ITERACOES_POR_CLIENTE

### Problema: Programa trava

Solução: Verifique se há deadlock em Logger.cs. Use timeout para threads.

### Problema: Erros de compilação

Solução: Certifique-se que Logger.cs e Configuracao.cs existem com as classes esperadas

## Notas

Este é um trabalho prático da disciplina de Sistemas Operacionais do 3º período de Sistemas da Informação.

