`index.php`: código inicial para simular gargalo e lentidão do PHP

O PHP por padrão utiliza em seus loops *Single-Threaded*, ou seja, não utiliza de threads para aumentar o desempenho além de ser interpretado pelo *Zend Engine*, para cada volta em um loop a máquina virtual do PHP tem que ler o opcode, alocar memória, checar tipos etc. Logo, um simples loop usando uma função pesada como a `password_hash()` do tipo *Key Derivation(Funções feitas para serem pesadas)* ja forçam a CPU dependendo quase ao máximo, exemplo do que ja forçaria uma CPU em código php:

```php
<?php //php começa com essa tag

//loop em php
for($i = 0; $i < 100; $i++) {
	//string em php
	$texto = "senha" . $i;
	//função key derivation
	$hash = password_hash($texto, PASSWORD_BCRYPT);
}
```

`<?php`: Apache começa a ler em php a partir daqui, tudo que vem antes ou depois é tratado apenas como texto

`$`: Seria tipo um *auto* em c++, ele define o tipo automaticamente analisando a linha de código escrita 

`.`: Seria o tipo para concatenar variáveis, a soma continua sendo o *"+"* porém ele não concatena, que concatena é o *"."*

`microtime()`: função do php do tipo *wall-clock time* que no caso qualquer coisa que acontecer no espaço tempo em que o cronômetro é iniciado vai ser contabilizado ao tmepo final, ele se equivale ao mesmo formato do *std::chrono* 
- `microtime() ou microtime(false)`: retorna uma string com o tempo final dividido em segundos e milissegundos
- `microtime(true)`: retorna um float em segundos + millissegundos -> *2.1s*

*Código para medir em php com microtime()*: 
```php
<?php
$s = microtime(true);
for($i = 0; $i < 100; $i++) {
	$t = "password" . $i;
	$h = password_hash($t, PASSWORD_BCRYPT);
}
$e = microtime(true);
echo "time duration: " . $e - $s . "s\n";
``` 