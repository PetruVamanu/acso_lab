# Laboator 6

# Instructiuni de lucru in virgula mobila
- unitatea de virgulă mobilă
    - prezentă încă de la primele procesoare Intel
    - set complex de instrucțiuni
    - dificil de programat
- extensii
    - MMX
    - SIMD
    - subseturi de instrucțiuni cu operații uzuale

## Extensile SIMD
SIMD - Single Instruction, Multiple Data
- capabile să aplice aceeași operație în paralel pe mai mulți operanzi
- aici ne interesează doar capacitatea de a lucra cu operanzi în virgulă mobilă

### Registri

- 8 regiștri dedicați
- XMM0, XMM1, ..., XMM7
- fiecare registru are 128 biți
    - poate reține simultan 4 valori în simplă precizie
    - există instrucțiuni care lucrează cu cele 4 valori în paralel
    - aici ne interesează instrucțiunile scalare - lucrează cu o singură valoare

### Atribuire

- se referă exclusiv la valori în virgulă mobilă
- sintaxa
```
movss xmm_i,xmm_j
movss xmm_i,adresa
movss adresa,xmm_i
```
- primul operand reprezintă destinația

Ex:

```
float f1,f2;
f1=f2;
_asm {
    movss xmm0,f2
    movss f1,xmm0
}
```

### Instructiuni de conversie

#### Trecerea valorilor întregi în virgulă mobilă
- sintaxa
```
cvtsi2ss xmm_i,adresa
cvtsi2ss xmm_i,registru
```
- registrul care apare în a doua formă a instrucțiunii este din setul de bază, pe 32 biți (EAX, EBX, ...)

#### Trecerea valorilor în virgulă mobilă în întregi
- sintaxa
```
cvtss2si registru,xmm_i
cvtss2si registru,adresa
```
- toate celelalte instrucțiuni lucrează exclusiv cu operanzi în virgulă mobilă

### Adunare
- sintaxa
```
addss xmm_i,xmm_j
addss xmm_i,adresa
```
- adună valorile celor doi operanzi și depune rezultatul în primul operand

### Scadere 
- sintaxa
```
subss xmm_i,xmm_j
subss xmm_i,adresa
```
- scade valoarea celui de-al doilea operand din valoarea primului și depune rezultatul în primul operand

### Inmultire / Impartire
- sintaxa
```
mulss xmm_i,xmm_j
mulss xmm_i,adresa
divss xmm_i,xmm_j
divss xmm_i,adresa
```
- primul operand este și destinația rezultatului

### Extragere radical

- sintaxa
```
sqrtss xmm_i,xmm_j
sqrtss xmm_i,adresa
```
- primul operand este destinația rezultatului
- al doilea operand furnizează valoarea a cărei rădăcină pătrată este calculată

### Instrucțiuni logice
```
andps xmm_i,xmm_j
andps xmm_i,adresa
orps xmm_i,xmm_j
orps xmm_i,adresa
xorps xmm_i,xmm_j
xorps xmm_i,adresa
```

### Comparatii
- sintaxa
```
comiss xmm_i,xmm_j
comiss xmm_i,adresa
```
- instrucțiunea setează indicatorii de condiții

### Valorile indicatorilor de condiții
| Relație între operanzi | Zero | Carry |
| :--- | :---: | :---: |
| operand 1 > operand 2 | 0 | 0 |
| operand 1 < operand 2 | 0 | 1 |
| operand 1 = operand 2 | 1 | 0 |


### Exemplu:
- calculul distanței între două puncte, date prin coordonatele lor
```cpp
float x1,y1; // primul punct
float x2,y2; // al doilea punct
float rez;
rez=sqrt((x2-x1)*(x2-x1)+(y2-y1)*(y2-y1));
```
```asm
_asm {
    movss xmm0,x2
    subss xmm0,x1
    mulss xmm0,xmm0
    movss xmm1, y2
    subss xmm1, y1
    mulss xmm1, xmm1

    addss xmm1,xmm0
    sqrtss xmm0,xmm1
    movss rez,xmm0
}
```


## Unitatea de virgulă mobilă

Deși programarea unității de virgulă mobilă este mai dificilă, are o serie de avantaje
- poate lucra și cu valori în dublă precizie
- instrucțiuni de inițializare cu unele constante
- instrucțiuni care implementează funcții trigonometrice
- funcțiile care returnează valori în virgulă mobilă folosesc vârful stivei pentru returnare


Floating Point Unit - FPU
- inițial - separată de procesor
    - inclusă în procesor începând cu Intel 486
- lucrează intern cu operanzi în precizie extinsă (80 de biți)
- orice operand de tip float sau double este convertit la această reprezentare

### Regiștrii interni
8 regiștri pe 80 de biți
    - denumiri: ST(0), ST(1), ..., ST(7)
- sunt organizați într-o stivă
    - ST(0) este întotdeauna vârful stivei
    - referit și ca ST
- orice operație de scriere în stivă schimbă ordinea (și numele) regiștrilor
    - fostul ST(0) devine ST(1) etc.

- la orice instrucțiune, ST(0) este unul dintre operanzi
    - sau singurul, la operațiile unare
- celălalt operand poate fi, după caz
    - o locație de memorie (valoare întreagă sau în virgulă mobilă)
    - un alt registru ST(i)
    - niciodată o constantă
- introducerea în stivă - prin instrucțiuni de încărcare a unor valori
    - locații de memorie
    - unele constante predefinite
- scoaterea din stivă
    - multe instrucțiuni de procesare au și o variantă care elimină ST(0) din stivă
    - numele acestor instrucțiuni se termină cu "p"

#### Încărcare din memorie
- sintaxa
```
fld adresa
fild adresa
```
- introduce în stivă operandul, care este
    - în primul caz - o variabilă în virgulă mobilă (float, double)
    - în al doilea caz - o variabilă întreagă (short int, int)
- pentru introducerea unei constante predefinite se folosesc:
    - fldz (0)
    - fld1 (1)
    - fldpi (pi)
    - există și alte constante (logaritmice)

Ex:
```
float f;
int i;
_asm {
    fld f
    fild i
    fld dword ptr [ebp+8]
}
```

#### Scriere în memorie
- sintaxa
```
fst adresa
fstp [adresa]
```
- scriere ST(0) într-o variabilă în virgulă mobilă
```
fist adresa
fistp adresa
```
- scriere ST(0) într-o variabilă întreagă


Ex:
```
float f;
int i;
_asm fstp f;
```
- copiază ST(0) în f și îl elimină din stivă
- fostul ST(1) devine ST(0) etc.
```
_asm fist i;
```
- copiază ST(0) în i, păstrându-l pe stivă

#### Interschimbare regiștri

- sintaxa
```
fxch st(i)
```
- i este o valoare constantă între 1 și 7
- schimbă între ele valorile regiștrilor ST(0) și ST(i)

#### Instrucțiuni trigonometrice

```
fsin
fcos
```
- calculează sinusul, respectiv cosinusul valorii din ST(0) și îl depune tot în ST(0)
```
fsincos
```
- depune sinusul valorii din ST(0) în ST(0), iar cosinusul îl adaugă în vârful stivei

```
fptan
```
- calculează tangenta valorii din ST(0) și o depune tot în ST(0)
- apoi adaugă valoarea 1.0 în vârful stivei

```
fpatan
```
- calculează arctangenta valorii ST(1)/ST(0)
- depune rezultatul în ST(1) și elimină vârful stivei


## Concluzii 
- de preferat lucrul cu instrucțiunile SIMD
- se apelează la FPU în cazuri rare
    - cel mai important - returnarea unei valori în virgulă mobilă de către o funcție
- dar valoarea poate fi calculată în prealabil prin instrucțiuni SIMD
- atenție: orice valoare încărcată în stiva FPU trebuie eliminată ulterior


## Exercitii:

1. Sa se scrie in limbaj de asamblare o functie definita astfel:

```
float computeFormula(float a, float b);
```

in cadrul functie se poate utiliza variabila aux pentru a transfera valori intre registrii SIMD si stiva FPU

Functia calculeaza si returneaza valoarea `sin(a + b)` utilizand urmatoarea formula:
```
sin(a + b) = sin(a) * cos(b) + cos(a) * sin(b)
```

```cpp
#include <iostream>
using namespace std;
float computeFormula(float, float)
{
    float aux;
    _asm
    {
    }
}

int main()
{
    float a = 0.7853f, b = 0.7853f; //pentru aceste valori, functia ar trebui sa returneze 1

    cout << "sin(a + b) = " << computeFormula(a, b);
    return 0;
}
```


2. Conversie de temperatură

Scrieți un program care convertește temperatura din grade Celsius în grade Fahrenheit folosind instrucțiuni SIMD (movss, mulss, addss).

Formula: $F = C \times 1.8 + 32$
```cpp
float celsius = 25.0f;
float factor = 1.8f;
float constant = 32.0f;
float fahrenheit;
```

3. Calculul ariei cercului

Folosiți FPU pentru a calcula aria unui cerc ($A = \pi \times r^2$). Utilizați instrucțiunea specială fldpi pentru a încărca valoarea lui $\pi$.

4. Rezolvarea ecuației de gradul I

Creati o functie care calculeaza $x$ din ecuația $ax + b = 0$, unde $a$ și $b$ sunt float, iar $a \neq 0$. 
Rezultatul trebuie returnat prin stiva FPU (așa cum fac funcțiile în C++).

Formula: $x = -b / a$