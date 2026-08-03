`CPU-Bound`: O script exige muito da CPU como por exemplo hash de senhas, cálculos pesados, Open SSL etc, fazendo a CPU atingir seu pico máximo(100%)
`I/O-Bound`: O script fica parado pois espera algo de fora como por exemplo uma query de um banco, requisição HTTP, json etc

`desvio padrão`: Seria para saber se um conjunto de fatores é muito junto ou separado ou para saber qual base do conjunto mais tem chance de ser, exemplo: 

a média de uma sala de aula é 15 anos
o desvio padrão dela é 2 anos
então a maioria das pessoas daquela sala tem entre 13 e 17 anos, pois deve ser somado e subtraido o valor principal a o desvio padrão

para achar o desvio padrão deve ser feito as seguintes etapas: 
*exemplo com valores: 13,14,15,16*
- ache a média deles: 13+14+16+17 = 60 / 4 = 15
- calcular distância até a média: 
	13: 13-15 = -2² = 4 
	14: 14-15 = -1² = 1
	16: 16-15 = 1² = 1
	17: 17-15 = 2² = 4
- fazer a média das distâncias: 4+1+1+4 = 10 / 4 = 2.5
- raiz quadrada da média encontrada:  √2.5 = 1.58
- desvio padrão = 1.58