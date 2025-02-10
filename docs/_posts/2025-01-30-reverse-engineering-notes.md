---
title: "Reverse engineering notes"
date: 2025-01-30
---


**Takaisinmallinnuksen Tärkeimmät Käsitteet ja Ominaisuudet**

| **Kategoria**                               | **Käsite**                        | **Selitys**                                                                                                |
| ------------------------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Muistin hallinta**                        | Pino (Stack)                      | Kasvaa alaspäin muistissa ja sisältää funktiokutsujen parametrit, paluuosoitteet ja paikalliset muuttujat. |
|                                             | Kehysosoitin (EBP)                | Käytetään funktioiden paikallisten muuttujien ja argumenttien hallintaan.                                  |
|                                             | Dynaaminen muisti                 | `malloc`, `HeapAlloc` ja muut funktiot varaavat muistia pinon ulkopuolelta.                                |
| **Ohjelman ohjausvirta**                    | Jumps (JMP, JNZ, JE)              | Hypyt ohjelmakoodissa, jotka voivat olla ehdollisia tai ehdottomia.                                        |
|                                             | Call / Ret                        | Funktiokutsut ja paluu niistä, tallentavat paluuosoitteen pinoon.                                          |
| **Keskeytykset ja system callit**           | INT 0x80                          | Vanhempi tapa kutsua Linuxin ydinpalveluita, kuten tiedostojen käsittelyä.                                 |
|                                             | Syscall                           | Moderni tapa suorittaa ydinpalveluita x86-64-arkkitehtuurissa.                                             |
|                                             | Windows API                       | Windows-käyttöjärjestelmässä käytetyt rajapinnat, kuten `CreateFileA`.                                     |
| **Dekompilointi ja disassemblointi**        | IDA Free                          | Suosittu työkalu binäärikoodin analysointiin.                                                              |
|                                             | Ghidra                            | NSA:n julkaisema avoimen lähdekoodin dekompilaattori.                                                     |
|                                             | Radare2                           | Tehokas komentorivipohjainen takaisinmallinnustyökalu.                                                     |
| **Koodin obfuskointi ja suojausmekanismit** | Packers (UPX, Themida)            | Pakkaavat ja salakirjoittavat ohjelmia haittaohjelmien suojaamiseksi.                                      |
|                                             | Anti-debugging                    | Tekniikat, jotka tunnistavat debuggereita (esim. `IsDebuggerPresent`).                                     |
|                                             | Code Injection                    | Koodin suorittaminen toisessa prosessissa, esim. `DLL Injection`.                                          |
| **Salaus ja enkoodaus**                     | XOR-salaus                        | Helppo ja nopea mutta helposti murrettavissa brute-force-menetelmällä.                                     |
|                                             | Base64                            | Tietojen muuntaminen ASCII-muotoon, ei varsinaista salausta.                                               |
|                                             | AES, RSA                          | Vahvat salausalgoritmit, joita käytetään tietojen suojaukseen.                                             |
| **Ohjelmistojen haavoittuvuudet**           | Buffer Overflow                   | Muistin yläkirjoitus, joka voi johtaa suoritusoikeuksien eskalointiin.                                     |
|                                             | Format String Exploit             | Haavoittuvuus, jossa ohjelma käyttää hallitsemattomia muotoiluoperaatioita.                                |
|                                             | Return-Oriented Programming (ROP) | Hyökkäystekniikka, jossa hyökkääjä ohjaa ohjelman suorituksen uudelleen ilman suoraa koodin suorittamista. |

**Rekisterikutsut ja niiden merkitys**

| **Rekisteri** | **Käyttötarkoitus** |
|--------------|----------------|
| **EAX** | Päärekisteri, käytetään usein funktioiden paluuarvona ja aritmeettisissa operaatioissa. |
| **EBX** | Yleiskäyttöinen rekisteri, jota käytetään mm. tietorakenteiden hallintaan. |
| **ECX** | Laskuri, jota käytetään usein toistolauseissa ja silmukoissa. |
| **EDX** | Käytetään laajennettuun aritmetiikkaan ja I/O-toimintoihin. |
| **ESI** | Lähdeosoitin, jota käytetään esimerkiksi merkkijonojen käsittelyssä. |
| **EDI** | Kohdeosoitin, jota käytetään usein tiedonsiirrossa ja kopioinnissa. |
| **EBP** | Kehysosoitin, joka viittaa nykyisen funktion pinoon tallennettuihin arvoihin. |
| **ESP** | Pino-osoitin, joka osoittaa pinoalueen nykyiseen yläpäähän. |
| **EIP** | Ohjelmalaskuri, joka sisältää seuraavan suoritettavan käskyn osoitteen. |
| **EFLAGS** | Tilan rekisteri, joka sisältää tilabitit, kuten nollolippu (ZF) ja kantolippu (CF). |

Tämän taulukon avulla voit hahmottaa takaisinmallinnuksen keskeisimmät aiheet ja tekniikat nopeasti.

## Takaisinmallinnus prosessi

### 1. Prosessorin rekisterit

+ Mitä rekistereitä ohjelma käyttää ja missä tilanteissa?
  +  EAX: Paluuarvot funktiokutsuista.
  +  ECX/EDX: Laskurit ja tietorakenteiden hallinta.
  +  ESP/EBP: Funktiokutsujen ja pinon hallinta.
+ Seuraa, miten rekistereiden arvot muuttuvat ohjelman suorituksen aikana.
+ Tunnista, milloin ohjelma käsittelee syötettä rekistereissä.

### 2. Muistin hallinta

+ Seuraa pino-operaatioita (push, pop, call, ret) – ymmärrä funktioiden sisäänmenot ja poistumiset.
+ Havaitse dynaamisen muistin varaukset (malloc, HeapAlloc jne.) – mistä ohjelma varaa muistia ja miten se sitä käyttää?
+ Etsi puskuriylivuotoja tai muuta muistinkäytön väärinkäyttöä.

### 3. Ohjelman ohjausvirta

+ Tunnista ehdolliset ja ehdottomat hypyt:
    + JNZ, JE, JMP – miten ohjelma tekee päätöksiä?
    + Ohjelman logiikan seuraaminen mahdollistaa esimerkiksi anti-debugging-menetelmien ja piilotettujen toiminnallisuuksien löytämisen.

### 4. Keskeytykset ja järjestelmäkutsut

+ Tarkastele, milloin ohjelma kutsuu käyttöjärjestelmän palveluita.
    + Windows: CreateFileA, VirtualAlloc, WriteProcessMemory → mahdollinen haittaohjelman käyttöjärjestelmän manipulointi.
    + Linux: INT 0x80, syscall → selvitä, mitä järjestelmäpalveluita ohjelma käyttää.
+ Havainnoi, manipuloiko ohjelma tiedostojärjestelmää, verkkoa tai muistia järjestelmäkutsujen kautta.

### 5. Dekompilointi ja disassemblointi

+ Käytä IDA Free/Ghidraa tutkiaksesi ohjelman rakennetta ja toimintaa.
+ Ymmärrä funktiorajat ja -kutsut:
    + Mitkä funktiot liittyvät mahdollisesti haitalliseen toimintaan?
    + Onko ohjelma pakattu tai obfuskoitu?

### 6. Suojausmekanismit ja haittaohjelmien taktiikat

+ Tunnista anti-debugging-tekniikat:
    + IsDebuggerPresent, CheckRemoteDebuggerPresent – yrittääkö ohjelma havaita tutkinnan?
    + Sleep, RDTSC – viivytystekniikat analyysin vaikeuttamiseksi.
+ Tarkastele koodin injektiota ja prosessien manipulointia:
    + CreateRemoteThread, VirtualAllocEx, WriteProcessMemory → mahdollinen prosessin kaappaus.

### 7. Salaus ja enkoodaus

+ Etsi XOR-salausta tai Base64-enkoodausta, joita haittaohjelmat käyttävät piilottamaan toimintansa.
+ Havaitse, missä ja miten ohjelma purkaa tietoa ennen käyttöä.
+ Tarkista, käyttääkö ohjelma kryptografisia funktioita (AES, RC4, MD5 jne.).