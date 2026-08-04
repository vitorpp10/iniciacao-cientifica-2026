# Protocolo de Pesquisa

> **Status:** A redigir na Semana 4 (22–28/ago/2026)
>
> **Seções obrigatórias:**
> - Hipótese (H0 e H1)
> - Variáveis independentes e dependentes
> - Ambiente de experimento
> - Procedimento de coleta de dados
> - Critérios de aceitação
> - Ameaças à validade

## Hipótese

- **H0:** Extensões C++ integradas ao Zend Engine não reduzem significativamente a latência das operações de conversão de documentos, validação criptográfica e indexação no Solr em relação à implementação PHP pura.
- **H1:** Extensões C++ integradas ao Zend Engine reduzem a latência dessas operações em no mínimo 2x (speedup ≥ 2).

## Variáveis

**Independente:** Implementação utilizada (PHP puro vs. extensão C++)

**Dependentes:**
- Tempo médio de execução (ms)
- Uso de CPU (user + system time)
- Uso de memória pico (MB)
- Throughput (operações/segundo)

---
*Completar na Semana 4*
