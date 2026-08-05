# 🗺️ Roadmap IC SEI Turbo — 52 Semanas (Versão Final Atualizada)

**Projeto:** Otimização de Performance em Sistemas de Gestão Pública via Módulos C++
**Aluno:** Vitor Pádua Moreira Justo (RA 22550776)
**Orientador:** Prof. Auto Tavares
**Ciclo:** 01/ago/2026 — 31/jul/2027
**Carga horária:** 20h/semana (seg–sex 3h + sáb 5h)
**Avaliação:** PIBIC 300/300 · PIBITI 298/300 ✅

> **Nota de Atualização:** Este roadmap foi atualizado em Ago/2026 após o mapeamento do repositório oficial `pengovbr/sei-docker`. Ele reflete o uso dos protocolos, schemas e configurações reais de produção do SEI.

---

## 📅 Calendário de Entregas Obrigatórias PIC/CEUB

| Entrega                   | Prazo Estimado                           | Tipo           |
| ------------------------- | ---------------------------------------- | -------------- |
| Fichas de Efetividade     | Dia 10 de **cada mês** (set/26 — jul/27) | Administrativo |
| Relatório Parcial         | ~Março/2027                              | Acadêmico      |
| Relatório Final + Resumos | ~Junho/2027                              | Acadêmico      |
| Congresso de IC da UnB/DF | ~Ago–Set/2027                            | Apresentação   |
| EnCUCA (CEUB)             | ~Out/2027                                | Apresentação   |

> [!IMPORTANT]
> As datas exatas do ciclo 2026-2027 seguem o padrão do ciclo anterior. Confirme com a coordenação do PIC assim que o edital for publicado. O roadmap assume os prazos típicos.

---

## Visão Geral das 8 Fases

```
AGO/26         SET          OUT          NOV          DEZ          JAN/27
┃━━ FASE 1 ━━┃━━ FASE 2 ━━━━━━━━━━━━━┃━━ FASE 3 ━━━━━━━━━━━━━┃
┃ Fundação    ┃ Ambiente & Baseline    ┃ 1º Protótipo C++ (JOD)┃
┃ 4 sem       ┃ 6 semanas             ┃ 8 semanas             ┃

FEV/27         MAR                     ABR          MAI          JUN
┃━━ FASE 4 ━━━━━━━━━━┃━━ FASE 5 ━━━━━━━━━━━━━━━━━━┃━━ FASE 6 ━━┃
┃ Validação &         ┃ Escalabilidade             ┃ Paper &    ┃
┃ Rel.Parcial (4 sem) ┃ Crypto+Solr (8 sem)        ┃ Rel.Final  ┃

JUN/27 (cont)  JUL/27
┃━━ FASE 7 ━━━━━━━━━━┃
┃ Apresentações &     ┃
┃ Encerramento (6 sem)┃
```

---

## ═══════════════════════════════════════════════
## FASE 1 — FUNDAÇÃO CONCEITUAL
## Semanas 1–4 · Agosto/2026 · 80h
## ═══════════════════════════════════════════════

**Objetivo:** Construir toda a base teórica, preparar o ambiente de trabalho, e ter o protocolo de pesquisa pronto antes de tocar em qualquer código.

---

### Semana 1 (01–07/ago) — Setup & Lei de Amdahl · 20h

| Dia | Foco                      | Tarefas | Horas |
| --- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| Seg | Setup do projeto          | Criar repo Git local `ic-sei-cpp-2026/`. Criar toda a árvore de diretórios (`/docs`, `/code`, `/benchmarks`, `/datasets`, `/results`, `/docker`, `/tests`). Escrever README.md inicial com objetivo, metodologia, e estrutura. Configurar `.gitignore` para C++, PHP, Docker, IDE. | 3h    |
| Ter | Lei de Amdahl (teoria)    | Ler o paper original de Amdahl (1967). Anotar: definição formal de speedup, premissas, limitações. Ler sobre extensões da lei (Lei de Gustafson). | 3h    |
| Qua | Lei de Amdahl (aplicação) | Calcular speedup teórico para 3 cenários do seu IC: (1) se p=70% da execução é crypto e s=5x → S=2.17x; (2) se p=40% é JOD e s=4x → S=1.43x; (3) cenário combinado. Documentar cálculos com fórmulas. | 3h    |
| Qui | Lei de Amdahl (escrita)   | Redigir documento `docs/02-lei-amdahl-estudo.md` (2-3 páginas). Seções: Formulação Matemática, Implicações para PHP, Cálculos de Speedup Teórico, Limitações da Lei, Referências. | 3h    |
| Sex | Revisão + Git             | Revisar tudo escrito. Fazer primeiro commit significativo. Ler tese `2024_tese_assousa.pdf`. | 3h    |

**Sábado (5h):** Estudar Docker a fundo. Testar subir container PHP 8.2 local.

---

### Semana 2 (08–14/ago) — Overhead do PHP (Parte 1) · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Zend Engine intro | Ler phpinternalsbook.com. Anotar o pipeline (lexer → parser → AST → opcodes). | 3h |
| Ter | Estrutura `zval` | Estudar a struct `zval` em profundidade. Entender overhead de ~16-24 bytes. | 3h |
| Qua | Memory Manager | Estudar ZendMM: malloc/free, pools. Entender overhead vs `malloc()` puro em C. | 3h |
| Qui | Type Juggling | Estudar o custo de type juggling: string→int, int→float, loose (`==` vs `===`). | 3h |
| Sex | OPcache e JIT | Estudar OPcache e JIT do PHP 8. Entender por que JIT não resolve tudo. | 3h |

**Sábado (5h):** Começar redação de `docs/01-analise-overhead-php.md` (seções 1-3).

---

### Semana 3 (15–21/ago) — Overhead do PHP (Parte 2) + Profiling · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Xdebug | Instalar Xdebug no container Docker. Configurar modo `profile`. Gerar `.cachegrind`. | 3h |
| Ter | Callgrind | Instalar QCachegrind. Aprender a ler tempos inclusivo/exclusivo, call graph. | 3h |
| Qua | SPX (alternativa) | Pesquisar SPX profiler. Comparar output com Xdebug. | 3h |
| Qui | Overhead doc (cont.) | Finalizar `docs/01-analise-overhead-php.md`. Total: 3-5 páginas. | 3h |
| Sex | Revisão geral | Revisar Amdahl + Overhead. Git commit. | 3h |

**Sábado (5h):** Prática de profiling. Script PHP simples rodando no Xdebug e analisado no QCachegrind.

---

### Semana 4 (22–31/ago) — Protocolo de Pesquisa · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Desenho experimental | Definir formalmente: Hipótese (H0/H1). Variáveis independentes/dependentes. | 3h |
| Ter | Controles | Definir grupo controle vs experimental, isolamento, iterações, outliers. | 3h |
| Qua | Protocolo formal | Redigir `docs/00-protocolo-pesquisa.md` (2 páginas). | 3h |
| Qui | Cronograma detalhado | Criar cronograma de 12 meses formato SGI/CEUB. | 3h |
| Sex | Reunião orientador | Reunião com Prof. Auto Tavares. Apresentar documentos, validar passos. | 3h |

**Sábado (5h):** Revisar tudo, organizar `.bib`. Git tag `v0.1-fundacao`.
> **Ficha #1:** entregar 10/set.

---

## ═══════════════════════════════════════════════
## FASE 2 — AMBIENTE & BASELINE
## Semanas 5–10 · Set–Out/2026 · 120h
## ═══════════════════════════════════════════════

**Objetivo:** Stack Docker baseada no repo oficial, simuladores calibrados com protocolos reais e baseline de performance.

---

### Semana 5 (01–05/set) — Docker Stack Completa (Otimizada) · 20h

> **ATUALIZAÇÃO:** Uso do repo oficial `sei-docker-main` economizando tempo.

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Docker-compose | Fazer fork de `sei-docker-main/dev/docker-compose.yml`. Configurar para o projeto. | 3h |
| Ter | Configuração PHP | Ajustar imagem pronta `app-dev-php8` com Xdebug já habilitado. | 3h |
| Qua | Serviços Extras | Habilitar JOD 4.4.8, Solr 9.6.1 e Memcached. Testar conectividade. | 3h |
| Qui | MySQL config | Verificar banco oficial subindo via `sei_5_0_0_BD_Ref_Exec.sql`. | 3h |
| Sex | Validação | `docker-compose up -d`. Criar `health-check.sh` validando portas (8000, 8983, 8080). | 3h |

**Sábado (5h):** Testar compilação C/C++ (`phpize`, `gcc`) dentro do container `app-dev-php8`.

---

### Semana 6 (08–12/set) — Rotina 1: JODSimulator · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Design JOD | Estudar protocolo POST http://jod:8080/conversion?format=pdf baseado no `ConfiguracaoSEI.php`. | 3h |
| Ter | JODSimulator.php | Implementar o simulador chamando o JOD HTTP. O gargalo simulado será a serialização/rede. | 3h |
| Qua | Dados de teste | Gerar `sample_docs`. 100+ arquivos representativos. | 3h |
| Qui | Benchmark JOD | `compare_jod.php`. 100 conversões x 10 repetições. Coletar CSV. | 3h |
| Sex | Profiling JOD | Analisar profiling. Separar tempo de HTTP (overhead alvo) do tempo de LibreOffice. | 3h |

**Sábado (5h):** Refinar simulador e tabelas.
> **Ficha #2:** entregar 10/out.

---

### Semana 7 (15–19/set) — Rotina 2: CryptoSimulator · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Certs de teste | Gerar certificados de teste X.509 auto-assinados. | 3h |
| Ter | CryptoSimulator.php | Implementar usando a lógica do campo `id_assin` do schema Solr. Usar `openssl_verify()`. | 3h |
| Qua | Benchmark crypto | 1000 validações x 10 repetições. Salvar CSV. | 3h |
| Qui | Profiling crypto | Identificar tempo de `openssl_verify()` vs overhead de variáveis PHP. | 3h |
| Sex | Análise | Comparar perfil de gargalo Crypto vs JOD. | 3h |

**Sábado (5h):** Completar e documentar baseline crypto.

---

### Semana 8 (22–26/set) — Rotina 3: SolrSimulator · 20h

| Dia | Foco | Tarefas | Horas |
|---|---|---|---|
| Seg | Solr API Real | Estudar `schema.xml` oficial: campos `id_prot`, `id_doc`, `desc`, pipeline PT-BR. | 3h |
| Ter | SolrSimulator.php | Fazer HTTP POST real para `/solr/sei-protocolos/update` enviando JSON. | 3h |
| Qua | Benchmark Solr | Indexar 10k docs x 10 repetições. Medir tempo `json_encode()` vs Rede. | 3h |
| Qui | Profiling Solr | Profiling SolrSimulator. Separar gargalo de rede e gargalo de JSON PHP. | 3h |
| Sex | Consolidar | Tabela unificada dos 3 baselines. | 3h |

**Sábado (5h):** Construir `MetricsCollector.php` final.

---

### Semanas 9 e 10 (29/set–10/out) — Relatórios e Automação · 40h

* **Semana 9:** Script Python matplotlib para gráficos. `ReportGenerator.php`. Análise final de profiling.
* **Semana 10:** Redigir `docs/04-relatorio-baseline.md`. Reunião orientador. Tag `v0.2-baseline`.
> **Ficha #3:** entregar 10/nov.

---

## ═══════════════════════════════════════════════
## FASE 3 — 1º PROTÓTIPO C++ (JOD)
## Semanas 11–18 · Out–Dez/2026 · 160h
## ═══════════════════════════════════════════════

**Objetivo:** Extensão C++ para JOD focada em pular a camada HTTP do JODConverter.

### Semanas 11 e 12 (13–24/out) — Zend API Fundamentos · 40h
* Ler "Basic Structure" do phpinternalsbook. Entender `zval`, arrays.
* Escrever extensão "hello world" (`phpize`, `make`).
* Entender memory management em extensões, error handling, try/catch.
* Entregar `docs/03-zend-api-fundamentals.md`.

### Semanas 13 e 14 (27/out–07/nov) — sei_turbo_jod.cpp · 40h
* **Foco (ATUALIZADO):** A extensão deverá conversar via socket direto com a UNO API do LibreOffice, ou realizar C++ system calls, demonstrando a remoção do overhead HTTP gerado pelo JOD.
* Unit tests C++ puros primeiro, depois bind para PHP com `PHP_FUNCTION`.
* `CMakeLists.txt` finalizado, compilando `sei_turbo_jod.so`.

### Semanas 15 e 16 (10–21/nov) — Benchmarks JOD · 40h
* `compare_jod.php` rodando as duas versões (HTTP original vs Extensão C++).
* Calcular Intervalos de Confiança 95%, speedup alcançado, e Amdahl effect.
* Tag `v0.3-jod-prototype`.

### Semanas 17 e 18 (24/nov–05/dez) — Relatório e Testes · 40h
* Redigir `docs/05-implementacao-jod.md` (Metodologia, Resultados, Análise).
* Valgrind `--leak-check=full`. Integrar ao Dockerfile estendido para automação.
* Tag `v0.4-jod-validated`.
> **Ficha #4 (10/dez)** e **Ficha #5 (10/jan)**.

---

## ═══════════════════════════════════════════════
## FASE 4 — VALIDAÇÃO INTEGRADA + TESTE DE CARGA
## Semanas 19–22 · Dezembro/2026 · 80h
## ═══════════════════════════════════════════════

### Semanas 19 e 20 (08–19/dez) — Apache Bench / JMeter · 40h
* **Referência Real:** Baseado no `AgendamentoTarefaSEI.php` rodando via cron, criar simulação de alta concorrência em Jmeter/locust.
* Coletar throughput, P95, P99 e medir robustez de memória sob carga.
* Redigir `docs/06-validacao-integracao.md`. Tag `v0.5-validated`.
> **Ficha #6:** entregar 10/jan.

### Semanas 21 e 22 (22/dez–02/jan) — Recesso Parcial · 40h
* Diminuir ritmo de código (10-15h). Foco em leitura: OpenSSL API em C++, RapidJSON.
* Organizar bibliografia.

---

## ═══════════════════════════════════════════════
## FASE 5 — ESCALABILIDADE: CRYPTO + SOLR
## Semanas 23–30 · Jan–Fev/2027 · 160h
## ═══════════════════════════════════════════════

### Semanas 23 a 26 (05–30/jan) — sei_turbo_crypto · 80h
* Estudar API C OpenSSL: `EVP_DigestVerify`, parse de X509.
* Binding C++ → `.so` com batch processing de validação criptográfica (pulando iterações lentas em loop PHP).
* Benchmarks e testes verdes.
> **Ficha #7:** entregar 10/fev.

### Semanas 27 a 30 (02–27/fev) — sei_turbo_solr · 80h
* **Foco (ATUALIZADO):** Serialização C++ focada no `schema.xml` oficial usando `RapidJSON` para substituir chamadas massivas de `json_encode` + `curl`.
* Testes unificados, relatórios gerados.
* Tag `v0.6-crypto-solr`. Tabela final com os 3 speedups comparados.
> **Ficha #8:** entregar 10/mar.

---

## ═══════════════════════════════════════════════
## FASE 6 — RELATÓRIO PARCIAL + FRAMEWORK
## Semanas 31–34 · Março/2027 · 80h
## ═══════════════════════════════════════════════

### Semanas 31 a 34 (02–27/mar)
* **Framework Documentado:** Explicar como qualquer dos 300 órgãos governamentais usando a stack PEN pode reutilizar esse método (`docs/07-framework-otimizacao.md`).
* Submeter o Relatório Parcial do CEUB/UnB com os avanços até a Fase 5.
> **Ficha #9:** entregar 10/abr.

---

## ═══════════════════════════════════════════════
## FASE 7 — PAPER CIENTÍFICO
## Semanas 35–42 · Abr–Mai/2027 · 160h
## ═══════════════════════════════════════════════

**Objetivo:** Paper formal detalhando resultados.

### Semanas 35 a 42 (Abril - Maio)
* **ATUALIZAÇÃO CRÍTICA (Metodologia):** Escrever claramente que o lab utilizou as imagens oficiais do `pengovbr` e os arquivos reais (`ConfiguracaoSEI.php`, `schema.xml`), validando a relevância governamental dos resultados.
* Produzir as seções: Resultados, Discussão (Amdahl), Conclusões.
* Gerar os resumos pro Congresso UnB e EnCUCA.
> **Ficha #10 (10/mai)** e **Ficha #11 (10/jun)**.

---

## ═══════════════════════════════════════════════
## FASE 8 — APRESENTAÇÕES & ENCERRAMENTO
## Semanas 43–52 · Jun–Jul/2027 · 200h
## ═══════════════════════════════════════════════

### Semanas 43 a 52 (Junho - Julho)
* **Slides e Defesa:** Elaborar slide "Validade do Lab" para destruir críticas sobre "código não real". Mostrar a cadeia de evidências (`pengovbr` → banco → configs).
* Refinar GitHub público. Adicionar README científico, apagar scripts scratch.
* Ensaiar defesa. Autoavaliação e fim de ciclo.
> **Ficha #12:** entregar 10/jul.

---
*Roadmap Finalizado - Cód. v2.0-SEI-Docker*
