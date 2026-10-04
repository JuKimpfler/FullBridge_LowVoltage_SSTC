# SSTC 2 – Vollbrücke 48 V DC

### Aufbaudokumentation & Bauteileliste

Basierend auf Guangyans SSTC 2 (loneoceans.com/labs/sstc2)\
Modifiziert: Vollbrücke · 48–60 V DC · Arduino-Interrupter · LWL-Strecke (TOTX173/TORX173) · UCC27424DR

---

## ⚠️ Sicherheitshinweise

> **Die Sekundärseite eines Tesla Coils erzeugt Hochspannung und starke HF-Felder.**\
> Auch bei 48 V Einspeisung sind die Ausgangsspannungen der Sekundärspule gefährlich.\
> Schrittmacher- und Implantatträger: **Mindestabstand 3 m**.\
> Elektronik in der Nähe kann durch HF-Einstrahlung beschädigt werden.\
> Nie alleine arbeiten. Kondensatoren vor Arbeiten am Gerät entladen.

---

## 1. Systemübersicht

```
[PC / USB]
    │
[Arduino]  ──►  [TOTX173]  ═══LWL═══  [TORX173]  ──►  [74HC14]  ──►  [UCC27424DR]
                                                              │
                                                    [Feedback-CT, 50 Wdg.]
                                                              │
                                                      [Gate-Trafo GDT]
                                                      Primär: 8 Wdg.
                                                      4× Sekundär: 8 Wdg.
                                                              │
                                          ┌───────────────────┴───────────────────┐
                                     [Q1 IRF540]                           [Q3 IRF540]
                                     [Q2 IRF540]                           [Q4 IRF540]
                                     Halbbrücke A                         Halbbrücke B
                                          └───────────┬──────────────────────┘
                                                 [Primär 6–7 Wdg., 14 AWG]
                                                 [4,7 µF Folien-C in Serie]
                                                      │
                                              [Sekundärspule]
                                              975 Wdg., AWG 34
                                              3,5" × 6,25"
                                                      │
                                                  [Toroid]
                                                8" × 2" Alu
```

**Taktprinzip Vollbrücke:**

- Takt 1: Q1 + Q4 leiten gleichzeitig
- Takt 2: Q2 + Q3 leiten gleichzeitig
- Q1/Q4 und Q2/Q3 dürfen **niemals** gleichzeitig leiten → Kurzschluss

---

## 2. Versorgungskonzept (3 getrennte Netzteile)

| Schiene | Spannung | Strom | Verwendung |
| --- | --- | --- | --- |
| NT1 | 5 V DC | ≥ 1 A | 74HC14, TORX173, Arduino (optional) |
| NT2 | 12 V DC | ≥ 1 A | UCC27424DR, UVLO-Schaltung |
| NT3 | 48–60 V DC | ≥ 5 A | Leistungsbrücke, Bus-Kondensatoren |

**Wichtige Regeln:**

- 5 V und 12 V müssen **gemeinsame Masse** haben (Logik-GND)
- Alle drei Massen nur an **einem einzigen Punkt** mit Schutzleiter verbinden
- Einschaltreihenfolge: erst NT1 + NT2, dann NT3
- Ausschalten: erst NT3, dann NT1 + NT2
- Am Eingang von NT1/NT2 je eine Ferritdrossel + 100 nF Keramikkondensator gegen HF

---

## 3. Bauteileliste

### 3.1 Leistungsbrücke

| Bauteil | Wert / Typ | Anzahl | Hinweise |
| --- | --- | --- | --- |
| Q1–Q4 | IRF540N (100 V / 33 A) | 4 | TO-220, isoliert auf Kühlkörper montieren |
| C-Bus | 4700 µF / 100 V Elko | 1–2 | Ripplestrom-Rating beachten, so nah wie möglich an Brücke |
| C-Snubber | 100 nF / 250 V Folie | 4 | Direkt Gate-Source je MOSFET |
| C-Prim | 4,7 µF / 250 V MKP Folie | 1 | DC-Blocking, in Serie mit Primärspule |
| TVS Gate | 16 V Zener P4SMAJ16A | 8 | Zwischen Gate und Source je MOSFET ( in beide Richtungen (bidirektional) |
| R-Gate | 10 kΩ | 4 | Gate nach Source, Pulldown |
| R-Gate-Serie | 6,8 Ω | 4 | In Serie Gate-Leitung vom GDT |
| D-Gate | 1N4148 | 4 | Parallel zu R-Gate-Serie, Kathode Richtung Gate |
| Sicherung | 5 A Träge | 1 | In 48-V-Zuleitung |
| Kühlkörper | ≥ 10 K/W | 1–2 | Alle 4 MOSFETs, Glimmerscheiben + Isolierbuchsen |

### 3.2 Gate-Treiber und Logik

| Bauteil | Wert / Typ | Anzahl | Hinweise |
| --- | --- | --- | --- |
| IC1 | UCC27424DR (SOIC-8) | 1 | Dual non-inverting, 4 A. Kanal A am GDT verpolt! |
| IC2 | 74HC14 (DIP-14 oder SOIC) | 1 | Hex Schmitt-Trigger-Inverter |
| C1 | 10 µF Keramik / 25 V | 1 | Direkt an VDD/GND des UCC |
| C2 | 100 nF Keramik | 2 | Je einer an VDD des UCC und der 74HC14 |
| R1 | 5,1 kΩ | 1 | Enable-Eingang UCC nach GND (Pulldown) |
| R2 | 1 kΩ | 1 | Feedback-Signal nach 74HC14 |
| C3 | 0,1 nF | 1 | Filter am Feedback-Eingang |
| D1 | 1N4148 | 2 | Feedback-Clamp an 5 V und GND |

### 3.3 Unterspannungsabschaltung (UVLO)

| Bauteil | Wert / Typ | Anzahl | Hinweise |
| --- | --- | --- | --- |
| IC3 | DS1233D-5+ (SOT-23) | 1 | Überwacht 12-V-Schiene |
| R3 | 470 Ω | 1 | Spannungsteiler oben |
| R4 | 560 Ω | 1 | Spannungsteiler unten |
| D2 | 1N4007 | 1 | Schutzdiode |

### 3.4 LWL-Strecke

| Bauteil | Wert / Typ | Anzahl | Hinweise |
| --- | --- | --- | --- |
| U1 | TOTX173 | 1 | Sender, Bedienseite (beim Arduino) |
| U2 | TORX173 | 1 | Empfänger, Coil-Seite |
| R5 | 330 Ω | 1 | In Serie zur LED des TOTX173 |
| C4 | 100 nF | 2 | Je einer an Vcc beider Module |
| Plastikfaser | Ø 1 mm, bis 10 m | 1 | Standard-POF-Kabel, F07 oder kompatibel |

**Wichtig:** Ruhezustand (kein Licht) muss am UCC-Enable **Low** ergeben.\
Prüfe Polarität des TORX173-Ausgangs mit Oszilloskop. Falls invertiert, ein freies Gatter des 74HC14 zwischenschalten.

### 3.5 Arduino-Interrupter

| Bauteil | Wert / Typ | Anzahl | Hinweise |
| --- | --- | --- | --- |
| Arduino | Uno / Nano | 1 | Versorgung über USB vom PC |
| Poti 1 | 10 kΩ linear | 1 | Frequenz (1–500 Hz) |
| Poti 2 | 10 kΩ linear | 1 | Pulsbreite (0–1500 µs) |
| R6 | 10 kΩ | 1 | Pulldown am Ausgangspin |

**Software-Limits im Sketch (zwingend einhalten):**

- Max. Pulsbreite: 1500 µs
- Max. Tastverhältnis: 15 %
- Ruhezustand bei Reset/Boot: Low (kein Puls)

---

## 4. Spulen und Trafos

### 4.1 Sekundärspule

| Parameter | Wert |
| --- | --- |
| Rohr | PVC, Außendurchmesser 3,5" (89 mm), Länge ca. 200 mm |
| Draht | Lackdraht (Kupfer-Magnet-Draht) AWG 34 (Ø 0,16 mm) |
| Windungen | ca. 975 (bei 98 % Füllung über 6,25") |
| Wickellänge | 159 mm (6,25") |
| Richtung | egal, aber konsistent zur Feedback-CT-Phasenlage |
| Isolation | 3–5 Schichten Polyurethan-Klarlack (Bootslack oder ähnlich) |
| Abschluss | Beide Enden auf Kupferstreifen anlöten, auf Endkappe aus Acryl |

**Wickeltipp:** Langsam und gleichmäßig wickeln, jede Lage soll flach anliegen.\
Keine Überlappungen. Windung für Windung unter leichter Spannung führen.\
Nach dem Wickeln sofort lackieren, damit nichts verrutscht.

**Resonanzfrequenz (Zielwert):** ca. 250 kHz mit 8" × 2"-Toroid.

---

### 4.2 Primärspule

| Parameter | Wert |
| --- | --- |
| Draht | Kupfer, AWG 14 (Ø 1,6 mm), Volldraht oder Litze |
| Windungen | 6–7 |
| Durchmesser | ca. 4,5–5" (115–127 mm), etwas größer als Sekundäre |
| Wickelform | Zylindrisch, direkt unter dem Beginn der Sekundärwicklung |
| Abstand | Mindestens 5 mm Luftspalt zur Sekundärspule (PVC-Abstandshalter) |
| Kopplung k | Zielwert ca. 0,20–0,25 |

**Hinweis:** Zu hohe Kopplung (k > 0,30) führt zu Racing Sparks (Überschläge) auf der Sekundärspule.\
Lieber eine Windung weniger und den Abstand etwas größer wählen.\
DC-Blocking-Kondensator (4,7 µF MKP) muss **in Serie** in die Primärleitung.

---

### 4.3 Gate-Drive-Trafo (GDT)

Der GDT überträgt die Schaltsignale des UCC27424DR galvanisch getrennt auf alle vier MOSFET-Gates.

**Kernauswahl:**

- Material: Ferrit, Typ N27, N30 oder äquivalent (kein Eisenpulver, kein Baumarkt-Ringkern!)
- Form: Ringkern (Toroid), Außendurchmesser 20–30 mm
- Empfehlung: Epcos/TDK B64290L0618X830 (N30, 25 mm) oder ähnlich
- Test vor dem Wickeln: Signalgenerator 250 kHz Rechteck anlegen, Ausgänge mit Oszilloskop prüfen – muss sauberes Rechteck liefern

**Wicklung:**

| Wicklung | Windungen | Draht | Anschluss | Phasenlage |
| --- | --- | --- | --- | --- |
| Primär | 8 | AWG 26 (0,4 mm) | UCC27424 Ausg. A + B | – |
| Sek. 1 (Q1) | 8 | AWG 26 | Gate/Source Q1 | normal (gleich wie Kanal B) |
| Sek. 2 (Q2) | 8 | AWG 26 | Gate/Source Q2 | **verpolt** (Anfang/Ende tauschen) |
| Sek. 3 (Q3) | 8 | AWG 26 | Gate/Source Q3 | **verpolt** (wie Sek. 2) |
| Sek. 4 (Q4) | 8 | AWG 26 | Gate/Source Q4 | normal (wie Sek. 1) |

**Phasenprinzip (da UCC27424 dual non-inverting):**

```
UCC Kanal A ──► Sek. 1 (normal)   ──► Q1 (oben, Seite A)  ┐ Takt 1
               Sek. 4 (normal)   ──► Q4 (unten, Seite B)  ┘

UCC Kanal B ──► Sek. 2 (verpolt) ──► Q2 (unten, Seite A)  ┐ Takt 2
               Sek. 3 (verpolt) ──► Q3 (oben, Seite B)    ┘
```

**Wickeltechnik:**

1. Alle 5 Wicklungen gemeinsam auf den Kern wickeln (bifilar/multifilar) – das verbessert die Kopplung
2. Oder: zuerst Primär, dann jede Sekundär einzeln, gleichmäßig über den Kern verteilt
3. Zwischen Primär und Sekundär: Lage Tesafilm oder Kaptonband zur Isolation
4. Drahtenden beschriften bevor man den Kern vom Tisch nimmt
5. **Abschlusskontrolle am Oszilloskop:** alle vier Gate-Signale prüfen, Gegenphasigkeit verifizieren, kein Überschwingen über 18 V an Gate-Source

---

### 4.4 Feedback-Stromtrafo (CT)

Der Feedback-CT sitzt am Fußpunkt der Sekundärspule und liefert das Selbstoszillations-Signal.

**Kernauswahl:**

- Kleiner Ferritringkern, Außendurchmesser 10–15 mm
- Material: N30 oder ähnlich (nicht Eisenpulver)
- Empfehlung: Epcos B64290L0004X830 oder beliebiger kleiner Ferrit-Ringkern

**Wicklung:**

| Parameter | Wert |
| --- | --- |
| Sekundärwindungen | 50 Windungen AWG 28–30 (Ø 0,25–0,32 mm) |
| Primär | 1 Windung = der Massedraht der Sekundärspule geht einmal durch den Ring |
| Übersetzung | 50:1 |

**Montage:**

- Den Massedraht der Sekundärspule durch die Öffnung des Ringkerns führen – das ist die Primärwicklung
- 50 Windungen gleichmäßig auf den Kern wickeln
- Zuleitungen verdrillen und abschirmen (Koaxialkabel oder verdrilltes Paar in Alufolie)
- Schirmung nur einseitig erden (Coil-Masse), sonst Masseschleife

**Phasing (Phasenlage):**

- Falls der Coil nicht anschwingt, die beiden Drähte der CT-Sekundärwicklung vertauschen
- Clamp-Dioden (2× 1N4148) begrenzen das Signal auf 0 V und +5 V vor dem 74HC14-Eingang

---

## 5. Aufbaureihenfolge

1. **Sekundärspule wickeln und lackieren** – Trocknungszeit einplanen (24–48 h)
2. **Feedback-CT wickeln**, am Sekundär-Fußpunkt montieren
3. **Gate-Trafo (GDT) wickeln und testen** (mit Signalgenerator, vor dem Einbau)
4. **Logikplatine aufbauen** (74HC14, UCC27424, UVLO, LWL-Empfänger)
5. **Leistungsbrücke aufbauen** (4× IRF540N auf Kühlkörper, Bus-C, Snubber)
6. **Arduino-Interrupter programmieren**, mit TOTX173 verbinden
7. **Erst-Test mit strombegrenztem Netzteil** (max. 1 A Strombegrenzung, 12 V)
8. **Gate-Signale mit Oszilloskop prüfen** – kein Überlapp, kein Überschwingen
9. **Spannungstest in Stufen**: 12 V → 24 V → 36 V → 48 V, je mit kurzen Pulsen
10. **Primärspule anpassen** (Windungszahl, Abstand) bis stabile Funken entstehen

---

## 6. Test und Inbetriebnahme

### Checkliste vor dem ersten Einschalten

- [ ] Alle vier Gate-Source-Pfade mit Ohmmeter auf Kurzschluss prüfen
- [ ] Bus-Kondensatoren korrekt gepolt
- [ ] Sicherung in 48-V-Leitung eingebaut
- [ ] UVLO aktiv (UCC bleibt beim Hochfahren gesperrt bis 12 V stabil)
- [ ] Pulldown-Widerstände am Enable-Eingang des UCC
- [ ] Arduino-Code: Pulsbreite max. 1500 µs, Tastverhältnis max. 15 %
- [ ] Sekundärspule mit Breakout-Punkt (angespitzte Schraube oben)
- [ ] Massedraht Sekundärspule mit Gehäuse-GND verbunden
- [ ] Kühlkörper-Isolierung aller vier MOSFETs geprüft (kein Kurzschluss zur Montagefläche)

### Erwartete Ergebnisse bei 48 V Vollbrücke

| Parameter | Erwarteter Wert |
| --- | --- |
| Resonanzfrequenz | ca. 240–260 kHz |
| Primärstrom (Spitze) | ca. 3–6 A |
| Funkenlänge | ca. 2–5 cm |
| Lautstärke Musikmodulation | gut hörbar im Raum, ca. 1–3 m |
| Gate-Spannung | 10–14 V (durch GDT-Verhältnis 1:1) |

---

## 7. Häufige Fehler und Lösung

| Problem | Mögliche Ursache | Lösung |
| --- | --- | --- |
| Coil schwingt nicht an | Feedback-CT falsch gepolt | CT-Sekundärdrähte tauschen |
| MOSFET stirbt sofort | Shoot-through (Querleitung) | Gate-Signale mit Oszi prüfen, GDT-Phasenlage korrigieren |
| Racing Sparks auf Sekundär | Kopplung zu hoch | Primärspule weiter nach außen, eine Windung weniger |
| Kein Signal am Coil, Treiber heiß | UCC Enable liegt High im Ruhezustand | Pulldown an Enable prüfen, TORX173-Polarität prüfen |
| Unregelmäßige Funken / Aussetzer | Masseschleife in Feedback-Leitung | Feedback-Draht abschirmen, Massepunkt prüfen |
| Gate-Überschwingen > 18 V | Zu hohe Leitungsinduktivität | Leitungen kürzen, TVS-Dioden an Gate-Source |

---

## 8. Quellenangaben und weiterführende Links

- Originalbeschreibung SSTC 2: [https://www.loneoceans.com/labs/sstc2/](https://www.loneoceans.com/labs/sstc2/)
- UCC27424 Datenblatt: [https://www.ti.com/product/UCC27424](https://www.ti.com/product/UCC27424)
- IRF540N Datenblatt: [https://www.infineon.com/dgdl/irf540n.pdf](https://www.infineon.com/dgdl/irf540n.pdf)
- JavaTC (Spulenrechner): [http://www.classictesla.com/java/javatc/javatc.html](http://www.classictesla.com/java/javatc/javatc.html)
- Hochspannungsforum (deutsch): [https://www.hobbyelektronik.org](https://www.hobbyelektronik.org)

---

*Dokument-Version 1.0 – Stand: Oktober 2026*\
*Basierend auf dem SSTC 2 Design von Gao Guangyan (loneoceans.com)*
