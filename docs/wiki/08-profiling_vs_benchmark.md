# Profiling vs Benchmarking

Ao otimizar uma aplicação ou criar uma extensão em C++, é crucial utilizar a ferramenta e métricas corretas para avaliar o desempenho. Medir o tempo de forma errada pode esconder os verdadeiros custos do sistema.

**Benchmark**: Mede o tempo total e o resultado global de uma aplicação. Seria equivalente à pergunta: "Quanto tempo meu sistema levou para terminar o processo totalmente?".

**Profiling**: Decompõe o custo interno da aplicação. Mede onde o maior custo foi gasto. Responde à pergunta: "Por que o sistema demorou todo esse tempo para executar totalmente?".

## Métricas de custos

Ao analisar um relatório, o tempo gasto em cada etapa pode ser lido de diferentes perspectivas:

**Wall-clock time**: Tempo real que passou no relógio do início até o fim da execução.

**CPU time**: Tempo contabilizado apenas enquanto a CPU esteve ativamente processando a aplicação (ignora tempos de espera, como rede, I/O etc).

Para saber onde o tempo foi gasto, por exemplo se ele foi gasto em uma função ou em outra, dividimos em dois tipos que medem essas especificações:

**Self time**: Conta apenas o tempo de execução da própria função, ignorando tempo gasto em outras funções que ela chamou.

**Inclusive time**: Conta o tempo em que foi gasto chamando outras funções somado ao próprio tempo de demora.

Exemplo prático:

```php
// Função intermediária (Alto inclusive time, baixo self time)

function a() {
    b();
} 

// Hotspot (Alto self time) 

function b() {
    heavy_work();
}
```

## Hotspot e Call Graphs

Durante o profiling, precisamos rastrear o caminho da execução. Para isso, utilizamos uma terminologia específica:

**Hotspot**: É o trecho ou função do código que tem alto custo de execução (tempo ou memória).

**Função intermediária**: Função que tem um alto inclusive time, ou seja, ela em si não é pesada e geralmente serve como uma ponte para outra função.

**Caller**: A função "pai", que faz chamada para outra função.

**Callee**: A função "filha", que é chamada por outra função.

**Calls**: Número total de vezes que uma função foi invocada.

``` 
A()
└── chama B()

Resultado:
A = Caller de B
B = Callee de A
```

A representação visual de todas essas funções (vértices) e de quem chamou quem (arestas) forma o **Call Graph** (Grafo de Chamadas), essencial para mapear o fluxo da aplicação.

## Tipos de Profiler e Fluxos

**Instrumentation Profiler**: Observa os eventos detalhadamente, analisando cada entrada e saída de funções (gera detalhes exatos, callee/caller, call graphs etc). Possui alto overhead, ou seja, deixa a aplicação mais lenta durante a medição. O Xdebug, por exemplo, é um profiler do tipo instrumentação.

**Sampling Profiler**: Gera um relatório estatístico de qual função foi vista ativa por mais tempo. Gera muito menos overhead já que ele não fica acompanhando cada função e seus detalhes, porém perde precisão em conexões específicas.

Para investigar o PHP, o fluxo de trabalho geralmente segue as etapas abaixo:

1. O Xdebug coleta os dados detalhados da execução.
2. Os dados são armazenados em um arquivo de formato padronizado (Cachegrind).
3. Uma interface gráfica (como o QCachegrind) lê o arquivo e interpreta os dados.
4. O desenvolvedor visualiza o Call Graph e as tabelas de Self/Inclusive time para encontrar o Hotspot e planejar a otimização em C++.
