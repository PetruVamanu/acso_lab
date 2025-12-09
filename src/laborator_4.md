# Laboator 4

## Lucrul cu stiva

Procesorul foloseşte o parte din memoria RAM pentru a o accesa printr-o disciplină de tip LIFO
- gestiune - adresa vârfului stivei
- memorată în registrul `ESP`
- structurile de tip stivă sunt necesare în
multe situaţii

Instrucţiunile care permit lucrul cu stiva sunt `push` şi `pop`.

### Instructiunea PUSH

Realizează introducerea unei valori în stivă.

Sintaxa: `push operand;`

Operandul poate fi registru, locaţie de memorie sau constantă numerică. Stiva lucrează doar cu valori de 2 sau 4 octeţi, pentru uniformitate preferându-se numai operanzi de 4 octeţi (varianta cu 2 se păstrază pentru compatibilitate cu procesoarele mai vechi).

Introducerea valorii în stivă se face astfel: se scade din `ESP` dimensiunea, în octeţi, a valorii care se vrea depusa în stivă, dupa care procesorul scrie valoarea operandului la adresa indicată de registrul `ESP` (vârful stivei); dimensiunea poate fi 2 sau 4 (se observă că se avansează "în jos", de la adresele mai mari la adresele mai mici); în acest mod, vârful stivei este pregătit pentru următoarea operaţie de scriere.

Exemple:
``` 
push eax
push dx
push dword ptr [200]
push word ptr [esi*2+ecx]
push dword ptr 5
push word ptr 14
```

De exemplu, instrucţiunea
```
push eax;
```
ar fi echivalentă cu:
```
sub esp, 4;
mov [esp], eax;
```
Prin folosirea lui push în locul secvenţei echivalente se reduce, însă, riscul erorilor.


### Instructiunea POP

Extrage vârful stivei într-un operand destinaţie.

Sintaxa: `pop operand`;

Operandul poate fi registru sau locaţie de memorie, de 2 sau 4 octeţi.

Exemple:
```
pop eax
pop cx
pop dword ptr [ebx+edi]
pop word ptr [edx+100]
```

Extragerea valorii din stivă se face prin depunerea în destinaţie a valorii aflate în vârful stivei (la adresa `[ESP]`) şi adunarea, la `ESP`, a numărului de octeţi ai operandului (acesta indică, practic, numărul de octeţi scoşi din stivă).


### Rolul stivei
Rolul stivei programului - stocarea de informaţii cu caracter temporar
- salvarea pentru un timp a valorii unui registru, pentru a-l folosi în alt scop
    - apoi valoarea salvată este restaurată în registru
- variabile locale
    - create la momentul apelului funcţiei și distruse la terminarea execuției funcţiei

#### De retinut!
Instrucţiunile de introducere în stivă trebuie riguros compensate de cele de scoatere din stivă
- ca număr de instrucţiuni
- ca dimensiune a operanzilor
Orice eroare afectează mai multe date decât pare la prima vedere

Exemplu:
```
push eax
push edx
mov eax,[esi]
mov edx,5
mul edx
mov [esi],eax

pop eax
```

O instrucţiune `pop` a fost uitată - efecte
- registrul `eax` primeşte altă valoare decât avea iniţial
- valoarea iniţială a registrului `edx` nu este restaurată

Similar dacă se execută prea multe instrucţiuni `pop`

Exemplu 2:
```
push edx;
push eax;
... //utilizare registri
pop  ax  //se recupereaza doar 2 octeti din valoarea anterioara a lui eax
pop edx  //nu se recupereaza edx, ci 2 octeti din eax, 2 din edx
         //decalajul se poate propaga astfel pana la capatul stivei
```

O altă eroare poate apărea atunci când registrul ESP este manipulat direct. De exemplu, pentru a aloca spaţiu unei variabile locale (neiniţializată), e suficient a scădea din ESP dimensiunea variabilei respective. Similar, la distrugerea variabilei, valoarea ESP este crescută. Aici nu se folosesc în general instrucţiuni push, repectiv pop, deoarece nu interesează valorile implicate, ci doar ocuparea şi eliberarea de spaţiu. Se preferă adunarea şi scăderea direct cu registrul ESP; evident că o eroare în aceste operaţii are consecinţe de aceeaşi natură ca şi cele de mai sus.

## Apeluri de functii

La prima vedere - ca o instrucţiune de salt
- se întrerupe execuţia liniară a programului şi se sare la altă adresă

Diferență - la terminarea funcţiei se revine la adresa de unde s-a făcut apelul
- deci aceasta a fost memorată
- informaţie temporară - tot pe stivă

Din moment ce într-un program se poate apela o funcţie de mai multe ori, din mai multe locuri, şi întotdeauna se revine unde trebuie, este clar că adresa la care trebuie revenit este memorată şi folosită atunci când este cazul. Cum adresa de revenire este în mod evident o informaţie temporară, locul său este tot pe stivă.

### Instructiunea CALL

Sintaxa `call adresa`
- în Visual C++, adresa este indicată folosind chiar numele funcţiei apelatea

Efectul instrucţiunii call: se introduce în stivă adresa instrucţiunii următoare (adresa de revenire) şi se face salt la adresa indicată. Aceste acţiuni puteau fi realizate şi cu instrucţiuni push şi jmp, dar din nou se preferă call pentru evitarea erorilor.

### Instructiunea RET

Sintaxa `ret`

Revenirea dintr-o funcţie se face prin instrucţiunea ret, care poate fi folosită fără operand. În acest caz, se preia adresa de revenire din vârful stivei (similar unei instrucţiuni pop) şi se face saltul la adresa respectivă. Din motive de conlucrare cu Visual Studio, nu vom folosi această instrucţiune.

#### Erori în lucrul cu stiva
- o instrucţiune pop omisă (sau una în plus)
    - la execuţia unei instrucţiuni ret se va prelua din stivă altă valoare decât adresa corectă de revenire
    - efect - blocarea programului sau terminarea sa
forţată

### Variabile locale
- create de obicei la începutul execuției
funcției/blocului
- fiind temporare, sunt plasate tot pe stivă
- ocuparea spațiului pe stivă se poate face
    - prin instrucțiuni push
    - prin scăderea valorii vârfului stivei (ESP)
- la terminarea funcției/blocului, trebuie
eliminate de pe stivă

Cod C
```
int a,b=3,c;
```

Cod ASM
```
sub esp,12 //alocare spațiu
mov dword ptr [esp+4],3 // b  3

...
add esp,12 //eliberare spațiu la final
```

### Transmiterea parametrilor
- sunt tot variabile locale - se găsesc pe stivă
- apelantul are responsabilitatea
    - de a-i pune pe stivă la apel
- folosind instrucțiunea push
    - de a-i scoate de pe stivă la revenirea din funcţia
apelată
- nu se folosește instrucțiunea pop, ci se adună la
ESP numărul total de octeţi ocupat de parametri
- în C/C++ parametrii trebuie puşi în stivă în ordine inversă celei în care se găsesc în lista de parametri
    - motiv - funcții cu număr variabil de parametri
- de ce trebuie scoşi din stivă la revenire?
    - la finalul fiecărei unități de program trebuie să lăsăm stiva așa cum am găsit-o la intrare
    - altfel apar erori de tipul celor descrise anterior

Exemplu:

• cod C
```c++
void dif(int a,int b)
{
    int c;
    c=a-b;
    cout<<c<<endl;
}
```
apelul dif(9,5) se traduce prin secvenţa
de mai jos:
```
push dword ptr 5
push dword ptr 9
call dif
add esp,8
```

### Accesul la parametri
- primul parametru din antetul funcţiei se găseşte
întotdeauna la adresa EBP+8
- următorii parametri, mergând spre dreapta în
lista parametrilor, sunt la adrese corespunzător
mai mari: EBP+12, EBP+16 etc.

Exemplu, o functie cu 3 parametri de tip int (o variabila de tip int are 4 octeti):

```c++
void functie(int a, int b, int c){
  _asm{
                
      mov eax, [ebp+8]  // muta in eax valoarea lui a
      mov ebx, [ebp+12] // muta in ebx valoarea lui b
      mov ecx, [ebp+16] // muta in ecx valoarea lui c

  };
}
```

Scris in aceasta maniera, exemplul de mai sus ar arata in felul urmator:
```c++
#include <stdio.h>

int compute_dif(int ,int ) { // nu mai este nevoie sa punem nume variabilelor, deoarece vom lucra direct cu stiva
  _asm{
                
    mov eax, [ebp+8]; 
    sub eax, [ebp+12]; 
                // in eax ramane rezultatul, care 
                // va fi preluat la termiarea functiei
  };
}

void main() {
  int c;
  _asm {
    push dword ptr 9
    push dword ptr 5
    call compute_dif  // se salveaza adresa de revenire pe stiva
    mov c, eax;
    add esp,8         // "stergerea" parametrilor din stiva
  }
  printf("Diferenta este %d.\n", c);
}
```


### Returnarea valorilor
-  convenție stabilită de compilator
- Visual C++ - rezultatul se depune într-un
registru - depinde de dimensiunea sa
    - tipuri de date de dimensiune 1 octet: al
    - tipuri de date de dimensiune 2 octeţi: ax
    - tipuri de date de dimensiune 4 octeţi: eax
    - tipuri de date de dimensiune 8 octeţi: edx şi eax

### Pastrarea valorilor registrilor
- programul pe care îl scriem este C/C++
- nu ştim cum foloseşte compilatorul regiştrii
- pentru orice funcţie pe care o scriem în limbaj de asamblare
    - la începutul execuţiei funcţiei trebuie salvați în stivă regiștrii pe care îi folosim în funcție
- la terminare, valorile lor trebuie restaurate
- excepție - eax şi edx (returnare valori)

### Functii recursive
- nu implică un mecanism diferit de funcțiile obișnuite (nerecursive)
- apelul se face către aceeași funcție, nu către alta
- în orice limbaj este scrisă, trebuie multă atenție la condiția de terminare a recursiei
    - risc - recursie infinită

- la scrierea în limbaj de asamblare, programatorul trebuie să implementeze ambele părți ale protocolului de comunicare
    - depunerea parametrilor în stivă - la efectuarea apelului
    - preluarea parametrilor din stivă - la execuția funcției

- atenție la preluarea rezultatului returnat de
apelul anterior

### Apeluri de întreruperi

- întreruperi software
    - instrucțiunea int
    - similară instrucțiunii call, dar mai complexă
- revenirea din rutina de tratare
    - instrucțiunea iret
    - din nou, similară cu ret, dar mai complexă

## Exercitii
### 1. Inversarea a 3 valori folosind stiva
Aveti 3 variabile in C: `a`, `b`, `c`.
Scrieti un bloc `_asm` care interschimba valorile acestor variabile folosind **doar** instructiuni `push` si `pop`, fara a folosi registre suplimentare (gen `eax` sau `ebx`) pentru stocarea temporara.

---
### 2. Evaluarea unei expresii folosind stiva
Calculati expresia `E = a - b + c` folosind stiva pentru a pastra rezultatele intermediare.

---

### 3. Functia "Suma a 3 numere"
Implementati o functie `int suma(int a, int b, int c)` complet in asamblare (corpul functiei) si apelati-o din `main` tot folosind asamblare.

---
### 4. Functia "Minimul dintre 2 numere"
Scrieti o functie care primeste doi parametri intregi si returneaza cel mai mic dintre ei.

---


### 5. Modificarea unei variabile prin pointer
Scrieti o functie `void dubleaza(int* p)` care primeste ca parametru adresa unei variabile si ii dubleaza valoarea.
- Atentie: Parametrul de pe stiva `[ebp+8]` nu este valoarea, ci **adresa** valorii.
- Trebuie sa incarcati adresa intr-un registru (ex: `mov ebx, [ebp+8]`), apoi sa operati asupra memoriei la care arata acel registru (ex: `add [ebx], ...`).

---
### 6. Alocare manuala pe stiva (Variabile locale)
In cadrul unei functii `asm_func`, alocati spatiu pe stiva pentru 2 variabile locale de tip int (scazand `esp`).
- Initializati aceste variabile locale cu valorile 10 si 20.
- Adunati-le in `eax`.
- La final, eliberati spatiul alocat (adunand la `esp`) inainte de a returna.

---

### 7. Factorialul unui numar (Recursiv)
Implementati functia recursiva `int factorial(int n)`.
Algoritm:
- Verificati conditia de oprire: daca `n <= 1`, returnati 1 (`mov eax, 1` si iesire).
- Pasul recursiv:
    - Calculati `n-1`.
    - Puneti `n-1` pe stiva (`push`).
    - Apelati `factorial`.
    - Curatati stiva (`add esp, 4`).
    - Rezultatul `factorial(n-1)` este acum in `eax`.
    - Inmultiti `eax` cu `n` (care se afla la `[ebp+8]`).
    - Returnati.

---
### 8. Salvarea Registrilor (Callee-Saved)
Scrieti o functie complexa care are nevoie de registrele `ebx`, `esi` si `edi` pentru calcule intermediare.
- Conform conventiei, aceste registre trebuie sa aiba aceeasi valoare la iesirea din functie ca la intrare.
- **La inceputul functiei:** Salvati registrele pe stiva (`push ebx`, `push esi`, etc.).
- **La finalul functiei:** Restaurati-le (`pop ...`) in ordine inversa.