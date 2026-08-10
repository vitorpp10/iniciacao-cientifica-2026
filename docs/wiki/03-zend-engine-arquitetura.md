"""
ler php-internals-book

explicar fases que o zend engine faz para ler um código PHP, como ele faz isso as funções etc

Lexing
Parsing
AST
Compilation & OpCode
"""

## *The Range Operator*

O operador do php chamado de *Range Operator (Operador de Alcance)* é um operador fictício testado no motor do PHP que é escrito em C para mostrar como alterar o ***Lexer, Parser, AST, Compilation & Opcode e Zend VM*** para fazer o PHP aceitar uma nova sintaxe que gera array em sequências númericas, exemplo:

`1 |> 4`: Começa do 1 e vai somando até chegar em 4, parando até o final (4).
Resultado: **`[1, 2, 3, 4]`**

Resulta em erro se o ponto final for menor que o inicial ou ser for texto:

***`5 |> 2 ou 1 |> 'texto`***, resultam em ***Error (Exception)***.

Com esse operador fictício o PHP precisa fazer **todas** as suas fases de compilação para entender o código:

**Lexer**: Avisa ao PHP que os símbolos `|` e `>` juntos não são um erro, mas sim uma nova forma de identação no código (um *Token*).

**Parser**: Ensina a regra gramatical, mostra que o operador precisa de um número na esquerda e na direita para funcionar.

**AST**: Organiza a linha de código digita em um mapa visual para o PHP entender quem vem primeiro.

**Compilation**: Traduz esse novo comando para o código interno que o motor do PHP lê chamado **OpCode**

**Zend VM**: É quem executa a ordem e cria a array na memória do computador

Abaixo irei explicar mais sobre cada uma das fases do compilador PHP no entendimento sobre as linhas de códigos e como cada etapa entendeu o novo operador `|>`:

## *Lexer*

Primeira fase do motor do php sobre o código, ele lê todas linhas de código dentro do parâmetro *`<?php`* e transforma eles em tokens do PHP como por exemplo, *`if`* vira *`T_IF`*, *`$a`* vira *`T_VARIABLE`* e descarta quebra de linhas e espaços em brancos. Ele sabe como conectar cada variável com seu token conforme as regras do arquivo *`zend_language_scanner.l`*, se a variável não estiver definida no mesmo ele ativa uma regra padrão de captura:

**Retorna o caractere bruto**: Ele entrega o próprio caractere textualmente para o Parser.

**O Parser decide o resto**: O parser vai olhar para aquele caractere isolado e como não existe nenhuma regra gramatical que aceite aquele símbolo naquela posição o PHP interrompe a execução e exibe o famoso erro: ***`Parse error: syntax error, unexpected '...'`***.

Como foi adicionado no arquivo `zend_language_scanner.l` o operador `|>`, então o lexer retornou ele como `T_RANGE` pois definimos isso:

```c
<ST_IN_SCRIPTING>"|>" {
    RETURN_TOKEN(T_RANGE);
}
```