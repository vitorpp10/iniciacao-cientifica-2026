# Estudo da Lei de Amdahl

> **Status:** A redigir na Semana 1 (01–07/ago/2026)
>
> **Seções planejadas:**
> - Formulação matemática
> - Implicações para PHP
> - Cálculos de speedup teórico para os 3 cenários do IC
> - Limitações da lei
> - Extensão de Gustafson

## Fórmula

S = 1 / ((1 - p) + p/s)

Onde:
- S = speedup total
- p = fração paralelizável/otimizável
- s = speedup da parte otimizada

## Cenários do IC (calcular na Semana 1)

1. JOD: se p=40% e s=4x → S = ?
2. Crypto: se p=70% e s=5x → S = ?
3. Combinado (3 extensões): S = ?

---
*Completar na Semana 1*
