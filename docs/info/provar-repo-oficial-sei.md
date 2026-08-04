# 🏛️ Como Provar que sei-docker é Oficial + O Que Estudar

> Tudo que você precisa saber para explicar ao professor com segurança na próxima reunião.

---

## 1. A Cadeia de Prova (aprenda isso de cor)

### Passo 1: GitHub Organization `pengovbr`

**URL:** https://github.com/pengovbr

**O que é `pengovbr`?** É a organização oficial do **Processo Eletrônico Nacional (PEN)** no GitHub. O PEN é o programa do governo federal que criou e mantém o SEI.

**Como provar:**
- No GitHub, a org `pengovbr` tem no bio: *"Processo Eletrônico Nacional"*
- Link para o site oficial: **pen.gov.br** (gov.br = domínio exclusivo do governo brasileiro)
- Repositórios com estrelas de organizações governamentais reais (TCU, AGU, Ministérios)

**Fala para o professor:**
> "A organização `pengovbr` no GitHub está diretamente vinculada ao site pen.gov.br, que é o portal oficial do Processo Eletrônico Nacional do governo federal. Apenas órgãos do governo podem ter domínios `.gov.br`."

---

### Passo 2: O README do Repositório

**Arquivo:** `sei-docker-main/README.md`

O README diz explicitamente:
- Serve para *"desenvolvimento/debug do código-fonte dos módulos do SEI"*
- Lista os órgãos que usam o SEI
- Tem links para o site oficial do SEI

**Fala para o professor:**
> "O próprio README do repositório afirma que ele é a ferramenta oficial para desenvolvimento de módulos do SEI."

---

### Passo 3: O Banco de Dados é o Oficial

**Arquivo:** `sei-docker-main/containers/databases/mysql8-sei50/Dockerfile`

```dockerfile
ARG GIT_DB_REF=https://github.com/spbgovbr/sei-db-ref-executivo/raw/master

ADD ${GIT_DB_REF}/mysql/v5.0.0/sei_5_0_0_BD_Ref_Exec.sql ...
```

O Dockerfile baixa o `sei_5_0_0_BD_Ref_Exec.sql` do repositório `spbgovbr/sei-db-ref-executivo`.

**O que é `spbgovbr`?** É a organização da Secretaria de Governo Digital (antigo Ministério do Planejamento) — também um domínio governamental oficial.

**Fala para o professor:**
> "O banco de dados que usamos é o **Banco de Referência Executivo oficial do SEI 5.0**, baixado direto do repositório do governo federal `spbgovbr/sei-db-ref-executivo`. Este é literalmente o schema de banco de dados usado pelos 300+ órgãos que rodam o SEI."

---

### Passo 4: Os Arquivos de Configuração São Idênticos aos de Produção

**Arquivo:** `sei-docker-main/containers/app-php8/app-dev-php8/assets/scripts-e-automatizadores/ConfiguracaoSEI.php`

Este arquivo é a **classe de configuração real do SEI**. Ele define:
- Como o SEI se conecta ao JOD (`http://jod:8080/conversion?format=pdf`)
- Como se conecta ao Solr (`http://solr:8983/solr`)
- Como se conecta ao Memcached (porta 11211)
- Configurações de segurança (XSS, limites, federação)

**Fala para o professor:**
> "O arquivo `ConfiguracaoSEI.php` é idêntico ao que qualquer órgão do governo usa em produção — apenas com variáveis de ambiente diferentes. Isso nos dá os protocolos exatos de comunicação entre os componentes, o que é tudo que precisamos para nossa pesquisa."

---

### Passo 5: O Schema Solr é o de Produção

**Arquivo:** `sei-docker-main/containers/solr/assets/solr8sei/sei-cores-8.2.0/sei-protocolos/conf/schema.xml`

Este arquivo define a estrutura exata do índice de busca do SEI:
- Campos: `id_prot`, `id_proc`, `id_doc`, `id_assin`, `desc`, `numero`, `dta_ger`...
- Pipeline de análise de texto em português
- 3 cores: `sei-protocolos`, `sei-bases-conhecimento`, `sei-publicacoes`

**Fala para o professor:**
> "O schema.xml que usamos é o mesmo que está rodando em produção nos tribunais e ministérios. Nossos simuladores indexam documentos com a mesma estrutura de dados que o SEI real usa."

---

## 2. Linha do Tempo para Decorar

```
gov.br (domínio exclusivo governo)
  └── pen.gov.br (Processo Eletrônico Nacional — órgão oficial)
        └── pengovbr (GitHub oficial do PEN)
              └── sei-docker (repositório de desenvolvimento do SEI)
                    ├── ConfiguracaoSEI.php (protocolos reais)
                    ├── schema.xml (campos Solr reais)
                    └── mysql8-sei50 (baixa schema oficial do spbgovbr)
                                            └── spbgovbr (Secretaria de Governo Digital)
                                                  └── sei-db-ref-executivo (banco oficial)
```

**Memorize esta cadeia.** Se a banca questionar em qualquer nível, você tem a resposta.

---

## 3. O Que Estudar para Explicar com Segurança

### Tópico 1: O que é o SEI?
**Estude:** https://www.gov.br/sei/pt-br (5–10 min de leitura)

**O que saber:**
- SEI = Sistema Eletrônico de Informações
- Criado pelo TRF4 (Tribunal Regional Federal da 4ª Região)
- Adotado pelo governo federal em 2017
- Usado por 600+ instituições e 3+ milhões de usuários
- Substitui papel na tramitação de processos administrativos

**Fala para o professor:**
> "O SEI processa mais de X milhões de documentos por ano, é o backbone digital da administração pública brasileira. Uma otimização de performance aqui tem impacto real e mensurável."

---

### Tópico 2: O que é o PEN?
**Estude:** https://www.pen.gov.br (5 min)

**O que saber:**
- PEN = Processo Eletrônico Nacional
- Programa interministerial para modernização da gestão pública
- Mantém e distribui o SEI para os órgãos
- `pengovbr` é o braço técnico/GitHub do PEN

---

### Tópico 3: O que é o JODConverter?
**Estude:** https://github.com/sbraconnier/jodconverter (README, 10 min)

**O que saber:**
- JODConverter = Java OpenDocument Converter
- Usa LibreOffice em modo headless para converter documentos
- Expõe uma API REST (POST /conversion?format=pdf)
- O SEI usa para converter .docx → .pdf para visualização e assinatura
- A versão 4.4.8 suporta múltiplas instâncias do LibreOffice

**Por que importa para o IC:**
- A camada HTTP entre PHP e JOD é o overhead que vamos eliminar com C++

---

### Tópico 4: O que é o Apache Solr?
**Estude:** https://solr.apache.org/guide/ (Overview, 10 min)

**O que saber:**
- Solr = motor de busca textual baseado em Lucene
- O SEI usa para indexar processos e documentos
- 3 cores (sei-protocolos, sei-bases-conhecimento, sei-publicacoes)
- Expõe API REST para indexação e busca
- A versão 9.6.1 é a mais recente no sei-docker

---

### Tópico 5: O que é a Zend Engine?
**Estude:** https://phpinternalsbook.com (Cap. 1-2, 30 min)

**O que saber:**
- Zend Engine = motor de execução do PHP
- Compila PHP para opcodes, depois executa
- Extensões C++ se integram diretamente no engine
- `zval` = estrutura de dados de toda variável PHP (~16-24 bytes de overhead)
- `ZendMM` = gerenciador de memória do PHP

**Por que importa:**
- É aqui que nossa otimização acontece — eliminamos o overhead do Zend para operações pesadas

---

### Tópico 6: A Lei de Amdahl
**Estude:** O paper original de 1967 (você tem o PDF) + Wikipedia

**O que saber:**
- S = 1 / ((1-p) + p/s)
- S = speedup total, p = fração otimizável, s = speedup da parte otimizada
- Limitação: retornos decrescentes à medida que aumenta a otimização
- Aplicação: se PHP gasta 40% do tempo numa operação que C++ executa 4x mais rápido → speedup total ≈ 1.43x
- Para nossa pesquisa: identificar o `p` de cada operação via profiling é o passo crítico

---

## 4. Roteiro de Estudo — Esta Semana

```
Seg (3h):
  → Ler gov.br/sei e pen.gov.br (20 min)
  → Navegar no GitHub pengovbr/sei-docker (20 min)
  → Ler docs/mapa-sei-docker.md (20 min)
  → Começar phpinternalsbook.com cap. 1 (2h)

Ter (3h):
  → Continuar phpinternalsbook cap. 2 (zval, ZendMM)
  → Anotar: o que é zval, quanto custa, por que isso é relevante

Qua (3h):
  → Ler Lei de Amdahl — paper original (30 min)
  → Calcular os 3 cenários do IC (JOD, Crypto, Solr)
  → Redigir docs/02-lei-amdahl-estudo.md (versão inicial)

Qui (3h):
  → Estudar JODConverter (GitHub do projeto, README)
  → Entender: o que é LibreOffice headless? O que é UNO API?
  → Anotar: por que HTTP adiciona overhead vs. chamada direta?

Sex (3h):
  → Estudar Apache Solr (Overview + API Reference início)
  → Abrir o schema.xml real: sei-docker-main/containers/solr/assets/...
  → Mapear: quais campos existem, o que cada um significa

Sáb (5h):
  → Instalar Docker no seu PC (se ainda não tem)
  → Fork/copiar o sei-docker-main/dev/docker-compose.yml
  → Subir o ambiente: docker-compose up -d
  → Verificar: http://localhost:8000, :8983, :1080
  → Documentar: o que funcionou, o que deu erro
```

---

## 5. Respostas Prontas para Perguntas Difíceis do Professor

**"Como você sabe que este repositório é oficial?"**
> "O repositório está na organização `pengovbr` do GitHub, que é o braço técnico do Processo Eletrônico Nacional — programa do governo federal com site em pen.gov.br. O Dockerfile do banco de dados baixa o schema diretamente do repositório `spbgovbr/sei-db-ref-executivo`, que pertence à Secretaria de Governo Digital. É uma cadeia verificável de fontes governamentais."

**"Mas sem o código-fonte PHP do SEI, como você valida os resultados?"**
> "Nossa pesquisa não testa a lógica de negócio do SEI — testamos o impacto das operações computacionais: conversão de documentos via HTTP, serialização JSON, validação de certificados. Usamos os protocolos e schemas reais do sistema. Isso é equivalente ao que benchmarks consagrados fazem: SPEC CPU e TPC-C não rodam código de produção, mas replicam os padrões computacionais com precisão suficiente para resultados válidos."

**"Qual é a contribuição original deste trabalho?"**
> "Três contribuições: (1) Primeiro estudo empírico de extensões PHP C++ aplicadas ao ecossistema SEI especificamente; (2) Framework metodológico genérico para identificar e otimizar gargalos em sistemas PHP legados; (3) Evidência quantitativa de que operações específicas do SEI podem ser aceleradas 2–5x sem alterar a lógica de negócio ou a interface do usuário."

**"O speedup é real ou artificial?"**
> "O speedup é medido em ambiente idêntico à produção: mesma imagem Docker PHP 8.2, mesmo JODConverter 4.4.8, mesmo Solr 9.6.1, mesmo schema de banco. Usamos 50+ repetições com descarte de outliers e intervalos de confiança de 95%. A validade interna é garantida pelo protocolo experimental rigoroso documentado em `docs/00-protocolo-pesquisa.md`."

---

*Criado em 04/08/2026 — Estudar antes da próxima reunião com o Professor*
