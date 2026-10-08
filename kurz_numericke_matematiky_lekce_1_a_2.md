# Kurz Numerické Matematiky (NUMA)

Tento dokument obsahuje ucelené zápisky z prvních dvou klíčových témat numerické matematiky, přesně podle dodaných materiálů.

---

## Lekce 1: O chybách a nepřesných číslech

**Co je cílem NUMA?**
Numerická matematika se snaží najít přibližné řešení matematických úloh (které často analyticky vyřešit nelze) pomocí základních aritmetických operací (sčítání, odčítání, násobení, dělení), protože to je to jediné, co umí procesor počítače vykonat.

### 1. Zdroje chyb
Do každého takového výpočtu vnášíme chyby, které se dělí do tří skupin:
1.  **Chyby matematického modelu:** Vzniknou zjednodušením reality do rovnic (např. ignorování odporu vzduchu).
2.  **Chyby numerické metody:** Vzniknou, když teoreticky nekonečný proces uřízneme v konečném čase.
3.  **Zaokrouhlovací chyby:** Vznikají v samotném počítači. Kvůli omezené paměti počítač nedokáže pracovat s nekonečným množstvím reálných čísel (neumí si přesně zapamatovat např. $\pi$ nebo $1/3$).

### 2. Měření chyb
Pokud $x$ je naprosto přesná (ale často neznámá) hodnota a $\tilde{x}$ je to, co máme uloženo v počítači, pak definujeme:

*   **Odhad absolutní chyby $\delta(\tilde{x})$:**
    Je to nejmenší možné číslo udávající maximální možnou odchylku.
    Platí: $|x - \tilde{x}| \le \delta(\tilde{x})$
    Zápis: $x = \tilde{x} \pm \delta(\tilde{x})$

*   **Odhad relativní chyby $\rho(\tilde{x})$:**
    Udává chybu v poměru k samotné velikosti čísla (obvykle v procentech).
    Vzorec: $\rho(\tilde{x}) \approx \frac{\delta(\tilde{x})}{|\tilde{x}|}$

### 3. Šíření chyb při výpočtech
Když počítač provádí operace s nepřesnými čísly, chyby se sčítají a rostou.

**a) Sčítání a odčítání**
Základní vzorec pro odhad absolutní chyby součtu a rozdílu zní:
$$\delta(\tilde{x} \pm \tilde{y}) = \delta(\tilde{x}) + \delta(\tilde{y})$$
**Kritické pravidlo NUMA:** Všimni si, že i když čísla odčítáme, jejich absolutní chyby se *sčítají*. To znamená, že **odčítání dvou přibližně stejně velkých čísel je pro počítač smrtící**, protože výsledek se blíží nule, ale chyba zůstává obrovská (relativní chyba letí nahoru). Algoritmus se takové operaci musí vždy vyhnout!

**b) Násobení a dělení**
Při násobení a dělení se naopak (přibližně) sčítají jejich relativní chyby:
$$\rho(\tilde{x} \cdot \tilde{y}) \approx \rho(\tilde{x}) + \rho(\tilde{y})$$
$$\rho\left(\frac{\tilde{x}}{\tilde{y}}\right) \approx \rho(\tilde{x}) + \rho(\tilde{y})$$

**c) Odhad chyby pro obecnou funkci**
Pokud dosadíme nepřesná čísla do libovolné funkce, chyba výsledku se odhadne pomocí parciálních derivací:
$$\delta(f(\tilde{x})) \approx \sum_{i=1}^n \left| \frac{\partial f}{\partial x_i}(\tilde{x}) \right| \delta(\tilde{x}_i)$$

### 4. Vzorový příklad na zkoušku (Katastrofa při odčítání)
**Zadání:** Mějme dvě zaokrouhlená čísla $x = 14,20 \pm 0,08$ a $y = 12,40 \pm 0,06$. Spočítejte výsledek jejich rozdílu $x - y$ a odhadněte výslednou relativní chybu v procentech.

**Postup:**
1.  Samotný rozdíl: $14,20 - 12,40 = 1,80$.
2.  Odhad absolutní chyby: Chyby se při odčítání sčítají, takže $\delta(x - y) = 0,08 + 0,06 = 0,14$.
3.  Výsledek je tedy **$1,80 \pm 0,14$**.
4.  Odhad relativní chyby: $\rho = \frac{\text{chyba}}{\text{výsledek}} = \frac{0,14}{1,80} \approx 0,0777$.
5.  **Závěr:** Relativní chyba je po převodu na procenta zhruba **8 %**. (Vidíme, jak pouhé odečtení velmi blízkých čísel zničilo přesnost).

---

## Lekce 2: Reprezentace čísel v počítači

Abychom pochopili, proč vznikají zaokrouhlovací chyby z první lekce, musíme se podívat pod kapotu počítače.

### 1. Systém s pohyblivou řádovou čárkou
Počítač nezná všechna reálná čísla, ale jen konečnou množinu tzv. čísel s pohyblivou řádovou čárkou (floating-point). Značíme ji $\mathbb{F}(\beta, t, L, U)$.
Každé číslo je zde uloženo jako:
$$x = \pm (d_1 \beta^{-1} + d_2 \beta^{-2} + \dots + d_t \beta^{-t}) \cdot \beta^e$$

**Co znamenají ty písmenka?**
*   $\beta$: Základ soustavy (pro počítač většinou binární, tedy 2).
*   $t$: Délka mantisy. Určuje "přesnost" počítače (např. kolik má číslo desetinných míst).
*   $d_1 \dots d_t$: Jsou samotné cifry mantisy.
*   $e$: Exponent, který musí být v nějakém rozmezí $L \le e \le U$.

### 2. Vlastnosti této množiny
Tento systém uložení čísel má tři zásadní omezení:
1.  **Čísel je jen konečně mnoho.** Počet všech možných čísel v počítači je:
    $2 \cdot (\beta - 1) \cdot \beta^{t-1} \cdot (U - L + 1) + 1$ (To $+1$ na konci je uložení nuly).
2.  **Mají svoje limity.**
    Největší kladné číslo, které jde uložit (OFL - Overflow), je $\beta^U(1-\beta^{-t})$.
    Nejmenší myslitelné kladné číslo kousek nad nulou (UFL - Underflow) je $\beta^{L-1}$.
3.  **Jsou nerovnoměrně rozložena.** Čím dále od nuly po ose jdeme, tím větší "mezery" mezi sebou v počítači čísla mají.

### 3. Strojové epsilon ($\varepsilon_M$)
Toto je extrémně důležitý pojem! Představ si, že chceš v počítači uložit číslo `1`. Které je **úplně to první následující** číslo, které je počítač vůbec schopen od jedničky rozeznat?
Rozdíl (vzdálenost) mezi číslem `1` a prvním nejbližším větším číslem se nazývá strojové epsilon.

*   Vzorec pro výpočet: $\varepsilon_M = \beta^{1-t}$
*   Při standardní "double" přesnosti v MATLABu (64 bitů) je toto číslo extrémně malé: $\approx 2,22 \cdot 10^{-16}$.
*   Jakékoliv číslo, které by spadlo "do mezery" mezi přesně uložená čísla, musí počítač zaokrouhlit. Maximální relativní chyba při takovém zaokrouhlení je $\frac{1}{2}\varepsilon_M$.

### 4. Kritické selhání: Počítačová aritmetika
Protože se čísla neustále zaokrouhlují, aby se vešla do mantisy, neplatí v počítači pravidla středoškolské algebry.

**Příklad zrádnosti (Neplatnost asociativního zákona):**
Pokud máme tři čísla $a, b, c$ a počítač je sčítá pomocí svých vlastních limitovaných operací (značíme je znakem $\oplus$ místo obyčejného plus), může se stát, že:
$(a \oplus b) \oplus c \neq a \oplus (b \oplus c)$

Například, když k obrovskému číslu přičteš nepatrné číslo (menší než $\varepsilon_M$), to nepatrné číslo při zaokrouhlování na mantisu prostě "zmizí", jako bys nepřičetl nic. Omezuje to stabilitu algoritmů.