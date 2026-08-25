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

