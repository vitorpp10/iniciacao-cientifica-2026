# Visão geral do projeto e do que foi estudado

Este documento consolida o que já foi escrito, aprendido e planejado no projeto de pesquisa. Ele funciona como mapa de leitura do repositório e como resumo do estado atual do IC.

## 1. Tema da pesquisa

O projeto investiga como reduzir a latência e o consumo de recursos em módulos críticos do ecossistema SEI usando extensões nativas em C++ integradas ao motor PHP/Zend Engine.

A ideia central é simples e objetiva:

- o SEI é um sistema de gestão pública com etapas pesadas de conversão de documentos, validação criptográfica e indexação;
- essas tarefas têm alta carga de processamento em PHP puro;
- o motor Zend e o tipo dinâmico do PHP criam overhead por conversão de tipos, alocações, parsing e gerenciamento de memória;
- extensões em C++ podem mover parte do processamento para um caminho nativo, mais rápido e mais previsível.

A pesquisa evita depender do código-fonte proprietário do SEI e trabalha com uma reprodução de padrões reais em laboratório, usando Docker e uma base de comparação controlada.

## 2. Objetivo do IC

O objetivo geral está descrito em `readme.md`:

- desenvolver um framework de otimização em baixo nível para sistemas PHP;
- focar no ecossistema SEI;
- criar extensões C++ para o Zend Engine;
- comparar PHP puro vs. extensão nativa;
- quantificar ganho de desempenho e eficiência com base na Lei de Amdahl.

Em resumo: o projeto busca demonstrar que parte do processamento pesado pode sair do código PHP interpretado e passar para módulos nativos, reduzindo tempo e memória.

## 3. O que o repositório já mostra

A estrutura do projeto foi organizada em blocos bem definidos:

- `readme.md`: visão geral do IC, objetivos, metodologia e cronograma;
- `structure.md`: mapa dos diretórios e como cada área do projeto se conecta;
- `docs/wiki/`: material técnico de estudo interno sobre PHP internals e otimização;
- `docs/info/`: documentação de planejamento, estratégia, roadmap e materiais de reunião;
- `src/`: código fonte em PHP e C++;
- `benchmarks/`: scripts e dados de benchmark;
- `docker/`: infraestrutura para montar o ambiente SEI em containers;
- `tests/`: testes de validação das extensões;
- `result_analysis/`: análise dos resultados e gráficos.

O repositório também referencia explicitamente o ambiente oficial do SEI em `sei-docker-main/`, usado como base de estudo e referência, mas não como área de alteração direta.

## 4. O que foi estudado no wiki

O material em `docs/wiki/` já cobre os pilares teóricos e técnicos do projeto.

### 4.1. Lei de Amdahl

Arquivo principal: `docs/wiki/02-lei-amdahl-estudo.md`

Aqui ficou claro que a pesquisa não trata de “acelerar tudo de uma vez”, mas de entender o limite real do ganho.

Principais aprendizados:

- a Lei de Amdahl mede o speedup global de uma otimização parcial;
- o ganho real depende da fração do sistema que pode ser melhorada;
- mesmo acelerando um gargalo muito forte, o restante sequencial continua limitando o total;
- em arquiteturas reais, a otimização deve olhar o sistema como um conjunto de módulos e não como um único bloco.

O documento mostra exemplos numéricos com:

- criptografia;
- JOD;
- cenário combinado com múltiplas otimizações.

A conclusão prática é que o projeto precisa escolher muito bem quais módulos são realmente críticos e o quanto cada um pesa no tempo total.

### 4.2. Arquitetura da Zend Engine

Arquivo: `docs/wiki/03-zend-engine-arquitetura.md`

Esse estudo detalha como o PHP processa código antes de executá-lo. O ponto central é que, para introduzir uma nova sintaxe ou um novo comportamento, o motor precisa passar por etapas de:

- Lexer;
- Parser;
- AST;
- Compilation;
- Zend VM.

O arquivo usa o exemplo fictício do operador `|>` para mostrar como o motor reconhece a sintaxe, monta a árvore de expressão, gera opcodes e entrega a instrução para a máquina virtual.

Essa parte foi muito importante porque conecta a pesquisa à ideia de criar extensões nativas e entender como a Zend Engine interpreta operações em baixo nível.

### 4.3. Estrutura da zval

Arquivo: `docs/wiki/04-zval-estrutura.md`

Esse estudo explica a base da representação de variáveis no PHP.

Aprendizado principal:

- a zval é a estrutura que representa qualquer valor no motor;
- ela guarda tipo, valor e metadados;
- PHP 5 e PHP 7+ mudaram bastante essa arquitetura;
- o uso de contagem de referências e estruturas complexas causou overhead;
- o PHP moderno melhorou muito, mas ainda há custo para tipos dinâmicos e estruturas complexas.

Também foi discutido o conceito de strings em `zend_string`, copy-on-write e o impacto do coletor de lixo em versões antigas.

### 4.4. Zend Memory Manager

Arquivo: `docs/wiki/05-zend-memory-manager.md`

Este documento descreve a camada de gerenciamento de memória do PHP.

O principal ponto é que o PHP não usa malloc/free padrão para tudo. Ele usa o Zend Memory Manager para:

- reduzir custo de alocação e liberação;
- controlar request-bound allocations;
- evitar vazamentos em requisições web;
- organizar pequenos blocos em bins e páginas fixas.

Esse estudo ajuda a entender por que operações em C++ podem ser mais eficientes: o custo de memória e a organização do layout em heap e stack passam a ser controlados melhor em código nativo.

### 4.5. Type Juggling

Arquivo: `docs/wiki/06.type-juggling.md`

Esse tema é essencial para a pesquisa e conecta diretamente com o gargalo de performance.

O documento mostra que:

- o PHP converte tipos implicitamente;
- strings numéricas podem ser tratadas como inteiros ou floats no contexto da operação;
- as operações em fast path são mais rápidas;
- as operações em slow path exigem inspeção, parsing e conversão;
- em loops e processamento massivo, esse overhead se torna relevante.

Essa é a base da hipótese do projeto: mover tarefas repetitivas para C++ reduz o custo de conversão e validação que o PHP faz em cada operação.

### 4.6. Protocolo de pesquisa

Arquivo: `docs/wiki/protocolo-pesquisa.md`

Este documento formaliza a base experimental do IC.

Ele define:

- hipótese nula e hipótese alternativa;
- variável independente: implementação PHP puro vs. extensão C++;
- variáveis dependentes: tempo, CPU, memória e throughput;
- a ideia de propor um ganho mínimo de 2x.

O protocolo ainda está em andamento, mas já organiza bem a direção do estudo experimental.

## 5. O que já ficou claro sobre o projeto

Depois de revisar o repositório e a documentação, o projeto está bem estruturado em torno de cinco ideias centrais:

1. O gargalo real não é “PHP em si”, e sim o custo de abstrações e processamento repetitivo em operações pesadas.
2. A Zend Engine e o type juggling criam overhead que pode ser eliminado ou reduzido em módulos nativos.
3. A Lei de Amdahl mostra que ganhos locais podem ter impacto global limitado, por isso a escolha dos módulos é crítica.
4. O ambiente SEI em Docker é a base experimental para simular o comportamento do sistema real.
5. A pesquisa tem uma linha clara: baseline em PHP, protótipo em C++, benchmarking e validação com métricas objetivas.

## 6. O que já foi feito e o que falta

### Já foi feito

- estudo do problema e escolhas metodológicas;
- organização do repositório e estrutura de documentação;
- estudo de Lei de Amdahl;
- estudos sobre Zend Engine, zval, memory manager e type juggling;
- definição do protocolo de pesquisa;
- preparação da infraestrutura Docker/SEI como referência;
- planejamento do benchmark e das extensões em C++.

### Ainda precisa ser concluído

- formalizar o protocolo completo;
- implementar os simuladores PHP para baseline;
- medir gargalos nas rotinas JOD, crypto e Solr;
- criar as extensões C++ e integrar ao ambiente;
- executar comparações reais de desempenho;
- coletar resultados устойчивes com CPU, memória e tempo;
- analisar se o ganho atende ao threshold esperado.

## 7. Conclusão resumida

O projeto já avançou bastante no lado conceitual e organizacional. O que foi escrito até agora mostra uma investigação sólida sobre:

- desempenho em PHP;
- arquitetura da Zend Engine;
- custo do tipo dinâmico;
- memória e alocação;
- benchmark e análise comparativa;
- impacto real de módulos nativos em C++.

A pesquisa está madura o suficiente para entrar na fase prática: montar o benchmark, testar os módulos, medir os gargalos e validar se a extensão nativa consegue entregar ganho real de desempenho no ecossistema SEI.

## 8. Links úteis dentro do projeto

- `readme.md`
- `structure.md`
- `docs/wiki/02-lei-amdahl-estudo.md`
- `docs/wiki/03-zend-engine-arquitetura.md`
- `docs/wiki/04-zval-estrutura.md`
- `docs/wiki/05-zend-memory-manager.md`
- `docs/wiki/06.type-juggling.md`
- `docs/wiki/protocolo-pesquisa.md`

Este resumo deve servir como ponto de partida para consolidar a narrativa do IC, tanto para o desenvolvimento técnico quanto para a escrita final do relatório e do artigo científico.
