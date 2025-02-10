---
title: "Firewall notes"
date: 2025-02-10
---


# 🔥 Palomuurin Perusteet – Muistilista

## 🛡️ 1. Mikä on palomuuri?
- Palomuuri on järjestelmä, joka **säätelee verkkoliikennettä** ennalta määritettyjen sääntöjen perusteella.
- Se voi olla **ohjelmistopohjainen (esim. iptables, Windows Firewall)** tai **laitteistopohjainen (esim. Cisco ASA, FortiGate).**
- Estää **luvattoman pääsyn** ja sallii vain sallitun liikenteen.

---

## 📜 2. Palomuurin säännöt ja toimintaperiaate  
Palomuuri käyttää sääntöjä liikenteen hallintaan. Jokaisella säännöllä on:  
✅ **Lähde-IP:** Mistä liikenne tulee?  
✅ **Kohde-IP:** Minne liikenne on menossa?  
✅ **Protokolla:** TCP, UDP, ICMP jne.  
✅ **Portti:** Mihin palveluun liikenne liittyy? (esim. HTTP → portti 80)  
✅ **Toiminto:** Sallitaanko vai estetäänkö liikenne?  

💡 **Sääntöjen prioriteetti:**  
- Säännöt käydään läpi ylhäältä alas. Ensimmäinen osuva sääntö määrittää kohtalon.  
- Monissa järjestelmissä **viimeisenä sääntönä on "DENY ALL"**, joka estää kaiken muun liikenteen.  

---

## 🔄 3. Liikenteen suunnat palomuurissa  
💬 **Mistä liikenne tulee ja minne se menee?**  

| 🔄 Liikenne | 🔍 Kuvaus | ✅ Sallitut | ❌ Estetyt |
|------------|----------|------------|------------|
| **Ingress (Sisääntuleva)** | Internetistä sisäverkkoon tuleva liikenne | Tarvittavat palvelut (esim. verkkosivut, VPN) | Suurin osa ulkopuolisesta liikenteestä |
| **Egress (Lähtevä)** | Sisäverkosta Internetiin lähtevä liikenne | Useimmat lähtevät yhteydet (esim. verkkoselailu) | Tietyt ulospäin menevät yhteydet (esim. haittaohjelmat, P2P) |
| **Lateral (Sisäinen)** | Laitteiden välinen liikenne samassa verkossa | Organisaation sisäverkon sallitut yhteydet | Epäilyttävä liikenne, joka yrittää levitä |

---

## 🔗 4. Esimerkki säännöistä  
📌 **Perusasetukset palomuurille:**  

```sh
ALLOW   TCP   192.168.1.0/24 → 443 (Internet)   # Salli HTTPS-selaus
ALLOW   TCP   192.168.1.10 → 22 (SSH)           # Salli SSH-yhteys palvelimelle
DENY    ALL   0.0.0.0/0 → 3389 (RDP)            # Estä etätyöpöytä Internetistä
DENY    ALL   0.0.0.0/0 → 0.0.0.0/0             # Estä kaikki muu liikenne
```

---

## 🔢 5. Yleisimmät portit palomuurisäännöissä

### Tunnetut portit ja palvelut:
| Portti | Protokolla | Käyttötarkoitus |
|------------|----------|------------|
| 80 | TCP | HTTP (Web) |
| 443 | TCP	| HTTPS (Suojattu Web) |
| 22 | TCP | SSH (Etäyhteys) |
| 53 | UDP/TCP | DNS (Nimipalvelu) |
| 3389 | TCP | RDP (Etätyöpöytä) |
| 3306 | TCP | MySQL (Tietokanta) |

---

## 📌 6. Tärkeimmät muistettavat asiat

✔ Oletussääntö: Estä kaikki, salli vain tarvittavat.
✔ Kirjaa ja analysoi liikennettä (logit auttavat tunnistamaan hyökkäykset).
✔ Käytä monitasoista suojausta (palomuuri + IDS/IPS + VPN).
✔ Päivitä säännöt ja tarkista asetukset säännöllisesti!
