# Roadmap Condensado — Para Colar no Gemini

> Cole este bloco dentro do prompt mestre onde está indicado [COLE O ROADMAP CONDENSADO AQUI]
> Versão compacta do roadmap de 52 semanas para caber no contexto do Gemini.

---

## Roadmap IC SEI Turbo — 52 Semanas (Ago/2026 – Jul/2027)

**Carga:** 20h/semana (seg–sex 3h + sáb 5h)
**Entregas obrigatórias:**
- Fichas de Efetividade: dia 10 de cada mês (set/26 – jul/27)
- Relatório Parcial: ~março/2027
- Relatório Final + Resumos: ~junho/2027
- Congresso IC UnB/DF: ~ago–set/2027
- EnCUCA (CEUB): ~out/2027

---

### FASE 1 — Fundação (Sem. 1–4 · Ago/2026 · 80h)
**Objetivo:** Base teórica, ambiente de trabalho, protocolo de pesquisa.

| Sem | Foco | Entrega |
|-----|------|---------|
| 1 | Lei de Amdahl (teoria, aplicação, escrita) + Docker básico | docs/02-lei-amdahl-estudo.md |
| 2 | Zend Engine: zval, ZendMM, type juggling, OPcache/JIT | Rascunho docs/01-analise-overhead-php.md |
| 3 | Xdebug (profile mode) + QCachegrind + finalizar docs overhead | docs/01-analise-overhead-php.md completo |
| 4 | Protocolo formal (H0/H1, variáveis) + reunião orientador | docs/00-protocolo-pesquisa.md + tag v0.1-fundacao |

**Ficha #1:** entregar 10/set — setup repo, Amdahl, overhead PHP, profiling

---

### FASE 2 — Ambiente & Baseline (Sem. 5–10 · Set–Out/2026 · 120h)
**Objetivo:** Stack Docker funcional, 3 simuladores PHP, baseline de performance.

| Sem | Foco | Entrega |
|-----|------|---------|
| 5 | Docker stack completa (fork do pengovbr/sei-docker/dev/) | docker/ funcionando |
| 6 | JODSimulator.php + datasets + benchmark + profiling | results/baseline-jod.csv |
| 7 | CryptoSimulator.php + certs de teste + benchmark | results/baseline-crypto.csv |
| 8 | SolrSimulator.php (campos reais!) + benchmark consolidado | results/baseline-solr.csv |
| 9 | MetricsCollector + ReportGenerator + generate_plots.py | Gráficos de baseline |
| 10 | Relatório de gargalos + reunião orientador + ajustes | docs/04-relatorio-baseline.md + tag v0.2-baseline |

**Ficha #2:** entregar 10/out — Docker, início simuladores
**Ficha #3:** entregar 10/nov — 3 simuladores, baseline, profiling

**Detalhes críticos dos simuladores:**
- JODSimulator: chama REAL endpoint POST http://jod:8080/conversion?format=pdf
- SolrSimulator: usa campos REAIS (id_prot, id_proc, id_doc, desc, dta_ger...) nos 3 cores reais
- CryptoSimulator: openssl_verify(), openssl_x509_parse() em batch

---

### FASE 3 — 1º Protótipo C++ / JOD (Sem. 11–18 · Out–Dez/2026 · 160h)
**Objetivo:** Aprender Zend API, implementar sei_turbo_jod.so, medir speedup real.

| Sem | Foco | Entrega |
|-----|------|---------|
| 11 | Zend API: zval, PHP_FUNCTION, extensão hello world, parâmetros | Extensão hello world compilando |
| 12 | Zend API avançado: memory management, strings, HashTable, errors | docs/03-zend-api-fundamentals.md |
| 13 | Design sei_turbo_jod: arquitetura, parsing C++, regex C++, unit tests | Classe JODTurbo standalone |
| 14 | Implementação: PHP binding, module entry, compilação, integração PHP | sei_turbo_jod.so carregável |
| 15 | Benchmark comparativo: PHP vs C++ (100 docs × 50 reps) + gráficos | results/results-jod-optimized.csv |
| 16 | Thread pool (se viável) + cleanup código + reunião orientador | tag v0.3-jod-prototype |
| 17 | Relatório técnico JOD (método, implementação, resultados, análise) | docs/05-implementacao-jod.md |
| 18 | PHPUnit (5+ testes) + Valgrind (0 errors) + Dockerfile integrado | tag v0.4-jod-validated |

**Ficha #4:** entregar 10/dez — Zend API, design da extensão
**Ficha #5:** entregar 10/jan — sei_turbo_jod, benchmarks, speedup

**Meta de speedup JOD:** ≥ 3x (maior gargalo é o protocolo HTTP, que C++ elimina)

---

### FASE 4 — Validação Integrada (Sem. 19–22 · Dez/2026 · 80h)
**Objetivo:** Provar que a extensão funciona sob carga.

| Sem | Foco | Entrega |
|-----|------|---------|
| 19 | JMeter/locust: teste de carga (100 req × 1000 total) PHP vs C++ | Resultados de carga |
| 20 | docs/06-validacao-integracao.md + README update + code review | Relatório validação |
| 21-22 | RECESSO — estudo leve: papers, OpenSSL C API, std::thread | Buffer natural |

**Ficha #6:** entregar 10/jan — testes de carga, validação, integração
Tag: v0.5-validated

---

### FASE 5 — Escalabilidade: Crypto + Solr (Sem. 23–30 · Jan–Fev/27 · 160h)
**Objetivo:** Repetir metodologia para as outras 2 rotinas.

| Sem | Foco | Entrega |
|-----|------|---------|
| 23 | OpenSSL C API: EVP_DigestVerify, X509_* — código standalone | C++ crypto funcionando |
| 24 | sei_turbo_crypto.cpp: binding PHP + compilação .so | Extensão crypto compilável |
| 25 | Benchmark crypto: 1000 validações × 50 reps + gráficos | CSVs + gráficos |
| 26 | Testes crypto + Valgrind + integração Docker | Testes verdes |
| 27 | RapidJSON + libcurl C API: serialização batch para Solr | Código C++ Solr funcionando |
| 28 | sei_turbo_solr.cpp: binding PHP + compilação .so | Extensão Solr compilável |
| 29 | Benchmark Solr: 10k docs × 50 reps + gráficos | CSVs + gráficos |
| 30 | Testes Solr + Valgrind + tabela unificada 3 extensões | Tabela comparativa final |

**Ficha #7:** entregar 10/fev — extensão crypto
**Ficha #8:** entregar 10/mar — extensão Solr, resultados das 3
**Meta de speedup:** Crypto: 2-3x | Solr: 1.5-2x

---

### FASE 6 — Relatório Parcial + Framework (Sem. 31–34 · Mar/2027 · 80h)
**⚠️ MARCO CRÍTICO: Relatório Parcial — início de março/2027**

| Sem | Foco | Entrega |
|-----|------|---------|
| 31 | docs/07-framework-otimizacao.md: decision tree, metodologia genérica | Framework documentado |
| 32 | Redigir relatório parcial: introdução, atividades Fases 1–5, resultados | Rascunho completo |
| 33 | Revisar com orientador + formatar + submeter via Google Form | ✅ Parcial submetido |
| 34 | Pausa + planejamento do paper + coleta de referências | Backlog organizado |

**Ficha #9:** entregar 10/abr — framework, relatório parcial

---

### FASE 7 — Paper Científico (Sem. 35–42 · Abr–Mai/27 · 160h)
**Objetivo:** Paper científico completo (20–25 páginas).

| Sem | Foco | Entrega |
|-----|------|---------|
| 35 | Introdução (2 pgs) + Fundamentação parte 1 (Amdahl, overhead PHP) | Seções 1-2 |
| 36 | Fundamentação parte 2 (Zend API, estado da arte) + Metodologia | Seções 2-3 |
| 37 | Resultados (4-5 pgs): tabelas, gráficos, análise de breakdown | Seção 4 |
| 38 | Discussão + Conclusão + Referências + Apêndices | Seções 5-6 + rascunho 1 |
| 39 | Revisão com orientador + feedback | Rascunho 2 |
| 40 | Revisão final: linguagem, formatação, citações, checklist PIC | Paper finalizado |
| 41 | Relatório Final do PIC (baseado no paper, formato CEUB) | Rascunho relatório |
| 42 | 2 Resumos: EnCUCA + Congresso IC UnB (300-500 palavras cada) | 2 resumos prontos |

**Ficha #10:** entregar 10/mai — paper em redação
**Ficha #11:** entregar 10/jun — paper finalizado, relatório final

---

### FASE 8 — Apresentações & Encerramento (Sem. 43–52 · Jun–Jul/27 · 200h)

| Sem | Foco | Entrega |
|-----|------|---------|
| 43 | Submeter relatório final + resumos | ✅ Submetido |
| 44 | Design slides (~25 slides, 20-25 min) | Rascunho slides |
| 45 | Refinar slides + script de apresentação | Slides refinados |
| 46 | Ensaios (3-5x) + feedback orientador + cronometrar | Apresentação pronta |
| 47 | Limpar GitHub: organizar, README final, remover temporários | Repo limpo |
| 48 | GitHub público (se aprovado) + LICENSE + Release v1.0 | ✅ GitHub público |
| 49 | Banner/pôster para EnCUCA + UnB (A0/A1) | Design do banner |
| 50 | Imprimir banner + ensaiar pitch de 3-5 min | Banner pronto |
| 51 | Revisão final de TUDO + organizar pasta de documentos | Checklist final |
| 52 | Autoavaliação + lições aprendidas | ✅ Fim do ciclo |

**Ficha #12:** entregar 10/jul — relatório final submetido, GitHub, apresentações

---

## Resumo de Speedups Esperados (para a Lei de Amdahl)

| Extensão | Operação | Speedup esperado | Justificativa |
|----------|----------|-----------------|---------------|
| sei_turbo_jod | Conversão de docs | 3–5x | Elimina HTTP overhead inteiro |
| sei_turbo_crypto | Validação X.509 | 2–3x | Batch em C++, sem overhead PHP |
| sei_turbo_solr | Indexação JSON | 1.5–2x | JSON em C++ (RapidJSON) vs json_encode |

---

*Este roadmap é o documento de referência. Quando me perguntar sobre progresso, baseie-se nele.*
