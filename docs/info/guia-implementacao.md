# 🛠️ Guia de Implementação — O Que Fazer e Como

> **Importante:** Este guia é um MAPA, não uma implementação. Você implementa, o guia orienta.

---

## O Que Você Tem Disponível (sem escrever 1 linha de código)

| Recurso | Onde está | O que é |
|---------|-----------|---------|
| Stack Docker completa | `sei-docker-main/dev/docker-compose.yml` | PHP 8.2 + MySQL + Solr + JOD + Memcached |
| Imagens prontas | Docker Hub `processoeletronico/*` | app-dev-php8, solr9.6.1, jod4.4.8, mysql8-sei50 |
| Protocolos reais | `sei-docker-main/containers/app-php8/app-dev-php8/assets/scripts-e-automatizadores/ConfiguracaoSEI.php` | Endpoints de JOD, Solr, Memcached |
| Schema Solr real | `sei-docker-main/containers/solr/assets/solr8sei/sei-cores-8.2.0/sei-protocolos/conf/schema.xml` | 30+ campos, 3 cores |
| Config PHP real | `sei-docker-main/containers/app-php8/app-dev-php8/assets/conf/sei.ini` | Charset, limites, includes |
| Config Xdebug | `sei-docker-main/containers/app-php8/app-dev-php8/assets/conf/xdebug.ini` | Pronto para copiar |
| Config JOD | `sei-docker-main/containers/jod4.4.8/assets/application.yaml` | 2 instâncias LibreOffice, porta 8080 |

---

## Sequência Recomendada de Implementação

### 🟡 AGOSTO: Fundação (Semanas 1–4)

```
[Semana 1 — já feita]
✅ Repo Git criado
✅ Estrutura de pastas criada
✅ README e structure.md

[Semana 2 — fazer agora]
→ Ler phpinternalsbook.com caps 1-3
→ Redigir docs/01-analise-overhead-php.md
→ Foco: zval, ZendMM, type juggling

[Semana 3]
→ Instalar Xdebug no Docker (copiar xdebug.ini do sei-docker)
→ Gerar primeiro .cachegrind
→ Abrir no QCachegrind

[Semana 4]
→ Redigir docs/00-protocolo-pesquisa.md
→ Redigir docs/02-lei-amdahl-estudo.md
→ Reunião com professor
→ Ficha de Efetividade #1 (entrega 10/set)
```

### 🟠 SETEMBRO: Docker + Simuladores (Semanas 5–8)

```
[Semana 5 — ~2 dias, não 5]
→ Copiar/fork docker-compose.yml do sei-docker-main/dev/
→ Adaptar para seu projeto em docker/
→ Subir: docker-compose up -d
→ Verificar: http://localhost:8000 (SEI), :8983 (Solr), :1080 (mail)

[Semana 6]
→ Implementar src/php/JODSimulator.php
→ USAR O ENDPOINT REAL: POST http://jod:8080/conversion?format=pdf
→ Criar datasets/sample_docs/generate_samples.php
→ Benchmarks: benchmarks/scripts/compare_jod.php

[Semana 7]
→ Implementar src/php/CryptoSimulator.php
→ openssl_verify(), openssl_x509_parse(), hash('sha256')
→ Criar datasets/certificates/generate_certs.sh

[Semana 8]
→ Implementar src/php/SolrSimulator.php
→ USAR CAMPOS REAIS: id_prot, id_proc, id_doc, desc, numero...
→ USAR ENDPOINTS REAIS: /sei-protocolos/update, /sei-bases-conhecimento/update
→ json_encode() + curl para o Solr real
```

### 🔴 SETEMBRO-OUTUBRO: MetricsCollector + Baseline (Semanas 9–10)

```
[Semana 9]
→ src/php/helpers/MetricsCollector.php
  Métricas: microtime(), memory_get_peak_usage(), getrusage()
→ src/php/helpers/ReportGenerator.php
  Output: CSV com (rotina, iteração, tempo_ms, cpu_user, ram_peak_mb)
→ benchmarks/scripts/generate_plots.py (matplotlib)

[Semana 10]
→ Rodar baseline completo (100 docs × 10 repetições)
→ Salvar: benchmarks/results/baseline-jod.csv
→        benchmarks/results/baseline-crypto.csv
→        benchmarks/results/baseline-solr.csv
→ Redigir docs/04-relatorio-baseline.md
→ Ficha de Efetividade #3 (entrega 10/nov)
```

### 🔵 OUTUBRO-DEZEMBRO: Extensão C++ JOD (Semanas 11–18)

```
[Semanas 11-12 — Zend API]
→ Ler phpinternalsbook.com caps 2-4
→ Escrever extensão "hello world" em C
→ Compilar: phpize && ./configure && make
→ Redigir docs/03-zend-api-fundamentals.md

[Semanas 13-14 — sei_turbo_jod.cpp]
→ Implementar src/cpp/sei_turbo_jod/jod_turbo.cpp
→ Design: JODTurbo class, parseDocument(), convertToPdf()
→ PHP binding: PHP_FUNCTION(sei_turbo_convert_document)
→ Criar src/cpp/sei_turbo_jod/CMakeLists.txt

[Semanas 15-16 — Benchmarks]
→ Comparativo: PHP (http://jod:8080) vs C++ (direto)
→ 100 docs × 50 repetições
→ Calcular speedup, intervalos de confiança 95%
→ Gráficos matplotlib

[Semanas 17-18 — Documentação e Testes]
→ Redigir docs/05-implementacao-jod.md
→ tests/test_sei_turbo_jod.php (PHPUnit, 5+ testes)
→ Valgrind --leak-check=full
→ docker/scripts/build-extensions.sh
→ Tag: v0.4-jod-validated
```

---

## Estrutura de Código a Criar

```
src/
├── php/
│   ├── JODSimulator.php          ← Semana 6
│   ├── CryptoSimulator.php       ← Semana 7
│   ├── SolrSimulator.php         ← Semana 8 (com campos reais!)
│   └── helpers/
│       ├── MetricsCollector.php  ← Semana 9
│       └── ReportGenerator.php   ← Semana 9
│
└── cpp/
    ├── sei_turbo_jod/            ← Semana 13-14
    │   ├── jod_turbo.cpp
    │   ├── jod_turbo.h
    │   ├── php_sei_turbo_jod.cpp  ← PHP binding (Zend API)
    │   └── CMakeLists.txt
    │
    ├── sei_turbo_crypto/         ← Semana 23-24
    │   ├── crypto_turbo.cpp       ← OpenSSL C API
    │   ├── php_sei_turbo_crypto.cpp
    │   └── CMakeLists.txt
    │
    └── sei_turbo_solr/           ← Semana 27-28
        ├── solr_turbo.cpp         ← RapidJSON + libcurl
        ├── php_sei_turbo_solr.cpp
        └── CMakeLists.txt

benchmarks/
├── scripts/
│   ├── compare_jod.php            ← PHP vs C++ (JOD)
│   ├── compare_crypto.php         ← PHP vs C++ (Crypto)
│   ├── compare_solr.php           ← PHP vs C++ (Solr)
│   ├── run_baseline.php           ← Roda tudo de uma vez
│   └── generate_plots.py          ← matplotlib (gráficos)
├── datasets/
│   ├── sample_docs/
│   │   └── generate_samples.php   ← Gera docs de teste
│   └── certificates/
│       └── generate_certs.sh      ← Gera X.509 de teste
└── results/
    ├── baseline-jod.csv
    ├── baseline-crypto.csv
    ├── baseline-solr.csv
    ├── results-jod-optimized.csv
    ├── results-crypto-optimized.csv
    └── results-solr-optimized.csv

docs/
├── 00-protocolo-pesquisa.md      ← Semana 4
├── 01-analise-overhead-php.md    ← Semana 2-3
├── 02-lei-amdahl-estudo.md       ← Semana 1-4
├── 03-zend-api-fundamentals.md   ← Semana 11-12
├── 04-relatorio-baseline.md      ← Semana 9-10
├── 05-implementacao-jod.md       ← Semana 17
├── 06-validacao-integracao.md    ← Semana 20
├── 07-framework-otimizacao.md    ← Semana 31
├── mapa-sei-docker.md            ← ✅ Criado
├── roadmap-vinculado.md          ← ✅ Criado
├── estrategia-mencao-honrosa.md  ← ✅ Criado
├── guia-implementacao.md         ← ✅ Este arquivo
├── proxima-reuniao-professor.md  ← ✅ Criado
└── references/
    ├── references.bib             ← ABNT/BibTeX
    └── fichamentos/               ← Notas de leitura

tests/
├── test_sei_turbo_jod.php        ← PHPUnit (Sem. 18)
├── test_sei_turbo_crypto.php     ← PHPUnit (Sem. 26)
└── test_sei_turbo_solr.php       ← PHPUnit (Sem. 30)

docker/
├── docker-compose.yml            ← Fork do sei-docker-main/dev/
├── configs/
│   ├── php.ini                   ← Baseado no sei.ini real
│   ├── xdebug.ini                ← Copiado do sei-docker
│   └── mysql-slow-query.cnf
└── scripts/
    ├── health-check.sh
    └── build-extensions.sh       ← Compila os 3 .so

result_analysis/
├── figures/                      ← Gráficos finais (publicação)
│   ├── fig1-baseline-comparison.png
│   ├── fig2-jod-speedup.png
│   ├── fig3-crypto-speedup.png
│   └── fig4-solr-speedup.png
└── notebooks/                    ← Jupyter (opcional)
```

---

## Dependências Externas a Instalar/Estudar

| Ferramenta | Quando | Para quê |
|-----------|--------|---------|
| `QCachegrind` ou `KCachegrind` | Semana 3 | Visualizar callgrind do Xdebug |
| `phpize`, `php-config` | Semana 11 | Compilar extensões PHP |
| `g++`, `cmake` | Semana 11 | Compilar C++ |
| `valgrind` | Semana 18 | Detectar memory leaks no .so |
| `PHPUnit` | Semana 18 | Testes unitários |
| `Apache JMeter` ou `locust` | Semana 19 | Testes de carga |
| `matplotlib` (Python) | Semana 9 | Gráficos de resultados |
| `nlohmann/json` ou `RapidJSON` | Semana 27 | JSON em C++ (extensão Solr) |
| `PCRE2` (opcional) | Semana 13 | Regex mais rápida que `<regex>` |

---

## Referências Técnicas Obrigatórias

1. **phpinternalsbook.com** — Leitura fundamental para Fases 1 e 3
2. **Zend API docs** — Para as extensões C++
3. **OpenSSL man pages** — Para sei_turbo_crypto
4. **Apache Solr Reference Guide** — Para sei_turbo_solr
5. **Amdahl (1967)** — Paper original (você já tem o PDF)
6. **pengovbr/sei-docker** — Citar como fonte da infraestrutura

---

*Documento criado em 04/08/2026 — Não implementar, apenas orientar*
