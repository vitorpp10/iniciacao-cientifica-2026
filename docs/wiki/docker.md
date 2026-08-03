## Docker

`image`: É um pacote que contém o sistema operacional de forma reduzida, dependências, bibliotecas e o software principal
- `image: php:8.2-apache`

`container`: Possui seu próprio sistemas de arquivos, rede, árvore de processos tudo de forma isolada 

`volumes`: É um mecanismo que permite mapear um diretório local da sua máquina(host) direto para o container, oque é alterado no host reflete diretamente no container
- `volumes: - "./meu_codigo_local:/caminho/no/container"`
- `volumes: - "./lab_sei:/var/www/html"`, onde *lab_sei* seria o diretório local que vai ser conectado ao contâiner e atualizado a cada alteração e */var/www/html* onde apache se conecta no caminho do container, para levar as alterações ate ele

`ports`: O container está em uma rede isolada para que o computador(host) consiga acessar o apache(no container) é preciso de uma porta TCP que redireciona e faz essa ligação
- `ports: - "PORTA_HOST:PORTA_CONTAINER"` 
- `ports: - "8080:80"`, onde *8080* seria a porta local onde vai ser transmitido o container e *80* a porta que o apache vai se conectar para transmitir

`command`:  executar comandos para que o docker o faça
- `command: tail -f /dev/null`: serve para deixar o docker rodando o contâiner sobre o terminal infinitamente consumindo 0% de CPU, como imagens de CLI não ficam ligadas para sempre usa-se esse método para manter o contâiner vivo de forma saudável
 
***Syntax base docker***:  
```yaml
version: '3.8'

services:
	nome_servidor: 
		image: php:8.2-apache
		container_name: nome_container
		ports: 
			- "PORTA_HOST:PORTA_APACHE"
		volumes: 
			- "./pasta_local:/caminho/apache/aqui"
		command: tail -f /dev/null
```

*Duas maneiras de executar script php via docker*: 
- Isolar testes entrando no contâiner:
	- `docker exec -it <nome_container> bash`
- Apenas executar no terminal direto no interpretador:
	- `docker exec -it <nome_container> php /var/www/html/index.php` 