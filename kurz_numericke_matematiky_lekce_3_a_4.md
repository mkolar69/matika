# Kurz Numerické Matematiky (NUMA) - Část 2

Tento dokument obsahuje ucelené zápisky z třetího a čtvrtého klíčového tématu, které se zaměřuje na matice, měření jejich chyb a algoritmické řešení soustav lineárních rovnic.

---

## Lekce 3: Normy vektorů a matic (Jak měřit velikost chyby)

### 1. Od intuice k formálnosti: Proč potřebujeme normy?
**Lidská myšlenka:** Když počítáme s jedním číslem, velikost jeho chyby změříme snadno pomocí absolutní hodnoty $|x|$. Co když ale pracujeme se soustavou rovnic, kde máme vektor chyb (např. o 100 hodnotách) nebo celou chybovou matici? Potřebujeme nějakou funkci, která vezme celý tento vektor nebo matici a "vyplivne" jediné nezáporné číslo, které bude reprezentovat jejich celkovou "velikost" nebo "délku". Tomuto číslu se říká **norma**.

**Formální definice (Vektory):**
Norma vektoru $\vec{x} \in \mathbb{R}^n$, značená jako $||\vec{x}||$, musí splňovat tři základní pravidla:
1. Jde o nezáporné číslo a $||\vec{x}|| = 0$ jedině tehdy, když je to nulový vektor.
2. Trojúhelníková nerovnost: $||\vec{x} + \vec{y}|| \le ||\vec{x}|| + ||\vec{y}||$
3. Násobení konstantou: $||\alpha\vec{x}|| = |\alpha| \cdot ||\vec{x}||$

Ve skriptech se definují tři nejpoužívanější vektorové $p$-normy:
*   **Manhattan norma ($p = 1$):** $||\vec{x}||_1 = \sum_{i=1}^n |x_i|$ (Prostý součet absolutních hodnot).
*   **Eukleidovská norma ($p = 2$):** $||\vec{x}||_2 = \sqrt{\sum_{i=1}^n x_i^2}$ (Klasická vzdálenost z geometrie, Pythagorova věta).
*   **Čebyševova norma ($p \to \infty$):** $||\vec{x}||_\infty = \max_i |x_i|$ (Vybere ten největší prvek z vektoru v absolutní hodnotě).

**Formální definice (Matice):**
Pro matice $A$ zavádíme tzv. indukovanou normu vztahem $||A|| = \max_{\vec{x} \neq \vec{o}} \frac{||A\vec{x}||}{||\vec{x}||}$. Prakticky počítáme dvě základní:
*   **Sloupcová norma ($p = 1$):** $||A||_1 = \max_j \sum_{i=1}^n |a_{ij}|$ (Uděláš součty absolutních hodnot v jednotlivých sloupcích a vybereš ten největší).
*   **Řádková norma ($p \to \infty$):** $||A||_\infty = \max_i \sum_j |a_{ij}|$ (Uděláš součty absolutních hodnot v jednotlivých řádcích a vybereš ten největší).

### 2. Vypočítané příklady s postupem

**Příklad 1: Normy vektoru**
Zadání: Vypočtěte 1, 2 a $\infty$ normu pro vektor $\vec{x} = (4, -5, 3)^T$.
*   $||\vec{x}||_1 = |4| + |-5| + |3| = 4 + 5 + 3 = \mathbf{12}$
*   $||\vec{x}||_2 = \sqrt{4^2 + (-5)^2 + 3^2} = \sqrt{16 + 25 + 9} = \sqrt{50} = \mathbf{5\sqrt{2}}$
*   $||\vec{x}||_\infty = \max(|4|, |-5|, |3|) = \max(4, 5, 3) = \mathbf{5}$

**Příklad 2: Řádková norma matice**
Zadání: Mějme matici $A = \begin{pmatrix} 3 & 3 \\ 4 & 5 \end{pmatrix}$. Spočtěte $||A||_\infty$.
*   Součet prvního řádku (v absolutních hodnotách): $|3| + |3| = 6$
*   Součet druhého řádku: $|4| + |5| = 9$
*   $||A||_\infty = \max(6, 9) = \mathbf{9}$

### 3. Kritická analýza: Číslo podmíněnosti matice $cond(A)$
K čemu jsou nám normy dobré? Potřebujeme odhadnout, jak moc se pokazí výsledek soustavy $A\vec{x} = \vec{b}$, když do vektoru $\vec{b}$ na vstupu zaneseme malou chybu (např. kvůli zaokrouhlení, označme ji $\Delta\vec{b}$).

Míru tohoto rizika udává **číslo podmíněnosti matice $cond(A)$**.
Vzorec: $cond(A) = ||A|| \cdot ||A^{-1}||$

*Omezení algoritmu:* Platí vztah, že relativní chyba výsledku je omezena takto: $\frac{||\Delta\vec{x}||}{||\vec{x}||} \le cond(A) \cdot \frac{||\Delta\vec{b}||}{||\vec{b}||}$. 
Pokud je $cond(A)$ obrovské číslo, systém je **špatně podmíněný**. Znamená to, že i nepatrná chyba $\Delta\vec{b}$ na pravé straně se při výpočtu obrovsky zvětší a výsledek $\vec{x}$ bude naprosto nepoužitelný!

*(Pozn. pro náš Příklad 2 výše, kde $A^{-1} = \frac{1}{3}\begin{pmatrix} 5 & -3 \\ -4 & 3 \end{pmatrix}$, je $||A^{-1}||_\infty = 8/3$. Číslo podmíněnosti je tedy $cond_\infty(A) = 9 \cdot \frac{8}{3} = \mathbf{24}$. To je malé číslo, systém je dobře podmíněný).*

---

## Lekce 4: Řešení soustav (Gaussova eliminace a LU rozklad)

### 1. Od intuice k formálnosti: Gaussova eliminace
**Lidská myšlenka:** Máme soustavu lineárních rovnic $A\vec{x} = \vec{b}$. Chceme pod hlavní diagonálou matice $A$ vytvořit samé nuly (získat tzv. horní trojúhelníkovou matici $U$). Jakmile to máme, rovnice se dají snadno vyřešit "odspodu nahoru", protože v poslední rovnici zbyde už jen jedna neznámá. Tomuto postupu se říká **zpětná substituce (Back substitution)**.

**Algoritmické kroky (Eliminace vpřed - Forward Elimination):**
1.  Vezmeme prvek na diagonále $a_{kk}$ (nazývá se **pivot**).
2.  Pro každý řádek $i$ pod ním spočítáme tzv. **multiplikátor**: $m = \frac{a_{ik}}{a_{kk}}$.
3.  Od celého $i$-tého řádku odečteme $m$-násobek $k$-tého řádku. Tím se prvek pod diagonálou vynuluje.

### 2. Kritická analýza: Proč Gauss bez pivotizace selhává?
Pokud algoritmus naprogramujeme přesně podle kroků výše, narazíme na fatální problém, který souvisí s chybami z 1. lekce.

**Důvod selhání 1:** Dělení nulou. Pokud je pivot $a_{kk} = 0$, algoritmus spadne.
**Důvod selhání 2 (Zaokrouhlovací chyby):** Představme si, že pivot není nula, ale je velmi malý. Tvé skripta uvádějí učebnicový příklad, kde na diagonále leží $\epsilon = 10^{-6}$:

$$ \begin{pmatrix} 10^{-6} & 1 & | & 1 \\ 1 & 1 & | & 0 \end{pmatrix} $$

Přesné analytické řešení je zhruba $x_1 \approx -1, x_2 \approx 1$. 
Jak ale postupuje počítač bez pivotizace?
1. Spočítá multiplikátor: $m = \frac{1}{10^{-6}} = 10^6$.
2. Upraví druhý řádek: od $1$ odečte $10^6 \cdot 1$. Dostane $1 - 10^6 = -999999$.
3. Počítačová mantisa uřízne malá čísla vedle obrovských (dojde k pohlcení, viz 1. lekce).
4. Výsledkem jsou obrovské zaokrouhlovací chyby a algoritmus "vyplyvne" nesmyslný výsledek.

**Řešení (Částečný výběr pivota - Partial Pivoting):** Před každým krokem eliminace najdeme ve sloupci (od diagonály dolů) prvek s největší absolutní hodnotou. Celý tento řádek prohodíme s aktuálním řádkem. Díky tomu budeme vždy dělit co největším možným číslem, multiplikátor $m$ bude malý (vždy $|m| \le 1$) a nedojde k explozi zaokrouhlovacích chyb.

### 3. Vylepšení: LU rozklad (Triangular Factorization)
**Proč to děláme:** Představ si, že v inženýrské praxi řešíš konstrukci mostu. Matice $A$ (tuhost mostu) je pořád stejná, ale vektor $\vec{b}$ (zatížení, vítr, auta) se mění každou hodinu. Znovu a znovu počítat složitou Gaussovu eliminaci pro každé nové $\vec{b}$ trvá hrozně dlouho.

**Myšlenka LU rozkladu:** Matici $A$ si jednou provždy rozložíme na součin dvou matic $A = LU$.
*   $L$ (Lower): Dolní trojúhelníková matice. Má na diagonále jedničky a pod ní jsou uloženy všechny naše **multiplikátory $m$**, které jsme spočítali během Gaussovy eliminace.
*   $U$ (Upper): Horní trojúhelníková matice. Je to ten výsledný tvar, který nám vyšel po Gaussově eliminaci.

**Postup výpočtu pro nová $\vec{b}$:**
Z rovnice $A\vec{x} = \vec{b}$ se stane $LU\vec{x} = \vec{b}$.
1. Nejprve spočítáme pomocný vektor $\vec{y}$ pomocí dopředné substituce (jde to bleskově): $L\vec{y} = \vec{b}$
2. Pak spočítáme výsledek $\vec{x}$ pomocí zpětné substituce (také bleskově): $U\vec{x} = \vec{y}$

### 4. Algoritmické myšlení: MATLAB kódy

Zde jsou oba klíčové algoritmy přesně tak, jak jsou uvedeny ve tvých anglických materiálech (Systems of Linear Algebraic Equations, Algorithm 3.2 a 5.1).

**A) Základní Gaussova eliminace (Bez pivotizace)**
```matlab
function x = simple_gauss(A, b)
    % Input: A je regulární matice (N x N)
    %        b je vektor pravých stran (N x 1)
    
    N = length(b);
    % Vytvoření horní trojúhelníkové matice (U|y) z (A|b)
    U = A; 
    y = b;
    
    % Eliminace po sloupcích
    for k = 1:N-1
        for i = k+1:N
            % Výpočet multiplikátoru
            m = U(i,k)/U(k,k); 
            U(i,k) = 0;
            % Úprava zbytku i-tého řádku
            U(i, k+1:N) = U(i, k+1:N) - m*U(k, k+1:N);
            % Úprava pravé strany
            y(i) = y(i) - m*y(k);
        end
    end
    
    % Zpětná substituce (Back substitution)
    x = zeros(N,1); 
    x(N) = y(N)/U(N,N);
    
    for k = N-1:-1:1
        % x(k) = (y(k) - suma(U(k, j)*x(j))) / U(k,k)
        x(k) = (y(k) - U(k, k+1:N)*x(k+1:N)) / U(k,k);
    end
end
```

**B) LU rozklad (Bez pivotizace)**
Tento kód vezme matici `A` a vrátí rozložené matice `L` a `U`.
```matlab
function [L, U] = LU_fact(A)
    % Inicializace
    [N, ~] = size(A); 
    U = A; 
    L = eye(N); % Jednotková matice s jedničkami na diagonále
    
    % Eliminace k-tého sloupce
    for k = 1:N-1
        % Eliminace i-tého řádku
        for i = k+1:N
            % Výpočet multiplikátoru
            m = U(i,k)/U(k,k);
            U(i, k) = 0;
            
            % Úprava řádku v matici U
            for j = k+1:N
                U(i,j) = -m*U(k,j) + U(i,j);
            end
            
            % Uložení multiplikátoru do matice L (přesně pod diagonálu)
            L(i, k) = m;
        end
    end
end
```