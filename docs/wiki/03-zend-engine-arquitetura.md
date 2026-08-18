# The Range Operator

O operador do php chamado de *Range Operator (Operador de Alcance)* é um operador fictício testado no motor do PHP que é escrito em C para mostrar como alterar o **Lexer, Parser, AST, Compilation & Opcode e Zend VM** para fazer o PHP aceitar uma nova sintaxe que gera array em sequências númericas, exemplo:

`1 |> 4`: Começa do 1 e vai somando até chegar em 4, parando até o final (4).
Resultado: `[1, 2, 3, 4]`.

Resulta em erro se o ponto final for menor que o inicial ou ser for texto:

`5 |> 2` ou `1 |> 'texto`, resultam em **Error (Exception)**.

Com esse operador fictício o PHP precisa fazer **todas** as suas fases de compilação para entender o código:

**Lexer**: Avisa ao PHP que os símbolos `|` e `>` juntos não são um erro, mas sim uma nova forma de identação no código (um *Token*).

**Parser**: Ensina a regra gramatical, mostra que o operador precisa de um número na esquerda e na direita para funcionar.

**AST**: Organiza a linha de código digita em um mapa visual para o PHP entender quem vem primeiro.

**Compilation**: Traduz esse novo comando para o código interno que o motor do PHP lê chamado **OpCode**.

**Zend VM**: É quem executa a ordem e cria a array na memória do computador.

Abaixo irei explicar mais sobre cada uma das fases do compilador PHP no entendimento sobre as linhas de códigos e como cada etapa entendeu o novo operador `|>`:

## Lexer

Primeira fase do motor do php sobre o código, ele lê todas linhas de código dentro do parâmetro `<?php` e transforma eles em tokens do PHP como por exemplo, `if` vira `T_IF`, `$a` vira `T_VARIABLE` e descarta quebra de linhas e espaços em brancos. Ele sabe como conectar cada variável com seu token conforme as regras do arquivo `zend_language_scanner.l`, se a variável não estiver definida no mesmo ele ativa uma regra padrão de captura:

**Retorna o caractere bruto**: Ele entrega o próprio caractere textualmente para o Parser.

**O Parser decide o resto**: O parser vai olhar para aquele caractere isolado e como não existe nenhuma regra gramatical que aceite aquele símbolo naquela posição o PHP interrompe a execução e exibe o famoso erro: `Parse error: syntax error, unexpected '...'`.

Como foi adicionado no arquivo `zend_language_scanner.l` o operador `|>`, então o lexer retornou ele como `T_RANGE` pois definimos isso:

```c
<ST_IN_SCRIPTING>"|>" {
    RETURN_TOKEN(T_RANGE);
}
```

## Parser

Depois de todos os passos para fazer o Lexer reconhecer nossa nova tipagem vamos atacar a segunda fase da arquitetura Zend engine, que seria a fase Parser.

Para fazer nossa tipagem ser reconhecida pelo Parser, devemos adicionar ela no arquivo `zend_language_parser.y`, um arquivo do tipo *YACC/GNU Bison*, específico para gerar analisadores de código (*parsers*). Dentro desse arquivo devemos adicionar uma linha de código que vai fazer o parser reconhecer o token:

```c
%token T_RANGE                  "|> (T_RANGE)"
```

A partir de agora o Parser reconhece o operador `|>`, porém ainda vai dar erro porque apenas definimos o reconhecimento sobre o operador e ainda está faltando a **regra gramatical**, ou seja, oque o PHP deve fazer quando encontra uma estrutura do tipo T_RANGE, essa regra é criada na próxima fase (*AST*).

O erro atual passa a ser:

`Parse error: syntax error, unexpected '|>' (T_RANGE) in...`

### Tokenizer

*Essa parte é opcional e serve apenas para boa didática.*

O compilador interno do PHP ja reconheceu o `T_RANGE`, mas o PHP possui uma extensão nativa chamada **Tokenizer** (Responsável pelas funções `token_get_all()` e `token_name()`). Essa extensão serve para que ferramentas de análise de código e scripts de usuários consigam ler os tokens.

Ou seja, até o momento se rodarmos a função em um script comum, ela não será reconhecida:

```php
echo token_name(token_get_all('<?php 1|>2;')[2][0]); // Retorna: UNKNOWN
```

Para atualizar a extensão e fazer o reconhecimento do novo token, devemos alterar arquivos internos escritos em C:

1. Navegar até a pasta `cd ext/tokenizer`
2. Executamos o script: `./tokenizer_data_gen.sh` (ele lê os tokens e atualiza o arquivo `tokenizer_data.c`).
3. Voltamos a raiz e recompilamos o PHP com o comando `make`.

Agora a extensão funciona em scripts comuns:

```php
echo token_name(token_get_all('<?php 1|>2;')[2][0]); // Retorna: T_RANGE
```

Agora no estágio atual vamos definir as **regras gramaticais** do operador as primeiras regras a serem definidas serão:

**Precedência (Prioridade)**: Define quem roda primeiro, na matemática por exemplo, a multiplicação (`*`) tem precedência maior que a soma (`+`). No nosso caso o autor definiu que o `|>` terá a mesma prioridade do operador *Spaceship* (`<=>`), que é um operador de comparação de variáveis com base em seu tamanho.

**Associatividade (Encadeamento)**: Define como ele vai se comportar em fila (*ex: `1 |> 3 |> 5`*).
- Alguns operadores leem da esquerda para a direita.
- O operador do autor `|>` retornará um array, ou seja, não faz sentido lógico tentar fazer um "intervalo de um array com outro número" isso daria erro.
- Por isso, ele foi definido como **Não associativo** (`%nonassoc`). Significa que **é proibido encadear o operador**, como por exemplo escrever `1 |> 3 |> 5` vai quebrar a regra gramatical definida.

Adicionamos essas regras no arquivo `Zend/zend_language_parser.y`, isso é feito adicionando a o `T_RANGE` na linha `%nonassoc`:

```c
%nonassoc T_IS_EQUAL T_IS_NOT_EQUAL T_IS_IDENTICAL T_IS_NOT_IDENTICAL T_SPACESHIP T_RANGE
```

Agora vamos fazer o Parser saber qual sintaxe usar. Para isso vamos procurar a regra chamada `expr_without_variable` que seriam expressões sem variáveis diretas. Adicioanamos a nossa nova regra usando o `|` (que significa "OU"):

```c
|   expr T_RANGE expr
        { $$ = zend_ast_create(ZEND_AST_RANGE, $1, $3); }
```

**`expr T_RANGE expr`**: O Parser entende que para o operador funcionar, ele obrigatoriamente precisa de uma expressão na esquerda (`$1`) e outra na direita (`$3`).

**`{...}`**: Se o user digitar exatamente essa estrutura, o Parser executa o códgio que está dentro das chaves.

**`$$`**: Representa o nó de resultado que será gerado

**`zend_ast_create(ZEND_AST_RANGE, $1, $3)`**: Esta função cria oficialmente o nó na **Árvore de Sintaxe Abstrata (AST)**. Ela dá o nome de `ZEND_AST_RANGE` para o nó e passa duas expressões, uma na esquerda (`$1`) e (`$3`) como filhas desse nó.

Para fechar o Parser tenta criar um nó chamado `ZEND_AST_RANGE`, mas o motor do PHP ainda não conhece essa palavra interna em C.

Para o PHP aceitar essa nova estrutura de árvore, precisamos abrir o arquivo `Zend/zend_ast.h` que seria o arquivo do cabeçalho do **AST** e registrar a nova constante no grupo de nós que possuem dois filhos. Adicionamos apenas a linha:

```c
ZEND_AST_RANGE,
```

A partir de agora, ao rodar `1 |> 2;`, o código **não vai mais dar erro de sintaxe**. Agora o Lexer lê, o Parser valida a gramática, o AST monta o mapa visual porém o motor ainda não sabe oque fazer na hora de executar, que seria fase de compilação/Zend VM, fazendo a compilação travar até o momento.

## Compilation

Agora que o Lexer, Parser e AST enxergam e sabem ler nosso operador `|>`, vamos compilar ele e fazer o PHP pegar o AST e percorrer o nó do nosso operador para gerar OpCodes que seria uma instrução de baixo nível para operações.

Basicamente primeiro devemos entrar dentro do arquivo `Zend/zend_compiler.c`, especificamente na função `zend_compile_expr` que possui o switch principal do sistema interno do PHP que olha nós da AST e retorna valores. No caso do operador do autor, vamos criar um novo case para quando ele encontrar uma váriavel do tipo token `ZEND_AST_RANGE` ele entrar chamar uma função terciária que vamos criar para identificar as duas variáveis obrigatórias da nossa função e retorna um resultado, então o código inicial fica assim:

```c
case ZEND_AST_RANGE:
zend_compile_range(result, ast);
return;
```

Agora sempre que o compilador do PHP encontrar com um nó do tipo token `ZEND_AST_RANGE` ele vai chamar a função e jogar oque ele encontrou nela.

Nossa função vai receber o resultado e o nó do compilador do PHP, onde vai funcionar em três passos:

```c
void zend_compile_range(znode *result, zend_ast *ast) {
    zend_ast *left_ptr = ast->child[0];
    zend_ast *right_ptr = ast->child[1];
    znode left_node, right_node;
zend_compile_expr(&left_node, left_ptr);
zend_compile_expr(&right_node, right_ptr);
zend_emit_op_tmp(result, ZEND_RANGE, &left_node, &right_node);
}
```

- 1° Passo: Pega os dados separando o nó da esquerda (`child[0]`) com o nó da direita (`child[1]`).

- 2° Passo: Ele chama a função `zend_compile_expr` para resolver os as variáveis dinâmicas que podem estar em ambos os lados do operador por exemplo `($a + 2 |> $minhaFuncao())`, ou seja, o PHP precisa compilar e resolver oque tem em cada lado primeiro.

- 3° Passo: Junta as duas partes resolvidas (`left_node, right_node`) e cria uma instrução real chamada `ZEND_RANGE` usando a função `zend_emit_op_tmp`.

`zend_ast`: É uma estrutura de árvore, representa o código em um formato visual e hierárquico.

`znode`: Estrutura usada exclusivamente durante a compilação para traduzir a árvore em instruções de máquina.

### OpCode

Opcodes são instruções reais de operações que possuem dois valores operandos (`op1`, `op2`) e um valor para guardar o resultado (`result`). Existem vários tipos de nós opcodes (`znode`), entre os principais estão:

**IS_CV**: Para valores comuns digitados pelo user (ex: `$a`), podem ser vistas no terminal com o comando `!0`.

**IS_VAR**: Expressões complexas de valores (ex: `$objeto->propriedade`)

**IS_CONST**: Valores literais fixos (ex: `2`, `"texto"` etc);

**IS_TMP_VAR**: Valores temporários criados pelo PHP durante contas. Eles duram muito pouco tempo e servem apenas para passar dados de uma linha para a outra. Podem ser vistas no terminal com o comando `~0`.

Então basicamente agora o PHP lê o código, monta a AST (Árvore de Sintaxe Abstrata) e gera os Opcodes necessários para compilar porém o Zend VM ainda não sabe oque é o `ZEND_RANGE`, então o programa vai travar.

## Zend VM

Agora que o nosso compilador sabe interpretar o nó `ZEND_AST_RANGE` e gerar o **OpCode** correspondente, precisamos ensinar a **Zend VM** a executá-lo. Para isso, criamos um *Handler* (manipulador).

Se tentarmos rodar o **PHP** neste estágio, o programa vai travar (causando um *Segmentation Fault*), pois a **VM** (Máquina Virtual) vai ler a instrução `ZEND_RANGE`, mas não saberá como processar a lógica por trás dela em tempo de execução.

O primeiro passo que deve ser feito para o reconhecimento é a assinatura do *handler* `ZEND_VM_HANDLER`:

```c
ZEND_VM_HANDLER(182, ZEND_RANGE, CONST|TMP|VAR|CV, CONST|TMP|VAR|CV) {}
```

Isso seria um pseudo-macro onde o arquivo `Zend/zend_vm_gen.php` vai ler o arquivo e gerar código **C** a partir dele.

* `182`: ID numérico do **opcode** (geralmente o último número do arquivo `zend_vm_opcodes.h`).
* `ZEND_RANGE`: Nome do **opcode**/instrução.
* `CONST|TMP|VAR|CV`: Os tipos possíveis de dados que pode aparecer no `op1` (Esquerda).
* `CONST|TMP|VAR|CV`: Mesma coisa, tipos possíveis porém para o outro lado `op2` (Direita).

Logo após assinatura do handler, antes de testarmos os casos que podem ocorrer para cada tipo recebido de ambos os parâmetros, a Zend VM precisa fazer uma etapa de preparação e captura de dados. O código realiza isso através do seguinte trecho:

```c
USE_OPLINE 
zend_free_op free_op1, free_op2;
zval *op1, *op2, *result, tmp;

SAVE_OPLINE();
op1 = GET_OP1_ZVAL_PTR_DEREF(BP_VAR_R);
op2 = GET_OP2_ZVAL_PTR_DEREF(BP_VAR_R);
result = EX_VAR(opline->result.var);
```

Oque esse trecho inicial faz:

**USE_OPLINE e SAVE_OPLINE()**: Ativam e salvam o ponteiro da linha de instrução atual que a VM está executando para o C saber onde olhar o script PHP.

**zend_free_op free_op1, free_op2**: Cria variáveis de controle de memória. Elas servem para registrar se os valores passados são temporários para apagar eles depois manualmente (Garbage Collector).

**zval *op1, *op2, *result, tmp**: Aloca os ponteiros dos contêineres de dados do PHP (`zval`). O `op1` representa o valor da esquerda, o `op2` representa o da direita, `result` guardará a array final e `tmp` será um valor temporário para cálculos.

**GET_OP1_ZVAL_PTR_DEREF...**: Essas macros vão até a memória da VM, pegam os valores que o usuário digitou no PHP (ex: o `1` e o `10` de `1 |> 10`) e os entrega mastigados para os nossos ponteiros `op1` e `op2`.

Após a criação e alocação de valores, vamos tratar dos vários casos que podem ocorrer ao ler os parâmetros.

O código vai chamar a função `Z_TYPE_P()` que serve para ler os dois operadores `op1` e `op2` para descobrir se oque está checando é válido ou não, dentro dessa função vai ser feito os seguintes passos:

1. Ambos são inteiros:

Se ambos forem inteiros (`IS_LONG`), escrevemos usando uma macro em C `Z_LVAL_P` (LVAL = Long Value) para valores inteiros.

Como o operador do autor foi projetado para ser um gerador de range exclusivamente crescente, o `op1` não pode ser maior que o `op2`, ou seja, vamos comparar `min` e `max`, onde `min` é o valor da esquerda e `max` o da direita. Logo, se `min > max`, quer dizer que não está em ordem crescente e a regra gramatical do operador foi quebrada, então retornamos erro com a função `zend_throw_error()` e pulamos a função `HANDLE_EXCEPTION()`.

Se não tiver com erro vamos fazer `max - min` para calcular o tamanho e ver se não passa do limite de memória do PHP (`HT_MAX_SIZE`), se tudo estiver certo incrementamos o tamanho em 1.

2. Floats com Inteiros:

Se tivermos floats no meio, o tratamento muda. O autor usa `long double` no C para armazenar os valores.

Usamos `long double` pois um simples `double` tem apenas 53 bits de precisão e pode perder dados ao converter ou misturar um inteiro muito grande. O `long double` garante segurança total. Extraímos os valores usando `Z_DVAL_P` (DVAL = Double Value).

3. Tipos não suportados (Else):

Se o usuário tentar fazer algo como `"texto" |> "texto"`, o código cai no else final e dispara uma exceção informando que os operadores usados não são suportados.

Uma vez que validamos os números e sabemos o tamanho da array, agora precisamos criá-lo. Arrays no PHP por baixo dos panos são chamadas de HashTables.

```c
Z_TYPE_INFO(tmp) = IS_LONG
array_init_size(result, size);
zend_hash_real_init(Z_ARRVAL_P(result), 1);
```

**Z_TYPE_INFO(tmp) = IS_LONG**: Define o tipo da variável temporária

**array_init_size(result, size)**: Define que o tamanho da array resultado vai ser esse.

**zend_hash_real_init(Z_ARRVAL_P(result), 1)**: Inicializa o hash table. O número 1 aqui é um detalhe crucial, ele diz ao PHP para criar uma Packed HashTable.

**Packed HashTable** é uma otimização em grande escala que ao invés de criar um array associativo pesado, ele cria uma array C real (chaves numéricas sequenciais e ordenadas), que gasta muito menos memória e é bem mais rápida.

Para preencher essa Array usamos um laço de repetição otimizado:

```c
ZEND_HASH_FILL_PACKED(Z_ARRVAL_P(result)) {
    for (i = 0; i < size; ++i) {
        Z_LVAL(tmp) = min + i;
        ZEND_HASH_FILL_ADD(&tmp); 
    }
} ZEND_HASH_FILL_END();
```

As macros `ZEND_HASH_FILL_PACKED` e `ZEND_HASH_FILL_END` delimitam um ambiente de inserção rápida na memória do nosso array (`result`). Dentro desse bloco, executamos um laço for padrão do C que roda o número exato de vezes correspondente ao tamanho calculado.

A cada iteração, o código realiza duas ações:

- Ele calcula o número atual fazendo `min + i` (o valor inicial mais o índice da volta atual) e injeta esse valor na variável `tmp` (temporário).

- A macro `ZEND_HASH_FILL_ADD(&tmp)` copia esse valor diretamente para dentro do bloco de memória sequencial do array e, de forma automática, avança o ponteiro interno para a próxima posição livre da memória.

No final do handler, o C faz o encerramento:

```c
FREE_OP1();
FREE_OP2();
ZEND_VM_NEXT_OPCODE_CHECK_EXCEPTION();
```

**FREE_OP...**: Se as variáveis `free_op1` e `free_op2`, forem valores temporários criados apenas para essa conta (`IS_TMP_VAR`), isso apaga elas da memória (Garbage Collection manual).

**ZEND_VM_NEXT_OPCODE**: Diz para a máquina virtual que o código foi finalizado e não houve erros, assim continuando para próximos scripts de usuários.

Com o código C escrito no arquivo, precisamos rodar o script gerador da VM no terminal:

```bash 
php Zend/zend_vm_gen.php
```

Isso lê o nosso cabeçalho e gera todas as variações da nossa função dentro do arquivo `zend_vm_opcodes.c`. Depois é só recompilar o PHP usando o comando `make`.

Embora o operador agora funcione se o usuário escrever algo como `assert(1 |> 2);`, o PHP vai retornar *Segmentation Fault* e travar. Quando um assert falha, o PHP tenta ler a AST de trás para frente para transformar o código de volta em texto (String) para mostrar na mensagem de erro, isso se chama **Pretty Printer**. Como ele não conhece o nosso operador, ele quebra.

Precisamos ir no arquivo `Zend/zend_ast.c`, procurar a função que exporta strings (`zend_ast_export_ex`) e ensinar a ela que o nosso nó `ZEND_AST_RANGE` deve ser impresso na tela como `" |> "`, com a mesma prioridade que definimos no Parser.

Com isso fechamos todo o ciclo de como um novo operador é criado de forma funcional em cada etapa do sistema interno do PHP para aceitar novas variáveis.