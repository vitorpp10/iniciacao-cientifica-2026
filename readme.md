# IC : Otimização via Extensões C++

**Pesquisa:** Otimização de Performance em Sistemas de Gestão Pública via Módulos C++: Um Estudo de Caso no Ecossistema SEI  
**Instituição:** CEUB (Programa de Iniciação Científica - PIC 2026/2027)  

## Objetivo
Desenvolver e validar um framework de otimização de baixo nível para sistemas de gestão pública em PHP (com foco no ecossistema SEI). A proposta consiste na injeção de bibliotecas dinâmicas nativas (C++) diretamente no motor da linguagem (Zend Engine) para reduzir a latência e o consumo de recursos em módulos pesados (conversão de documentos, validação criptográfica e indexação).

## Metodologia (Como faremos?)
A pesquisa não depende do código-fonte proprietário do SEI. Utilizaremos uma replicação de dados dos padrões de processamento em um ambiente de laboratório controlado.

**Montagem do Laboratório**: Criação de um ambiente totalmente isolado usando Docker, deixando pronto o servidor com Apache, PHP 8.2, banco de dados MySQL e o motor de busca Solr.
**Descobrindo os Gargalos**: Desenvolvimento de 3 rotinas de teste em PHP puro (simulando processos de JOD, criptografia e buscas no Solr) para mapear os pontos mais lentos do sistema com a ajuda do Xdebug.
**C++**: Criação de extensões nativas em C++ (arquivos .so) integradas diretamente no motor do PHP para assumir o trabalho pesado e eliminar o peso extra que o PHP consome com memória e conversão de tipos.
**Testes**: Testes práticos de alta carga usando Apache Bench ou JMeter para comparar o PHP puro contra a nova extensão em C++, calculando o ganho real de velocidade e eficiência com base na Lei de Amdahl.

## Stack
- **Linguagens:** PHP 8.2, C++ (GCC/CMake)
- **Infraestrutura:** Docker, Docker Compose
- **Análise & Profiling:** Xdebug, QCachegrind, Valgrind
- **Testes & Métricas:** JMeter, PHPUnit, Python (Matplotlib para gráficos)

## Cronograma Resumido (Ago/2026 a Jul/2027)

| Fase | Período | Objetivo Principal |
| :--- | :--- | :--- |
| **1. Fundação** | Ago/26 | Estudo de baseline, Lei de Amdahl, overhead PHP e protocolo. |
| **2. Baseline** | Set - Out/26 | Criação dos simuladores PHP e coleta de métricas de gargalo. |
| **3. Protótipo C++** | Out - Dez/26 | Estudo da Zend API e desenvolvimento da extensão `sei_turbo_jod.so`. |
| **4. Validação** | Dez/26 | Testes de carga, integração Docker e análise Valgrind. |
| **5. Escalabilidade** | Jan - Fev/27 | Desenvolvimento das extensões adicionais (Crypto e Solr). |
| **6. Relatório Parcial**| Mar/27 | Entrega do Relatório Parcial PIC e documentação do framework. |
| **7. Paper Científico** | Abr - Mai/27 | Redação do artigo final (20-25 pgs) e relatório final. |
| **8. Encerramento** | Jun - Jul/27 | Entrega de Resumos, polimento do GitHub e apresentações (EnCUCA/UnB). |
