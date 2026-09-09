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

## Previsibilidade: JIT e OPcache

Para que o JIT consiga compilar um código nativo eficiente, ele depende de um fator crucial: a previsibilidade. O comportamento das variáveis durante a execução dita se o JIT vai acelerar o sistema ou se vai ser parado em verificações de segurança (guards).

**Type Stability (Estabilidade de Tipos)**: Ocorre quando uma função ou loop tem variáveis com tipos constantes e previsíveis. Como o tipo não muda, o JIT assume que pode gerar um código de máquina direto e altamente otimizado para aquele tipo específico.

**Type Instability (Instabilidade de Tipos)**: Ocorre quando as variáveis mudam de tipo dinamicamente (frequentemente causado por type juggling e coerções). Como o motor não consegue prever o que vai acontecer no runtime, o JIT é obrigado a inserir guards (verificações extras) e criar caminhos alternativos, perdendo grande parte do ganho de memória.

Exemplo prático de estabilidade vs instabilidade:

```php
// Type Stability 
// O tipo é constante, o JIT gera código de máquina nativo rápido.
for ($i = 0; $i < 10000; $i++) {
	$x = $integer + 1;
}

// Type Instability 
// A variável alterna entre string e int. O JIT sofre com coerções e caminhos dinâmicos.
for ($i = 0; $i < 10000; $i++) {
	if ($i % 2 == 0) {
		$value = "10";
	} else {
		$value = 10;
	}
	$x = $value + 10;
}
```

### Cold Cache vs Warm Cache

Devido à natureza do Tracing JIT e do OPcache, avaliar o desempenho de um script PHP moderno exige metodologia. O estado da memória no momento do teste altera os resultados:

**Cold Cache (Cache Frio)**: Ocorre na primeira execução ou quando o arquivo ainda não está cacheado na memória. O JIT ainda não possui histórico de uso, ou seja, não mapeou os hot paths (caminhos quentes) nem os hot loops, e não consegue fazer previsões precisas.

**Warm Cache (Cache Quente)**: O sistema já executou o código diversas vezes (já passou pelo warm-up). Os hot paths já foram identificados, o bytecode já está no OPcache e o JIT já compilou os trechos críticos para código nativo. É aqui que o motor está no seu máximo.

Por isso não se pode medir o tempo real de execução de uma aplicação baseando-se apenas na primeira rodada (Cold Cache). Para obter métricas científicas válidas de otimização de CPU, é obrigatório separar o tempo da "otimização inicial" do tempo de "execução em massa" (Warm Cache), executando ciclos de aquecimento antes da medição final.
