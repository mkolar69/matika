# Uceleny Kurz Numericke Matematiky - Cast 1

Tento material obsahuje kompletni teorii a priklady k uvodnim lekcim numericke matematiky. Probirame zdroje chyb, jejich sireni a algoritmicka omezeni zpusobena tim, jak pocitace ukladaji realna cisla. 

---

## Lekce 1: Uvod do chyb a jejich sireni

### 1. Od intuice k formalnosti: Proc vubec resime chyby?
**Lidska myslenka:** V klasicke matematice pracujeme s presnymi hodnotami (napr. pi, odmocnina ze 2). Realny pocitac ma ale jen omezenou pamet a zvlada pouze konecny pocet scitani, odcitani, nasobeni, deleni a porovnavani. Pokud chceme na pocitaci resit slozity problem (napr. integrovat nebo resit obri soustavu rovnic), musime jej pretvorit na konecny algoritmus, ktery se sklada prave z techto zakladnich operaci. Do tohoto procesu se nevyhnutelne vnaseji **chyby**.

**Formalni rozdeleni:**
Podle skript se pri numerickem reseni problemu setkavame se tremi zakladnimi druhy chyb:
1. **Chyby matematickeho modelu:** Vzniknou zjednodusenim reality do rovnic (napr. inzenyr zanedba odpor vzduchu).
2. **Chyby numericke metody:** Vzniknou, kdyz teoreticky nekonecny matematicky proces (napr. vypocet limity nebo integralu) nahradime konecnym poctem kroku.
3. **Zaokrouhlovaci chyby:** Vznikaji primo uvnitr pocitace pri reprezentaci cisel na konecny pocet desetinnych mist a kumuluji se behem kazde pocetni operace.

**Mereni chyb (Matematicky zapis):**
Mejme presnou (ale casto neznamou) hodnotu `x` a jeji aproximaci `x_approx` (to, co je ulozeno v pocitaci). Zavedeme dve miry:
* **Odhad absolutni chyby delta(x_approx):** Cislo udavajici maximalni moznou odchylku, aby platilo `|x_approx - x| <= delta(x_approx)`. Zapisujeme jako `x = x_approx +- delta(x_approx)`.
* **Odhad relativni chyby rho(x_approx):** Udava velikost chyby v pomeru k velikosti samotneho cisla (casto uvadena v procentech). Pocita se jako `rho(x_approx) = delta(x_approx) / |x_approx|`.

### 2. Sireni chyb v aritmetickych operacich
Zajima nas, jak se chyba vstupnich dat promitne do vysledku nasi metody.

**a) Scitani a odcitani**
Podle skript plati vzorec: `delta(x_approx +- y_approx) = delta(x_approx) + delta(y_approx)`
* **Kriticka analyza algoritmu (Ztrata platnych cislic):** Z tohoto vzorce vyplyva nejvetsi hrozba numericke matematiky. Vsimnete si, ze i kdyz cisla odcitame, jejich absolutni chyby se scitaji. Zlate pravidlo pro navrh numerickych algoritmu zni: **Algoritmus se musi vyhnout odcitani dvou priblizne stejne velkych cisel.** Pokud to udela, vysledek rozdilu bude cislo blizke nule, ale jeho absolutni chyba zustane obrovska. Nasledny vypocet relativni chyby (chyba delena vysledkem blizicim se nule) "vystreli" nepresnost do obrovskych procent a znici vysledek.

**b) Nasobeni a deleni**
Pri techto operacich se absolutni chyby chovaji privetiveji, zato se jednoduse (priblizne) scitaji jejich relativni chyby:
* `rho(x_approx * y_approx) = rho(x_approx) + rho(y_approx)`
* `rho(x_approx / y_approx) = rho(x_approx) + rho(y_approx)`

### 3. Vypocitany priklad s postupem: Katastrofa pri odcitani
Tento ucebnicovy priklad ukazuje logiku predchozi kriticke analyzy v praxi.
* **Zadani:** Mejme dve cisla s odhadem absolutni chyby: `x = 14,20 +- 0,08` a `y = 12,40 +- 0,06`. Spocitejte vysledek jejich rozdilu `x - y` a odhadnete vyslednou relativni chybu rho.
* **Postup reseni:**
    1. Spocitame samotny rozdil: `14,20 - 12,40 = 1,80`.
    2. Odhadneme absolutni chybu. Podle vzorce se chyby pri odcitani scitaji: `delta(x - y) = 0,08 + 0,06 = 0,14`.
    3. Vysledek rozdilu i s chybou je tedy: **1,80 +- 0,14**.
    4. Vypocitame relativni chybu: `rho = chyba / vysledek = 0,14 / 1,80 = 0,0777`.
* **Zaver:** Z pomerne presnych cisel jsme pouhym odectenim vyrobili vysledek, ktery ma relativni chybu **temer 8 %**.

---

## Lekce 2: Reprezentace cisel v pocitaci

Abychom pochopili zaokrouhlovaci chyby z prvni lekce, musime analyzovat zpusob, jakym pocitac uklada cisla.

### 1. System s pohyblivou radovou carkou (Floating-point)
Pocitac nezna vsechna realna cisla, ale operuje s konecnou mnozinou cisel, kterou znacime `F(beta, t, L, U)`.
Kazde takove cislo ma tvar:
`x = +-(d1*beta^-1 + d2*beta^-2 + ... + dt*beta^-t) * beta^e`

**Vysvetleni parametru:**
* **beta:** Zaklad soustavy (pro pocitac temer vzdy 2 - binarni soustava).
* **t:** Delka mantisy. Udava pocet cislic a definuje "presnost" systemu.
* **di:** Samotne cislice mantisy. Plati pro ne, ze `d1 != 0` (takove cislo pak nazyvame "normalizovane").
* **e:** Exponent, omezeny zdola i shora limity `L <= e <= U`.

### 2. Vlastnosti mnoziny F (Kriticka omezeni pro algoritmizaci)
Tato mnozina ma tri fatalni vlastnosti pro klasickou matematiku:
1. **Je konecna.** Zdaleka nepokryva vsechna cisla. Lze dokazat (viz domaci ukol 1), ze obsahuje presne `2 * (beta - 1) * beta^(t-1) * (U - L + 1) + 1` prvku.
2. **Je ohranicena.** NejMensi reprezentovatelne kladne cislo kousek nad nulou (UFL - Underflow) je `beta^(L-1)`. Nejvetsi predstavitelne cislo (OFL - Overflow) je `beta^U * (1 - beta^-t)`.
3. **Je nerovnomerna.** Mezery mezi jednotlivymi reprezentovatelnymi cisly se smerem od nuly k nekonecnu stale zvetsuji!

### 3. Strojove epsilon (epsilon_M)
Jedna se o naprosto klicovy parametr daneho pocitace. Strojove epsilon definujeme jako **rozdil (vzdalenost) mezi cislem 1 a prvnim nejblizsim vetsim cislem, ktere je pocitac schopen rozeznat.**
* Vzorec pro vypocet: `epsilon_M = beta^(1-t)`.
* Vzdalenost prvniho rozeznatelneho cisla od jednicky tedy neni nekonecne mala, ale rovna se prave epsilon_M.
* Dusledek pro zaokrouhlovani: Maximalni relativni chyba, ktera vznikne zaokrouhlenim jakehokoliv cisla do pameti pocitace (tzv. zariznutim mantisy), je omezena hodnotou `epsilon_M / 2`.

### 4. Analyza selhani: Kdy neplati klasicka stredoskolska algebra?
V pocitacove aritmetice (kdy operace budeme znacit napr. symbolem [+]) dochazi kvuli neustalemu zaokrouhlovani mantisy k tomu, ze **neplati asociativni zakon pro scitani:**
`(a [+] b) [+] c != a [+] (b [+] c)`

**Ukazkovy priklad selhani (Pocitac vs Analytika):**
Predstavme si, ze v pocitaci bezi obycejna Gaussova eliminace a pro vypocet multiplikatoru ma provest operaci `m = -89 / 47`. 
Pocitac, ktery ma presnost (delku mantisy) nastavenou napr. na `t=3` cifry, cislo `m` (ktere je analyticky rovno -1,893617...) orizne a ulozi si hodnotu -1,89. Pri naslednem nasobeni radku timto "oriznutym" cislem narusta chyba natolik, ze vysledne reseni `x`, `y` soustavy vyjde zcela spatne (viz slide "Presny vypocet" a "Vypocet v systemu s plovouci radovou carkou").

Timto se otevira nutnost uprav algoritmu tak, aby chybam v ulozeni cisel predchazely (napr. vyberem pivota, jak bude ukazano v Lekci 4).
```eof

```markdown:Lekce_3_4.md
# Uceleny Kurz Numericke Matematiky - Cast 2

Tento material navazuje na teorii chyb. Nyni se ucime jak tyto chyby hromadne merit v maticich (soustavach o mnoha promennych) a nasledne rozebirame prvni algoritmicke postupy, jak takove soustavy resit primymi metodami.

---

## Lekce 3: Mereni chyb vektoru a matic pomoci norem

### 1. Od intuice k formalnosti: Co je to norma a proc ji potrebujeme?
**Lidska myslenka:** Z prvni lekce vime, ze velikost chyby jednoho cisla zmerime snadno jeho absolutni hodnotou `|x|`. V praxi ale pocitame s obrovskymi vektory a maticemi. Potrebujeme nejakou specialni matematickou funkci, ktera vezme cely vektor nebo celou matici a "slisuje" ji do jedineho nezaporneho cisla, ktere bude reprezentovat jeji "velikost" nebo "delku". Pomoci tohoto cisla budeme moci odhadnout, jak moc se nam zblaznila cela soustava rovnic, kdyz se na vstupu projevila zaokrouhlovaci chyba.

**Formalni definice (Vektorove normy):**
Norma vektoru `x` v prostoru `R^n`, znacena `||x||`, musi splnovat tato pravidla:
1. `||x|| >= 0` a je nulova jedine tehdy, kdyz `x` je nulovy vektor.
2. Trojuhelnikova nerovnost: `||x + y|| <= ||x|| + ||y||`.
3. `||alpha * x|| = |alpha| * ||x||`.

**Vypocet tri nejpouzivanejsich vektorovych norem (podle p-normy):**
* **Manhattan norma (p = 1):** `||x||_1 = suma |x_i|` (Prosty soucet absolutnich hodnot).
* **Eukleidovska norma (p = 2):** `||x||_2 = odmocnina(suma (x_i^2))` (Klasicka "Pythagorova veta").
* **Cebysevova norma (p -> nekonecno):** `||x||_nekonecno = max |x_i|` (Vybere absolutne nejvetsi prvek).

**Formalni definice (Maticove normy):**
Matici bereme jako soustavu, takze pocitame tzv. indukovanou normu `||A|| = max (||A*x|| / ||x||)` pro `x != 0`. Prakticke vzorce ze skript:
* **Sloupcova norma matice (p = 1):** `||A||_1` Spocitame soucty absolutnich hodnot v jednotlivych sloupcich a vybereme ten nejvetsi.
* **Radkova norma matice (p -> nekonecno):** `||A||_nekonecno` Spocitame soucty absolutnich hodnot v jednotlivych radcich a vybereme ten nejvetsi.

### 2. Kriticka analyza a Cislo podminenosti ulohy cond(A)
Normy matic nevyuzivame k nicemu mensimu nez k analyze stability algoritmu. 
Vime, ze resime soustavu `A*x = b`. Zajima nas: co se stane s vysledkem `x`, pokud je vstupni prava strana `b` zatizena malou chybickou `delta_b`?

Pro mereni teto citlivosti zavadime **cislo podminenosti matice**:
`cond(A) = ||A|| * ||A^-1||`

* **Pravidlo stability:** Ze skript vyplyva zasadni nerovnost:
`||delta_x|| / ||x|| <= cond(A) * (||delta_b|| / ||b||)`
* Znamena to, ze relativni chyba vysledku je omezena (shora) relativni chybou vstupu vynasobenou cislem podminenosti.
* **Dusledek:** Pokud nam po vypoctu normy vyjde cislo `cond(A)` obrovske (napr. 100 000), uloha je tzv. **spatne podminena**. Znamena to, ze i zanedbatelne mala chybicka zaokrouhleni na prave strane muze zpusobit destruktivni, stotisickrat vetsi chybu ve vyslednem reseni.

---

## Lekce 4: Prime metody reseni soustav (Gauss a LU rozklad)

### 1. Od intuice k formalnosti: Gaussova eliminace
**Lidska myslenka:** Mame obrovskou soustavu rovnic `A*x = b`. Cilem primych metod je soustavu chytre upravovat scitanim a odcitanim radku tak dlouho, dokud se z leve dolni casti matice (pod hlavni diagonalou) nestanou same nuly (ziskavame Horni trojuhelnikovou matici `U`). Z te pak uz snadno vycteme reseni odzadu (tzv. zpetna substituce).

**Algoritmus Gaussovy eliminace bez uprav:**
Chceme v k-tem sloupci vynulovat vsechny radky pod k-tym radkem.
1. Oznacime si prvek na diagonale `a_kk` (tzv. **pivot**).
2. Spocitame pro spodni radky **multiplikator** `m_ik = a_ik / a_kk`.
3. Od i-teho radku odecteme m-nasobek k-teho radku.
4. Nasledne po uprave do matice `U` provedeme Zpetnou substituci vzorcem pro kazde `x_i` odzadu.

### 2. Kriticka chyba: Proc cisty Gauss na pocitaci selhava?
Na pocitaci s pohyblivou radovou carkou (kvuli pravidlum z Lekce 1 a 2) narazi tento postup na dva fatalni problemy:
1. **Deleni nulou:** Pokud je pivot `a_kk` nahodou 0, vzorec `m_ik = a_ik / 0` program shodi.
2. **Katastroficke ruseni (Ztrata platnych cislic):** Predstavte si, ze pivot neni nula, ale velmi male cislo (napr. `10^-15`). Vynikne zrudne velky multiplikator (napr. `m = 10^15`). Pri odcitani m-nasobku radku zacne pocitac odcitat obrovska cisla. Kvuli omezene delce mantisy "odrizne" mala cisla. Vlivem ztraty platnych cislic z minule lekce vyjde naprosty nesmysl.

**Lek: Castecny vyber pivota (Partial Pivoting):** 
Pred kazdym delenim v k-tem sloupci najdeme radek s nejvetsim cislem v absolutni hodnote. Cely tento radek prohodime s aktualnim. Budeme tak delit co nejvetsim cislem a multiplikator `|m|` nikdy nepresahne 1.

### 3. MATLAB Kody

**Zpetna substituce (Pro vypocet x z upravene matice U):**
```matlab
% Predpokladame, ze U je horni trojuhelnikova matice (N x N) a y je prava strana (N x 1)
x = zeros(N, 1);
x(N) = y(N) / U(N, N); % Vyreseni posledni promenne
for i = N-1:-1:1
    suma = 0;
    for j = i+1:N
        suma = suma + U(i, j) * x(j);
    end
    x(i) = (y(i) - suma) / U(i, i);
end
for k = 1:N-1
    % Nalezeni nejvetsiho pivota
    [max_val, r_index] = max(abs(A(k:N, k))); 
    r = k + r_index - 1; 
    
    % Prohozeni radku
    if r > k
        temp = A(k, :); A(k, :) = A(r, :); A(r, :) = temp;
        temp_b = b(k); b(k) = b(r); b(r) = temp_b;
    end
    
    % Eliminace (ziskavame U z matice A)
    for i = k+1:N
        m = A(i, k) / A(k, k);
        A(i, k:N) = A(i, k:N) - m * A(k, k:N);
        b(i) = b(i) - m * b(k);
    end
end
% ... Nasleduje kod Zpetne substituce z bloku vyse
```markdown:Lekce_5.md
# Kurz Numericke Matematiky (NUMA) - Cast 3

Tento dokument obsahuje ucelene zapisky k nejnovejsimu tematu: **Iteracni metody reseni soustav linearnich rovnic (SLAR)**.

## Lekce 5: Iteracni metody (Jacobi, Gauss-Seidel, SOR)

### 1. Od intuice k formalnosti: Proc nepocitat presne?

**Lidska myslenka:** Prime metody (jako Gaussova eliminace) funguji skvele pro male matice. Predstav si ale, ze pocitas teplotu uvnitr 3D modelu motoru auta. Vznikne ti matice o velikosti napr. 1 000 000 x 1 000 000. Primy vypocet by trval roky a kvuli zaokrouhlovacim chybam by stejne nevysel.
Iteracni metody na to jdou jinak:

1. "Tipneme" si jakykoliv pocatecni vysledek (napr. same nuly) - znacime `x_krok_0`.
2. Vlozime ho do specialniho vzorce, ktery nam vyplivne trosku presnejsi "tip" `x_krok_1`.
3. Tento proces opakujeme (iterujeme), dokud se vysledek neprestane menit.

**Formalni matematika:**
Zakladni soustavu `A*x = b` rozlozime na tvar `A = M - N`, kde matice `M` je regularni.
Z toho odvodime vztah: `M*x = N*x + b`, a po osamostatneni `x` ziskame zakladni rovnici vsech iteracnich metod:

`x = B*x + c`

*(Kde pomocna matice `B = M^-1 * N` a pomocny vektor `c = M^-1 * b`.)*

Z toho vznika iteracni proces, kde novy krok (k+1) pocitame ze stareho kroku (k):

`x_krok_novy = B * x_krok_stary + c`

### 2. Konvergence: Kdy to bude fungovat?

Tento proces bohuzel nefunguje vzdy. Nekdy se nase "tipy" misto priblizovani ke spravnemu vysledku zacnou zvetsovat do nekonecna (divergence).
**Veta o konvergenci:** Posloupnost iteraci konverguje ke spravnemu reseni jedine tehdy, pokud nejaka norma matice `B` (napr. radkova, viz lekce 3) je **ostre mensi nez 1**.

`||B|| < 1`

### 3. Konkretni metody podle skript

Abychom odvodili konkretni metody, rozdelime si puvodni matici A na tri casti: `A = L + D + U`.
* **L (Lower):** Vse pod hlavni diagonalou.
* **D (Diagonal):** Jen hlavni diagonala.
* **U (Upper):** Vse nad hlavni diagonalou.

#### A) Jacobiho metoda

Jedna z nejstarsich a nejjednodussich metod.

* **Vzorec (Matice):** `x_krok_novy = -D^-1 * (L+U) * x_krok_stary + D^-1 * b`
* **Vzorec (Po prvcich v kodu):** Abychom spocitali novou promennou `x_i`, vezmeme prislusny prvek z prave strany `b_i`, odecteme vsechny ostatni promenne (pouzijeme stare hodnoty z kroku k) a vydelime prvkem na diagonale `a_ii`.
* **Kdy konverguje:** Kdyz je matice A tzv. **ostre radkove diagonalne dominantni**. To znamena, ze cislo na hlavni diagonale v danem radku musi byt (v absolutni hodnote) vetsi, nez je soucet vsech ostatnich cisel v tom samem radku!

#### B) Gauss-Seidelova metoda

Vylepseni Jacobiho metody. Pocitac hodnoty pocita poporade (`x_1`, pak `x_2`, atd.). Zatimco Jacobi bere pro vypocet VZDY stare hodnoty z kroku k, Gauss-Seidel si uvedomi: *"Hej, `x_1` uz jsem prece pred vterinou spocital v novem kroku k+1, tak ho hned pouziju pro vypocet `x_2`!"*
Diky tomu se k vysledku dostaneme rychleji (mene iteraci).

* **Vzorec (Matice):** `x_krok_novy = -(L+D)^-1 * U * x_krok_stary + (L+D)^-1 * b`
* **Kdy konverguje:** Ma stejne podminky jako Jacobi (diagonalni dominance), ale *navic* konverguje i tehdy, kdyz je matice A **pozitivne definitni**!

#### C) Relaxacni metody (SOR - Successive Overrelaxation)

Gauss-Seidelova metoda se da jeste urychlit tak, ze "uhodneme" smer, kterym se vysledek pohybuje, a skocime tam rychleji pomoci relaxacniho parametru `omega`.

* Novy vypocet se posklada jako linearni kombinace predchoziho kroku a noveho Gauss-Seidelova kroku:
`x_i_novy = (1 - omega) * x_i_stary + omega * (vysledek z Gauss-Seidela)`
* **Pravidlo stability:** Metoda SOR muze fungovat pouze tehdy, kdyz zvolime parametr omega v otevrenem intervalu (0, 2).
```eof
