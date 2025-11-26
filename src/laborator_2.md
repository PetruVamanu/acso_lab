# Laboator 2

## Instrucţiuni aritmetice: add, sub, inc, dec, mul/imul, div/idiv 

### Add / Sub

#### Sintaxa
- add op1,op2 // op1 += op2
- sub op1,op2 // op1 -= op2
- sunt posibile aceleași combinații de operanzi ca în cazul atribuirii
- indicatorii de condiții primesc valori conform rezultatului operației

#### Ex.
```
add eax,ebx
add dl,3
add si,[ecx]
add [eax+edi],ebp
add byte ptr [esi+10],14
add word ptr [esi+10],14
add dword ptr [esi+10],14
```

#### Operatii cu si fara semn
```
unsigned int a;
int b;
_asm add a,20;
_asm add b,20;
```
Procesorul nu stie ce tip de date au a,b(el stie ca sunt locatii din memorie).

Astfel trebuie testati indicatorii Carry/Overflow. Acestia sunt setati automat de procesor, dar trebuie verificati de programatori.

#### Adunare/Scadere cu transport
```
adc op1,op2 // op1=op1+op2+Carry
sbb op1,op2 // op1=op1-op2-Carry
```
- la adunare/scădere participă și valoarea anterioară a indicatorului Carry
- rol - operații cu numere de dimensiuni prea mari pentru a fi stocate în variabile scalare

### Inc / Dec

#### Sintaxa
```
inc op // op++
dec op // op--
```
- operandul poate fi
    - registru (orice dimensiune)
    - locație de memorie - dimensiunea trebuie precizată explicit

### Mul/Imul

#### Sintaxa
```
sintaxa
mul op // numere fără semn
imul op // numere cu semn
```
Efect: destinatie_implicita = operand_implicit * op

- operandul explicit nu poate fi o constantă
- dacă operandul explicit este o locație de memorie, trebuie precizată și dimensiunea sa
- destinația rezultatului este tot implicită și necesită o dimensiune dublă față de operanzi

| dimensiune operand explicit | operand implicit | destinație rezultat |
| :--- | :---: | :---: |
| 1 octet | `al` | `ax` |
| 2 octeți | `ax` | `(dx,ax)` |
| 4 octeți | `eax` | `(edx,eax)` |

#### Ex
```
mul ebx // eax * ebx ->(edx,eax)
mul cx // ax * cx ->(dx,ax)
mul al // se ridică al la pătrat
mul dword ptr [esi]
// eax[esi] ->(edx,eax)
// operanzi pe 4 octeți
imul cx // operanzi cu semn
```
```
#include <stdio.h>

void main(){
  _asm {
    mov ax, 60000; //in baza 16: EA60
    mov bx, 60000; //in baza 16: EA60
    mul bx;        //rezultatul inmultirii este 3600000000; 
                   //in baza 16: D693A400, plasat astfel:
                   //in registrul dx - partea cea mai semnificativa: D693
                   //in registrul ax - partea cea mai putin semnificativa: A400

  }
}

```
```
#include <stdio.h>

void main(){
  _asm {
    mov eax, 60000; //in baza 16: 0000EA60
    mov ebx, 60000; //in baza 16: 0000EA60
    mul ebx;        //rezultatul inmultirii este 3600000000; 
                    //in baza 16: D693A400, plasat astfel:
                    //in edx - partea cea mai semnificativa: 00000000
                    //in eax - partea cea mai putin semnificativa: D693A400
  }
}


```
```
void main(){
  _asm {
    mov ax, 0xFFFF; 
    mov bx, 0xFFFE;
    mul bx;        //rezultatul inmultirii numerelor FARA SEMN:
                   //65535 * 65534 = 4294770690; 
                   //in baza 16: FFFD0002, plasat astfel:
                   //in dx - partea cea mai semnificativa: FFFD
                   //in ax - partea cea mai putin semnificativa: 0002

    mov ax, 0xFFFF; 
    mov bx, 0xFFFE;
    imul bx;       //rezultatul inmultirii numerelor CU SEMN:
                   //-1 * -2 = 2; 
                   //in baza 16: 00000002, plasat astfel:
                   //in dx - partea cea mai semnificativa: 0000
                   //in ax - partea cea mai putin semnificativa: 0002

  }
}
```

### Div/Idiv

#### Sintaxa
```
div op // numere fără semn
idiv op // numere cu semn
```
Efect: cat_implicit, rest_implicit = deimpartit_implicit : op

- este indicat explicit doar împărțitorul
- poate fi registru sau locație de memorie
- deîmpărțitul este implicit și depinde de dimensiunea împărțitorului
- împărțitorul nu poate fi o constantă
- dacă împărțitorul este o locație de memorie, trebuie precizată și dimensiunea sa
- două rezultate: câtul și restul
- destinațiile acestora sunt tot implicite

| dimensiune împărțitor | deîmpărțit | cât | rest |
| :---: | :---: | :---: | :---: |
| 1 | `ax` | `al` | `ah` |
| 2 | `(dx,ax)` | `ax` | `dx` |
| 4 | `(edx,eax)` | `eax` | `edx` |

#### Impartirea la 0
- impartitorul = 0 -> eroare de executie
- problema poate aparea si in alte situatii

```
_asm {
    mov eax,1
    mov edx,1
    mov ebx,1
    div ebx
}
```

Secventa produce eroare
Motivul este acela că se încearcă împărţirea numărului 0x100000001 la 1, câtul fiind 0x100000001. Acest cât trebuie depus în registrul eax, însă valoarea lui depăşeşte valoarea maximă ce poate fi pusă în acest registru, adică 0xFFFFFFFF. Mai concret, în cazul în care câtul nu încape în registrul corespunzător, se obţine eroare:
```
(edx*2^32 + eax) / ebx ≥ 2^32 <=>
edx*2^32 + eax ≥ ebx*2^32 <=>
eax ≥ (ebx - edx) * 2^32 <=>
ebx ≤ edx
```
Cu alte cuvinte, vom obţine cu siguranţă eroare dacă împărţitorul este mai mic sau egal cu partea cea mai semnificativă a deîmpărţitului. Pentru a evita terminarea forţată a programului, trebuie verificată această situaţie înainte de efectuarea împărţirii.

#### Ex.
```
#include <stdio.h>

void main(){
  _asm {
    mov ax, 35;
    mov dx, 0;  //nu trebuie uitata initializarea lui (e)dx!
                //(in general, initializarea partii celei mai
                //  semnificative a deimpartitului)
    mov bx, 7;  
    div bx;    //rezultat: ax devine 5, adica 0x0005 (catul)
               //          dx devine 0 (restul)
               

    mov ax, 35;
    mov dx, 0; 
    mov bx,7  
    idiv bx     // acelasi efect, deoarece numerele sunt pozitive
    
    
    mov ax, -35; //in hexa (complement fata de 2): FFDD
    mov dx, 0;
    mov bx,7  
    div bx      //deimpartitul este (dx, ax), adica 0000FFDD
                //in baza 10: 65501
                //rezultat: ax devine 0x332C, adica 13100 (catul)
                //          dx devine 0x0001  (restul)
                
    mov ax, -35; //in hexa (complement fata de 2): FFDD
    mov dx, 0;
    mov bx,7  
    idiv bx     //deimpartitul este (dx, ax), adica 0000FFDD
                //este un mumar pozitiv, adica, in baza 10, 65501
                //rezultat: ax devine 0x332C, adica 13100 (catul)
                //          dx devine 0x0001  (restul)
                //(efectul este acelasi ca la secventa de mai sus)
                
    mov ax, -35; //in hexa (complement fata de 2): FFDD
    mov dx, -1;  //in hexa (complement fata de 2): FFFF
    mov bx,7  
    idiv bx     //deimpartitul este (dx, ax), adica FFFFFFDD
                // - numar negativ, reprezentat in complement fata de 2
                //in baza 10: -35
                //rezultat: ax devine 0xFFF9, adica -5 (catul)
                //          dx devine 0  (restul)
    
    mov ax, -35; //in hexa (complement fata de 2): FFDD
    mov dx, -1;  //in hexa (complement fata de 2): FFFF
    mov bx,7  
    div bx      //deimpartitul este (dx, ax), adica FFFFFFDD
                // - numar pozitiv (deoarece folosim div)
                // in baza 10: 4294967261
                //rezultat: EROARE, deoarece FFFF > 0007,
                //   catul (613566751, adica 2492491F) nu incape in ax
    
  }
}
```

## Instrucţiuni logice: not, or, and, xor, test, shl, shr, sar, sal, rol, ror, rcl, rcr

### And, or, not, xor, test

Implementeaza functiile booleene elementare
- instrucțiunile au aceleași nume ca și
funcțiile pe care le implementează
- execuție: se aplică funcția, în paralel, pe toți
biții operanzilor

Instrucţiunea test are acelaşi efect ca şi and (execută AND intre biţii celor doi operanzi, modifică la fel indicatorul ZERO), dar nu alterează valoarea primului operand. Aceasta modifica indicatorul ZERO fara a altera valoarea sursei.

```
not eax
and bx,16
or byte ptr [edx],100
xor [esi],ecx
test al,ah
```
- și aici, primul/singurul operand este și
destinația rezultatului


### Instructiuni de deplasare(shl, shr, sar, sal)
Sunt instrucţiuni care permit deplasarea biţilor în cadrul operanzilor cu un număr precizat de poziţii.

Deplasările pot fi aritmetice sau logice. Deplasările aritmetice pot fi utilizate pentru a înmulţi sau împărţi numere prin puteri ale lui 2. Deplasările logice pot fi utilizate pentru a izola biţi în octeţi sau cuvinte.


Dintre modificările pe care deplasările le fac asupra indicatorilor:

- Carry Flag (CF) = ultimul bit deplasat în afara operandului destinaţie;
- Sign Flag (SF) = bitul cel mai semnificativ din operandul destinaţie;
- Zero Flag (ZF) = 1 dacă operandul destinaţie devine 0, 0 altfel.
Instrucţiunile de deplasare sunt:
```
shr dest, count
shl dest, count
sar dest, count
sal dest, count
```

count precizează cu cîte poziţii se face deplasarea; poate fi constantă numerică sau registrul cl:
    - shl ebx, cl

#### SHR (SHift Right) - deplasare la dreapta
Efect: deplasarea la dreapta a biţilor din dest cu numărul de poziţii precizat de count; completarea la stânga cu 0; plasarea în CF (Carry Flag) a ultimului bit ieşit.
```
mov bl, 33; //binar: 00100001
shr bl, 3;  //bl devine 00000100
            //Carry devine 0
shr bl, 3   //bl devine 00000000
            //Carry devine 1
```

#### SHL (SHift Left) - deplasare la stanga
Efect: deplasarea la stânga a biţilor din dest cu numărul de poziţii precizat de count; completarea la dreapta cu 0; plasarea în CF (Carry Flag) a ultimului bit ieşit.
```
mov bl, 33; //binar: 00100001
shl bl, 3;  //bl devine 00001000
            //Carry devine 1
shl bl, 1   //bl devine 00010000
            //Carry devine 0
```

#### SAR (Shift Arithmetic Right) - deplasare aritmetica la dreapta
Efect: deplasarea la dreapta a biţilor din dest cu numărul de poziţii precizat de count; bitul cel mai semnificativ îşi păstrează vechea valoare, dar este şi deplasat spre dreapta (extensie de semn); plasarea în Carry a ultimului bit ieşit.
```
mov bl, -36; //binar: 11011100
sar bl, 2;   //bl devine 11110111
             //Carry devine 0
```
Trebuie menţionat că sar nu furnizează aceeaşi valoare ca şi idiv pentru operanzi echivalenţi, deoarece idiv trunchiază toate câturile către 0, în timp ce sar trunchiază câturile pozitive către 0 iar pe cele negative către infinit negativ.
```
mov ah, -7; //binar: 11111001
sar ah, 1;  //teoretic, echivalent cu impartirea la 2
            //rezultat: 11111100, adica -4
            //idiv obtine catul -3
```

#### SAL (Shift Arithmetic Left) - deplasare aritmetica la stanga
Analog cu `SAR`


### Instructiuni de rotire(rol, ror, rcl, rcr)
- similare instrucțiunilor de deplasare
- diferență - biții care "ies" nu sunt pierduți, ci sunt introduși la celălalt capăt
- rotirea se poate face luând în considerare și bitul Carry
    - în acest caz rotirea sa face practic pe 9/17/33 biți în loc de 8/16/32

#### Sintaxa 
- rotire fără Carry
```
rol op1,op2 // rotire spre stânga
ror op1,op2 // rotire spre dreapta
```
- rotire prin Carry
```
rcl op1,op2 // rotire spre stânga
rcr op1,op2 // rotire spre dreapta
```

### Conversii: movzx, movsx 

`movzx` - move with zero extend: extinde cu valori de zero
`movsx` - move with sign extend: extinde cu valori in functie de semn

[Docs movzx](https://www.felixcloutier.com/x86/movzx)

[Docs movsx](https://www.felixcloutier.com/x86/movsx:movsxd)



## Exercitii:
### 1. Fie următorul program care calculează factorialul unui număr. Să se înlocuiască linia de cod din interiorul buclei for (f = f * i) cu un bloc de cod asm, cu obţinerea aceluiaşi efect. Pentru simplificare, vom considera că rezultatul nu depăşeşte 4 octeţi.

```
#include <stdio.h>

void main(){
  unsigned int n = 10, i, f = 1;
  for(i=1;i<=n;i++) {
    f = f * i;
  }
  printf("%u\n",f);
}
```

### 2. Fie următorul program. Să se înlocuiască liniile 4 şi 5 cu un bloc de cod asm, cu obţinerea aceluiaşi efect.
```
#include <stdio.h>

void main(){
   unsigned a=500007,b=10,c,d;
   c=a/b;
   d=a%b;
   printf("%u %u\n",c,d);
}
```

### 3. Calculul unei expresii matematice
Să se implementeze în limbaj de asamblare calculul următoarei expresii: $E = 2 \cdot x^2 + 3 \cdot x - 5$.
Se consideră `x` o variabilă de tip `int` (cu semn). Rezultatul va fi stocat în variabila `result`.

### 4. Optimizarea înmulțirii prin deplasări (Shift)
Să se scrie o secvență de cod care înmulțește variabila `a` (fără semn) cu valoarea 32, folosind instrucțiuni de deplasare pe biți (`shl`) în locul instrucțiunii clasice de înmulțire (`mul`), pentru a optimiza viteza de execuție.

### 5. Determinarea parității (Fără salturi)
Verificați paritatea unui număr `n` folosind doar operații logice (`and`, `test`), fără a utiliza instrucțiuni de salt sau structuri condiționale (fără `if/else` sau etichete). Rezultatul (0 pentru par, 1 pentru impar) se va stoca într-o variabilă `este_impar`.

### 6. Media aritmetică
Calculați media aritmetică a două numere `a` și `b` fără semn. Programul trebuie să gestioneze corect pregătirea registrelor pentru instrucțiunea `div` (extinderea deîmpărțitului la `EDX:EAX` cu zero), presupunând că suma `a+b` încape într-un registru de 32 de biți.

### 7. Interschimbarea valorilor (XOR Swap)
Să se interschimbe valorile a două variabile `var1` și `var2` folosind doar instrucțiunea `xor`, fără a utiliza un registru auxiliar suplimentar sau o variabilă temporară.