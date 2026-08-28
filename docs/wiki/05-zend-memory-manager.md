# Zend Memory Manager

O PHP opera sob o modelo Share-Nothing. Na prática, isso significa que um mesmo processo pode tratar centenas ou milhares de requisições, mas, por padrão, o motor do PHP esquece qualquer informação sobre a requisição atual assim que ela é finalizada.

Se a Zend Engine dependesse diretamente do alocador dinâmico tradicional da `libc` (`malloc` e `free`), ela encontraria dois gargalos críticos:

1. **Memory Leak**: Em aplicações de alto tráfego, alocar memória dinâmica de forma tradicional abre uma brecha perigosa para esquecer de liberar o buffer. Acumular esses vazamentos esgotaria rapidamente a RAM do servidor (Out of Memory).
2. **Syscalls**: Solicitar e liberar memória no Kernel Space para milhares de variáveis (zvals) criadas e destruídas por segundo geraria um gargalo absurdo de latência.

Para resolver esses problemas, foi criado o **Zend Memory Manager (ZendMM)**. Ele atua como uma camada em C para alocar e liberar memória dinâmica, substituindo o alocador da `libc` ao copiar a sua API.

O ZendMM divide o gerenciamento de memória em duas estratégias lógicas bem definidas:

## 1. Alocações Request-Bound

Representam cerca de 95% das alocações dinâmicas de extensões e do motor. São memórias utilizadas exclusivamente enquanto o PHP processa uma requisição específica (como strings, arrays e objetos do script).

O desenvolvedor C utiliza a API espelhada do ZendMM, que geralmente adiciona o prefixo `e` às chamadas. Essa camada funciona como uma rede de segurança: no fim da requisição, o ZendMM é desligado e limpa qualquer buffer alocado que não tenha sido explicitamente liberado pelo programador, evitando vazamentos mortais.

Syntax:

```c
emalloc(size_t)
ecalloc(size_t nmemb, size_t size)
efree(void *)
```

## 2. Alocações Persistentes

São informações raras que precisam permanecer inalteradas e sobreviver a múltiplos ciclos de requisições. Um exemplo é a string `_SERVER`, que é reutilizada em todas as requisições, ou o caminho do executável do PHP.

O desenvolvedor utiliza a API persistente (prefixo `pe` ou um parâmetro extra), e essa alocação contorna o ZendMM, sendo direcionada para uma chamada tradicional de `malloc()` da `libc`.

Syntax:

```c
// O '1' no final sinaliza que a estrutura ignora o ZendMM e usa libc
pemalloc(size, 1)
pefree(ptr, 1)
pestrdup(str, 1)
```


## Chunks, Pages e Bins

Para evitar o custo de solicitar memória ao Sistema Operacional a cada variável, o ZendMM aloca blocos gigantes de memória de uma vez e gerencia a separação internamente. A estrutura por trás disso é a `_zend_mm_heap`.

Ela gerencia o estado global do alocador, sabe quais partes dos grandes blocos (chunks) estão livres para serem usados e onde alocar novos espaços. Conceitualmente, sua hierarquia é dividida assim:

```
_zend_mm_heap
 ├── main_chunk      (O bloco principal de trabalho atual)
 ├── cached_chunks   (Blocos mantidos vivos entre requisições)
 ├── huge_list       (Lista para alocações absurdamente grandes)
 └── free_slots[BIN] (Compartimentos para pequenos pedaços)
```

Para organizar essa memória, o ZendMM divide as alocações em três subníveis físicos: Chunks, Pages e Bins.

### Chunks e Pages

Uma Chunk é um bloco contíguo de 2 MiB que serve para facilitar a contabilidade de espaço. O ZendMM divide esse Chunk em Pages de 4 KiB (4096 bytes) cada.

A alocação funciona assim:

- 1 Chunk = 2 MiB = 2.097.152 bytes
- 1 Page = 4 KiB = 4.096 bytes
- Total: 2.097.152 / 4096 = 512 pages por Chunk.

A primeira página de cada Chunk é reservada para os metadados do próprio bloco. Quando o motor pede, por exemplo, um `emalloc(12000)`, o ZendMM divide isso em 3 páginas contíguas para satisfazer a alocação:

`12000 / 4096 ≃ 2,93`.

Para localizar essas páginas livres rapidamente, o Chunk utiliza dois mapas nos seus metadados:

**`free_map`**: Um mapa de 512 bits onde cada bit (0 ou 1) representa se a página está livre ou ocupada.

**`map`**: Um array que descreve o que exatamente está armazenado em cada página ocupada.

### Bins

Gastar uma página inteira de 4096 bytes para alocar uma string de 24 bytes causaria um desperdício massivo. A Bin foi criada para resolver este problema.

Um Bin permite que várias alocações pequenas compartilhem uma mesma Page. O ZendMM possui 30 classes de Bins predefinidas, com tamanhos exatos: 8, 16, 24, 32... até 3072 bytes.

Se o PHP precisa criar um objeto de 32 bytes, a página de 4096 bytes é fatiada da seguinte forma:

``` 
Page de 4096 bytes

┌────32────┐
│ slot     │
├────32────┤
│ slot     │
├────32────┤
│ slot     │
├────32────┤
│ slot     │
│ ...      │
└──────────┘
``` 

Isso permite que uma única página acomode 128 variáveis de 32 bytes de forma perfeitamente organizada, facilitando a reutilização de slots assim que elas não são mais usadas.

Essa arquitetura de compartimentos de tamanhos fixos introduz um conceito chamado **fragmentação interna**.

Se o motor solicitar `emalloc(2000)`, o ZendMM não criará um slot exato de 2000 bytes. Ele encaixará essa variável no Bin de 2048 bytes. Os 48 bytes que vão sobrar ficarão inutilizados. O motor aceita esse pequeno desperdício de RAM em troca de não gastar ciclos de CPU calculando e reajustando ponteiros matematicamente dinâmicos.

### Três Categorias de Alocação

Com essa arquitetura, toda vez que o PHP chama `emalloc()`, o tamanho do dado define seu caminho físico imediato:

```
emalloc(size)
     │
     ├── ≤ 3072 B
     │      ↓
     │     BIN (Alocação Pequena - Fatias dentro de uma Page)
     │
     ├── > 3072 B (E menor que 2 MiB)
     │      ↓
     │    PAGES (Alocação Grande - Múltiplas Pages contíguas do Chunk)
     │
     └── Grande demais para um Chunk
            ↓
          mmap() (Alocação Enorme - Pedido direto ao SO)
            ↓
        huge_list
``` 