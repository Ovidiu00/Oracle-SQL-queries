
```sql
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--h. Numele beneficiarilor care au cumparat cel putin doua tipuri de produse?


SELECT
    nume
FROM
    beneficiar b
WHERE
    (
        SELECT
            COUNT(DISTINCT codp)
        FROM
            tranzactii t
        WHERE
            b.codb = t.codb
    ) >= 2;
    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- k. Numele beneficiarilor care au cereri pentru toate tipurile de produse?

-- se obtine produsele cerute de fiecare beneficiar , apoi acestea sunt comparate folosind minus , minus scoate elementele din primul query din al doilea, 
daca rezultatul este 0 atunci cele 2 query-uri sunt identice


SELECT
    b.nume
FROM
    beneficiar b
WHERE
    (
        SELECT
            COUNT(rezultat_operatie_minus.nume)
        FROM
            (
                ( SELECT DISTINCT
                    p.nume
                FROM
                    produs p
                )
                MINUS
                ( SELECT
                    p.nume
                FROM
                    produs   p
                    INNER JOIN cerere   c ON p.codp = c.codp
                WHERE
                    c.codb = b.codb
                )
            ) rezultat_operatie_minus
    ) = 0


--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- m. Numele beneficiarilor care nu au cereri pentru toate tipurile de produse


SELECT
    b.nume
FROM
    beneficiar b
WHERE
    (
        SELECT
            COUNT(rezultat_operatie_minus.nume)
        FROM
            (
                ( SELECT DISTINCT
                    p.nume
                FROM
                    produs p
                )
                MINUS
                ( SELECT
                    p.nume
                FROM
                    produs   p
                    INNER JOIN cerere   c ON p.codp = c.codp
                WHERE
                    c.codb = b.codb
                )
            ) rezultat_operatie_minus
    ) != 0

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- n. Numele beneficiarilor care au cumparat toate tipurile de produse pentru care au cereri - INCA NU E GATA?!?!?!?!?!?!?!?!?!


--produsele pt care beneifciaru  a facut cerere
select p.nume from produs p
inner join cerere c on c.codp = p.codp
where c.codb = 1;


--produsele cumparate de beneficiar
select p.nume from produs p
inner join tranzactii t on t.codp = p.codp
where t.codb = 1;




--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- 3. Lista perechilor furnizor-benefiicar (codf, nume furnizor, codb si nume beneficiar) intre care nu s-a incheiat nicio tranzactie 
SELECT
    f.nume,
    b.nume
FROM
    furnizor     f
    CROSS JOIN beneficiar   b
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            tranzactii t
        WHERE
            t.codf = f.codf
            AND b.codb = t.codb
    );

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Numele furnizorilor care au ca beneifciari pe toti beneficiarii din bucuresti 

-- logica : returneaza numele furnizorului daca nu  exista nici un beneficiar din bucuresti care sa nu fie beneficiar al furnizorului curent
SELECT
    f.nume
FROM
    furnizor f
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            beneficiar b
        WHERE
            b.oras = 'Zalau'
            AND NOT EXISTS (
                SELECT
                    *
                FROM
                    tranzactii t
                WHERE
                    t.codf = f.codf
                    AND t.codb = b.codb
            )
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
 -- numele furnizorilor care oferta toate tipurile de produse
 
 --logica : afisam furnizorul pentru care nu exista nici un produs care sa nu fie oferit de acesta
SELECT
    f.nume
FROM
    furnizor f
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            produs
        WHERE
            NOT EXISTS (
                SELECT
                    *
                FROM
                    oferta o
                WHERE
                    o.codf = f.codf
                    AND p.codp = o.codp
            )
    )    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Numele beneficiarilor care au cumparat toate tipurile de produse pt care au cereri


--logica : numele benficiarilor pt care nu exista un produs pe care sa l fi cerut si nu l o cumparat
SELECT
    b.nume
FROM
    beneficiar b
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            cerere c
        WHERE
            c.codb = b.codb
            AND NOT EXISTS (
                SELECT
                    *
                FROM
                    tranzactii t
                WHERE
                    t.codb = b.codb
                    AND t.codp = c.codp
            )
    )
    
--4.Sa se stearga, din tabela Oferte, toate ofertele de automobile.

CREATE OR REPLACE VIEW vw_ delete
FROM
    oferte
WHERE
    o.codb IN (
        SELECT
            o.cod
        FROM
            oferte     o
            INNER JOIN produs   p ON p.codp = o.codp
        WHERE
            p.nume = 'ETC'
    )
    
    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--1. Orasele furnizorilor al caror nume incepe cu litera ‚A’

select oras from furnizor 
where oras like 'C%'

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--2. Numele furnizorilor din Cluj-Napoca care vand automobile.

SELECT
    f.nume
FROM
    furnizor   f
    INNER JOIN oferte     o ON o.codf = f.codf
    INNER JOIN produs     p ON p.codp = o.codp
WHERE
    p.nume = 'Automobil'

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--3. Numele beneficiarilor din Bucuresti ordonate alfabetic.

SELECT
    b.nume
FROM
    beneficiar b
WHERE
    b.oras = 'Zalau'
ORDER BY
    b.nume ASC;

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--4. Numele beneficiarilor, orasele lor si produsele cerute de fiecare beneficiar.

SELECT
    b.nume,
    b.oras,
    p.nume
FROM
    beneficiar   b
    INNER JOIN cerere       c ON c.codb = b.codb
    INNER JOIN produs       p ON p.codp = c.codp


--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--Numele furnizorilor din Cluj si numele beneficiarilor din Bucuresti care au incheiat intre ei tranzactii, numele produsului tranzactionat, cantitatea, pretul si data tranzactiei. 
--Ordonare dupa: numele furnizorului, apoi dupa numele beneficiarului, apoi dupa produs, respectiv pret.

SELECT
    *
FROM
    furnizor     f
    INNER JOIN tranzactii   t ON t.codf = f.codf
    INNER JOIN beneficiar   b ON b.codb = t.codb
    INNER JOIN produs       p ON p.codp = t.codp
ORDER BY
    f.nume,
    b.nume,
    p.nume,
    t.pret;

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- Numele furnizorilor din Bucuresti si numarul de oferte de calculatoare pentru fiecare furnizor

SELECT
    f.nume,
    COUNT(f.codf)
FROM
    furnizor   f
    INNER JOIN oferte     o ON o.codf = f.codf
    INNER JOIN produs     p ON p.codp = o.codp
WHERE
    p.nume = 'Paine'
GROUP BY
    f.nume

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  

--2. Numele beneficiarilor din Cluj-Napoca si valoarea medie a tranzactiilor incheiate cu furnizorii din Bucuresti pe anul 2014.

SELECT
    b.nume,
    AVG(t.pret * t.cantitate)
FROM
    furnizor     f
    INNER JOIN tranzactii   t ON t.codf = f.codf
    INNER JOIN beneficiar   b ON t.codb = b.codb
WHERE
    f.oras = 'Cluj'
    AND b.oras = 'Zalau'
    AND EXTRACT(YEAR FROM t.data_tranzactie) = '2021'
GROUP BY
    b.nume

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--3. Numele produselor si pretul minim la care este oferit produsul respectiv, impreuna cu furnizorul care ofera acest produs si orasul acestuia.


CREATE OR REPLACE VIEW vw_produs_pret_minim AS
    SELECT
        p.nume,
        MIN(o.pret) AS pret_minim
    FROM
        produs     p
        INNER JOIN oferte     o ON o.codp = p.codp
        INNER JOIN furnizor   f ON f.codf = o.codf
    GROUP BY
        p.nume
SELECT
    p.nume,
    vw_produs_pret_minim.pret_minim,
    f.nume,
    f.oras
FROM
    vw_produs_pret_minim
    INNER JOIN produs     p ON p.nume = vw_produs_pret_minim.nume
    INNER JOIN oferte     o ON o.codp = p.codp
                           AND o.pret = vw_produs_pret_minim.pret_minim
    INNER JOIN furnizor   f ON o.codf = f.codf
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--4. Numele furnizorului din Cluj-Napoca ce ofera cele mai multe ape.


create or replace view vw_apa_cantitate_maxima as
select p.nume, max(o.cantitate) as cantitate_maxima from furnizor f
inner join oferte o on f.codf = o.codf
inner join produs p on p.codp = o.codp
where p.nume = 'Apa'
group by p.nume


select * from furnizor f
inner join oferte o on f.codf = o.codf 
inner join produs p on p.codp = o.codp
inner join vw_apa_cantitate_maxima on o.cantitate = vw_apa_cantitate_maxima.cantitate_maxima and p.nume = 'Apa'


--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- 5. Numele beneficiarilor si numarul de tranzactii incheiate de fiecare beneficiar, doar pentru beneficiarii care au incheiat cel putin doua tranzactii.

select b.nume,count(b.nume) as nr_tranzactii from beneficiar b
inner join tranzactii t on t.codb = b.codb
group by b.nume
having count(b.nume) > 2
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- Numele beneficiarilor din Bucuresti?

SELECT
    nume,oras
FROM
    beneficiar
WHERE
    nume IN (
        SELECT
            b.nume
        FROM
            beneficiar b
        WHERE
            b.oras = 'Zalau'
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- d. Numele produselor pentru care s-a incheiat cel putin o tranzactie?


SELECT
    nume
FROM
    produs
WHERE
    nume IN (
        SELECT
            p.nume
        FROM
            produs       p
            INNER JOIN tranzactii   t ON t.codp = p.codp
    )


--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--e. Numele beneficiarilor din Bucuresti care au cumparat calculatoare?


SELECT DISTINCT
    b.nume
FROM
    beneficiar   b
    INNER JOIN tranzactii   t ON t.codb = b.codb
    INNER JOIN produs       p ON p.codp = t.codp
WHERE
    p.nume = 'Bomboana'
    
   
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--f. Numele beneficiarilor care au cereri de calculatoare?

SELECT
    b.nume
FROM
    beneficiar   b
    INNER JOIN cerere       c ON c.codb = b.codb
    INNER JOIN produs       p ON p.codp = c.codp
WHERE
    p.nume = 'Bomboana'

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--g. Numele beneficiarilor care au cereri de calculatoare si automobile?


(SELECT
    b.nume
FROM
    beneficiar   b
    INNER JOIN cerere       c ON c.codb = b.codb
    INNER JOIN produs       p ON p.codp = c.codp
WHERE
    p.nume = 'Bomboana')

Intersect

(SELECT
    b.nume
FROM
    beneficiar   b
    INNER JOIN cerere       c ON c.codb = b.codb
    INNER JOIN produs       p ON p.codp = c.codp
WHERE
    p.nume = 'Conserva')

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--i. Numele beneficiarilor care nu au nicio cerere


SELECT
    b.nume
FROM
    beneficiar b
WHERE
    b.nume NOT IN (
        SELECT
            b1.nume
        FROM
            beneficiar b1
            INNER JOIN cerere c ON c.codb = b1.codb
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--j. Numele beneficiarilor care nu au cereri de paine, nici apa


( SELECT
    b.nume
FROM
    beneficiar b
WHERE
    b.nume NOT IN (
        SELECT
            b1.nume
        FROM
            beneficiar   b1
            INNER JOIN cerere       c ON c.codb = b1.codb
            INNER JOIN produs       p ON p.codp = c.codp
        WHERE
            p.nume = 'Apa'
    )
)
INTERSECT
( SELECT
    b.nume
FROM
    beneficiar b
WHERE
    b.nume NOT IN (
        SELECT
            b1.nume
        FROM
            beneficiar   b1
            INNER JOIN cerere       c ON c.codb = b1.codb
            INNER JOIN produs       p ON p.codp = c.codp
        WHERE
            p.nume = 'Paine'
    )
)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- a. Numele produsului si numarul de cereri pentru fiecare produs.

SELECT
    p.nume,
    COUNT(p.nume)
FROM
    produs   p
    INNER JOIN cerere   c ON p.codp = c.codp
GROUP BY
    p.nume

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
--b. Numele produsului si valoarea totala a tranzactiilor incheiate pentru acel produs.

SELECT
    p.nume,
    SUM(t.pret * t.cantitate)
FROM
    produs       p
    INNER JOIN tranzactii   t ON p.codp = t.codp
GROUP BY
    p.nume

--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
-- c. Numele produsului si valoarea maxima a tranzactiilor incheiate pentru fiecare produs, precum si numele furnizorului, 
respectiv al beneficiarului care au incheiat tranzactia respectiva


CREATE OR REPLACE VIEW vw_produs_pretmaxim AS
    SELECT
        p.nume,
        MAX(t.pret) AS pret_maxim
    FROM
        produs       p
        INNER JOIN tranzactii   t ON t.codp = p.codp
    GROUP BY
        p.nume

/
 SELECT
      p.nume,
      t.pret,
      f.nume   AS nume_furnizor,
      b.nume   AS nume_beneficiar
  FROM
      produs       p
      INNER JOIN tranzactii   t ON t.codp = p.codp
      INNER JOIN produs       p ON p.codp = t.codp
      INNER JOIN furnizor     f ON t.codf = f.codf
      INNER JOIN beneficiar   b ON t.codb = b.codb
      INNER JOIN vw_produs_pretmaxim ON vw_produs_pretmaxim.nume = p.nume
                                        AND vw_produs_pretmaxim.pret_maxim = t.pret    
    

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
-- e. Numele furnizorilor si denumirile produselor oferite de fiecare furnizor. Pentru furnizorii fara produse se va trece „Fara produse oferite”.


SELECT
    f.nume,
    nvl(p.nume, 'Fara produse oferite')
FROM
    furnizor   f
    LEFT JOIN oferte     o ON o.codf = f.codf
    LEFT JOIN produs     p ON p.codp = o.codp

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--f. Numele furnizorilor si numele beneficiarilor care au incheiat tranzactii cu furnizorii mentionati. Pentru beneficiarii fara tranzactii se va afisa 
--„Fara tranzactii” in loc de numele furnizorului.


SELECT
    b.nume,
    nvl(f.nume, 'Fara tranzactii')
FROM
    beneficiar   b
    LEFT JOIN tranzactii   t ON t.codb = b.codb
    LEFT JOIN furnizor     f ON f.codf = t.codf

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--g. Numele produselor si valoarea minima a ofertelor pentru fiecare produs. Pentru produsele fara oferte se va afisa valoarea 0.


SELECT
    p.nume,
    nvl(MIN(o.pret), 0) AS pret_minim
FROM
    produs   p
    LEFT JOIN oferte   o ON o.codp = p.codp
GROUP BY
    p.nume
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--orasele beneficiarilor care nu sunt orase ale furnizorilor

SELECT DISTINCT
    oras
FROM
    beneficiar
MINUS
SELECT DISTINCT
    oras
FROM
    furnizor

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- furnizorii care au cel putin o oferta

SELECT
    f.nume
FROM
    furnizor f
WHERE
    EXISTS (
        SELECT
            *
        FROM
            oferte o
        WHERE
            o.codf = f.codf
    );
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- numele produselor pt care s a incheiat cel putin o tranzactie

SELECT
    p.nume
FROM
    produs p
WHERE
    EXISTS (
        SELECT
            *
        FROM
            tranzactii t
        WHERE
            t.codp = p.codp
    )

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--numele beneficiarilor care nu au cereri de Paine si nici de Conserva

CREATE OR REPLACE VIEW vw_cereri_calculator AS
    SELECT DISTINCT
        b.nume
    FROM
        beneficiar   b
        INNER JOIN cerere       c ON b.codb = c.codb
        INNER JOIN produs       p ON p.codp = c.codp
    WHERE
        p.nume = 'Paine'
/

CREATE OR REPLACE VIEW vw_cereri_conserva AS
    SELECT DISTINCT
        b.nume
    FROM
        beneficiar   b
        INNER JOIN cerere       c ON b.codb = c.codb
        INNER JOIN produs       p ON p.codp = c.codp
    WHERE
        p.nume = 'Conserva'
/

CREATE OR REPLACE VIEW vw_beneficiari_fara_paine AS
    SELECT
        b.nume
    FROM
        beneficiar b
    WHERE
        b.nume NOT IN (
            SELECT
                *
            FROM
                vw_cereri_calculator
        );

CREATE OR REPLACE VIEW vw_beneficiari_fara_paine AS
    SELECT
        b.nume
    FROM
        beneficiar b
    WHERE
        b.nume NOT IN (
            SELECT
                *
            FROM
                vw_cereri_conserva
        );

SELECT
    nume
FROM
    (
        SELECT
            nume
        FROM
            vw_beneficiari_fara_paine
        INTERSECT
        SELECT
            nume
        FROM
            vw_cereri_conserva
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- numele beneficiarilor care au cumparat cel putin  2 produse distincte

SELECT
    *
FROM
    beneficiar b
WHERE
    (
        SELECT
            COUNT(*)
        FROM
            (
                SELECT DISTINCT
                    *
                FROM
                    tranzactii t
                WHERE
                    t.codb = b.codb
            )
    ) >= 2

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- furnizorii care nu au nici o oferta	

SELECT
    f.nume
FROM
    furnizor f
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            oferte o
        WHERE
            o.codf = f.codf
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- numele beneficiarilor din cluj si valoarea medie a tranzactiilor incheiate cu furnizorii din bucuresti pe anul 2014

SELECT
    b.nume,
    AVG(t.pret * t.cantitate)
FROM
    beneficiar   b
    INNER JOIN tranzactii   t ON t.codb = b.codb
    INNER JOIN furnizor     f ON f.codf = t.codf
WHERE
    EXTRACT(YEAR FROM t.data_tranzactie) = 2021
GROUP BY
    b.nume
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- numele produselor si pretul minim la care este oferit produsul respectiv,impreuna cu furnizorul care ofera acest produs si orasul acestuia

CREATE OR REPLACE VIEW vw_produs_pret_minim AS
    SELECT
        p.nume,
        MIN(o.pret) AS pret_minim
    FROM
        produs     p
        INNER JOIN oferte     o ON o.codp = p.codp
        INNER JOIN furnizor   f ON f.codf = o.codf
    GROUP BY
        p.nume

/ SELECT
      p.nume,
      f.nume,
      f.oras,
      vw_produs_pret_minim.pret_minim
  FROM
      produs     p
      INNER JOIN oferte     o ON o.codp = p.codp
      INNER JOIN furnizor   f ON f.codf = o.codf
      INNER JOIN vw_produs_pret_minim ON vw_produs_pret_minim.nume = p.nume
                                         AND vw_produs_pret_minim.pret_minim = o.pret

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--numele furnizorului din cluj napoca ce ofera cele mai multe biciclete
SELECT
    *
FROM
    (
        SELECT
            o.cantitate,
            f.codf,
            f.nume
        FROM
            furnizor   f
            INNER JOIN oferte     o ON f.codf = o.codf
            INNER JOIN produs     p ON p.codp = o.codp
        WHERE
            p.nume = 'Apa'
        ORDER BY
            o.cantitate DESC
    )
WHERE
    ROWNUM = 1;

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Pentru fiecare tranzactie sa se afiseze codt,codp,nume prod,pret ,pret-min-pe-produs tranzactionat
SELECT
    t.codt,
    p.codp,
    p.nume,
    t.pret,
    vw_produs_pret_minim.pret_minim
FROM
    tranzactii   t
    INNER JOIN produs       p ON p.codp = t.codp
    INNER JOIN vw_produs_pret_minim ON vw_produs_pret_minim.nume = p.nume


create or replace view vw_produs_pret_minim
    AS
        SELECT
            p.nume,
            MIN(t.pret) AS pret_minim
        FROM
            produs       p
            INNER JOIN tranzactii   t ON t.codp = p.codp
        GROUP BY
            p.nume
--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
--a. Numele furnizorilor si media valorilor ofertelor pentru fiecare furnizor.

CREATE OR REPLACE VIEW vw_numefurnizor_mediaoferta AS
    SELECT
        f.nume,
        AVG(o.pret) AS medie_pret
    FROM
        furnizor   f
        INNER JOIN oferte     o ON f.codf = o.codf
    GROUP BY
        f.nume

--------------------------------------------------------------------------------------------------------------------------------------------------------------------       
-- b. Numele beneficiarilor din Oradea care au cumparat Paine, dar nu au cumparat Bomboana.
SELECT
    b.nume
FROM
    beneficiar   b
    INNER JOIN tranzactii   t ON t.codb = b.codb
    INNER JOIN produs       p ON p.codp = t.codp
WHERE
    p.nume = 'Paine'
INTERSECT
SELECT
    vw_persoane_care_nu_au_cumparat_bomboana.nume
FROM
    vw_persoane_care_nu_au_cumparat_bomboana



create or replace view vw_persoane_care_nu_au_cumparat_bomboana as
select b.nume from beneficiar b
where b.nume not in (select vw_persoane_care_au_cumparat_bomboana.nume from vw_persoane_care_au_cumparat_bomboana)




create or replace view vw_persoane_care_au_cumparat_bomboana
    AS
        SELECT
            b.nume
        FROM
            beneficiar   b
            INNER JOIN tranzactii   t ON t.codb = b.codb
            INNER JOIN produs       p ON p.codp = t.codp
        WHERE
            p.nume = 'Bomboana'
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Sa se afiseze pentru fiecare oferta care ar fi cea mai buna cerere - adica cererea cu pret maxim
SELECT
    *
FROM
    oferte   o
    INNER JOIN produs   p ON p.codp = o.codp
    INNER JOIN vw_produs_cerere_maxima ON p.nume = vw_produs_cerere_maxima.nume


--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Pentru fiecare beneficiar sa se afiseze numarul de produse ofertate de furnizori care sunt din acelasi oras cu beneficiarul

SELECT
    *
FROM
    beneficiar b
    LEFT JOIN offer ON b.oras = offer.oras


create or replace view offer
    AS
        SELECT
            COUNT(*) AS nr,
            f.oras
        FROM
            oferte     o
            INNER JOIN furnizor   f ON f.codf = o.codf
        GROUP BY
            f.oras
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Generare de intervale de 30 min!


WITH intervale AS (
    SELECT
        trunc(sysdate) + ( level * 30 ) / ( 24 * 60 ) c_time
    FROM
        dual
    CONNECT BY
        level <= ( 24 * 60 ) / 30
)
SELECT
    to_char(c_time, 'hh24:mi') starttime,
    to_char(c_time + 30 /(24 * 60), 'hh24:mi') endtime
FROM
    intervale

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--13 numele beneficiarilor care nu cumpara paine

CREATE OR REPLACE VIEW vw_beneficiari_paine AS
    SELECT DISTINCT
        b.nume
    FROM
        beneficiar   b
        INNER JOIN cerere       c ON c.codb = b.codb
        INNER JOIN produs       p ON p.codp = c.codp
    WHERE
        p.nume = 'Paine'
/

SELECT
    b.nume
FROM
    beneficiar b
WHERE
    b.nume NOT IN (
        SELECT
            *
        FROM
            vw_beneficiari_paine
    )
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--15 Numele beneficiarilor  care au furnizori doar in Cluj - sergiu si ovidiu

SELECT DISTINCT
    b.nume
FROM
    beneficiar   b
    INNER JOIN tranzactii   t ON b.codb = t.codb
    INNER JOIN furnizor     f ON t.codf = f.codf
WHERE
    f.oras = 'Cluj'
    AND b.nume NOT IN (
        SELECT
            b.nume
        FROM
            beneficiar   b
            INNER JOIN tranzactii   t ON b.codb = t.codb
            INNER JOIN furnizor     f ON t.codf = f.codf
        WHERE
            f.oras != 'Cluj'
    )

--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
-- Lista perechiilor furnizor - beneficiar care au incheiat cel putin 2 tranzactii

SELECT
    f.nume,
    b.nume
FROM
    furnizor     f
    INNER JOIN tranzactii   t ON f.codf = t.codf
    INNER JOIN beneficiar   b ON t.codb = b.codb
GROUP BY
    f.nume,
    b.nume
HAVING
    COUNT(f.nume) > 1
--------------------------------------------------------------------------------------------------------------------------------------------------------------------   
--1. Numele furnizorilor si numarul de tranzactii incheiate de fiecare furnizor cu beneficiarii din Bucuresti. Pentru furnizorii fara asemenea tranzactii se va trece 0.

SELECT
    f.nume,
    COUNT(t.codt) AS nr_tranzactii
FROM
    furnizor     f
    LEFT JOIN tranzactii   t ON t.codf = f.codf
    LEFT JOIN beneficiar   b ON b.codb = t.codb
WHERE
    b.oras = 'Zalau'
    OR b.oras IS NULL
GROUP BY
    f.nume
    
    ```
