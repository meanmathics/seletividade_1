# Análise Computacional de Seletividade em Antibióticos no PyMOL (PDB: 3WJ6)

Este repositório documenta uma análise de química computacional sobre o mecanismo de inibição de antibióticos, focando na seletividade. O objetivo é ir além da simples análise geométrica ("chave-fechadura") e investigar a física da atração eletrostática que guia a ligação do fármaco.

### A Lógica da Seletividade

A ideia é identificar quantitativamente por que a Ampicilina é letal para um micro-organismo, mas completamente **inócua** para as células humanas. A resposta é simples:

* **Células Humanas (Eucariotas):** Possuem uma membrana plasmática flexível.
* **Bactérias (Procariotas):** Possuem, além da membrana, uma parede celular externa rígida, composta de **peptidoglicano**.

Esta parede é crucial para proteger a bactéria da Lise Osmótica (acúmulo excessivo de água no interior da célula).

A parede de peptidoglicano é construída por um conjunto de enzimas. A enzima-chave que faz a "costura" final (ligação cruzada) é a **DD-Transpeptidase** (também chamada de PBP - Proteína Ligadora de Penicilina). Esta enzima utiliza um resíduo de **Serina (SER)** em seu sítio ativo para atacar o substrato natural da bactéria (um peptídeo D-Ala-D-Ala) e formar esta ligação cruzada.

A estrutura da Ampicilina (um antibiótico beta-lactâmico) foi evoluída para mimetizar o substrato natural das enzimas PBP: o dipeptídeo D-Alanil-D-Alanina (D-Ala-D-Ala).

O resíduo de Serina do sítio ativo "confunde" a Ampicilina com o substrato natural, atacando o anel beta-lactâmico (um anel quadrado de 4 membros, altamente tensionado). Esta reação forma uma **ligação covalente** permanente, inativando a enzima. Como resultado, a bactéria não consegue mais construir sua parede celular, levando à lise e morte de forma completamente seletiva, já que as células humanas não possuem esta maquinaria.

### Modelo de Estudo: O Dímero (Antes e Depois)

Utilizamos a estrutura de cristal da DD-transpeptidase (PBP5) de *E. coli* (PDB ID: `3WJ6`). Este arquivo é um modelo de estudo ideal, pois contém um **dímero** na unidade assimétrica que captura os dois estados em um único arquivo:

* **Cadeia A (O "Depois"):** A enzima em seu estado "ocupado", ligada covalentemente ao inibidor Ampicilina.
* **Cadeia B (O "Antes"):** A enzima em seu estado "apo", ou vazia, fornecendo uma linha de base perfeita para comparação.

---

## Análise 1: A Comparação Geométrica (O Ajuste Induzido)

O primeiro passo é entender a mudança *estrutural* que ocorre na enzima. Para isso, criamos objetos separados para cada cadeia e as sobrepusemos estruturalmente.

O comando `super (enzima_B and name CA), (enzima_A and name CA)` revela um **RMSD de 0.309 Å**, provando que as duas cadeias são quase idênticas estruturalmente e formam um par de comparação "antes e depois" perfeito.

Após isolar o sítio ativo (`pocket_bound` na Cadeia A) e a região correspondente na enzima vazia (`pocket_apo` na Cadeia B), podemos visualizar o "ajuste induzido":

![Imagem da Comparação Geométrica](https://raw.githubusercontent.com/meanmathics/seletividade_1/refs/heads/main/img/pocket-2.png)

A imagem acima mostra como o bolso da enzima ocupada (cinza) se "fecha" e se molda ao redor do inibidor, em comparação com a forma do bolso vazio da enzima B (em vermelho).

No entanto, a geometria sozinha não explica *por que* o inibidor foi atraído para este local.

## Análise 2: A Física da Atração (O Potencial Eletrostático - MEP)

Aqui, investigamos o mapa eletrostático da molécula. A hipótese é que o bolso vazio possui uma "assinatura de carga" de alta energia que é estabilizada ou neutralizada pela ligação do inibidor.

### O MEP "Antes": O Convite Eletrostático

Primeiro, calculamos o MEP (usando o plugin APBS - Adaptive Poisson-Boltzmann Solver) para a superfície do bolso vazio (`pocket_apo`). O APBS calcula o Potencial de Superfície Eletrostático (ESP) da molécula num ambiente de solvente, uma metodologia central no Design de Fármacos Baseado em Estrutura (SBDD).

![Imagem do MEP "Antes"](https://raw.githubusercontent.com/meanmathics/seletividade_1/refs/heads/main/img/before_1.png)

O resultado é um campo de alto contraste, com fortes regiões de potencial negativo (vermelho, rico em elétrons) e positivo (azul, pobre em elétrons). Este é o "mapa de atração" que o inibidor "vê" ao se aproximar.

### O MEP "Depois": A Neutralização da Carga

Em seguida, calculamos o MEP para o complexo ativo (`complexo_ativo`, que é `pocket_bound + inibidor`).

![Imagem do MEP "Depois"](https://raw.githubusercontent.com/meanmathics/seletividade_1/refs/heads/main/img/after_1.png)

O resultado é uma superfície muito mais "estável" e eletricamente neutra. As fortes manchas vermelhas e azuis que existiam no bolso vazio desapareceram. Isso é a prova visual de que as cargas complementares do inibidor se encaixaram e **neutralizaram** o potencial de alta energia do bolso.

**A ligação é, portanto, impulsionada tanto pela forma quanto, crucialmente, pela complementaridade eletrostática.**

---

## Conclusão da Análise Computacional

Os dados demonstram conclusivamente que a inibição da PBP não segue um modelo rígido de "chave-fechadura". Em vez disso, a ligação do antibiótico explora a flexibilidade inerente da enzima (mecanismo de *induced fit*).

Esta mimetização não é apenas estérica (relacionada à forma), mas é fundamentalmente eletrostática. Esta distinção é crucial para validar metodologias computacionais (como o uso do APBS) no design de fármacos [01, 02]. O ponto crucial da seletividade é que esta maquinaria de síntese, vital para a bactéria, está completamente ausente nas células humanas.

Isso não nos leva a uma simples conclusão, mas a uma nova pergunta interessante:

> **E se pudéssemos desenhar computacionalmente moléculas que se ligassem ainda mais seletivamente, prevendo sua afinidade eletrostática antes mesmo de sintetizá-las?**

---

## Referências Utilizadas

* [01] Song, Y., et al. (2010). Structural insights into the mechanism of action of penicillin-binding protein 5 from Escherichia coli. *J Am Chem Soc.*, 132(16), 5866-5874.
* [02] Montaner, M., et al. (2023). PBP Target Profiling by β-Lactam and β-Lactamase Inhibitors... *Microbiol Spectr.*, 11(1), e0303822. 

## Apêndice: Script de Comandos PyMOL para Reprodução

```python
# --- 1. CONFIGURAÇÃO INICIAL (PARA SOBREPOR O DÍMERO) ---
fetch 3WJ6
remove solvent
hide everything, 3WJ6
create enzima_A, 3WJ6 and chain A
create enzima_B, 3WJ6 and chain B
disable 3WJ6
show cartoon, enzima_A
show cartoon, enzima_B
color white, enzima_A
color wheat, enzima_B
super (enzima_B and name CA), (enzima_A and name CA)

# --- 2. CONSTRUINDO O MECANISMO (Enzima A - Ocupada) ---
select inibidor, enzima_A and organic
select serina_chave, byres (resn SER and enzima_A within 4 of inibidor)
select pocket_bound, (byres (enzima_A within 5 of inibidor)) and not inibidor and not serina_chave
show sticks, inibidor or serina_chave
zoom inibidor
show surface, pocket_bound
util.cbaw inibidor
color cyan, serina_chave
color gray80, pocket_bound
set transparency, 0.3, pocket_bound

# --- 3. MOSTRANDO AS INTERAÇÕES ---
dist hbonds, (inibidor and (elem N,O)), (pocket_bound and (elem N,O)), 3.6
hide labels, hbonds
color white, hbonds
set dash_width, 2
dist covalent_bond, (inibidor within 2.0 of (serina_chave and name OG)), (serina_chave and name OG), 2.0
hide labels, covalent_bond
color hotpink, covalent_bond
set dash_width, 4
set dash_gap, 0, covalent_bond
hide cartoon, serina_chave

# --- 4. A COMPARAÇÃO "ANTES" (Enzima B - Vazia) ---
select pocket_apo, (byres (enzima_B within 5 of inibidor))
show surface, pocket_apo
color tv_red, pocket_apo
set transparency, 0.3, pocket_apo

# --- 5. PRONTO PARA ANÁLISE MEP ---
select complexo_ativo, pocket_bound or inibidor
# (O usuário deve agora usar o plugin APBS nas seleções 'pocket_apo' e 'complexo_ativo')

# --- FIM DO SCRIPT ---
