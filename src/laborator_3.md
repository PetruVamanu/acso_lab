# Laboator 3

## Instructiuni de salt

Instrucţiunile de salt modifică valoarea registrului contor program (EIP), astfel încât următoarea instrucţiune care se execută să nu fie neapărat cea care urmează în memorie. Sunt utile pentru implementarea, la nivel de limbaj de asamblare, a structurilor de control (testări sau bucle).

Salturile pot fi:
- neconditionate (prin instructiunea `jmp`)
- conditionate (instructiuni de forma `j<conditie>`)

Pentru orice instrucțiune de salt trebuie indicată adresa la care se va face saltul

Exprimarea adresei se poate face prin urmatoarele moduri:
- valoare constanta
- continutul unui registru
- continutul unei locatii din memorie

În practică, adresa de salt este aproape întotdeauna o valoare constantă
- dar de unde știm care este adresa dorită?

Raspuns: precizarea adreselor - prin etichete
- similar limbajului C (instrucțiunea `goto`)
- se pot face salturi din blocuri de cod `_asm` în blocuri de cod C și invers


## Salt neconditionat

### Sintaxa
`jmp adresa`
- nu necesita informaii suplimentare
- utilitate redusă - execuția programului nu este ramificată în două variante

Exemplu:
```c++
#include <stdio.h>

void main()
{
  int i;
  _asm {
    mov i, 11
    jmp eticheta
    sub i, 3;     // aceasta instructiune nu se executa
  eticheta:
    add i, 4
  }
  printf ("%d\n", i);
}
```
Este util, folosit împreună cu salturi condiţionate, pentru reluarea unei secvenţe de de cod într-o buclă, aşa cum se va vedea într-un exemplu ulterior.

## Salt conditionat
Introduc o ramificaţie în program, deoarece avem două variante:

- condiţia de salt este adevărată – se face saltul la adresa indicată
- condiţia de salt este falsă – se continuă cu instrucţiunea următoare din memorie ca şi cum nu ar fi existat instrucţiune de salt.

### Salturi pentru indicatori individuali

Cele mai utile la acest nivel sunt cele care testează indicatorii:
- Carry
- Overflow
- Sign
- Zero

Pentru fiecare indicator există două instrucţiuni de salt condiţionat:
- una care face saltul când indicatorul testat are valoarea 1
- una care face saltul când are valoarea 0

| indicator testat | salt pt. valoarea 1 | salt pt. valoarea 0 |
| :--- | :--- | :--- |
| Carry | `jc` (Jump if Carry) | `jnc` (Jump if Not Carry) |
| Overflow | `jo` (Jump if Overflow) | `jno` (Jump if Not Overflow) |
| Zero | `jz` (Jump if Zero) | `jnz` (Jump if Not Zero) |
| Sign | `js` (Jump if Sign) | `jns` (Jump if Not Sign) |
| Parity | `jp` (Jump if Parity) | `jnp` (Jump if Not Parity) |

Exemplu
```c++
#include <stdio.h>

void main()
{
  int a, b, s=0;
  printf("a=");
  scanf("%x", &a);
  printf("b=");
  scanf("%x", &b);
  
  _asm {
    mov eax, a;
    add eax, b;
    jc semnaleaza_depasire; //in Visual C++,
                            //putem sari la o eticheta din codul C
    mov s, eax;
    jmp afiseaza_suma;      //sau in alt bloc asm
  }
semnaleaza_depasire: 
  printf ("S-a produs depasire!\n");
  return;
  _asm {
    afiseaza_suma:
  }
  printf ("%x + %x = %x\n", a, b, s);
}
```

Chiar daca poate fi folosit uneori, nu este suficient pentru a face teste pe baza operatorilor cu care sunt testati relatiile in limbajul C: <, <=, ==, !=, >, >=.

### Comparare
A fost introdusa pentru a rezolva "lipsul" salturilor pentru indicatori individuali

#### Sintaxa
`cmp op1, op2`

Intern, se realizeaza o scadere pentru care rezultatul nu se scrie/pastreaza nicaieri. In schimb sunt setati indicatorii de conditii.

Instrucțiunea trebuie urmată de un salt care testează o anumită relație între op1 și op2

### Salturi care testeaza relatii

| relație | operanzi fără semn | operanzi cu semn |
| :--- | :--- | :--- |
| `op1 < op2` | `jb` (Jump if Below) | `jl` (Jump if Less) |
| `op1 <= op2` | `jbe` (Jump if Below or Equal) | `jle` (Jump if Less or Equal) |
| `op1 > op2` | `ja` (Jump if Above) | `jg` (Jump if Greater) |
| `op1 >= op2` | `jae` (Jump if Above or Equal) | `jge` (Jump if Greater or Equal) |
| `op1 == op2` | `je` (Jump if Equal) | `je` (Jump if Equal) |
| `op1 != op2` | `jne` (Jump if Not Equal) | `jne` (Jump if Not Equal) |

Sunt necesare instrucţiuni diferite pentru numere fără semn, respectiv cu semn, deoarece indicatorii ce trebuie verificaţi diferă. De exemplu, comparând 00100110 şi 11001101, ar trebui să obţinem relaţia 00100110 < 11001101 dacă sunt numere fără semn, şi 00100110 > 11001101 dacă sunt numere cu semn.


## Structuri de control
- în limbajul C nu se recomandă utilizarea instrucțiunii `goto`
- în schimb, se folosesc structuri de control
- dar implementarea acestora la nivelul procesorului se face tot prin instrucțiuni de salt

### Structura If
#### Implementare - salt condiționat pe condiție inversă celei din limbajul C
- motivație
- în limbajul C, se execută o secvență de cod dacă o condiție este adevărată
- în limbajul de asamblare, instrucțiunile de salt pot evita execuția unei secvențe de cod

```c++
/// C
    int a,x;
    // tip cu semn
    ...
    if(x>5)
        a=2;

/// ASM
    cmp x,5
    jle maimare
    mov a,2
    maimare:
```
#### Ramura else
dacă avem şi o ramură else, trebuie
utilizate două instrucţiuni de salt
- a doua instrucţiune se asigură că, la
terminarea execuţiei instrucţiunilor din
prima ramură, nu vor fi executate şi
instrucţiunile din a doua ramură
- salt necondiționat

```c++
/// C
    if(x>5)
        x--;
    else
        x++;
/// ASM
    cmp x,5
    jle nu
    dec x
    jmp afara
nu:
    inc x
afara:
```

### Structura Switch
similară unei structuri if-else
- organizare
- se compară valoarea testată, pe rând, cu toate
constantele corespunzătoare cazurilor din
blocul switch
- pentru fiecare caz se face un salt condiționat la
codul aferent

```c++
/// C
    switch (i) {
        case 0:
            a = 5; break;
        case 1:
            a = 7; //break - omis intenționat
        default:
            a = 10; 
    }
/// ASM
    cmp i,0
    je case_0
    cmp i,1
    je case_1
    jmp case_default

case_0:
    mov a,5
    jmp case_final

case_1:
    mov a,7
    //aici nu este break

case_default:
    mov a,10
    case_final:
```

### Structura While
- două instrucţiuni de salt
    - prima pentru a decide dacă se execută o nouă
    iteraţie sau se părăseşte bucla
        - saltul - pe condiţia inversă celei din codul C
    - a doua pentru a relua execuția de la început
        - la finalul corpului buclei
        - salt necondiţionat
```c++
/// C
    while(x<10)
        x++;
/// ASM
bucla:
    cmp x,5
    jge afara
    inc x
    jmp bucla
afara:
```

### Structura Do While
- testul de continuare/ieşire - la final
- se testează aceeaşi condiţie ca în limbajul C
- în ambele limbaje, condiția adevărată duce la
reluarea buclei
```c++
/// C
    do {
        s+=i;
        i++;
    } while(i<=5);

/// ASM
bucla:
    mov eax,s
    add eax,i
    mov s,eax
    inc i
    cmp i,5
    jle bucla

```


### Structura For
- similară structurii While
- instrucţiuni auxiliare - delimitează clar
    - iniţializarea variabilelor
    - actualizarea valorilor variabilelor pentru iteraţia următoare
    
```c++
/// C
    for(i=0;i<=5;i++)
        s+=3;

/// ASM
    mov i,0
    bucla: 
        cmp i,5
        jg afara
        add s,3
        inc i
        jmp bucla
    afara:
```

---
## Exercitii

### 1. Verificarea semnului
Scrieti un program C care citeste un numar intreg `x` de la tastatura. Folosind un bloc `_asm`, verificati daca numarul este negativ sau pozitiv/zero.
- Daca `x < 0`, variabila `rezultat` va lua valoarea -1.
- Daca `x >= 0`, variabila `rezultat` va lua valoarea 1.
Afisati rezultatul.

---

### 2. Maximul a doua numere
Se dau doua variabile `a` si `b`. Implementati in limbaj de asamblare logica pentru a pune in variabila `max_val` valoarea cea mai mare dintre cele doua.

---

### 3. Functie matematica pe ramuri
Implementati urmatoarea functie definita pe ramuri, pentru un `x` citit de la tastatura:
- Daca `x` este par, `y = x + 5`
- Daca `x` este impar, `y = x - 1`

---
### 4. Incadrare in interval
Se da un numar `x`. Verificati daca `x` apartine intervalului `[10, 20]`.
- Daca da, variabila `valid` devine 1.
- Altfel, variabila `valid` devine 0.

---


### 5. Mini-calculator
Se citesc doua numere `a` si `b` si un cod de operatie `op` (intreg).
- Daca `op == 1`, calculati `res = a + b`
- Daca `op == 2`, calculati `res = a - b`
- Daca `op == 3`, calculati `res = a * b` (presupuneti ca incape in 32 biti)
- Pentru orice alt cod, `res = 0`
Implementati folosind comparatii succesive (simulare switch).

---


### 6. Suma numerelor 
Calculati suma primelor `n` numere naturale (1 + 2 + ... + n), unde `n` este citit de la tastatura.
Folositi registrul `ecx` pe post de contor si o structura de tip bucla.

---
### 7. Numararea bitilor de 1 
Se da un numar intreg `n`. Numarati cati biti de 1 are acest numar in reprezentarea sa binara si salvati rezultatul in variabila `k`.

---
### 8. Impartirea prin scaderi repetate 
Implementati impartirea intreaga a lui `deimpartit` la `impartitor` folosind doar scaderi (fara instructiunea `div` sau `idiv`).
- Scadeti `impartitor` din `deimpartit` atata timp cat `deimpartit >= impartitor`.
- Numarati de cate ori ati scazut (acesta va fi catul).
- Ce ramane in `deimpartit` la final este restul.
Folositi o structura de tip `do-while` (verificarea la final) sau `while`.

---
### 9. Sirul lui Fibonacci 
Generati al `n`-lea termen din sirul lui Fibonacci, unde `n` este dat.
Sirul: 0, 1, 1, 2, 3, 5, 8, 13...
- F0 = 0, F1 = 1
- Fn = F(n-1) + F(n-2)

---
### 10. Verificare numar prim 
Verificati daca un numar `n` (n > 2) este prim.
- Creati o bucla cu un divizor `d` care porneste de la 2 si merge pana la `n/2` (sau `n-1`).
- In interiorul buclei, calculati restul impartirii lui `n` la `d` (folosind `div` sau `idiv`).
- Daca restul este 0, numarul nu este prim: setati un flag si folositi un salt (`break`) pentru a iesi fortat din bucla.
- Daca bucla se termina fara sa gasiti divizori, numarul este prim.

---
### 11. Cel mai mare divizor comun 
Implementati algoritmul lui Euclid prin scaderi repetate pentru a gasi CMMDC al numerelor `a` si `b`.
Algoritm:
```text
Cat timp (a != b) executa:
    Daca a > b atunci
        a = a - b
    Altfel
        b = b - a
Sfarsit Cat timp
Resultat = a
```