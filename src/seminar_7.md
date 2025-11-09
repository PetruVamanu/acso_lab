# Seminar 7

## Exercitii

### 1. Legea lui Amdahl:
- a. Avem un sistem ce foloseste CPU-ul 40% din timp, HDD-ul 30% din timp Avem de ales intre a cumpara un CPU de 4 ori mai bun si un HDD de 5. Stiind ca putem alege o singura componenta, ce alegem?
- b. Avem un sistem ce foloseste CPU-ul 25% din timp, HDD-ul 40% din timp, GPU-ul 15% din timp si SDD 10% din timp. Avem de ales intre a cumpara un CPU de 2 ori mai bun, un HDD de 1.5 ori mai bun, un GPU de 3 ori mai bun si un SDD de 5 ori mai bun. Stiind ca putem alege o singura componenta, ce alegem?
- c. Într-un procesor, componenta cea mai utilizată este unitatea aritmetico-logică (ALU), care este folosită 70% din timp. Dacă se proiectează o ALU cu 75% mai rapidă decât cea existentă, dar astfel pretul procesorului creste cu 150%, este sau nu justificat să înlocuim vechiul procesor cu cel nou? De ce?
- d. Într-un sistem de calcul, 50% din timpul de calcul al procesorului este folosit pentru a aştepta datele de la memoria RAM. Este justificată adăugarea de memorie cache, care măreşte preţul procesorului cu 75%, dar reduce timpul de acces la memorie cu 50%?


### 2. Memoria Cache
- a. Un procesor are o memorie cache de 256 Ko, cu următoarele caracteristici: H = 90%, Tc = 4 ns, Tm = 18 ns, Tp = 16 ns; Mai există două variante ale procesorului, cu cache de 512 Ko şi respectiv 1 Mo. La fiecare dublare a capacitatii cache-ului, rata de insucces se injumatateste, în schimb timpul de acces la cache şi timpul de acces la memorie în cazul unei ratări în cache cresc fiecare cu câte 2 ns. Care variantă de cache este mai rapidă?

- b. Demonstrati daca se justifica dotarea unui procesor cu un cache cu urmatoarele caracteristici: T c = 0.25 ns, Tp = 8 ns, Tm = 12 ns si H = 80%

- c. Un procesor are o memorie cache de 256 Ko, cu următoarele caracteristici: H = 80%, Tc = 2 ns, Tm = 6 ns, Tp = 8 ns; Mai există două variante ale procesorului, cu cache de 512 Ko şi respectiv 1 Mo. La fiecare dublare a capacitatii cache-ului, rata de insucces se injumatateste, în schimb timpul de acces la cache şi timpul de acces la memorie în cazul unei ratări în cache se dubleaza. Care variantă de cache este mai rapidă?

- d. Ce Miss-Ratio ar trebui sa avem pentru un cache cu urmatoarele caracteristici, astfel incat sa
se justifice dotarea procesorului cu acesta: Tc = 0.1 ns, Tp = 8 ns, Tm = 10 ns, T = 75% * Tp


### Setup ASM

<iframe src="assets/IDE-ASM.doc.pdf" width="100%" height="800px">
    <p>Your browser does not support PDFs. 
    <a href="assets/IDE-ASM.doc.pdf">Download the PDF</a>.</p>
</iframe>
