# O Alvo Seletivo: Análise de Design de Fármacos no PyMOL (PDB: 3WJ6)

Este repositório documenta uma análise de química computacional sobre o mecanismo de inibição de antibióticos, focando na seletividade. O objetivo é ir além da simples análise geométrica ("chave-fechadura") e investigar a física da atração eletrostática que guia a ligação do fármaco.

**Modelo de Estudo:** Utilizamos a estrutura de cristal da DD-transpeptidase de *E. coli* (PDB ID: `3WJ6`). Este arquivo é um modelo de estudo ideal, pois contém um dímero que captura dois estados em um único arquivo:
* **Cadeia A (O "Depois"):** A enzima em seu estado "ocupado", ligada covalentemente ao inibidor Ampicilina.
* **Cadeia B (O "Antes"):** A enzima em seu estado "apo", ou vazia, fornecendo uma linha de base perfeita para comparação.

---

## Análise 1: A Comparação Geométrica (O Ajuste Induzido)

O primeiro passo é entender a mudança *estrutural* que ocorre na enzima. Para isso, separamos as duas cadeias em objetos distintos e as sobrepusemos estruturalmente.

O comando `super (enzima_B and name CA), (enzima_A and name CA)` revela um **RMSD de 0.309 Å**, provando que as duas cadeias são quase idênticas estruturalmente e formam um par de comparação "antes e depois" perfeito.

Após isolar o sítio ativo (`pocket_bound` na Cadeia A) e a região correspondente na enzima vazia (`pocket_apo` na Cadeia B), podemos visualizar o "ajuste induzido":

![Insira aqui a Imagem 1: Comparação das superfícies pocket_apo (vermelha) e pocket_bound (cinza)](./img/geometric_fit.png)

A imagem acima mostra como o bolso da enzima (cinza) se "fecha" e se molda ao redor do inibidor (amarelo) em comparação com a forma do bolso vazio (vermelho).

No entanto, a geometria sozinha não explica *por que* o inibidor foi atraído para este local.

## Análise 2: A Física da Atração (O Potencial Eletrostático - MEP)

Aqui, investigamos a "equação" eletrostática. A hipótese é que o bolso vazio possui uma "assinatura de carga" de alta energia que é "satisfeita" ou "neutralizada" pela ligação do inibidor.

### O MEP "Antes": O Convite Eletrostático

Primeiro, calculamos o MEP (usando o plugin APBS) para a superfície do bolso vazio (`pocket_apo`).

![Insira aqui a Imagem 2: O MEP do bolso 'pocket_apo' vazio, mostrando as cores vermelhas e azuis](./img/mep_apo.png)

O resultado é um campo de alto contraste, com fortes regiões de potencial negativo (vermelho, rico em elétrons) e positivo (azul, pobre em elétrons). Este é o "mapa de atração" que o inibidor "vê" ao se aproximar.

### O MEP "Depois": A Neutralização da Carga

Em seguida, calculamos o MEP para o complexo ativo (`complexo_ativo`, que é `pocket_bound + inibidor`).

![Insira aqui a Imagem 3: O MEP do 'complexo_ativo' ocupado, mostrando uma superfície mais neutra](./img/mep_complex.png)

O resultado é uma superfície muito mais "estável" e eletricamente neutra. As fortes manchas vermelhas e azuis que existiam no bolso vazio desapareceram. Isso é a prova visual de que as cargas complementares do inibidor se encaixaram e **neutralizaram** o potencial de alta energia do bolso.

**A ligação é, portanto, impulsionada tanto pela forma quanto, crucialmente, pela complementaridade eletrostática.**

---

## Conclusão: Uma Pergunta Complexa

A análise prova que o bolso está eletricamente "pré-programado" para atrair e orientar o inibidor para a posição exata, permitindo o ataque covalente da Serina 44.

Isso não nos leva a uma simples conclusão, mas a uma pergunta complexa:

> **E se desenhássemos um novo antibiótico no computador que se ligasse ainda melhor, prevendo sua seletividade e afinidade eletrostática antes mesmo de sintetizá-lo?**

---

## Apêndice: Script de Comandos PyMOL para Reprodução

O script abaixo permite que qualquer usuário do PyMOL reproduza esta análise do início ao fim.

*(Nota: Os comandos para rodar o APBS são manuais (via plugin) e estão descritos na narrativa acima.)*

```python
# --- 1. CONFIGURAÇÃO INICIAL ---
fetch 3WJ6
remove solvent
hide all, 3WJ6
create enzima_A, 3WJ6 and chain A
create enzima_B, 3WJ6 and chain B
disable 3WJ6
show cartoon, enzima_A
show cartoon, enzima_B
color palecyan, enzima_A
color slate, enzima_B
super (enzima_B and name CA), (enzima_A and name CA)

# --- 2. CONSTRUINDO O MECANISMO (Enzima A - Ocupada) ---
select inibidor, enzima_A and organic
# O PyMOL seleciona a Ser44 corretamente com o seletor `byres`
select serina_chave, byres (resn SER and enzima_A within 4 of inibidor)
select pocket_bound, (byres (enzima_A within 5 of inibidor)) and not inibidor and not serina_chave
show sticks, inibidor or serina_chave
zoom inibidor
show surface, pocket_bound
# Colore o inibidor por elemento (C=verde, O=vermelho, N=azul)
util.cbaw inibidor
color cyan, serina_chave
color gray80, pocket_bound
set transparency, 0.3, pocket_bound

# --- 3. MOSTRANDO AS INTERAÇÕES ---
# Ligações de Hidrogênio (Velcro)
dist hbonds, (inibidor and (elem N,O)), (pocket_bound and (elem N,O)), 3.6
hide labels, hbonds
color white, hbonds
set dash_width, 2

# Ligação Covalente (Serina -> Inibidor)
dist covalent_bond, (inibidor within 2.0 of (serina_chave and name OG)), (serina_chave and name OG), 2.0
hide labels, covalent_bond
color hotpink, covalent_bond
set dash_width, 4
set dash_gap, 0, covalent_bond

# Conecta a Serina ao "esqueleto" da enzima
hide cartoon, serina_chave

# --- 4. A COMPARAÇÃO "ANTES" (Enzima B - Vazia) ---
select pocket_apo, (byres (enzima_B within 5 of inibidor))
show surface, pocket_apo
color tv_red, pocket_apo
set transparency, 0.3, pocket_apo

# --- 5. PRONTO PARA ANÁLISE MEP ---
# O usuário agora pode rodar o APBS em 'pocket_apo' e 'complexo_ativo'
select complexo_ativo, pocket_bound or inibidor

# --- FIM DO SCRIPT ---
