# 🔗 Roadmap Vinculado ao sei-docker-main

> **Documento de análise:** Como cada fase do IC se conecta ao que existe no repositório oficial `pengovbr/sei-docker`  
> Elaborado em: 04/08/2026

---

## Diagnóstico Geral: O Roadmap Está Correto?

**✅ SIM. O roadmap está bem estruturado e as fases fazem sentido.**

O mapeamento do sei-docker-main revelou **3 ajustes** que deixam o projeto mais forte:

1. **Semana 5 pode ser comprimida** — o Docker stack já existe pronto no repo
2. **Os simuladores PHP ficam mais reais** — temos os protocolos exatos (endpoint JOD, schema Solr, campos reais)
3. **A argumentação fica mais sólida** — podemos provar que nossa infraestrutura é idêntica à de produção

---

## Fase 1 — Fundação Conceitual (Sem. 1–4 · Ago/2026)

**Status:** ✅ Mantida integralmente

| Atividade do Roadmap | Arquivo do sei-docker | O que usar |
|---|---|---|
| Estudar overhead PHP | `containers/app-php8/app-dev-php8/assets/conf/sei.ini` | Config PHP real de produção |
| Entender arquitetura | `README.md` do repositório | Confirma propósito e componentes |
| Protocolo de pesquisa | `ConfiguracaoSEI.php` | 3 serviços (JOD, Solr, Memcached) como hipóteses |
| Setup do repositório Git | — | Estrutura já criada ✅ |

---

## Fase 2 — Ambiente & Baseline (Sem. 5–10 · Set–Out/2026)

**Status:** ⚠️ Ajuste importante — **Semana 5 pode ser economizada**

### Ajuste Crítico — Semana 5

O roadmap diz: *"Criar docker/docker-compose.yml... Basear na arquitetura do sei-docker mas simplificado."*

**Agora sabemos:** não precisa criar do zero! Use diretamente:

```
sei-docker-main/dev/docker-compose.yml          ← Copie/fork este
sei-docker-main/dev/envs/env-mysql-sei5.env     ← Use como referência
sei-docker-main/containers/app-php8/app-dev-php8/ ← Imagem já publicada no Docker Hub
```

Isso poupa 3–5 dias de trabalho.

### Semana 6 — JODSimulator.php

**Protocolo real confirmado** (via `ConfiguracaoSEI.php`):
```php
// Isso é o que o SEI faz de verdade:
$url = 'http://jod:8080/conversion?format=pdf';
// POST multipart com o arquivo .docx → recebe PDF binário
```

**Descoberta adicional** (`application.yaml` do JOD 4.4.8):
```yaml
port-numbers: [2002, 2003]  # JOD já usa 2 instâncias LibreOffice!
```
O gargalo é o **overhead de protocolo HTTP**, não só single-threading. Isso fortalece a argumentação da extensão C++.

### Semana 7 — CryptoSimulator.php

O campo `id_assin` no `schema.xml` do Solr confirma que o SEI armazena IDs de assinatura digital. O simulador deve usar `openssl_verify()`, `openssl_x509_parse()`, `openssl_pkey_get_public()`.

### Semana 8 — SolrSimulator.php (com campos reais!)

**Estrutura REAL do documento SEI no Solr** (do `schema.xml` oficial):

```php
$doc = [
    'id'          => 'prot_' . $id,
    'id_prot'     => $id_protocolo,    // long
    'id_proc'     => $id_processo,     // long
    'id_doc'      => $id_documento,    // long
    'id_assin'    => $id_assinatura,   // string
    'id_org_ger'  => $id_orgao,        // int
    'id_uni_ger'  => $id_unidade,      // int
    'desc'        => $descricao,       // text_general
    'numero'      => $numero,          // text_general
    'nome_arvore' => $nome_arvore,     // text_general
    'dta_ger'     => $data_geracao,    // date ISO 8601
    'prot_pesq'   => $protocolo_pesq,  // string
    'prot_proc'   => $protocolo_proc,  // string
    'prot_doc'    => $protocolo_doc,   // string
    'sta_prot'    => $status,          // string
];
```

**Pipeline de análise de texto real** (do `schema.xml`):
```
Whitespace → StopWords(pt_BR) → WordDelimiter → FlattenGraph → LowerCase → ASCIIFolding
```

**Endpoints reais:**
- `POST http://solr:8983/solr/sei-protocolos/update`
- `POST http://solr:8983/solr/sei-bases-conhecimento/update`
- `POST http://solr:8983/solr/sei-publicacoes/update`

---

## Fase 3 — 1º Protótipo C++ / JOD (Sem. 11–18 · Out–Dez/2026)

**Status:** ✅ Mantida — com foco refinado

### O que o SEI faz (confirmado)

```
PHP → POST http://jod:8080/conversion?format=pdf → [JODConverter JAR → LibreOffice] → PDF → PHP
```

### O que sua extensão C++ vai substituir

```
PHP → sei_turbo_convert(file_path) → [C++ → LibreOffice UNO API direto] → PDF path → PHP
```

### Headers PHP disponíveis (confirmado via `install.sh`)
```bash
dnf install -y php-devel  # php-config, phpize, headers em /usr/include/php/
```

### Argumento acadêmico forte para o paper
> "Enquanto o JODConverter 4.4.8 já implementa paralelismo básico com 2 instâncias LibreOffice (portas 2002/2003), identificamos que o overhead de protocolo HTTP representa X% do tempo total de conversão. Nossa extensão C++ elimina essa camada intermediária através de chamadas diretas à UNO API do LibreOffice."

---

## Fase 4 — Validação Integrada (Sem. 19–22 · Dez/2026)

**Status:** ✅ Mantida

**Cron do SEI real** (do `cron.conf`) — referência para simular carga:
```cron
* * * * *  root  php /opt/sei/scripts/AgendamentoTarefaSEI.php
```
A cada minuto. Seus testes de carga devem simular múltiplas execuções simultâneas.

---

## Fase 5 — Escalabilidade: Crypto + Solr (Sem. 23–30 · Jan–Fev/27)

**Status:** ✅ Mantida — com campos reais disponíveis

### sei_turbo_crypto.so

```cpp
// OpenSSL C API disponível (confirmado: openssl instalado no container)
// EVP_DigestVerify, X509_verify_cert, validação em batch
PHP_FUNCTION(sei_turbo_validate_cert_batch) { ... }
```

### sei_turbo_solr.so

```cpp
// Usar RapidJSON ou nlohmann/json para serializar campos reais:
struct SeiProtocolo {
    long id_prot, id_proc, id_doc;
    std::string desc, numero, nome_arvore;
    // etc. (baseado no schema.xml real)
};
// POST para http://solr:8983/solr/sei-protocolos/update
```

---

## Fase 6 — Relatório Parcial + Framework (Sem. 31–34 · Mar/2027)

**Status:** ✅ Mantida

**Argumento extra para o framework:** o SEI é usado por **mais de 300 órgãos do governo federal**. A metodologia é replicável por qualquer órgão que use o `pengovbr/sei-docker` como base.

---

## Fase 7 — Paper Científico (Sem. 35–42 · Abr–Mai/27)

**Status:** ✅ Mantida

### Trechos de argumentação prontos para o paper

**Seção 3.1 - Ambiente de Experimento:**
> "Utilizamos a infraestrutura oficial de desenvolvimento do SEI (pengovbr/sei-docker, versão 3.6.7) que replica fielmente o ambiente de produção dos órgãos governamentais."

**Seção 3.2 - Calibração dos Simuladores:**
> "Os simuladores foram calibrados com os protocolos reais do SEI: (1) endpoint JOD http://jod:8080/conversion?format=pdf, (2) campos Solr id_prot, id_proc, id_doc conforme schema.xml oficial, (3) validação criptográfica baseada no campo id_assin."

---

## Fase 8 — Apresentações & Encerramento (Sem. 43–52 · Jun–Jul/27)

**Status:** ✅ Mantida

### Slide obrigatório: "Validade do Lab"

Prepare um slide específico respondendo: *"Como você sabe que seu laboratório reflete o sistema real?"*

```
Evidência 1: ConfiguracaoSEI.php (arquivo real de configuração de produção)
             → protocolos exatos de JOD, Solr, Memcached

Evidência 2: schema.xml do Solr (configuração usada em produção)
             → campos usados nos simuladores: id_prot, id_doc, id_assin...

Evidência 3: mysql8-sei50 Dockerfile
             → baixa sei_5_0_0_BD_Ref_Exec.sql do GitHub oficial do governo

Evidência 4: application.yaml do JOD 4.4.8
             → protocolo REST exato: POST /conversion?format=pdf
```

---

## Tabela Consolidada: Original vs. Ajustado

| Fase | Semanas | Ajuste | Impacto |
|------|---------|--------|---------|
| 1 — Fundação | 1–4 | ✅ Sem mudança | — |
| 2 — Docker Stack | 5 | ⚡ Comprimida (repo já tem tudo) | +3–5 dias livres |
| 2 — Simuladores | 6–8 | ⚡ Usar campos/protocolos reais | Paper mais forte |
| 3 — JOD C++ | 11–18 | ⚡ Foco em eliminar HTTP overhead | Argumento mais preciso |
| 4 — Validação | 19–22 | ✅ Sem mudança | — |
| 5 — Crypto+Solr | 23–30 | ⚡ Usar schema.xml real para Solr | Metodologia mais sólida |
| 6 — Parcial | 31–34 | ✅ Sem mudança | — |
| 7 — Paper | 35–42 | ⚡ Adicionar seção sei-docker como evidência | Banca mais convencida |
| 8 — Apresentações | 43–52 | ⚡ Preparar slide "Validade do Lab" | Defesa mais robusta |

---

## Para a Próxima Reunião com o Professor (Checklist)

- [ ] Mostrar o `mapeamento_sei_docker.md` com a análise completa do repo
- [ ] Explicar: `ConfiguracaoSEI.php` revela os 3 protocolos exatos
- [ ] Explicar: vamos usar a infraestrutura Docker real, não construir do zero
- [ ] Explicar: os simuladores usarão campos e protocolos reais (schema.xml, application.yaml)
- [ ] Explicar: sem o código-fonte PHP do SEI — metodologia de simuladores é padrão na área
- [ ] Perguntar: o professor quer mirar EnCUCA (CEUB) + Congresso IC da UnB?
- [ ] Confirmar: início com JOD Simulator em setembro conforme planejado

---

*Documento gerado em 04/08/2026*
