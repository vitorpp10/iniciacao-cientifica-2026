# 📋 Próxima Reunião — Prof. Auto Tavares

> Data sugerida: esta semana (Ago/2026)  
> Duração: ~30 min  
> Objetivo: validar a descoberta do sei-docker-main e confirmar metodologia

---

## Pauta

### 1. Novidade: Mapeamos o Repositório Oficial do SEI (5 min)

**O que dizer:**

> "Professor, eu e a IA mapeamos completamente o repositório `pengovbr/sei-docker`, que é o repositório oficial do Processo Eletrônico Nacional para desenvolvimento e debug de módulos do SEI. Isso resolveu a pergunta que o senhor fez sobre como garantir que o laboratório vai ser igual ao módulo real."

**Evidências a mostrar** (abrir o arquivo `docs/mapa-sei-docker.md`):

- `ConfiguracaoSEI.php` — contém os protocolos **exatos** que o SEI usa em produção:
  - JOD: `http://jod:8080/conversion?format=pdf`
  - Solr: `http://solr:8983/solr`, com 3 cores: `sei-protocolos`, `sei-bases-conhecimento`, `sei-publicacoes`
  - Memcached: porta 11211, timeout 1s
- `schema.xml` — estrutura **exata** dos campos indexados no Solr (`id_prot`, `id_doc`, `id_assin`, `desc`, etc.)
- `application.yaml` do JOD 4.4.8 — revela que o JOD já usa **2 instâncias do LibreOffice** (ports 2002/2003)
- `mysql8-sei50/Dockerfile` — baixa o banco de referência executivo do SEI 5.0 do GitHub oficial do governo

---

### 2. Resposta à Pergunta Matadora (10 min)

**Pergunta do professor (reunião anterior):** "Como você vai garantir que o lab vai ser igual ao módulo real?"

**Resposta a apresentar:**

> "Nosso laboratório usa a **mesma infraestrutura oficial** de desenvolvimento do SEI. Não precisamos do código-fonte PHP (que é propriedade do TRF4) porque nossa pesquisa mede o impacto das **operações computacionais** — não da lógica de negócio.
>
> Nossos simuladores vão replicar os *padrões computacionais* usando os *protocolos reais*:
> - JODSimulator chama o endpoint real `POST http://jod:8080/conversion?format=pdf`
> - SolrSimulator indexa documentos com os campos reais (`id_prot`, `id_doc`, `desc`...)
> - CryptoSimulator valida certificados X.509 com as funções OpenSSL que o SEI usa
>
> Isso é metodologicamente equivalente ao que a literatura faz: benchmarks como SPEC CPU, TPC-C e YCSB não usam código de produção — eles criam cargas controladas que replicam os padrões computacionais. Nossa vantagem é que usamos os **campos e protocolos reais** do sistema."

---

### 3. Confirmação de Metodologia (5 min)

**Pontos a confirmar com o professor:**

- [ ] A abordagem de simuladores com protocolos reais é metodologicamente válida para o PIC?
- [ ] Posso usar `pengovbr/sei-docker` como referência bibliográfica no paper?
- [ ] O professor conhece algum pesquisador ou órgão que usa SEI para possível validação externa?

---

### 4. Próximos Passos Concretos (5 min)

**Apresentar o cronograma de agosto:**

| Semana | Data | Entrega |
|--------|------|---------|
| Sem. 1 | 01–07/ago | ✅ Setup do repo — FEITO |
| Sem. 2 | 08–14/ago | Estudo overhead PHP (zval, ZendMM, OPcache) |
| Sem. 3 | 15–21/ago | Xdebug + profiling de scripts PHP |
| Sem. 4 | 22–28/ago | Protocolo de pesquisa formal + cronograma SGI |

**Ficha de Efetividade #1** — entregar até 10/setembro. Conteúdo: setup do repositório, estudo teórico, configuração do ambiente de profiling.

---

### 5. Perguntas para o Professor

- O professor quer mirar congressos: **EnCUCA (CEUB)** + **Congresso de IC da UnB**?
- Qual o formato esperado para as reuniões quinzenais? (presencial/remoto, relatório prévio?)
- Há algum servidor do CEUB com SEI rodando que poderia visitar para validação?

---

## Materiais para Levar

- [ ] `docs/mapa-sei-docker.md` — análise completa do repositório oficial
- [ ] `docs/estrategia-mencao-honrosa.md` — estratégia para premiação
- [ ] `docs/roadmap-vinculado.md` — roadmap ajustado com evidências
- [ ] `readme.md` — visão geral do projeto

---

*Criado em 04/08/2026*
