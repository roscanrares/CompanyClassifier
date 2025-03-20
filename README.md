### Company Classifier

![image](https://github.com/user-attachments/assets/220341d5-a27e-40f4-8383-e05c9c6390c5)

Am primit 2 fisiere, un fisier [insurance_taxonomy.xlsx](https://github.com/roscanrares/Veridion-CompanyClassifier/blob/main/insurance_taxonomy.xlsx) si [ml_insurance_challenge.csv](https://github.com/roscanrares/Veridion-CompanyClassifier/blob/main/ml_insurance_challenge.csv). Scopul task-ului era de a atrbiui label-urile din insurance_taxonomy potrivite firmelor din ml_insurance_challenge.

Formatul fisierului cu companii era urmatorul: description (un string cu o descriere in limbaj natural a companiei), bussiness_tags (o lista cu tag-uri sub forma ['tag1', 'tag2']), sector (sectoru pe care activeaza firma, de ex. 'Services' sau 'Manufacturing'), category (categoria firmei) si niche (nisa pe care activeaza firma respectiva). In fisier avem circa 10000 de companii si 221 de label-uri.

![image](https://github.com/user-attachments/assets/980e94bd-b594-49db-a67c-566f5d09faff)

## Thought-process

Veridion pune accent pe scalabilitate. Compania lucreaza cu foarte multe date, astfel este asteptata o scalabilitate pentru a putea procesa miliarde de date. Astfel am incercat sa reduc cat mai mult posibil costul computational.
Am analizat cu atentie setul de date; primul pas a fost de a analiza putin companiile. Fiind foarte multe companii, in analiza am pus accent pe sector si categorii, unde am observat cateva lipsuri in legatura cu label-uri. Ce vreau sa zic este ca sunt foarte multe companii care nu sunt eligibile in primirea unui label.


Mi-am permis sa citesc toate cele 220 de label-uri. Printre acestea se regasesc doua label-uri foarte nisate: 'Swimming pool installation sevrices' si 'Swimming pool maintenance services'. Pe de alta parte, avem companii care se ocupa productia de masini, sau universitati/colegii. Cele din urma nu au un label care le poate fi atribuit in lista de 200 de label-uri. Am inteles de aici faptul ca veridion cand mentiona ca ei doresc sa proceseze datele a mult mai multor companii, ca o sa existe si mult mai multe label-uri (posibil mii de label-uri). In contextul acesta, **este imposibil verificarea directa a unei companii cu fiecare label in parte**. 

Primul meu gand a fost sa clustarizez label-urile, dar am realizat ca nu este fezabil. Motivul este faptul ca multe label-uri au cuvinte comune, dar care nu ne ajuta. Label-urile prezinta domeniul cu care se ocupa firma, dar se difera prin categorie, avem spre exemplu 'Residential Snow Removal' si 'Commercial Snow Removal' sau 'Commercial Construction Services'. Din aceste exemple in teorie ar trebui sa extragem cuvintele 'residential', 'commercial' si 'services'. Astfel aici ramanem cu doua, 'snow' si 'construction'. Problema este ca noi avem si label-urile 'dock and pier construction' si 'pipeline construction services'. Asta inseamna ca nu putem elimina cuvantul 'construction' din acestea. Folosirea de embeddings nu este fiabila deoarece are un cost computational ridicat (asta daca nu vrem sa verificat fiecare label in parte). M-am gandit sa incerc "augmentarea" label-urilor (adica sa adaug sinonime la fiecare label pentru a putea avea un rezultat cu acuratete mai mare in ruma comparatiei dar dupa ce am incercat asta cu wordnet am realizat ca unele label-uri nu primeau sinonime, pe cand altele primeau sinonime foarte proaste; asta ignorand faptul ca crestea costul computational).

Am renuntat la ideea de a grupa label-urile astfel. Mi-a venit o alta idee in schimb, sa creez un arbore cu toate label-urile. Ideea a venit in incercarea mea de a intelege problema adevarata a companiei. Ma refer la faptul ca daca noi avem doar 10% din label-uri, am incercat sa gasesc solutia pentru acel 100% din label-uri. Nu am reusit sa gasesc de unde sunt extrase aceste label-uri, asa ca am analizat codurile CAEN pentru firmele romanesti drept inspiratie. Ideea mea de la bun inceput a fost sa grupez label-urile in functie de domeniul de activitate. Dupa ce am analizat si codurile CAEN am observat ca acestea sunt deja structurate intr-un arbore oarecum (adica se ramifica). 

Ideea este in felul urmator. Luam drept exemplu codurile CAEN. Prima categorie de cod este agricultura-silvicultura-si-pescuit; aceasta se ramifica ulterior in 3 copii: agricultura, silvicultura si pescuit. Ulterior acestea se mai ramifica de cateva ori. Scopul arborelui este de a compara compania 'x' cu toate domeniile de activitate (pt. codurile CAEN exista 27 de domenii principale), in cazul in care aveam acuratete sa spunem >0.6 la unul sau mai multe domenii, le aprofundam si incepeam sa comparam cu copii lor. Adica daca aveam o firma de pescuit ar fi aratat ceva de genu 'peste.srl' -> 'agricultura'silvicultura'si'pescuit' -> 'pescuit' (asta fiind o generalizare). Ulterior in functie de puterile computationale pe care le avem, dupa ce am gasit domeniul de activitate pescuit, daca sub pescuit avem 3 copii, putem concatena intr-o forma sau alta (in functie de modelul pe care vrem sa il folosim) nodul 'pescuit' pentru a avea acuratete reala mai buna (adica sa nu dea miss-match sau sa nu depaseasca threshold-ul).

Ok, acum ca am ideea as putea sa o implementez, dar nu mi se pare fezabil pentru 220 de label-uri. In primul rand, neavand toate label-urile mi se pare complicat sa reusesc sa creez o structura cu sens si logica, iar ulterior la vederea celorlalte label-uri trebuie luat de la zero. Desigur, ideal ar fi ca label-urile sa fie deja structurate ca codurile CAEN, dar in cazul in care nu se poate, putem implementa o metoda automata (dar reprt dupa vederea tuturor label-urilor).

## Preprocesare

Aici am avut cele mai mari probeleme. Am folosit embeddings pentru a filtra datele gresite din companii.

![image](https://github.com/user-attachments/assets/89e8dbdd-8f34-40a9-92d8-ed62900a2a77)

Da, poate nu era neaparat cel mai eficient mod, dar tag-urile pot fi problematice. De exemplu compania 'Kyoto' vinde legume, dar are sectorul atribuit 'manufacturing' si tag-ul 'iron casting manufacturing' ceea ce intelegem ca nu este normal :).
