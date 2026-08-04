# 🎯 Método de Estudo — Meta: Top 1 / Menção Honrosa

> **Premissa:** Menção honrosa é aceitável. Top 1 é o objetivo.
> Esse método não é para "passar" — é para dominar o projeto a ponto de
> conseguir explicar qualquer parte para qualquer pessoa.

---

## O Princípio Central: Não Memorize, Construa

A diferença entre quem ganha top 1 e quem ganha menção honrosa não é inteligência — é **profundidade de entendimento**.

Quem ganha top 1 consegue:
- Explicar o projeto para uma criança de 12 anos E para um especialista da área
- Responder "por que você fez assim e não assado?" sem hesitar
- Antecipar as perguntas da banca e já ter as respostas na apresentação
- Relacionar cada decisão técnica com o objetivo da pesquisa

**Seu lema:** *"Se não consigo ensinar, não entendi."*

---

## Estrutura Semanal (20h/semana — não negociável)

```
SEG  TER  QUA  QUI  SEX  SAB
 3h   3h   3h   3h   3h   5h  = 20h
```

### Rotina Diária (3h, seg–sex)

```
00:00 – 00:15  → Revisão do dia anterior (o que fiz, o que aprendi)
00:15 – 02:30  → Trabalho focado (Pomodoro: 25min on + 5min off)
02:30 – 03:00  → Documentar: o que aprendi hoje? Próximo passo?
                 Git commit com mensagem descritiva
```

### Rotina do Sábado (5h)

```
00:00 – 00:30  → Revisão semanal: o que conquistei esta semana?
00:30 – 03:30  → Bloco pesado: coding longo, escrita de documento, benchmark
03:30 – 04:30  → Revisão do que produziu: está bom? O que melhorar?
04:30 – 05:00  → Planejamento da próxima semana: 3 objetivos concretos
```

---

## As 5 Técnicas de Estudo

### 1. Técnica Feynman (para cada conceito novo)

**Quando usar:** Para tópicos teóricos (Amdahl, zval, Zend API, OpenSSL)

**Como:**
1. Estude o conceito (livro, vídeo, phpinternalsbook)
2. Feche o material
3. Explique em voz alta como se fosse ensinar para alguém que nunca viu
4. Onde você travou = o que não entendeu de verdade
5. Volte ao material só para aquele ponto específico
6. Repita até conseguir explicar sem parar

**Teste:** Se você consegue explicar o porquê de `zval` custar ~24 bytes de overhead e por que isso importa para sua extensão C++, você entendeu. Se não consegue, não entendeu.

---

### 2. Build-Measure-Learn (para implementação)

**Quando usar:** Para código (simuladores, extensões C++, benchmarks)

**Como:**
1. **Build mínimo:** escreva o menor código que pode funcionar (MVP)
2. **Measure:** rode, meça, veja o que acontece
3. **Learn:** o que o resultado te diz? O que precisa mudar?
4. Repita

**Regra de ouro:** nunca escreva mais de 50 linhas sem testar o que escreveu.

---

### 3. Documentação Como Aprendizado

**Quando usar:** Sempre, para tudo

**Como:** Cada coisa que você aprende vira um documento. Não para o professor — para você mesmo. Escrever força o entendimento.

**Exemplos:**
- Entendeu zval? → Escreva uma seção em `docs/01-analise-overhead-php.md`
- Compilou uma extensão? → Documente os passos exatos em `docs/wiki/php.md`
- Encontrou um bug? → Escreva o diagnóstico e a solução no commit message

**Por que funciona:** daqui a 3 meses, quando for escrever o paper, você terá toda a matéria-prima pronta.

---

### 4. Estudo Orientado a Perguntas

**Quando usar:** Antes de cada sessão de estudo

**Como:** Antes de abrir qualquer material, escreva 3 perguntas que quer responder hoje. Exemplo:
- "Por que o JIT do PHP não resolve o problema que minha extensão resolve?"
- "Qual é a diferença entre `emalloc` e `malloc` no contexto de extensões?"
- "Por que 50 repetições com descarte de outliers > 3 repetições?"

Só termine a sessão quando tiver respondido as 3 perguntas.

---

### 5. Revisão Espaçada (para não esquecer)

**Quando usar:** Para manter o conhecimento ao longo dos 12 meses

**Como:**
- Depois de estudar algo, revise em: 1 dia, 3 dias, 1 semana, 2 semanas, 1 mês
- Use Anki (app de flashcards) ou simplesmente uma pasta `docs/wiki/` bem organizada
- **Mínimo:** no início de cada sábado, passe 30 min revisando notas da semana anterior

---

## O Que Estudar e Em Que Ordem (Fase 1)

### Esta Semana (Semana 2: 08–14/ago)

**Segunda:**
- [ ] phpinternalsbook.com — Capítulo 1: Basic Structure of PHP
- [ ] Anotar: o pipeline de execução do PHP (lexer → parser → AST → opcodes → executor)
- [ ] Responder: "o que acontece entre `<?php echo 'hello'; ?>` e aparecer na tela?"

**Terça:**
- [ ] phpinternalsbook.com — Capítulo 2: zval
- [ ] Anotar: o que é `zval`, quantos bytes ocupa, o que é copy-on-write
- [ ] Responder: "por que `$a = $b` pode não copiar a variável imediatamente?"

**Quarta:**
- [ ] phpinternalsbook.com — Zend Memory Manager
- [ ] Anotar: diferença entre `emalloc()` e `malloc()`, o que são pools
- [ ] Responder: "por que o alocador do PHP é mais rápido que malloc() puro para objetos pequenos, mas mais lento para grandes?"

**Quinta:**
- [ ] Estudar type juggling: `==` vs `===`, string→int implícito
- [ ] Anotar: quais operações do PHP envolvem conversão de tipo implícita
- [ ] Responder: "por que `1 == '1a'` é true em PHP? O que acontece internamente?"

**Sexta:**
- [ ] Estudar OPcache e JIT do PHP 8
- [ ] Anotar: o que OPcache guarda em cache? Por que JIT não resolve o problema da sua extensão?
- [ ] Redigir primeiros parágrafos de `docs/01-analise-overhead-php.md`

**Sábado:**
- [ ] Instalar Xdebug 3.3.2 no container Docker
- [ ] Configurar `xdebug.ini`: mode=profile, profiler_output_dir=/tmp/cachegrind
- [ ] Criar script PHP simples (loop com string ops, array, math)
- [ ] Rodar script → gerar arquivo `.cachegrind`
- [ ] Instalar QCachegrind (ou KCachegrind) no host
- [ ] Abrir o `.cachegrind` e identificar: top 5 funções mais custosas
- [ ] Documentar o processo com screenshots/notas

---

## Como Usar o Gemini (Regras de Ouro)

**Regra 1: O Gemini é um assistente, não um professor.**
Você precisa entender. Se o Gemini explicou algo e você não entendeu de verdade, peça para explicar de novo com outro enfoque. Não passe para o próximo tópico sem entender.

**Regra 2: Sempre teste o entendimento.**
Depois que o Gemini explicar algo, feche o chat e tente explicar para si mesmo. Se não conseguir, não entendeu.

**Regra 3: Use o prompt certo.**
- Para estudar: "modo estudo: [tópico]"
- Para implementar: "modo impl: [tarefa]"
- Para revisar seus documentos: "modo revisão: [cole o texto]"
- Para se preparar para o professor: "modo reunião: [tema]"

**Regra 4: Cole código real, erros reais, resultados reais.**
O Gemini é muito mais útil com contexto concreto. Nunca pergunte em abstracto se puder mostrar o que está acontecendo.

---

## Como Usar o Git (Disciplina Essencial)

```bash
# Commit todo dia que trabalhar (mesmo que pequeno)
git add .
git commit -m "feat(docs): adiciona seções 1-3 de analise-overhead-php"

# Formato do commit message:
# tipo(escopo): o que fez
# tipos: feat, fix, docs, bench, test, refactor
# escopo: docs, src/php, src/cpp, docker, benchmarks

# Tags em cada milestone:
git tag -a v0.1-fundacao -m "Fase 1 completa: docs teoricos"
git push origin main --tags
```

---

## Como Preparar a Apresentação Final (Meta Top 1)

A apresentação é onde você separa menção honrosa de top 1. Comece a pensar nisso desde agora.

### O que os Top 1 fazem diferente:

**1. Contam uma história, não listam resultados**
- Não: "implementamos X e obtivemos speedup de 2.7x"
- Sim: "o governo federal processa milhões de documentos no SEI. Identificamos que a conversão de documentos gastava Y segundos por chamada. Nossa extensão reduz isso para Z segundos. Em um órgão com 1000 usuários, isso significa..."

**2. Antecipam as perguntas na própria apresentação**
- Se a banca provavelmente vai perguntar "como você valida?", responda isso NO SLIDE antes de perguntarem
- Isso demonstra maturidade científica

**3. Dominam os números**
- Saibam de cor: speedup de cada extensão, número de repetições, nível de confiança, desvio padrão
- Saibam explicar: por que esse número faz sentido? O que seria surpreendente?

**4. Conectam com impacto real**
- Traduzam os números técnicos em impacto: "X% de redução de tempo = Y mil horas economizadas por ano nos órgãos"

**5. Demo ou vídeo (se possível)**
- Se der, tenha um vídeo de "antes vs depois" mostrando a extensão funcionando
- Alternativa: gráfico animado mostrando o speedup em tempo real

---

## Checklist Mensal de Qualidade

No início de cada mês, responda:

- [ ] Consegui explicar o projeto inteiro em 2 minutos para alguém de outra área?
- [ ] Todos os meus resultados estão em CSVs no repositório?
- [ ] Fiz commit pelo menos 4x essa semana?
- [ ] Entreguei (ou estou no prazo para) a ficha de efetividade?
- [ ] Meu orientador sabe o que estou fazendo? (reunião quinzenal!)
- [ ] Tenho pelo menos 1 resultado concreto (número, gráfico) para mostrar?

---

## Referências de Estudo por Fase

### Fase 1 (Ago/2026)
- phpinternalsbook.com — capítulos 1-4
- Amdahl, G.M. (1967) — você tem o PDF
- Documentação oficial do Xdebug: xdebug.org/docs

### Fase 2 (Set–Out/2026)
- Documentação PHP `microtime()`, `memory_get_peak_usage()`, `getrusage()`
- QCachegrind: kcachegrind.github.io
- Documentação Apache Solr: solr.apache.org/guide/

### Fase 3 (Out–Dez/2026)
- phpinternalsbook.com — capítulos sobre extensões
- PHP source code: github.com/php/php-src (referência de como extensões são feitas)
- CMake docs: cmake.org/documentation

### Fase 5 (Jan–Fev/27)
- OpenSSL C API: openssl.org/docs/man3.0
- RapidJSON: rapidjson.org
- libcurl C API: curl.se/libcurl/c

---

*"A menção honrosa é garantida se você entregar o que está no roadmap.
O top 1 vem quando você entende tão bem o projeto que consegue
explicar qualquer parte com confiança, conectar com impacto real,
e apresentar os resultados de forma que a banca não tem como negar."*

---

*Criado em 04/08/2026*
