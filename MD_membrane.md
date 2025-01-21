# Simulações de Dinâmica Molecular de Proteínas em Membranas com Ligantes Utilizando GROMACS
## Tutorial Dinâmica Molecular de Proteínas Associadas a Membranas em Complexo com Ligantes

Este documento descreve os passos necessários para realizar simulações de dinâmica molecular de membrana utilizando o software GROMACS, abrangendo as etapas de **minimização de energia**, **equilíbrio**, e **produção**, além de análises comuns. 

Os arquivos de entrada são gerados a partir do [CHARMM-GUI](https://charmm-gui.org/), que facilita a construção de sistemas proteína-ligante imersos em membranas lipídicas. O tempo de simulação da produção deverá ser modificado no arquivo ```step7_production.mdp```. Para um simulação de ```500ns```, ```nsteps``` deve ser igual a ```250000000```.

Ou se preferir, faça a equação:

```nsteps = tempo desejado (em ps) / dt (em ps)```

```​500 ns = 500.000 ps```

```dt = 0.002 ps```

---

## Etapas da Simulação

### 1. **Minimização de Energia**
Execute o comando abaixo para realizar a minimização de energia:
```bash
gmx grompp -f step6.0_minimization.mdp -o step6.0_minimization.tpr -c step5_input.gro -r step5_input.gro -p topol.top -n index.ndx

gmx mdrun -nt 12 -v -deffnm step6.0_minimization &
```

### 2. **Equilíbrio**
Realize a etapa de equilíbrio (repetir o processo até step6.6):
```bash
gmx grompp -f step6.1_equilibration.mdp -o step6.1_equilibration.tpr -c step5_input.gro -r step5_input.gro -p topol.top -n index.ndx

gmx mdrun -nt 12 -v -deffnm step6.1_equilibration &

```

### 3. **Produção**
Produção utilizando GPU

```bash
gmx grompp -f step7_production.mdp -o step7_production.tpr -c step6.6_equilibration.gro -t step6.6_equilibration.cpt -p topol.top -n index.ndx

gmx mdrun -nt 12 -v -deffnm md_400 -cpi state.cpt -nb gpu -gpu_id 1 & -append
```

Produção sem GPU

```bash
gmx mdrun -nt 22 -v -deffnm step7_production &
```

### 1. **Centralizar a Trajetória**
Gere um arquivo centrado:

```bash
gmx trjconv -s step7_production.tpr -f md_400.xtc -o md_400_center.xtc -pbc mol -center
```

### 2. **Gerar Index**
Crie um arquivo de índices personalizado:

```bash
gmx make_ndx -f md_400.gro -o index.ndx
```

Para definir grupos específicos, use os comandos:

```bash
13 & ! a H*

name 32 UNL_Heavy

q
```

### Análises
### 1. **RMSD (Root Mean Square Deviation)**
Para calcular o RMSD da proteína sozinha:

```bash
gmx rms -s step7_production.tpr -f md_400_center.xtc -o rmsd_protein.xvg -tu ns
```

Para calcular o RMSD do ligante utilizando o índice:

```bash
gmx rms -s step7_production.tpr -f md_400_center.xtc -n index.ndx -o rmsd_ligand.xvg -tu ns
```

### 2. **RMSF (Root Mean Square Fluctuation)**
Para calcular o RMSF por resíduo:

```bash
gmx rmsf -s step7_production.tpr -f md_400_center.xtc -o rmsf_protein.xvg -res
```

### 3. **Ligações de Hidrogênio (H-Bond)**
Calcule as ligações de hidrogênio:

```bash
gmx hbond -s step7_production.tpr -f md_400_center.xtc -n index.ndx -num hbond.xvg -b 0 -tu ns
```
