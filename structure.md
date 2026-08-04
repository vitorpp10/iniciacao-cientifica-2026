# Arquitetura do Repositório

Este documento descreve a finalidade de cada pasta e arquivo da pesquisa.
**Atualizado em:** 04/08/2026

---

## Raiz do Projeto

```
ic-sei-cpp-2026/
├── readme.md          ← Apresentação geral: objetivo, metodologia, cronograma
├── structure.md       ← Este arquivo: mapa de cada pasta
├── .gitignore         ← Ignora binários C++, cache PHP, arquivos IDE
└── .vscode/           ← Configurações do VS Code (C++ IntelliSense, launch)
```

---

## docs/ — Documentação Científica

Todos os documentos de pesquisa, relatórios, análises e referências.

```
docs/
│
│ ── Documentos de planejamento (criados em ago/2026)
├── mapa-sei-docker.md             ← Árvore completa do sei-docker-main + conexões com IC
├── roadmap-vinculado.md           ← Roadmap 52 semanas vinculado ao sei-docker-main
├── estrategia-mencao-honrosa.md   ← Estratégia para premiação (EnCUCA + UnB)
├── guia-implementacao.md          ← O que implementar, quando e como
├── proxima-reuniao-professor.md   ← Pauta e material para reunião com orientador
│
│ ── Documentos técnicos (a criar ao longo do IC)
├── 00-protocolo-pesquisa.md       ← Hipóteses, variáveis, protocolo formal [Sem. 4]
├── 01-analise-overhead-php.md     ← Estudo: zval, ZendMM, type juggling, JIT [Sem. 2-3]
├── 02-lei-amdahl-estudo.md        ← Estudo: Lei de Amdahl aplicada ao IC [Sem. 1]
├── 03-zend-api-fundamentals.md    ← Estudo: Zend API para extensões C++ [Sem. 11-12]
├── 04-relatorio-baseline.md       ← Relatório: gargalos identificados [Sem. 9-10]
├── 05-implementacao-jod.md        ← Relatório: sei_turbo_jod.so [Sem. 17]
├── 06-validacao-integracao.md     ← Relatório: testes de carga + Valgrind [Sem. 20]
├── 07-framework-otimizacao.md     ← Framework: metodologia genérica reutilizável [Sem. 31]
│
├── references/                    ← Referências bibliográficas
│   ├── references.bib             ← Arquivo BibTeX / ABNT
│   └── fichamentos/               ← Notas de leitura de papers
│
└── wiki/                          ← Base de conhecimento de suporte
    ├── docker.md                  ← Cheatsheet Docker
    ├── php.md                     ← Cheatsheet PHP internals
    ├── python.md                  ← Cheatsheet Python (plots)
    └── general.md                 ← Notas gerais
```

---

## src/ — Código Fonte

Todo o código desenvolvido na pesquisa.

```
src/
├── php/                           ← Simuladores e helpers PHP
│   ├── JODSimulator.php           ← Simula conversão de documentos [Sem. 6]
│   ├── CryptoSimulator.php        ← Simula validação de assinaturas [Sem. 7]
│   ├── SolrSimulator.php          ← Simula indexação no Solr [Sem. 8]
│   └── helpers/
│       ├── MetricsCollector.php   ← Coleta: tempo, CPU, RAM [Sem. 9]
│       └── ReportGenerator.php    ← Gera CSVs padronizados [Sem. 9]
│
└── cpp/                           ← Extensões PHP em C++
    ├── sei_turbo_jod/             ← Extensão de conversão JOD [Sem. 13-14]
    │   ├── jod_turbo.cpp          ← Lógica C++ (parsing, conversão)
    │   ├── jod_turbo.h
    │   ├── php_sei_turbo_jod.cpp  ← Binding Zend API (PHP_FUNCTION)
    │   └── CMakeLists.txt
    │
    ├── sei_turbo_crypto/          ← Extensão de validação crypto [Sem. 23-24]
    │   ├── crypto_turbo.cpp       ← OpenSSL C API
    │   ├── php_sei_turbo_crypto.cpp
    │   └── CMakeLists.txt
    │
    └── sei_turbo_solr/            ← Extensão de indexação Solr [Sem. 27-28]
        ├── solr_turbo.cpp         ← RapidJSON + libcurl
        ├── php_sei_turbo_solr.cpp
        └── CMakeLists.txt
```

---

## benchmarks/ — Scripts e Dados de Benchmark

```
benchmarks/
├── scripts/                       ← Scripts de medição
│   ├── compare_jod.php            ← PHP vs C++ (JOD) [Sem. 15]
│   ├── compare_crypto.php         ← PHP vs C++ (Crypto) [Sem. 25]
│   ├── compare_solr.php           ← PHP vs C++ (Solr) [Sem. 29]
│   ├── run_baseline.php           ← Executa todos os benchmarks [Sem. 9]
│   └── generate_plots.py          ← Gráficos matplotlib [Sem. 9]
│
├── datasets/                      ← Dados de entrada para testes
│   ├── sample_docs/
│   │   └── generate_samples.php   ← Gera .docx simulados [Sem. 6]
│   └── certificates/
│       └── generate_certs.sh      ← Gera X.509 de teste [Sem. 7]
│
└── results/                       ← CSVs com resultados
    ├── baseline-jod.csv           ← Baseline PHP (JOD) [Sem. 6]
    ├── baseline-crypto.csv        ← Baseline PHP (Crypto) [Sem. 7]
    ├── baseline-solr.csv          ← Baseline PHP (Solr) [Sem. 8]
    ├── results-jod-optimized.csv  ← C++ (JOD) [Sem. 15]
    ├── results-crypto-optimized.csv ← C++ (Crypto) [Sem. 25]
    └── results-solr-optimized.csv ← C++ (Solr) [Sem. 29]
```

---

## docker/ — Infraestrutura

```
docker/
├── docker-compose.yml             ← Fork do sei-docker-main/dev/ [Sem. 5]
│                                     Serviços: PHP 8.2, MySQL 8, Solr 9.6.1,
│                                     JOD 4.4.8, Memcached, Mailcatcher
├── configs/
│   ├── php.ini                    ← Baseado em sei.ini real do sei-docker
│   ├── xdebug.ini                 ← Copiado do sei-docker (port 9003, mode=profile)
│   └── mysql-slow-query.cnf       ← Log de queries lentas [Sem. 5]
└── scripts/
    ├── health-check.sh            ← Verifica todos os serviços [Sem. 5]
    └── build-extensions.sh        ← Compila e instala os 3 .so [Sem. 18]
```

---

## tests/ — Testes Unitários

```
tests/
├── test_sei_turbo_jod.php         ← PHPUnit: 5+ testes da extensão JOD [Sem. 18]
├── test_sei_turbo_crypto.php      ← PHPUnit: testes da extensão Crypto [Sem. 26]
└── test_sei_turbo_solr.php        ← PHPUnit: testes da extensão Solr [Sem. 30]
```

---

## result_analysis/ — Análise de Resultados

```
result_analysis/
├── figures/                       ← Gráficos finais (qualidade de publicação)
│   ├── fig1-baseline-comparison.png
│   ├── fig2-jod-speedup.png
│   ├── fig3-crypto-speedup.png
│   └── fig4-solr-speedup.png
└── notebooks/                     ← Jupyter Notebooks (análise exploratória)
```

---

## sei-docker-main/ — Repositório Oficial SEI (referência, não modificar)

```
sei-docker-main/                   ← pengovbr/sei-docker (repositório oficial)
│                                     NÃO MODIFICAR — usar apenas como referência
├── containers/                    ← Receitas Docker de todos os componentes
│   ├── app-php8/app-dev-php8/    ← Imagem PHP 8.2 + Xdebug (O QUE USAMOS)
│   ├── jod4.4.8/                 ← JOD com 2 instâncias LibreOffice
│   ├── solr-9.6.1/               ← Solr com 3 cores reais do SEI
│   └── databases/mysql8-sei50/   ← MySQL com schema oficial do SEI 5.0
├── dev/                           ← Ambiente de desenvolvimento (O QUE USAMOS)
│   ├── docker-compose.yml        ← Stack completa (forkar para docker/)
│   └── envs/env-mysql-sei5.env   ← Variáveis para SEI 5 + MySQL
└── infra/                         ← Infraestrutura de produção (referência)
```

---

## Portas Expostas pelo Docker Stack

| Serviço | Porta | URL |
|---------|-------|-----|
| SEI (Apache/PHP) | 8000 | http://localhost:8000/sei |
| Solr Admin | 8983 | http://localhost:8983/solr |
| Mailcatcher | 1080 | http://localhost:1080 |
| MySQL | 3306 | localhost:3306 (usuário: sei_user / sei_user) |
| Memcached | 11211 | localhost:11211 |
| JOD | 8080 | http://jod:8080/conversion?format=pdf (interno) |

---

## Tags Git Planejadas

| Tag | Quando | Significado |
|-----|--------|-------------|
| `v0.1-fundacao` | Semana 4 | Fase 1 completa: docs teóricos |
| `v0.2-baseline` | Semana 10 | Fase 2 completa: simuladores + baseline |
| `v0.3-jod-prototype` | Semana 16 | Fase 3 parcial: JOD protótipo |
| `v0.4-jod-validated` | Semana 18 | Fase 3 completa: JOD validado |
| `v0.5-validated` | Semana 20 | Fase 4 completa: testes de carga |
| `v0.6-crypto-solr` | Semana 30 | Fase 5 completa: 3 extensões |
| `v1.0-final` | Semana 47-48 | Versão pública final |
