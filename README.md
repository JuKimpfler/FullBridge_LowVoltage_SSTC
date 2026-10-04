# SSTC 2 – Vollbrücke 48 V DC

### Aufbaudokumentation & Bauteileliste · Version 1.3

Basierend auf Guangyans SSTC 2 (loneoceans.com/labs/sstc2)\
Modifiziert: Vollbrücke · 48 V DC (max. 55 V) · Arduino-Interrupter · LWL (TOTX173/TORX173) · UCC27424DR · IRF540N

> **Status:** Dies ist eine *Ableitung* des Originals und wurde nicht gebaut oder getestet.\
> Werte mit „(Schätzung)“ sind Hochrechnungen aus den Messwerten des Originalautors.\
> Alle Gate-Signale, Phasenlagen und Spannungsspitzen **müssen** vor dem Leistungsbetrieb mit einem Oszilloskop geprüft werden.

---

## 0. Was sich gegenüber dem Original ändert

| Punkt | Original (SSTC 2) | Dieser Aufbau |
| --- | --- | --- |
| Bus | 340 V (Spannungsverdoppler aus 120 V AC) | 48 V DC aus Netzteil / Akku |
| Brücke | Halbbrücke, 2× IGBT HGTG30N60A4D | **Vollbrücke, 4× IRF540N** |
| Spannung am Primärkreis | ±170 V | **±48 V** |
| Gate-Treiber | UCC27425 (A invertierend, B nicht invertierend) | **UCC27424DR** (beide nicht invertierend) + 1 Gatter des 74HC14 als Inverter an INA |
| Gate-Trafo | 8 Wdg. primär, 2× 12 Wdg. sekundär (±18 V) | 8 Wdg. primär, **4× 8 Wdg. sekundär (±12 V)** |
| Interrupter | ATtiny85 im Gehäuse | **Arduino, per LWL** (TOTX173 → TORX173) |
| Logikversorgung | 2 Trafos + 7812 + 7805 | **3 externe Netzteile** (48 V, 12 V, 5 V) |
| UVLO | DS1233D-5 | bleibt unverändert |

---

## 1. Sicherheit

> **Die Sekundärseite erzeugt Hochspannung bei etwa 250 kHz.**\
> Die Einspeisung mit 48 V schützt vor Netzspannung, nicht vor dem Ausgang der Spule.

**Realistische Gefahren (nach Wahrscheinlichkeit):**

1. **HF-Verbrennungen** bei Berührung von Funken oder von Metallteilen, die in der Nähe liegen (auch ohne Funkenkontakt). Sie schmerzen oft nicht sofort und reichen tief.
2. **Störung und Zerstörung von Elektronik** in der Umgebung (Handy, Laptop, Messgeräte).
3. **Brandgefahr** durch Funken auf Papier, Kunststoff, Kleidung.
4. **Energie im Bus-Kondensator:** ½ · C · U² ≈ 11,5 J bei 10 mF und 48 V (ca. 15 J bei 55 V). Ein Kurzschluss mit Werkzeug kann Metall verdampfen oder verschweißen.
5. **Implantate:** Träger von Herzschrittmachern oder ICD halten **mindestens 5 m Abstand** (besser: gar nicht anwesend).

**Regeln:**

- Nie allein arbeiten, nie unbeaufsichtigt laufen lassen.
- Nicht barfuß, nicht auf leitendem Boden; nichts Geerdetes anfassen, während der Coil läuft.
- Funken nur mit isoliertem Griff und Metallspitze abnehmen. Eine Hand benutzen, die andere weg vom Körper.
- Sekundärspule **immer** mit kurzer, dicker Leitung erden (sonst suchen sich die Funken andere Wege, auch über Elektronik und Menschen).
- **Not-Aus** in der 48-V-Leitung, griffbereit.
- Bus-Kondensatoren über Bleeder-Widerstand entladen lassen, vor Arbeiten mit Multimeter prüfen.
- Das Gehäuse der Logik muss geerdet sein; Netzteile mit Schutzleiter verwenden.
- Rechtliche Vorgaben zu Funkstörungen (EMVG) beachten; kurze Läufe, möglichst im Keller oder geschirmten Raum.

---

## 2. Systemübersicht

```
BEDIENSEITE (USB)                       │  COIL-SEITE (HF, Hochspannung)
                                        │
PC ─USB─► Arduino ─► TOTX173 ═══ LWL ═══╪═► TORX173 ─► 74HC14 G3 ─► G4 ─► 1 kΩ ─┬─► ENBA + ENBB
                                        │                                4,7 kΩ ─┤   (UCC27424DR)
                                        │                 UVLO (DS1233) ─► Diode ─┘
                                        │
Sekundär-Fuß ─► Feedback-CT (50 Wdg.) ─► Klemmdioden ─► 100 nF ─► 1 kΩ ─► 74HC14 G1 ─┬─► INA (invertiert)
                                                                                     └─► G2 ─► INB

UCC27424DR:  OUTA ─► GDT-Primär (8 Wdg.) ─► 1 µF ─► OUTB      (gegenphasig → ±12 V am GDT)
GDT:         4 × Sekundär (8 Wdg.) ─► Gate-Netzwerk ─► Q1 · Q2 · Q3 · Q4
```

**Leistungsstufe (Vollbrücke):**

```
 +48 V ──────┬───────────────────────────────┬──────
             │                               │
           [Q1]                            [Q3]
             │                               │
   A ────────┴──[C-Prim]──[Primärspule]───────┴────── B
             │                               │
           [Q2]                            [Q4]
             │                               │
   0 V ──────┴───────────────────────────────┴──────
```

- Takt 1: **Q1 + Q4** leiten gleichzeitig
- Takt 2: **Q2 + Q3** leiten gleichzeitig
- Q1/Q2 und Q3/Q4 dürfen **nie** gleichzeitig leiten (Kurzschluss durch die Brücke).

**Warum ein Gatter des 74HC14 an INA?**\
Der UCC27424 hat zwei *nicht invertierende* Kanäle. Beide Ausgänge würden gleichzeitig schalten, die GDT-Primärwicklung zwischen OUTA und OUTB bekäme dann dauerhaft 0 V. Damit der GDT ±12 V sieht, muss einer der Eingänge invertiert werden (INA über Gatter G1). Das Verpolen von Sekundärwicklungen ersetzt dies **nicht**. Die exakte Verdrahtung steht in Abschnitt 4.7. Die gegensinnigen Sekundärwicklungen (S2/S3) sind zusätzlich für die diagonale Brücke nötig.

---

## 3. Versorgung (3 getrennte Netzteile)

| Schiene | Spannung | Strom | Verwendung |
| --- | --- | --- | --- |
| NT1 | 5 V DC | ≥ 0,5 A | 74HC14, TORX173 |
| NT2 | 12 V DC | ≥ 1 A | UCC27424DR, UVLO, Lüfter (optional) |
| NT3 | 48 V DC (max. 55 V) | ≥ 10 A Dauer-/Spitzenleistung | Leistungsbrücke |

- Der Arduino wird über USB vom PC versorgt (Bedienseite).
- 5 V und 12 V: **gemeinsame Logik-Masse**.
- Logik-Masse nur an **einem** Punkt mit Schutzleiter verbinden. Mehrere Erdverbindungen erzeugen Masseschleifen und Aussetzer.
- **Einschaltreihenfolge:** erst NT1 + NT2, dann NT3. **Ausschalten:** erst NT3, dann NT1 + NT2.
- Die Netzteile mit Ferritdrossel + 100 nF Keramik am Ausgang gegen HF absichern; möglichst im geerdeten Metallgehäuse.
- Ein Labornetzteil kann keine Energie aufnehmen. Beim Abschalten der Pulse kann die Busspannung kurz ansteigen. Bus-Kondensator großzügig dimensionieren und die Spitzen messen.
- Inrush beim Zuschalten der 48 V auf 10 mF: Vorladung über 22 Ω / 5 W (τ ≈ 0,22 s, nach ca. 1–2 s überbrücken) oder Netzteil mit Softstart. Ohne Vorladung können Netzteil-Schutz, Sicherung oder Schalter auslösen bzw. verschweißen.

---

## 4. Bauteileliste

### 4.1 Leistungsbrücke

| Bauteil | Wert / Typ | Anz. | Hinweise |
| --- | --- | --- | --- |
| Q1–Q4 | **IRF540N** (100 V, 33 A, Rds(on) ≤ 44 mΩ, Vgs max ±20 V) | 4 | TO-220. Drain-Lasche liegt auf Potential → **isoliert** auf den Kühlkörper (Glimmer/Silikonpad + Isolierbuchse). Alle vier aus derselben Charge. |
| C-Bus | **Elko 10 mF (10 000 µF), 70 V** (Vorhandenes Bauteil; Alternative: 2200–4700 µF, ≥ 80 V, low-ESR) | 1 | Nah an der Brücke. Nur bis **max. 55 V Bus** verwenden; das Netzteil darf 70 V nie erreichen (Überschwinger, Defekt). Polung beachten. Bei langer Lagerung vorher mit Strombegrenzung „formieren“ (langsam auf Nennspannung bringen). Rippelstrom und ESR laut Datenblatt prüfen. |
| C-Brücke | **KEMET X7R 1206, 1 µF, ≥ 100 V Nennspannung** (50-V-Typen sind bei 48–55 V Dauerspannung zu knapp) | 8–10 | **Direkt** an den Brückenanschlüssen, je Halbbrückenzweig (Q1/Q2 und Q3/Q4) die Hälfte. Wegen DC-Bias bleiben bei 48 V nur etwa 40–60 % der Kapazität (Schätzung, Kennlinie im Datenblatt prüfen). Optional zusätzlich 1 MKP-Folie 1–2,2 µF. Siehe 4.8. |
| C-Prim | **KEMET X7R 1206, 1 µF, ≥ 50 V** (100 V empfohlen) | 8–10 | In Serie zur Primärspule (DC-Sperre), alle parallel auf einer kleinen Platine. Siehe 4.8. Robustere Alternative: 1× MKP-Folie 4,7 µF, ≥ 250 V. |
| R-Bleeder | 3,3 kΩ, ≥ 2 W | 1 | Parallel zum Bus-C. Verlustleistung dauerhaft ca. 0,7 W bei 48 V. Entladezeit: τ = 33 s, unter 5 V nach ca. 75 s (ein 10-kΩ-Widerstand wäre mit τ = 100 s zu langsam). |
| R-Vorladung | 22 Ω, 5 W (Drahtwiderstand) + Überbrückungsschalter | 1 | Begrenzt den Einschaltstrom auf ca. 2 A (ohne Vorladung fließt beim Zuschalten ein sehr hoher Impulsstrom in 10 mF). Nach ca. 1–2 s überbrücken. |
| Sicherung | 6,3 A träge + Halter | 1 | In +48 V. |
| Not-Aus | Schalter / Schütz | 1 | In +48 V. |
| Kühlkörper | Rth ≤ 10 K/W (kleiner ist besser) | 1–2 | Lüfter optional. |

**Gate-Netzwerk (je MOSFET, 4×):**

| Bauteil | Wert | Anz. | Hinweise |
| --- | --- | --- | --- |
| R-Gate | 6,8 Ω, ½ W | 4 | In Serie zwischen GDT-Sekundärwicklung und Gate. |
| D-Gate | 1N4148 | 4 | **Parallel zu R-Gate, Anode am Gate** (Kathode zum GDT). Beschleunigt das Ausschalten. |
| R-GS | 10 kΩ | 4 | Gate–Source, hält den MOSFET sicher aus. |
| TVS | **P4SMAJ16A** (unidirektional, SMA/DO-214AC) | **8** | **Je Gate 2 Stück in Antiserie** (Kathode an Kathode) zwischen Gate und Source, direkt am MOSFET. |

> ⚠️ **Hinweis zu P4SMAJ16A:** Der Typ hat laut Datenblatt eine Durchbruchspannung von etwa 17,8–19,7 V. In Antiserie kommen ca. 0,7 V Flussspannung hinzu, bei Stromfluss steigt die Klemmspannung weiter. Das Gate des IRF540N verträgt maximal ±20 V. Der Schutz greift also erst *an* dieser Grenze und fängt nur sehr kurze Spitzen ab.\
> **Besser:** P4SMAJ12A oder P4SMAJ13A (Antiserie klemmt dann bei etwa 14–16 V) oder alternativ Z-Dioden 15 V (z. B. BZX85C15) antiseriell. Wenn du bei P4SMAJ16A bleibst, die Gate-Spannung mit dem Oszilloskop prüfen (Spitzen müssen deutlich unter 18 V bleiben) und nicht über ±12 V Treiberamplitude gehen.

### 4.2 Gate-Treiber und Logik

| Bauteil | Wert / Typ | Anz. | Hinweise |
| --- | --- | --- | --- |
| IC1 | **UCC27424DR** (SOIC-8, dual nicht invertierend, ±4 A, VDD 4–15 V) | 1 | SMD → Adapterplatine. Pins: 1 ENBA, 2 INA, 3 GND, 4 INB, 5 OUTB, 6 VDD, 7 OUTA, 8 ENBB. Enable-Pins sind intern mit 100 kΩ nach VDD gezogen (offen = *aktiv*). |
| IC2 | 74HC14 (DIP-14 oder SOIC) | 1 | G1, G2: Feedback. G3, G4: LWL-Puffer. 2 Gatter frei. |
| C-VDD | 10 µF Keramik (≥ 25 V) + **1 µF X7R 1206 (≥ 25 V)** statt 100 nF | je 1 | Direkt an VDD/GND des UCC. |
| C-5V | **1 µF X7R 1206 (≥ 16 V)** oder 100 nF | 2 | An 74HC14 und TORX173. |
| C-GDT | **KEMET X7R 1206, 1 µF, ≥ 25 V** | 2 (parallel) | **In Serie zur GDT-Primärwicklung** (zwischen OUTA und OUTB). Fehlte in v1.0. |
| R-EN | 1 kΩ | 1 | Von G4 zum Enable-Knoten (ENBA + ENBB verbunden). |
| R-EN-PD | 4,7 kΩ | 1 | Enable-Knoten → GND. Sorgt für „aus“, wenn 5 V oder LWL ausfallen (intern zieht sonst 100 kΩ nach VDD = Dauerbetrieb). |
| R-TORX | 100 kΩ | 1 | Eingang G3 → GND (definierter Ruhepegel). |

**Feedback-Pfad (je 1×):** 2× 1N4148 als Klemmdioden (nach +5 V und GND; Schottky wie 1N5818 sind besser), 100 nF Koppelkondensator (**nicht** durch 1 µF ersetzen, siehe 4.8), 1 kΩ Serienwiderstand. Optional 0,1 nF gegen MHz-Oberwellen (der Autor rät, ihn eher vor die Inverter zu legen, weil er Verzögerung einführt).

### 4.3 Unterspannungsabschaltung (UVLO)

| Bauteil | Wert / Typ | Anz. | Hinweise |
| --- | --- | --- | --- |
| IC3 | DS1233D-5(+) | 1 | Original-Schaltplan zeigt TO-92. Gehäuse beim Kauf prüfen. |
| R3 | **560 Ω** | 1 | Von **+12 V** zum Vcc-Knoten (oberer Teilerwiderstand). |
| R4 | **470 Ω** | 1 | Vom Vcc-Knoten nach **GND** (unterer Teilerwiderstand). |
| D2 | 1N4007 | 1 | **Anode am Enable-Knoten, Kathode am /RST-Ausgang** des DS1233. Entkopplungsdiode, zieht Enable auf Low. |

Funktion: Der Teiler gibt bei 12 V etwa 5,5 V an Vcc. Fällt die 12-V-Schiene unter etwa 10 V, sinkt Vcc unter die Schwelle des DS1233, /RST geht auf Low und zieht über D2 den Enable-Knoten nach unten (Treiber aus). (In v1.0 waren R3/R4 vertauscht.)

### 4.4 LWL-Strecke

| Bauteil | Wert / Typ | Anz. | Hinweise |
| --- | --- | --- | --- |
| U1 | TOTX173 | 1 | Sender, 5 V, bei dem Arduino. |
| U2 | TORX173 | 1 | Empfänger, 5 V, auf der Coil-Seite im geerdeten Metallgehäuse. |
| Kabel | Standard-TOSLINK-Kabel (POF) | 1 | 1–5 m. |
| C | **1 µF X7R 1206 (≥ 16 V)** oder 100 nF an jedem Modul | 2 | Direkt an den Versorgungspins. |

**Wichtig:**

- Die verwandte Baureihe TORX170 gibt laut Datenblatt *High bei Licht, Low ohne Licht* (6 Mbit/s). **Für den TORX173 im Datenblatt prüfen**, ebenso die Polarität des Senders (Licht bei Eingang High oder Low).
- Der Ausgang des TORX ist schwach (Datenblatt der Baureihe: VOH bei nur −60 µA spezifiziert). Deshalb **nicht** direkt auf den Enable-Knoten, sondern über die Puffer G3/G4 des 74HC14.
- Ruhezustand muss „Treiber aus“ sein (kein Licht, Kabel gezogen, Arduino im Reset). Ist der TORX173 invertierend, entfällt ein Gatter (G3 oder G4) bzw. wird hinzugefügt, bis dieser Zustand erreicht ist. Ohne diese Prüfung droht Dauerbetrieb und damit Zerstörung der Brücke.

### 4.5 Arduino-Interrupter

| Bauteil | Wert / Typ | Anz. | Hinweise |
| --- | --- | --- | --- |
| Arduino | Uno / Nano (5 V-Logik) | 1 | 3,3-V-Boards nur mit Pegelwandler. |
| Poti | 10 kΩ linear | 2 | Frequenz (Original: 1–254 Hz) und Pulsbreite (0–1500 µs). |
| R6 | 10 kΩ | 1 | Pulldown am TOTX173-Eingang (sofern Licht bei „High“). |

**Software-Limits (zwingend):**

- Pulsbreite max. 1500 µs, Tastverhältnis max. 15 % (Original: ca. 10 %).
- Ruhezustand bei Reset und Boot: Ausgang Low.
- Timer-basierte Pulserzeugung (Timer1), nicht per `delay()`.
- Wegen des Ring-up-Verhaltens bei niedriger Spannung sind 200–500 µs Pulsbreite typisch.

### 4.6 Spulenmaterial und Sonstiges

| Bauteil | Wert / Typ | Hinweise |
| --- | --- | --- |
| PVC-Rohr | 3,5" Außendurchmesser (89 mm, „3 Zoll“), Länge ca. 7" (180 mm) | Sekundärspulenkörper. |
| Lackdraht | 34 AWG (ca. 0,16 mm), ca. 280 m | Etwa 50 g Kupfer. |
| Primärdraht | 14 AWG (ca. 1,6 mm Ø), ca. 3 m | Volldraht oder Litze. |
| Toroid | gestanzter Alu-Toroid 8" × 1,9" | Alternativ Alu-Flexrohr oder Schaumstoff mit Alufolie. |
| Endkappen | Acryl, mit 2-56-Schrauben |  |
| Lack | Polyurethan-Klarlack, mehrere dünne Schichten |  |
| Gehäuse | geerdetes Metallgehäuse für Logik | Masseleitung des Sekundärfußes **innen** führen. |

### 4.7 Verdrahtung: Gate-Invertierung, LWL-Puffer und Treiber (Pin für Pin)

**Prinzip:** INA bekommt das *invertierte* Feedback-Signal, INB das *nicht invertierte*. Dadurch schaltet OUTA immer gegenphasig zu OUTB, und die GDT-Primärwicklung zwischen beiden sieht ±12 V.

```
Feedback-CT ─► Klemmdioden ─► 100 nF ─► 1 kΩ ─► [HC14 Pin 1 ▷○ Pin 2] ─┬─► UCC Pin 2 (INA)  = NICHT Feedback
                                                                        └─► [HC14 Pin 3 ▷○ Pin 4] ─► UCC Pin 4 (INB) = Feedback

TORX173 ─► [HC14 Pin 5 ▷○ Pin 6] ─► [HC14 Pin 9 ▷○ Pin 8] ─► 1 kΩ ─┬─► UCC Pin 1 (ENBA)
                                                                   ├─► UCC Pin 8 (ENBB)
                                                     4,7 kΩ ─► GND ┤
                               UVLO: D2, Anode an diesem Knoten ────┘

UCC Pin 7 (OUTA) ─► GDT-Primär Anfang (●) ... Ende ─► 1 µF ─► UCC Pin 5 (OUTB)
```

**a) 74HC14 (Pinbelegung Standard: 1A=1, 1Y=2, 2A=3, 2Y=4, 3A=5, 3Y=6, GND=7, 4Y=8, 4A=9, 5Y=10, 5A=11, 6Y=12, 6A=13, VCC=14)**

| Pin | Funktion | Verbindung |
| --- | --- | --- |
| 1 | Eingang G1 | vom 1-kΩ-Widerstand des Feedback-Netzwerks (siehe d) |
| 2 | Ausgang G1 (= NICHT Feedback) | → **UCC Pin 2 (INA)** und → **74HC14 Pin 3** |
| 3 | Eingang G2 | ← Pin 2 |
| 4 | Ausgang G2 (= Feedback) | → **UCC Pin 4 (INB)** |
| 5 | Eingang G3 | ← TORX173-Ausgang; zusätzlich **100 kΩ nach GND** |
| 6 | Ausgang G3 | → Pin 9 |
| 7 | GND | Logik-GND |
| 8 | Ausgang G4 | → **1 kΩ** → Enable-Knoten |
| 9 | Eingang G4 | ← Pin 6 |
| 10, 12 | Ausgänge G5, G6 | offen lassen |
| 11, 13 | Eingänge G5, G6 | **an GND** (unbenutzte Eingänge nie offen lassen) |
| 14 | VCC | +5 V; 1 µF X7R (oder 100 nF) direkt zwischen Pin 14 und Pin 7 |

**b) UCC27424DR (SOIC-8)**

| Pin | Name | Verbindung |
| --- | --- | --- |
| 1 | ENBA | Enable-Knoten (EN) |
| 2 | INA | ← 74HC14 **Pin 2** (invertiertes Feedback) |
| 3 | GND | Logik-GND |
| 4 | INB | ← 74HC14 **Pin 4** (Feedback) |
| 5 | OUTB | → 1 µF → GDT-Primär **Ende** |
| 6 | VDD | +12 V; 10 µF + 1 µF X7R direkt zwischen Pin 6 und Pin 3 |
| 7 | OUTA | → GDT-Primär **Anfang (●)** |
| 8 | ENBB | Enable-Knoten (EN), mit Pin 1 verbinden |

**c) Enable-Knoten (EN)** – hier treffen sich: UCC Pin 1 und Pin 8, der 1-kΩ-Widerstand von 74HC14 Pin 8, ein **4,7-kΩ-Widerstand nach GND** und die **Anode von D2** (UVLO; Kathode am /RST-Ausgang des DS1233).

**d) Feedback-Netzwerk**

| Von | Nach |
| --- | --- |
| CT-Sekundär, Ende 1 | Logik-GND (Schirm des Kabels nur hier erden) |
| CT-Sekundär, Ende 2 | Knoten **FB1** |
| 1N4148 (oben) | Anode an FB1, Kathode an +5 V |
| 1N4148 (unten) | Anode an GND, Kathode an FB1 |
| FB1 | → 100 nF → 1 kΩ → **74HC14 Pin 1** |
| optional (empfohlen für sicheren Start): 100 kΩ | Pin 1 → +5 V und 100 kΩ Pin 1 → GND (legt den Arbeitspunkt in die Mitte) |

**e) LWL-Pfad:** TORX173: Vcc → +5 V, GND → Logik-GND, Ausgang → 74HC14 Pin 5. Ist der TORX173 **nicht invertierend** (Licht = High), gilt die Beschaltung oben (G3 + G4). Ist er **invertierend** (Licht = Low), nur G3 verwenden: Pin 6 → 1 kΩ → EN, und Pin 9 nach GND legen, G4 entfällt.

**f) Zustandstabelle (Primärspannung = OUTA − OUTB)**

| EN | Feedback | INA | INB | OUTA | OUTB | GDT-Primär |
| --- | --- | --- | --- | --- | --- | --- |
| Low | beliebig | beliebig | beliebig | 0 V | 0 V | 0 V (alle Gates aus) |
| High | Low | High | Low | 12 V | 0 V | +12 V |
| High | High | Low | High | 0 V | 12 V | −12 V |

Welcher Brückentakt zu welchem Feedback-Pegel gehört, ist unkritisch. Die Phasenlage stellst du über die Richtung des Massedrahts im Feedback-CT ein (siehe 5.4).

**g) Zeitverhalten:** INB wird um eine Gatterlaufzeit des 74HC14 (ca. 10–20 ns) später geschaltet als INA. In dieser kurzen Zeit liegen beide Ausgänge auf dem gleichen Pegel, die GDT-Primärwicklung sieht 0 V. Das ist harmlos und wirkt wie eine winzige Pause zwischen den Takten.

**h) Prüfung am Oszilloskop (vor dem Anschließen des Busses)**

1. EN auf High legen, 250-kHz-Rechteck (0–5 V) auf 74HC14 Pin 1. Pin 2 und Pin 4 müssen gegenphasig sein.
2. UCC Pin 7 und Pin 5: Rechtecke 0/12 V, gegenphasig. Differenz (OUTA − OUTB): ±12 V.
3. An den vier Gates: S1/S4 gleichphasig, S2/S3 dazu gegenphasig, jeweils ±12 V.
4. EN auf Low legen: OUTA und OUTB bleiben bei 0 V.

**Typische Fehler:**

| Fehler | Folge |
| --- | --- |
| INA und INB am selben Signal (kein Inverter) | GDT bekommt 0 V, keine Gate-Signale |
| Unbenutzte HC14-Eingänge offen | unkontrolliertes Schwingen, Stromaufnahme, Störungen |
| UCC Pin 1 und Pin 8 nicht verbunden | ein Kanal bleibt per Pull-up aktiv (Dauerbetrieb dieses Kanals) |
| INA/INB vertauscht | alle Gates in umgekehrter Phase (mit CT-Richtung korrigierbar) |
| 1 µF fehlt | GDT kann bei Unsymmetrie sättigen |

### 4.8 Einsatz der KEMET X7R 1206, 1 µF

Der vorhandene Keramikkondensator (KEMET X7R, Bauform 1206, 1 µF) lässt sich an vielen Stellen verwenden. **Entscheidend ist seine Nennspannung** (gängig: 16, 25, 50 oder 100 V; 1 µF in 1206 gibt es nicht mit 250 V).

| Stelle | Verwenden? | Anzahl | Mindest-Nennspannung | Hinweis |
| --- | --- | --- | --- | --- |
| C-Brücke (an +48 V / 0 V) | ja | 8–10 | **100 V** | DC-Bias beachten (effektiv ca. 40–60 % der Kapazität). |
| C-Prim (Serie zur Primärspule) | ja, heikel | 8–10 | 50 V (100 V empfohlen) | Strom verteilt sich auf alle Kondensatoren; Temperatur prüfen. |
| C-GDT (Serie zur GDT-Primärwicklung) | ja | 2 | 25 V | Nur wenige Volt DC, kaum Bias-Verlust. |
| VDD-Entkopplung UCC (statt 100 nF) | ja | 1 | 25 V | Zusätzlich zum 10-µF-Keramikkondensator. |
| 5-V-Entkopplung (74HC14, TORX173, TOTX173) | ja | je 1 | 16 V | Alternativ 100 nF. |
| Feedback-Koppelkondensator | **nein** | – | – | 100 nF beibehalten (Zeitkonstante mit Bias-Widerständen). |
| C-Bus (10 mF) | nein | – | – | Elko. |

Gesamtbedarf bei voller Nutzung: etwa 22–26 Stück.

**C-Prim: 8–10 Stück parallel (8–10 µF)**

- Reaktanz bei 252 kHz: etwa 0,06–0,08 Ω, das sind weniger als 1 % der 11,7 Ω Spulenreaktanz. Er bleibt ein reiner DC-Blocker.
- Der Primärstrom (ca. 4 A stationär, ca. 8 A Spitze, Schätzung) teilt sich auf alle Kondensatoren auf, jeder trägt nur etwa 0,3–0,6 A (effektiv). Die Verlustleistung pro Kondensator liegt dadurch im Milliwatt-Bereich.
- Mit weniger als 6 Stück sinkt die Reserve spürbar; bei einem einzelnen Kondensator würde er überlastet.
- **Aufbau:** Alle Kondensatoren nebeneinander auf einer kleinen starren Platine (kupferkaschiert, mit breiten Kupferflächen) verlöten und die Platine mechanisch fest montieren. Keine frei hängende Verdrahtung, keine Biegebelastung (Keramik reißt leicht).
- **Zuleitung:** dicke Leitungen (mindestens 1,5 mm²), kurz.
- **Temperaturcheck:** Nach dem ersten Lauf von etwa 30 Sekunden die Kondensatoren von Hand oder mit IR-Thermometer prüfen; sie sollten höchstens handwarm sein (\< 60 °C).
- Fehlerfall: Bleibt ein Schalter dauerhaft eingeschaltet, kann bis zur vollen Busspannung Gleichspannung am Kondensator anliegen. Deshalb ist 100 V Nennspannung besser als 50 V.

**C-Brücke:** Die Kondensatoren so nah wie möglich an Drain von Q1/Q3 und Source von Q2/Q4 löten (kürzeste Wege), nicht an den Elko. Die Hälfte auf Zweig A, die Hälfte auf Zweig B.

**Nachteile von X7R:** Kapazitätsverlust bei Gleichspannung (DC-Bias), Temperaturabhängigkeit (±15 %), mögliche Mikrofonie (hörbares Knistern bei gepulstem Strom im Audiobereich, harmlos) und Rissgefahr bei mechanischer Belastung. MKP-Folienkondensatoren sind an den Stellen C-Prim und C-Brücke weiterhin die robustere Wahl, falls du welche hast.

---

## 5. Spulen und Trafos

### 5.1 Sekundärspule (Daten wie im Original)

| Parameter | Wert |
| --- | --- |
| Rohr | PVC 3,5" Außendurchmesser, ca. 7" lang |
| Draht | Lackdraht 34 AWG |
| Wickellänge | **6,25" (159 mm)** |
| Windungen | ca. **975** (Original: ca. 159 Wdg. pro Zoll, 98 % Füllung) |
| Isolation | mehrere dünne Schichten Polyurethan-Lack |
| Anschlüsse | Drahtenden auf Kupferstreifen löten, Streifen auf Endkappen kleben |
| Resonanzfrequenz | ca. 250 kHz mit 8" × 1,9"-Toroid (Original: 249–252 kHz; JavaTC: 257 kHz) |

Der Draht hat einen Abstand von nur etwa 0,16 mm pro Windung. Das gelingt nur mit dünn isoliertem Lackdraht und exakt aneinander liegenden Windungen.

**Wickeltipp:** Gleichmäßig unter leichter Spannung wickeln (Original: ca. 1,5 Stunden von Hand). Danach sofort mit mehreren dünnen Lackschichten fixieren.

### 5.2 Primärspule (Daten wie im Original)

| Parameter | Wert |
| --- | --- |
| Draht | 14 AWG |
| Windungen | **6** (bei Racing Sparks eine Windung weniger oder Durchmesser vergrößern) |
| Durchmesser | **4,56" (ca. 116 mm)** |
| Wickelhöhe | **0,65" (ca. 16,5 mm)** |
| Lage | leicht unterhalb des Beginns der Sekundärwicklung |
| Induktivität / Kopplung | ca. 7,4 µH / k ≈ 0,25 (JavaTC) |
| Reaktanz bei 252 kHz | ca. 11,7 Ω |

Mit dem größeren Durchmesser lässt sich die Kopplung reduzieren; sie ist ein Kompromiss gegen Racing Sparks. Der Sperrkondensator C-Prim (siehe 4.1 und 4.8) liegt in Serie zur Primärspule.

### 5.3 Gate-Trafo (GDT)

Aufgabe: die gegenphasigen Ausgangssignale des UCC27424 potentialgetrennt auf vier Gates übertragen. Die Primärseite sieht ±12 V, jede Sekundärseite ebenfalls (1:1).

**Kern:** Ferrit-Ringkern, Außendurchmesser ca. 22–30 mm. Material: Leistungsferrit der Klasse N87/N30/3C90/3F3. Eisenpulver und Baumarkt-Ringkerne gehen **nicht**.\
**Test vor dem Wickeln** (wie im Original): ein paar Probewindungen aufbringen, 250 kHz-Rechteck vom Signalgenerator einspeisen und die Sekundärseite am Oszilloskop prüfen. Das Signal muss annähernd rechteckig bleiben.

**Wicklung:**

| Wicklung | Wdg. | Draht | Anschluss | Polarität |
| --- | --- | --- | --- | --- |
| P | 8 | AWG 24–26 | OUTA – (1 µF) – OUTB | Bezug (Anfang = „●“) |
| S1 | 8 | AWG 24–26 | Gate-Netzwerk Q1 / Source Q1 | Anfang → Gate-Netzwerk (**gleichsinnig**) |
| S4 | 8 | AWG 24–26 | Gate-Netzwerk Q4 / Source Q4 | Anfang → Gate-Netzwerk (**gleichsinnig**) |
| S2 | 8 | AWG 24–26 | Gate-Netzwerk Q2 / Source Q2 | Anfang → **Source** (**gegensinnig**) |
| S3 | 8 | AWG 24–26 | Gate-Netzwerk Q3 / Source Q3 | Anfang → **Source** (**gegensinnig**) |

**Vorgehen:**

1. Fünf Drähte (je eine Farbe) **gemeinsam verdrillen** und 8 Windungen gleichmäßig über den ganzen Umfang des Kerns wickeln. Gleiche Wickelzahl und gleiche Lage ergeben gleiche Streuinduktivität und damit gleichzeitiges Schalten aller vier Gates.
2. Anfänge sofort markieren (Farbe, Schrumpfschlauch).
3. Isolation: Lackdraht Grad 2 oder Teflon/Kapton-isoliert, zusätzlich Kaptonband zwischen Kern und Wicklung. Die Sekundärwicklungen liegen auf unterschiedlichen Potentialen (bis 48 V gegeneinander).
4. Fertigen GDT mit Signalgenerator testen: alle vier Ausgänge ±12 V, S1/S4 gleichphasig, S2/S3 dazu gegenphasig.

**Kontrolle der Flussdichte:** Bei ±12 V, 250 kHz, 8 Windungen und einem Kern von ca. 50 mm² Querschnitt liegt die Flussdichte bei etwa 55 mT (Schätzung), weit unter der Sättigung.

### 5.4 Feedback-Stromtrafo (CT)

| Parameter | Wert |
| --- | --- |
| Kern | kleiner Ferrit-Ringkern (ca. 15–25 mm Außendurchmesser, Leistungsferrit) |
| Sekundärwicklung | **50 Windungen**, Draht AWG 28–32 |
| Primärwicklung | **1 Windung** = der Massedraht des Sekundärfußes, einmal durch den Ring |
| Phasenlage | bei falscher Phase die Richtung des Massedrahts durch den Ring **umdrehen** (Original) |

- Ausgang verdrillt führen und schirmen (Schirm nur einseitig erden).
- Der Autor beobachtete Aussetzer ab etwa 80 V Bus, weil die Masseleitung des Sekundärfußes Störungen vom Primärstrom aufnahm. Abhilfe: Masseleitung **innerhalb des geerdeten Gehäuses** führen.

---

## 6. Aufbau und Inbetriebnahme

### 6.1 Reihenfolge

1. Sekundärspule wickeln und lackieren (Trocknungszeit einplanen).
2. GDT wickeln und mit Signalgenerator testen.
3. Feedback-CT wickeln.
4. Logikplatine aufbauen (74HC14, UCC27424, UVLO, LWL-Empfänger).
5. Brücke mit Kühlkörper aufbauen (kurze, breite Leitungen, Bus-C nah an den MOSFETs).
6. Arduino mit Interrupter-Sketch + TOTX173.

### 6.2 Inbetriebnahme (Stufen)

1. **Nur Logik** (NT1 + NT2 an, NT3 **aus**): UVLO prüfen (12 V langsam hochdrehen; Enable bleibt unterhalb von etwa 10 V Low). Ruhezustand testen: LWL abziehen, Arduino reset → Enable Low.
2. **Gate-Signale ohne Bus** (Signalgenerator-Rechteck 250 kHz in den Feedback-Pfad einspeisen): an allen vier Gates ±12 V, Phasen wie in 5.3, **kein Überlappen** von Q1/Q2 bzw. Q3/Q4, keine Spitzen über etwa 16 V.
3. **Bus 12 V, Strombegrenzung 0,5 A** (alternativ Glühlampe in Reihe). Coil starten, Primärstrom mit Stromwandler messen (Original: 300 Wdg., 47 Ω Bürde).
4. **Phasenlage prüfen:** Der Primärstrom soll der Brückenspannung leicht **nacheilen** (induktiv). Eilt der Strom vor, schaltet die Body-Diode des IRF540N hart (Reverse-Recovery-Spitzen, Gefahr für die MOSFETs). Dann CT-Phase prüfen und ggf. eine kleine Verzögerung ins Feedback legen.
5. Bus schrittweise erhöhen: 12 → 24 → 36 → 48 V, jeweils mit kurzen Pulsen (100–300 µs, 50–100 Hz). Nach jedem Schritt Temperaturen und Spannungsspitzen an den Drains prüfen (Ziel: deutlich unter 80 V).
6. Erst dann längere Pulse und höhere Pulsrate.

### 6.3 Checkliste vor dem ersten Einschalten

- [ ] Alle Gate-Source-Pfade mit Multimeter geprüft (kein Kurzschluss)
- [ ] Polung Bus-Elko und Dioden kontrolliert
- [ ] Sicherung und Not-Aus eingebaut
- [ ] UVLO funktioniert (Enable Low unter etwa 10 V)
- [ ] Ruhezustand Enable = Low bei gezogenem LWL und bei Arduino-Reset
- [ ] Polarität von TOTX173/TORX173 nach Datenblatt verifiziert
- [ ] Software-Limits aktiv (max. 1500 µs, max. 15 %)
- [ ] Alle vier MOSFET-Drains elektrisch isoliert vom Kühlkörper
- [ ] Sekundärfuß kurz und dick geerdet, Breakout-Spitze am Toroid
- [ ] Masseleitung des Sekundärfußes innerhalb des Gehäuses
- [ ] Bleeder (3,3 kΩ) am Bus-Kondensator, Spannung per Multimeter nach dem Ausschalten gemessen
- [ ] Vorladung (22 Ω) vorhanden, Bus-Kondensator richtig gepolt, Netzteil nie über 55 V
- [ ] Alle unbenutzten 74HC14-Eingänge (Pin 11, 13) an GND; UCC Pin 1 und Pin 8 verbunden

---

## 7. Erwartete Werte (Schätzungen)

Grundlage: Messwerte des Originals (Halbbrücke, ±170 V, X_L ≈ 11,7 Ω, 14,2 A stationär, Spitze ca. 28 A vor Streamer-Belastung, Funken ca. 9") und die lineare Skalierung mit der Primärspannung.

| Größe | Original (±170 V) | Dieser Aufbau (±48 V) | bei 55–60 V |
| --- | --- | --- | --- |
| Primärstrom stationär | 14,2 A (gemessen) | ca. 4 A | ca. 5 A |
| Primärstrom Spitze | ca. 28 A | ca. 8 A | ca. 10 A |
| Funkenlänge | ca. 9" (22 cm) | ca. 4–8 cm | etwas länger |
| Gate-Spannung | ±18 V | ±12 V | ±12 V |
| Lautstärke | „quite loud“ | etwa 11 dB leiser (Faktor (48/170)² ≈ 0,08 in der Leistung) | +1–2 dB |

Zur Einordnung: Der Autor erreichte bei ±40 V (Halbbrücke am 80-V-Bus) mit einer ungünstigeren Zweitspule und 400+ µs Pulsen 2,5–3" Funken.

**Musik:** Die Töne entstehen aus der Pulsfolge (Interrupter). Gut hörbar im ruhigen Raum auf wenige Meter. Mit 200–500 µs Pulsbreite sind tiefe bis mittlere Töne (bis etwa 500 Hz) möglich; höhere Töne werden leiser, weil die Pulse kürzer sein müssen.

---

## 8. Fehlersuche

| Problem | Mögliche Ursache | Lösung |
| --- | --- | --- |
| Coil schwingt nicht an | Feedback-CT falsch gepolt; HC14-Eingang ohne Anregung | Richtung des Massedrahts durch den Ring umdrehen; kurzer Startimpuls oder Bias am HC14-Eingang |
| MOSFET stirbt sofort | Überlappung der Gate-Signale, falsche GDT-Phase, Strom eilt vor | Gate-Signale mit Oszi prüfen; S2/S3 gegensinnig; Phasenlage korrigieren |
| UCC27424 wird heiß | Gate-Ladeleistung (vier Gates bei 250 kHz, ca. 1 W) | Temperatur beobachten; ggf. VDD 10–12 V, größere Gate-Widerstände, Kühlfläche |
| Racing Sparks auf der Sekundärspule | Kopplung zu hoch, kein Breakout-Punkt | Primärspule weiter nach außen, 1 Windung weniger, scharfe Spitze |
| Aussetzer ab bestimmter Spannung | Störungen in der Masseleitung des Feedbacks | Masseleitung ins geerdete Gehäuse, Leitungen verdrillen und schirmen |
| Dauerbetrieb statt Pulsen | Enable offen / TORX-Polarität falsch | R-EN-PD prüfen (4,7 kΩ nach GND); Polarität nach Datenblatt |
| Gate-Spitzen über 16 V | Leitungsinduktivität, GDT-Streuung | Leitungen kürzen, Treiberspannung senken, niedrigere TVS (P4SMAJ12A/13A) |
| C-Prim-Kondensatoren werden heiß oder knistern | zu wenige parallel, Strom zu hoch, X7R-Mikrofonie | Anzahl erhöhen (8–10), Leitungen kürzen; ggf. MKP-Kondensator verwenden |
| Spannungsspitzen an den Drains über 80 V | zu lange Leitungen, Bus-C zu weit weg | 1-µF-X7R-Kondensatoren direkt an der Brücke; ggf. TVS/RC-Snubber über Drain–Source |

---

## 9. Quellen

- Originalbeschreibung SSTC 2: [https://www.loneoceans.com/labs/sstc2/](https://www.loneoceans.com/labs/sstc2/)
- UCC27423/4/5 Datenblatt und Produktseite: [https://www.ti.com/product/UCC27424](https://www.ti.com/product/UCC27424)
- JavaTC (Spulenrechner): [http://www.classictesla.com/java/javatc/javatc.html](http://www.classictesla.com/java/javatc/javatc.html)
- IRF540N, P4SMAJ16A, TOTX173/TORX173, DS1233: aktuelle Datenblätter beim Hersteller bzw. Händler abrufen und die Werte aus diesem Dokument dagegen prüfen.

---

## 10. Änderungen

### Version 1.3

- KEMET X7R 1206 1 µF in der Bauteilliste an den möglichen Stellen eingetragen (C-Brücke, C-Prim, C-GDT, Entkopplung); neuer Abschnitt 4.8 mit Stückzahlen, Mindest-Nennspannung, DC-Bias, Aufbauhinweisen und Temperaturcheck. Feedback-Koppelkondensator bleibt 100 nF.

### Version 1.2

- Neuer Abschnitt 4.7: Pin-für-Pin-Verdrahtung der Gate-Invertierung (74HC14, UCC27424DR, Enable-Knoten, Feedback-Netzwerk, LWL-Pfad), Zustandstabelle, Oszilloskop-Prüfung, typische Fehler.
- Bus-Kondensator: vorhandener 10-mF/70-V-Elko zugelassen (max. 55 V Bus), Bleeder auf 3,3 kΩ geändert, Vorladung 22 Ω ergänzt, Energieangabe auf ca. 11,5 J korrigiert.

### Version 1.1 (gegenüber Version 1.0)

- Signalkette korrigiert: TORX173 → Enable, Feedback → 74HC14 → INA/INB (nicht umgekehrt).
- **Treiber:** UCC27424 liefert zwei gleichphasige Ausgänge → ein Gatter des 74HC14 invertiert INA; sonst sieht der GDT 0 V.
- GDT: 1-µF-Sperrkondensator in Serie zur Primärwicklung ergänzt; Windungen 8:8:8:8:8 (±12 V); Polarität pro Wicklung eindeutig beschrieben.
- Gate-Netzwerk: Diode **Anode am Gate**; fälschlich genannte 100-nF-Kondensatoren an den Gates entfernt (würden das Schalten blockieren); 15-V-bidirektionale TVS durch je zwei P4SMAJ16A in Antiserie ersetzt, inklusive Warnhinweis.
- UVLO: Teilerwiderstände waren vertauscht (560 Ω oben, 470 Ω unten), Diodenpolarität präzisiert.
- Enable-Beschaltung: 1 kΩ Serie + 4,7 kΩ Pulldown (interner Pull-up von 100 kΩ nach VDD beachtet); 5,1-kΩ-Pulldown aus v1.0 ersetzt.
- LWL: unnötigen 330-Ω-Vorwiderstand am Sender entfernt; Standard-TOSLINK-Kabel; Polaritätswarnung.
- Spulen: Daten an das Original angeglichen (Primär 6 Wdg., 4,56", Toroid 1,9" × 8", Rohrlänge 7").
- Entfernt: nicht verifizierte Bestellnummern von Ferritkernen und nicht verifizierte Weblinks.
- Erwartungswerte neu aus den Messwerten des Originals berechnet; Sicherheitshinweise an die korrigierte Risikobewertung angepasst.

---

*Dokument-Version 1.3 · Stand: Oktober 2026*\
*Basierend auf dem SSTC-2-Design von Guangyan (loneoceans.com)*