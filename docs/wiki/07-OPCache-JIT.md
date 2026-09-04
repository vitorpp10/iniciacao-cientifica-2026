# OPCache e JIT

Para entender por que uma extensão em C++ é necessária em cenários críticos de CPU, precisamos separar otimizações nativas do PHP em duas frentes: OPCache (Compilação) e JIT (Execução).

## OPCache 

Melhora o desempenho armazenando o bytecode (opcodes) de scripts pré-compilados em memória compartilhada. Isso remove a necessidade do motor carregar, analisar (parse) e compilar o script a cada nova requisição.

A limitação fundamental do OPCache é que ele não acelera códigos CPU-Bound (operações que exigem muita CPU). O que estiver escrito no script não sofre mutaçao de performance lógica; ele apenas reduz o overhead de compilação repetida. O OPCache não elimina o custo de executar esses opcodes, ele apenas os entrega mais rápido para o executor.

```
SOURCE PHP
│
├─ lexer / parser / compilação repetida
│       ↑
│    OPcache ataca principalmente aqui (Evita o retrabalho)
│
└─ OPCODES
        ↓
     execução
        ↓
  zval / operações / memória / chamadas etc.
```

## JIT (Just-In-Time Compiler)

Enquanto o OPCache para na entrega dos opcodes, o JIT tenta evitar parte do custo de interpretar esses opcodes transformando trechos específicos diretamente em código de máquina nativa.

No entanto, o JIT não compila o script inteiro às cegas. Ele possui regras estritas de funcionamento baseadas em observações:

- Hot code: É o trecho de código que é executado com frequência suficiente para que o custo de otimizá-lo compense. O JIT só acelera o que está efetivamente rodando repetidas vezes e que ele consiga otimizar.

- Tracing JIT: É a estratégia que observa ativamente os "caminhos" de execução do código. Quando o Tracing JIT percebe que certas funções ou caminhos executados pela Zend VM aparecem muitas vezes, ele extrai esse caminho e o compila para código nativo.

- Warm-up: É o tempo de "aquecimento". Representa a janela de tempo entre as execuções iniciais do código até o momento em que o Tracing JIT identifica que a repetição é alta o suficiente para valer a pena convertê-lo. Durante a warm-up, o código continua rodando de forma interpretada.