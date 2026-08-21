# Zval

A zval (Zend Value) é uma estrutura de dados escrita em C que representa qualquer variável criada no PHP. Como o PHP é dinamicamente tipado, a zval serve para gerenciar o tipo e o valor do dado na memória.

## Zval no PHP 5

Antigamente a zval era alocada na Heap (memória de longo prazo), que exigia uma alocação mais difícil e muito custosa. A estrutura base da zval era definida da seguinte forma:

```c
typedef struct _zval_struct {
    zvalue_value value;
    zend_uint refcount__gc;
    zend_uchar type;
    zend_uchar is_ref__gc;
} zval;
```

A partir dessa estrutura, a zval antiga trabalhava com quatro conceitos principais:

**Union**: Um espaço de memória reservado para apenas um tipo de valor por vez, além de que era alocado com o tamanho do maior tipo de dado suportado pela zval.

**Refcount**: Contador de ponteiros por referência. Exemplo, se `$a = $b = 42`, o refcount seria 2, pois existem dois ponteiros apontando para o 42, não sendo necessário copiar o 42 novamente para a memória.

**Type**: Uma tag utilizada para informar ao sistema qual tipo de dado estava ativo na Union.

**is_ref**: Flag para variáveis referenciais. Serve para fazer uma variável acompanhar o mesmo valor de outra constantemente. Por exemplo, se temos `$b = 20` e depois fazemos `$a = &$b` (usando a referência), o `$a` será igual ao `$b` a qualquer custo, mesmo se o valor mudar depois.

Veja que o tipo do `type` e da flag `is_ref` usam o tipo char do C (`zend_uchar`), ocupando 1 byte, já o `refcount__gc` usa um inteiro de 4 bytes. Isso prova que mesmo sem o valor em si, a estrutura já gastava memória RAM apenas com metadados.

### Coletor de Lixo e o Copy-on-Write (CoW)

O contador de referências (`Refcount`) do PHP 5 tinha uma falha fatal onde ele não conseguia detectar e liberar referências cíclicas (como um array que aponta para si mesmo), basicamente o erro ocorria assim:

```php
$a = []; 
$a[0] = &$a; 
//Array a, aponta para si mesma.
```

Quando fazíamos uma variável apontar para si mesma, o `Refcount` aumentava para 2 (um para o ponto de acesso externo `$a` e o outro para o acesso interno `[0]`). Quando apagava a variável externa (`unset($a)`), o contador caía de 2 para 1, como o contador ainda é 1 por causa do índice interno que aponta para o próprio array, o PHP 5 pensava que ainda existia alguma parte do código usando aquilo, ou seja, o contador de referências ficava preso em 1, contando uma variável que já estava inacessível.

Para resolver isso, o motor do PHP usava um coletor de lixo cíclico adicional que aumentava ainda mais a estrutura real alocada na Heap chamada `zval_gc_info`:

```c
typedef struct _zval_gc_info {
    zval z;
    union {
        gc_root_buffer *buffered;
        struct _zval_gc_info *next;
    } u;
} zval_gc_info;
```

Essa estrutura envelopava a zval comum e injetava um ponteiro extra (`u`) de 8 bytes (em sistemas 64-bits) apenas para monitorar os ciclos na Heap, o que tornava a estrutura de memória (contêiner) ainda mais pesada.

Além desse peso, o PHP 5 dependia fortemente do mecanismo Copy-on-Write (CoW). Múltiplas variáveis podiam compartilhar a mesma zval na memória para economizar espaço, subindo o `Refcount`. Porém, no momento que qualquer uma delas tentasse alterar o seu valor, o motor era obrigado a duplicar toda aquela estrutura complexa e criar uma nova zval no Heap. Quando isso envolvia estruturas cíclicas, o sistema perdia muito tempo repetindo as mesmas tarefas e gerava peso sobre o contêiner, causando vazamento de memória (`memory leak`) e sobrecarregando o coletor de lixo (`garbage collector`).

Resumindo, a zval antiga aloca uma estrutura de dados completa na Heap para armazenar apenas um valor por vez, o que era muito custoso para performance.

## Zval no PHP 7+

Do PHP 7 para frente mudou tudo. Com a chegada do novo parâmetro `zend_refcounted`, as zvals não eram mais alocadas isoladamente na Heap e começaram a ser alocadas na Stack (memória de execução rápida).

A grande sacada foi perceber que tipos simples (`int`, `float`, `bool`, `null`) não precisam mais de contagem de referência. Eles passaram a ser copiados diretamente de uma variável para outra. Enquanto a estrutura antiga passava dos 24 bytes na Heap e gerava grande sobrecarga, a nova zval foi otimizada para ter um tamanho fixo de exatos 16 bytes na Stack.

Desses 16 bytes, 8 bytes são usados para armazenar o valor em si e o restante guarda a tag de tipo e metadados internos. Os engenheiros do PHP perceberam que copiar essas variáveis diretamente na memória era mais eficiente do que gerenciar ponteiros e contadores para cada uma delas.

Com isso, as únicas variáveis que continuaram precisando de ponteiros e contadores foram os tipos complexos (`string`, `array`, `object`, etc).

Os tipos complexos ganharam estruturas de dados próprias e um cabeçalho padrão chamado `zend_refcounted`, que fez a contagem de referência (`Refcount`) sair de dentro da zval. Atualmente, a zval guarda apenas a tag indicando o tipo do valor e um ponteiro direto apontando para qual estrutura representa ela na memória (como a `zend_string`).

## Otimização de strings no PHP 7+

No PHP 5, as strings eram apenas um ponteiro para um texto solto (`char*`) e um número inteiro (`len`) guardados na zval. No PHP 7, elas ganharam uma estrutura própria chamada `zend_string`, definida da seguinte forma:

```c
struct _zend_string {
    zend_refcounted   gc;
    zend_ulong        h;
    size_t            len;
    char              val[1];
};
```

Com essa modelagem, a estrutura passou a gerenciar quatro campos essenciais:

**zend_refcounted**: Um cabeçalho para contar quantas variáveis apontam para essa estrutura e permitir o compartilhamento de strings sem depender diretamente da zval.

**Cache de hash (`h`)**: Para procurar strings dentro de arrays de forma super rápida.

**Len**: Tamanho da string.

**val[1]**: O valor do texto em si.

### Struct Hack

Quando o motor em C vai criar uma string de 100 caracteres, ele aloca na memória o tamanho da estrutura `zend_string` + 100 bytes. Como o valor `val[1]` (array de 1 caractere) é a última coisa declarada na estrutura, o C permite que o motor continue escrevendo o texto além do tamanho original. Isso resulta em um cache hit (dado encontrado com sucesso na cache line). Como o cabeçalho e o texto ficam no mesmo bloco contínuo de memória (cache line), a leitura é muito rápida para o processador, pois ele consegue ler tudo de uma vez.

No contexto da otimização do projeto (Lei de Amdahl [[02-lei-amdahl-estudo]]), a grande sacada de usar C++ é justamente pular todo esse overhead (sobrecarga) de criar zvals e zend_strings, para manipular os buffers de memória (blocos únicos de memória) diretamente.