# Arquitetura do repositório

Este documento descreve e explica a finalidade de cada documento e pasta referente a pesquisa

## Raiz do Projeto

`.gitignore`: Para impedir o envio ao github de arquivos binários em c++, dependências PHP ou cache locais 

`structure.md`: Para mostrar para que serve cada pasta e arquivos da pesquisa

`readme.md`: Apresentação geral da pesquisa, planos, instruções etc

## docs 

Pasta referente a documentações científicas da pesquisa, relatórios etc

`references/`: Arquivos de citação .bib e fichamentos 

`wiki/`: Base de conhecimento aprendidos/usados na pesquisa para suporte e entendimento do processo 

## docker

Arquivos para funcionamento e configurações docker para garantir que o uso seja idêntico para cada máquina

`docker-compose.yml`: Configurações gerais dos contâiners 

`configs/`: Arquivos de inicialização do ambiente que determinam o comportamento das linguagens e configurações iniciais do sistema 

## src

Onde desenvolvimento do projeto acontece 

`php/`: Simuladores/Arquivos no formato PHP

`cpp/`: Módulos/Arquivos no formato C++

## benchmarks

Scripts e ferramentas que simulam cenários de alta demanda para coleta de dados

`scripts/`: Automações de testes 

`datasets/`: Cargas de testes estáticas 

`results/`: Arquivos no formato de planilha(.csv) medindo os resultados gerados

## tests

Validações que garantem o funcionamento dos códigos antes dos testes reais 

## results_analysis

Gráficos para mostrar resultados gerados e comparações entre testes feitos