`subprocess`: biblioteca do python que serve para capturar saída de outra linguagem + lançar comandos, iremos usar essa biblioteca para executar o php e pegar sua saída(tempo para medir benchmark)

*syntax*:
```python
import subprocess

res = subprocess.run(
    ["docker", "exec", "lab-sei", "php", "/var/www/html/index.php"],
    capture_output=True,
    text=True,
)

t = res.stdout
``` 

tiramos o *-it* do comando pois ele serve para abrir o terminal de forma interativa, como nosso código é para automatizar e executar de forma autônoma então precisa disso, usamos *check=False* para se der erro ele apenas continua a automatização sem precisar parar para checar o erro, *text* para pegar o texto e *capture_output* para capturar a saída do código php 

---

`split`: biblioteca do python que serve para dividir os valores de uma lista em vários índices, iremos usar isso para pegar somente o tempo total de cada benchmark ao invés de pegar seu texto todo 

*syntax*:
```python
t = res.stdout

a = t.split()
b = a[-1]
c = b.replace("s", "")
d = float(c)
```

dividimos o texto original em uma lista com índices divididos por espaços, depois pegamos o ultimo valor dessa lista *a[-1]*, onde -1 pega o último valor de uma lista independente do seu tamanho, depois tiramos o "s" que tinha no texto deixando somente os números centrados do teste, no final transformamos em *float*

---

`slicing`: para começar de uma lista a partir x ponto, iremos usar essa técnica para começar na lista a partir do índice 1, ja que o índice é apenas o *warm-up*

*syntax*
```python
list = [1.80, 1.20, 1.22, 1.21, ...]
benchmarks = list[1:]
```

pegamos a lista final somente com os valores de cada rodado anteriormente e começamos pelo índice um com a técnica de *slicing(:1)*, pois o índice 0 é apenas nosso *warm-up*

---

`statistics`: biblioteca do python que serve para fazer médias de testes de qualquer código/lista e métricas como média total, desvio padrão etc 

*syntax*:
```python
import statistics

media = statistics.mean(benchmarks)
stdevv = statistics.stdev(benchmarks)

print(f"media: {media:.4f}s\n stdev: {stdevv:.4f}s\n")
```

usamos *mean* que serve para calcular a mede de uma lista inteira, no caso desse código simulado acima a variável *benchmarks* seria nossa lista com as 21 amostras, depois pegamos o desvio padrão com *stdev* da mesma e guardamos na variável *detour* 

---

***Código final de automação 1***
```python
import statistics
import subprocess

l = []
for i in range(21):
    res = subprocess.run(
        ["docker", "exec", "lab", "php", "/var/www/html/index.php"],
        check=False,
        capture_output=True,
        text=True,
    )
    a = r.stdout
    b = a.split()
    c = b[-1]
    bench = float(c.replace("s", ""))
    l.append(bench)
benchmarks = l[1:]
media = statistics.mean(benchmarks)
stdevv = statistics.stdev(benchmarks)
print(f"media: {media:.4f}s \n stdev: {stdevv:.4f}s \n")
``` 