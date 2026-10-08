# Kurz Numerické Matematiky (NUMA) - Část 3

Tento dokument obsahuje ucelené zápisky k nejnovějšímu tématu: **Iterační metody řešení soustav lineárních rovnic (SLAR)** podle dodané české prezentace.

---

## Lekce 5: Iterační metody (Jacobi, Gauss-Seidel, SOR)

### 1. Od intuice k formálnosti: Proč nepočítat přesně?
**Lidská myšlenka:** Přímé metody (jako Gaussova eliminace) fungují skvěle pro malé matice. Představ si ale, že počítáš teplotu uvnitř 3D modelu motoru auta. Vznikne ti matice o velikosti např. $1 000 000 \times 1 000 000$. Přímý výpočet by trval roky a kvůli zaokrouhlovacím chybám by stejně nevyšel. 
Iterační metody na to jdou jinak:
1. "Tipneme" si jakýkoliv počáteční výsledek (např. samé nuly) - značíme $\vec{x}^{(0)}$.
2. Vložíme ho do speciálního vzorce, který nám vyplivne trošku přesnější "tip" $\vec{x}^{(1)}$.
3. Tento proces opakujeme (iterujeme), dokud se výsledek nepřestane měnit.

**Formální matematika:**
Základní soustavu $A\vec{x} = \vec{b}$ rozložíme na tvar $A = M - N$, kde matice $M$ je regulární. 
Z toho odvodíme vztah: $M\vec{x} = N\vec{x} + \vec{b}$, a po osamostatnění $\vec{x}$ získáme základní rovnici všech iteračních metod:

$$ \vec{x} = B\vec{x} + \vec{c} $$

*(Kde pomocná matice $B = M^{-1}N$ a pomocný vektor $\vec{c} = M^{-1}\vec{b}$.)*

Z toho vzniká iterační proces, kde nový krok $k+1$ počítáme ze starého kroku $k$:
$$ \vec{x}^{(k+1)} = B\vec{x}^{(k)} + \vec{c} $$

### 2. Konvergence: Kdy to bude fungovat?
Tento proces bohužel nefunguje vždy. Někdy se naše "tipy" místo přibližování ke správnému výsledku začnou zvětšovat do nekonečna (divergence).
**Věta o konvergenci:** Posloupnost iterací konverguje ke správnému řešení jedině tehdy, pokud nějaká norma matice $B$ (např. řádková, viz lekce 3) je **ostře menší než 1**.
$$ ||B|| < 1 $$

### 3. Konkrétní metody podle skript

Abychom odvodili konkrétní metody, rozdělíme si původní matici $A$ na tři části: $A = L + D + U$.
*   $L$ (Lower): Vše pod hlavní diagonálou.
*   $D$ (Diagonal): Jen hlavní diagonála.
*   $U$ (Upper): Vše nad hlavní diagonálou.

#### A) Jacobiho metoda
Jedna z nejstarších a nejjednodušších metod.
*   **Vzorec (Matice):** $\vec{x}^{(k+1)} = -D^{-1}(L+U)\vec{x}^{(k)} + D^{-1}\vec{b}$
*   **Vzorec (Po prvcích v kódu):** Abychom spočítali novou proměnnou $x_i$, vezmeme příslušný prvek z pravé strany $b_i$, odečteme všechny ostatní proměnné (použijeme staré hodnoty z kroku $k$) a vydělíme prvkem na diagonále $a_{ii}$.
*   **Kdy konverguje:** Když je matice $A$ tzv. **ostře řádkově diagonálně dominantní**. To znamená, že číslo na hlavní diagonále v daném řádku musí být (v absolutní hodnotě) větší, než je součet všech ostatních čísel v tom samém řádku!

#### B) Gauss-Seidelova metoda
Vylepšení Jacobiho metody. Počítač hodnoty počítá popořadě ($x_1$, pak $x_2$, atd.). Zatímco Jacobi bere pro výpočet VŽDY staré hodnoty z kroku $k$, Gauss-Seidel si uvědomí: *"Hej, $x_1$ už jsem přece před vteřinou spočítal v novém kroku $k+1$, tak ho hned použiju pro výpočet $x_2$!"*
Díky tomu se k výsledku dostaneme rychleji (méně iterací).
*   **Vzorec (Matice):** $\vec{x}^{(k+1)} = -(L+D)^{-1}U\vec{x}^{(k)} + (L+D)^{-1}\vec{b}$
*   **Kdy konverguje:** Má stejné podmínky jako Jacobi (diagonální dominance), ale *navíc* konverguje i tehdy, když je matice $A$ **pozitivně definitní**!

#### C) Relaxační metody (SOR - Successive Overrelaxation)
Gauss-Seidelova metoda se dá ještě urychlit tak, že "uhodneme" směr, kterým se výsledek pohybuje, a skočíme tam rychleji pomocí relaxačního parametru $\omega$ (omega).
*   Nový výpočet se poskládá jako lineární kombinace předchozího kroku a nového Gauss-Seidelova kroku:
    $$ x_i^{(k+1)} = (1 - \omega)x_i^{(k)} + \omega \cdot (\text{výsledek z Gauss-Seidela}) $$
*   **Pravidlo stability:** Metoda SOR může fungovat pouze tehdy, když zvolíme parametr $\omega$ v otevřeném intervalu **$(0, 2)$**.