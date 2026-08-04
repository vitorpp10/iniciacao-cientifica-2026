# Mapeamento do Repositório `sei-docker-main` e Conexões com o Projeto de IC (C++)

> [!NOTE]
> Este documento apresenta o mapeamento completo da estrutura de arquivos e diretórios do repositório `sei-docker-main`, detalhando a função de cada componente e a sua conexão direta com a pesquisa de Iniciação Científica (IC) voltada à análise de métricas, profiling e otimização/reescrita de gargalos do SEI em C++.

---

## 1. Visão Geral da Arquitetura e Propósito

O repositório `sei-docker-main` provê o ambiente parametrizado e containerizado para execução, desenvolvimento, testes e infraestrutura do **Sistema Eletrônico de Informações (SEI)** e seu módulo de permissões (**SIP**).

Para a Iniciação Científica, este repositório é a **plataforma fundamental de experimentação**, pois:
1. Permite reproduzir com exatidão o ambiente de produção do SEI 5.0 (PHP 8.2, Solr 9.6.1, JODConverter 4.4.8, MySQL 8).
2. Oferece o ponto de injeção de código (`${SEI_PATH}:/opt/`) para compilação e teste de extensores nativos em C++ ou microsserviços otimizados.
3. Disponibiliza ferramentas de profiling (Xdebug 3.3.2, headers `php-devel`, `gcc`) prontas para medição de desempenho e análise de gargalos.

---

## 2. Árvore de Arquivos e Mapeamento de Conexões com a IC

```
sei-docker-main/
├── .gitignore
├── README.md                    ← Documentação oficial (confirma repo para dev/debug)
├── PIC.pdf                      ← Plano de IC anterior (referência acadêmica)
├── 5722.pdf                     ← Documento governamental de referência
├── plano_trabalho_2017.Fernanda.docx.pdf  ← Plano de trabalho de pesquisa anterior
│
├── containers/                  ← RECEITAS DOCKER DE TODOS OS COMPONENTES
│   ├── Makefile                 ← Automação de build das imagens
│   ├── README.md
│   ├── envcontainers.env.modelo ← Template de variáveis de ambiente
│   │
│   ├── app/                     ← Container PHP legado (PHP 7.x, CentOS)
│   ├── app-php8/                ← ⭐ CONTAINER PHP 8.2 (AMBIENTE PRINCIPAL DA IC)
│   │   ├── app-base-php8/       ← Imagem base da aplicação
│   │   │   ├── Dockerfile       ← Base: Rocky Linux 9.3
│   │   │   └── assets/
│   │   │       ├── install.sh   ← ⭐ INSTALA: php:remi-8.2, php-devel, gcc, libgearman-devel...
│   │   │       ├── copy-packages.sh
│   │   │       └── ca-certIN.pem
│   │   │
│   │   ├── app-dev-php8/        ← ⭐ CONTAINER DE DESENVOLVIMENTO (com Xdebug)
│   │   │   ├── Dockerfile       ← Herda de app-base-php8
│   │   │   └── assets/
│   │   │       ├── install.sh   ← Instala Development Tools, cronie, Xdebug 3.3.2
│   │   │       ├── conf/
│   │   │       │   ├── sei.ini      ← Config PHP (ISO-8859-1, limites 500M)
│   │   │       │   ├── xdebug.ini   ← Config Xdebug (porta 9003)
│   │   │       │   ├── sei.conf     ← VirtualHost Apache (porta 8000)
│   │   │       │   ├── deflate.conf ← Compressão HTTP
│   │   │       │   ├── cron.conf    ← ⭐ Agendador do SEI (* * * * * php ...)
│   │   │       │   └── info.php     ← phpinfo()
│   │   │       └── scripts-e-automatizadores/
│   │   │           ├── command.sh          ← Startup do container (SEI, SIP, Apache)
│   │   │           ├── ConfiguracaoSEI.php ← ⭐⭐⭐ MAPEAMENTO DE SERVIÇOS E PROTOCOLOS
│   │   │           ├── ConfiguracaoSip.php ← Config do SIP
│   │   │           └── ConfiguracaoSip.old.php
│   │   │
│   │   ├── app-ci-php8/              ← Container para CI
│   │   └── app-ci-php8-agendador/    ← ⭐ Container do AGENDADOR em CI
│   │       ├── Dockerfile
│   │       └── assets/               ← entrypoint-agendador.sh
│   │
│   ├── jod/                     ← JOD LEGADO (CentOS 7 + Tomcat + JOD 2.2.2)
│   ├── jod4.4.8/                ← ⭐ JOD ATUAL (Alpine 3.21 + JOD 4.4.8)
│   │   ├── Dockerfile           ← Alpine + LibreOffice + OpenJDK 17
│   │   └── assets/
│   │       └── application.yaml ← ⭐ CONFIGURAÇÃO DE POOLING E PORTAS REST
│   │
│   ├── solr/                    ← SOLR 8.2.0 (Legado)
│   ├── solr-9.4.0/              ← Solr 9.4.0 (Intermediário)
│   ├── solr-9.6.1/              ← ⭐ SOLR ATUAL PARA SEI 5.0
│   │   ├── Dockerfile
│   │   └── assets/
│   │       └── solr9.6.1sei/    ← Cores atualizados do Solr 9
│   │           └── sei-cores-8.2.0/ (ou 9.6.1)
│   │               └── sei-protocolos/
│   │                   └── conf/
│   │                       ├── schema.xml     ← ⭐ SCHEMA REAL DOS CAMPOS E PIPELINE
│   │                       └── solrconfig.xml ← Config do Lucene
│   │
│   ├── databases/               ← BANCOS DE DADOS
│   │   ├── mysql8-sei50/        ← ⭐ MYSQL 8 PARA SEI 5.0
│   │   │   ├── Dockerfile       ← Downloads do repositório oficial do SEI no GitHub
│   │   │   └── assets/          ← pre-install.sql, pos-install.sql, my.cnf
│   │   ├── mariadb-*/
│   │   ├── oracle-*/
│   │   ├── postgres-*/
│   │   └── sqlserver-*/
│   │
│   ├── memcached/               ← Cache em memória
│   ├── mailcatcher/             ← SMTP de teste
│   ├── base-centos/ / base-rocky93/
│   ├── traefik/ / haproxy/
│   ├── openldap/ / phpldapadmin/ / phpmemcachedadmin/ / dbadminer/
│   └── tests/
│
├── dev/                         ← ⭐ AMBIENTE DE DESENVOLVIMENTO LOCAL
│   ├── docker-compose.yml       ← ⭐ ORQUESTRAÇÃO DE SERVIÇOS E VOLUME MOUNT
│   ├── Makefile                 ← Comandos rápidos (make up, make destroy)
│   ├── README.md
│   ├── envs/
│   │   ├── env-mysql-sei5.env   ← ⭐ CONFIGURAÇÃO DA COMBINAÇÃO ALVO DA IC
│   │   └── env-*.env
│   └── tests/
│       └── Selenium/            ← Testes end-to-end automatizados (Python)
│
└── infra/                       ← INFRAESTRUTURA DE PRODUÇÃO / CI-CD
    ├── Makefile / envlocal.env / certificado.pem / generatebase64.sh
    ├── docs/                    ← Instruções avançadas e duplo SEI
    ├── orquestrators/           ← Templates Kubernetes / Rancher / Docker Compose
    └── jenkins/                 ← Pipelines de CI/CD
```

---

## 3. Detalhamento de Cada Seção e Conexão Técnica com a IC

### 3.1. Raiz do Repositório (`/`)

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `README.md` | Documentação oficial confirmando que o repositório destina-se a desenvolvimento e depuração de módulos do SEI. | **Validação de Escopo:** Confirma o ecossistema padrão mantido pela comunidade governamental para expansão do SEI, onde nossas extensões PHP/C++ ou microsserviços serão injetados e testados. |
| `PIC.pdf` | Plano de Iniciação Científica anterior presente na raiz. | **Histórico Acadêmico:** Comprova o uso acadêmico prévio desta infraestrutura Docker para investigações no SEI, fornecendo referências metodológicas e de benchmark. |
| `5722.pdf` | Documentação oficial/normativa governamental relacionada ao SEI. | **Requisitos de Negócio:** Fornece o arcabouço normativo que rege o fluxo de documentos, temporalidade e requisitos de auditoria a serem preservados nas otimizações. |
| `plano_trabalho_2017.Fernanda.docx.pdf` | Plano de trabalho de pesquisa correlata em anos anteriores. | **Metodologia de Medição:** Serve como base de validação para métricas de desempenho e delimitação de módulos críticos sob avaliação. |

---

### 3.2. Diretório `containers/` — Receitas Docker dos Componentes

#### A. Ambiente PHP 8.2 (`containers/app-php8/`) — Núcleo de Execução da Aplicação

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `app-base-php8/Dockerfile` | Define a imagem base sobre Rocky Linux 9.3 com repositório Remi PHP 8.2. | **Ambiente Base de Produção:** Garante a paridade da versão do PHP 8.2 utilizada nas medições de baseline e testes com extensões nativas C++. |
| `app-base-php8/assets/install.sh` | Script de instalação da imagem base. Instala: `php:remi-8.2`, `php-devel`, `php-pecl-gearman`, `php-pecl-memcached`, `php-pecl-apcu`, `gcc`, `libgearman-devel`, `openssl`, `ffmpeg`, `wkhtmltox-0.12.6.1`. | **⭐ AMBIENTE DE COMPILAÇÃO C++:** A presença explícita de **`php-devel`**, **`gcc`** e **`libgearman-devel`** confirma que o container já possui os cabeçalhos C/C++ da Zend API (`php.h`, `zend.h`) e o compilador necessários para construir extensões dinâmicas PHP (`.so`) em C++ ou integradores de fila sem necessidade de alterar a imagem base. |
| `app-dev-php8/Dockerfile` | Herda de `app-base-php8` e adiciona pacotes de desenvolvimento e profiling. | **Container Alvo da IC:** É o container que roda a aplicação durante a fase de experimentos de profiling e benchmarking. |
| `app-dev-php8/assets/install.sh` | Instala `Development Tools` (`make`, `g++`, `gdb`), `cronie`, cliente `mysql` e `Xdebug 3.3.2`. | **⭐ FERRAMENTAS DE PROFILING E DEBUG:** Xdebug 3.3.2 permite gerar arquivos de trace e profiling (`cachegrind`) para localizar cu-de-sacs de CPU em scripts PHP, enquanto as ferramentas C++ (`g++`, `gdb`) permitem depurar extensões compiladas nativas acopladas ao Apache/PHP-FPM. |
| `app-dev-php8/assets/conf/sei.ini` | Configurações PHP do SEI: `include_path`, charset `ISO-8859-1`, `upload_max_filesize=500M`, `post_max_size=505M`. | **Tratamento de Encoding em C++:** O uso de `ISO-8859-1` exige que módulos em C++ tratem adequadamente a codificação de strings durante parsing, indexação ou geração de resumos para evitar erros de encode no Solr (que opera em UTF-8). |
| `app-dev-php8/assets/conf/xdebug.ini` | Define `xdebug.mode=debug`, porta 9003 e protocolo DBGp. | **Depuração Nativa:** Permite depurar a transição de chamadas do PHP para código C++ via gdb ou IDEs conectadas à porta 9003. |
| `app-dev-php8/assets/conf/cron.conf` | Configuração do Crontab no container:<br>`* * * * * php /opt/sei/scripts/AgendamentoTarefaSEI.php`<br>`* * * * * php /opt/sip/scripts/AgendamentoTarefaSip.php` | **⭐ AGENDADOR E TAREFAS DE BACKGROUND:** Revela que o agendador do SEI roda a cada 1 minuto. Processos pesados em background (indexação em lote, expurgo, estatísticas) são disparados aqui, tornando-o um candidato primário para otimização ou reescrita de rotinas em C++. |
| `app-dev-php8/assets/scripts-e-automatizadores/ConfiguracaoSEI.php` | Arquivo PHP de configuração global do SEI. Mapeia:<br>- `'Solr' => 'http://solr:8983/solr'`<br>- `'JODConverter' => 'http://jod:8080/conversion?format=pdf'`<br>- `'CacheSEI' => ['Servidor'=>'memcached', 'Porta'=>'11211']`<br>- Parâmetros de Banco de Dados, E-mail, Limites e Federação. | **⭐⭐⭐ PONTO CENTRAL DE ARQUITETURA E INTERCEPTAÇÃO:** É o arquivo mais importante do sistema. Define as URLs, portas e conectores de todos os microsserviços. Permite redirecionar chamadas nativas do PHP para novos microsserviços ou extensões compiladas em C++ (ex: substituir o cliente de conversão ou indexação). |
| `app-ci-php8-agendador/` | Container otimizado para rodar exclusivamente o Agendador de Tarefas do SEI em ambientes de integração contínua. | **Isolamento de Benchmarking:** Permite isolar o consumo de CPU/Memória das tarefas agendadas em background sem a interferência do tráfego web Apache. |

---

#### B. Conversor de Documentos (`containers/jod4.4.8/`) — Processamento de PDFs e Documentos

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `jod4.4.8/Dockerfile` | Imagem Alpine 3.21 com LibreOffice e OpenJDK 17, executando `java -Xmx2G -jar jodconverter-4.4.8.jar`. | **Identificação do Gargalo de Conversão:** O JODConverter consome até 2GB de RAM por instância e depende do LibreOffice headless, sendo um dos componentes de maior latência no SEI. |
| `jod4.4.8/assets/application.yaml` | Arquivo de configuração do Spring Boot JOD:<br>`port-numbers: [2002, 2003]` (2 instâncias LibreOffice)<br>`server.port: 8080` (API REST)<br>`max-file-size: 1024MB` | **⭐ ARQUITETURA DA API E POOLING DE CONVERSÃO:** Revela que a conversão ocorre via API REST HTTP na porta 8080 com pooling de portas de processo (2002, 2003). Permite projetar substitutos de alta performance em C++ utilizando bibliotecas como `pdfium` ou `poppler` para conversões/extrações diretas sem o overhead do JVM/LibreOffice. |

---

#### C. Motor de Busca (`containers/solr-9.6.1/` e `containers/solr/`) — Indexação e Pesquisa

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `solr-9.6.1/Dockerfile` | Container do Apache Solr 9.6.1 configurado especificamente para o SEI 5.0. | **Motor de Busca Oficial:** Define a versão exata do Solr usada no SEI 5.0 para avaliação de latência de indexação e consultas. |
| `solr/assets/solr8sei/.../sei-protocolos/conf/schema.xml` | Define o esquema de campos do Solr:<br>- Campos: `id_prot`, `id_proc`, `id_doc`, `id_assin`, `id_org_ger`, `desc`, `numero`, `dta_ger`...<br>- Pipeline de texto: `WhitespaceTokenizer` → `StopFilterFactory` (pt) → `WordDelimiterFilterFactory` → `LowerCaseFilterFactory` → `ASCIIFoldingFilterFactory`. | **⭐ ESTRUTURA DE DADOS E TOKENIZAÇÃO:** O `schema.xml` revela a estrutura exata dos documentos indexados pelo SEI e a pipeline de análise de texto. É indispensável para construir parsers de documentos e indexadores customizados em C++ compatíveis com o Solr ou para otimizar o envio de payloads via cURL/C++. |
| `sei-bases-conhecimento/` & `sei-publicacoes/` | Cores do Solr para bases de conhecimento e publicações oficiais. | **Mapeamento Complementar:** Permite analisar as demandas de indexação nos outros dois núcleos de pesquisa do SEI. |

---

#### D. Banco de Dados (`containers/databases/mysql8-sei50/`) — Persistência Racional

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `mysql8-sei50/Dockerfile` | Faz o download automático do script `sei_5_0_0_BD_Ref_Exec.sql` diretamente do repositório oficial do SEI no GitHub (`https://github.com/spbgovbr/sei`). | **⭐ ESQUEMA DE BANCO OFICIAL:** Garante que o ambiente de testes possui exatamente a estrutura relacional oficial do SEI 5.0 (tabelas `protocolo`, `documento`, `atividade`, etc.). |
| `mysql8-sei50/assets/my.cnf` | Configurações do MySQL 8 (buffers, innodb, charsets). | **Tuning de Banco e Profiling de Queries:** Permite analisar o impacto de I/O de banco de dados e comparar com acessos otimizados via C++ MySQL Connector ou Connection Pools nativos. |

---

### 3.3. Diretório `dev/` — Ambiente de Desenvolvimento e Orquestração Local

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `dev/docker-compose.yml` | Orquestra a pilha completa:<br>- `httpd` (app-dev-php8)<br>- `database` (mysql8-sei50)<br>- `memcached`<br>- `solr` (solr-9.6.1)<br>- `jod` (jod4.4.8)<br>- `smtp` (mailcatcher)<br>**Volume Mount:** `${SEI_PATH}:/opt/`<br>**Portas expostas:** 8000 (SEI Web), 8983 (Solr), 11211 (Memcached), 1080 (Mail). | **⭐⭐ PONTO CENTRAL DE INJEÇÃO E BENCHMARKING:** O docker-compose é o orquestrador do ambiente de testes da IC. A montagem de volume `${SEI_PATH}:/opt/` permite injetar o código-fonte do SEI e as extensões/módulos C++ compilados sem necessidade de reconstruir as imagens Docker. |
| `dev/Makefile` | Automação com alvos `make up`, `make destroy`, `make logs`. | **Automação de Testes:** Facilita a execução reprodutível de ciclos de teste e reset de ambiente entre diferentes baterias de testes de carga. |
| `dev/envs/env-mysql-sei5.env` | Define as variáveis de ambiente padrão para o SEI 5.0:<br>`APP_IMAGE=app-dev-php8`<br>`SOLR_IMAGE=solr9.6.1`<br>`JOD_IMAGE=jod4.4.8`<br>`DATABASE_IMAGE=mysql8-sei50` | **⭐ CONFIGURAÇÃO ALVO DA PESQUISA:** Estabelece o conjunto exato de imagens que compõem o baseline de medição de desempenho da IC. |
| `dev/tests/Selenium/PythonExported/test_suiteBasics.py` | Suíte de testes Selenium em Python para automação de login, criação de processos e inclusão de documentos. | **Carga Sintética e Validação de Regressão:** Usado para simular interações de usuários e medir o tempo de resposta ponta-a-ponta (E2E) antes e depois das otimizações em C++. |

---

### 3.4. Diretório `infra/` — Infraestrutura para Produção e Testes em Lote

| Arquivo / Pasta | Descrição Detalhada | 🔗 Conexão com a IC |
| :--- | :--- | :--- |
| `infra/Makefile` & `envlocal.env` | Automação de deploy e configuração de variáveis para produção. | **Paridade com Produção:** Permite verificar como as alterações de código e módulos C++ se comportam sob configurações de infraestrutura reais. |
| `infra/docs/duploSEI/duplosei.md` | Guia de configuração de instâncias múltiplas do SEI na mesma VM. | **Estudo de Escalabilidade:** Ajuda no planejamento de testes de multi-tenancy e disputa de recursos entre instâncias. |
| `infra/orquestrators/` | Templates para Kubernetes (`rancher-kubernetes`) e Docker Compose avançado. | **Deploy de Microsserviços C++:** Oferece os manifestos necessários caso partes do SEI sejam reescritas como microsserviços C++ independentes em containers. |

---

## 4. Matriz de Conexão: Arquivos Chave vs. Ações da IC

| Arquivo Chave | Função no SEI | Impacto Direto na IC em C++ |
| :--- | :--- | :--- |
| `ConfiguracaoSEI.php` | Mapeamento central de URLs e portas do Solr, JOD e Memcached | Interceptação de chamadas; substituição de drivers PHP por bindings C++ ultra-rápidos. |
| `application.yaml` (JOD) | Configuração do serviço de conversão REST (porta 8080, pool 2002/2003) | Referência para benchmarking e criação de microsserviço C++ equivalente (ex: usando Poppler/PDFium). |
| `schema.xml` (Solr) | Estrutura de campos e pipeline de filtros de texto (StopWords, ASCIIFolding) | Guia para implementação de tokenizadores/parsers nativos em C++ com suporte a ISO-8859-1 / UTF-8. |
| `install.sh` (`app-base-php8`) | Instalação de pacotes Linux (`php-devel`, `gcc`, `libgearman-devel`) | Validação de que o container possui o toolchain C++ pronto para compilar Zend Extensions (`.so`). |
| `cron.conf` (`app-dev-php8`) | Execução periódica do AgendamentoTarefaSEI.php (a cada 1 min) | Identificação e perfilamento de tarefas de background em lote para paralelização em C++. |
| `docker-compose.yml` (`dev/`) | Orquestração local e volume mount `${SEI_PATH}:/opt/` | Injeção dinâmica do código C++ e dos módulos de medição sem custos de rebuild. |
| `env-mysql-sei5.env` | Seleção de imagens para o stack SEI 5.0 (PHP 8.2 + Solr 9.6 + JOD 4.4 + MySQL 8) | Definição rigorosa do ambiente de baseline para benchmarking científico. |
| `Dockerfile` (`mysql8-sei50`) | Download automático da base SQL de referência oficial do SEI 5.0 | Garantia de dados de teste idênticos aos usados em homologação oficial do Governo Federal. |

---

## 5. Próximos Passos Recomendados para a Pesquisa

1. **Subir o Ambiente Baseline:** Executar `make up` dentro de `dev/` utilizando o arquivo `env-mysql-sei5.env`.
2. **Coleta de Métricas Baseline:** Rodar os testes sintéticos em `dev/tests/Selenium/` com Xdebug profiler ativado para mapear os hotspots de CPU e memória do PHP.
3. **Desenvolvimento do Módulo C++:** Compilar uma extensão PHP de teste usando `php-devel` e `gcc` dentro de `app-dev-php8`, conectando-a via `ConfiguracaoSEI.php`.
4. **Benchmarking Comparativo:** Medir o tempo de execução e throughput (operações/segundo) entre a versão original em PHP e a versão otimizada em C++.
