# GFS Physik LK: Festkörper-Teslaspule (SSTC, vollbrückengetrieben, 48 V)

Projekt: https://github.com/JuKimpfler/FullBridge_LowVoltage_STTC
Dauer: bis zu 2 Schulstunden (ca. 90 min) inklusive Vorführung
Team: Person A und Person B

---

## 1. Aufteilung der Themen

### Person A: Vom Strom zur Hochspannung (Physik der Spule)

- Aufbau der Teslaspule: Primärspule, Sekundärspule, Toroid
- Sekundärspule als LC-Schwingkreis, Resonanzfrequenz (ca. 250 kHz)
- Induktion und Kopplung zwischen Primär und Sekundär (k ≈ 0,25), Spannungsübersetzung
- Primärkreis mit Serienkondensator (C-Prim als DC-Sperre), Phasenlage von Strom und Spannung
- Hochspannungsphysik: Durchschlagsfeldstärke, Koronaentladung, Streamer, Funken
- Erwartete Werte (Abschnitt 7 im README): Primärstrom, Funkenlänge, Skalierung von ±170 V (Original) auf ±48 V
- Eigene Messung: berechnete gegen gemessene Resonanzfrequenz

### Person B: Von der Steuerung zum Schalten (Elektronik und Ansteuerung)

- Funktionsprinzip der Vollbrücke: Gleichspannung wird zur Wechselspannung (Q1+Q4 gegen Q2+Q3)
- MOSFET-Grundlagen (IRF540N): Gate, Schaltverhalten, Schutz vor Querstrom
- Gate-Treiber (UCC27424DR) und Gate-Trafo: Potentialtrennung, vier Gates gleichzeitig ansteuern
- Rückkopplung: Wie findet die Schaltung selbst die Resonanzfrequenz? (Stromwandler, Schmitt-Trigger 74HC14)
- Interrupter mit Arduino und Lichtwellenleiter: Warum pulst man, warum optische Trennung?
- Schutzschaltungen: Unterspannungsabschaltung (UVLO), Vorladung, Sicherung, Not-Aus
- Eigene Messung: Gate-Signale am Oszilloskop

### Gemeinsame Teile

- Einstieg (A) und Fazit (B), oder umgekehrt
- Sicherheit: kurz im Überblick (B), gilt für beide
- Schnittstelle der Teile: "Die Brücke erzeugt ein Rechteck, die Spule schwingt darauf." Hier übergibt B an A bzw. A an B.
- Jede Person muss die Grundidee des anderen Teils erklären können (Rückfragen!).

### Ausgleich bei Bedarf

- A ist theorielastiger, B technik- und bauteillastiger. Wenn einer sich im anderen Bereich sicherer fühlt, einfach tauschen.
- Rechnungen aus Abschnitt 7 des README (Skalierung 170 V auf 48 V) können B übernehmen, wenn A zu viel hat.

---

## 2. Zeitplan der GFS (ca. 90 min)

| Zeit   | Inhalt                                                              | Wer   |
|--------|---------------------------------------------------------------------|-------|
| 5 min  | Einstieg: Was ist eine Festkörper-Teslaspule? Foto/Video des Originals | A     |
| 5 min  | Sicherheit und Blockschaltbild (Gesamtüberblick)                    | B     |
| 25 min | Teil A: Schwingkreis, Induktion, Resonanz, Funkenentstehung        | A     |
| 25 min | Teil B: Vollbrücke, Gate-Ansteuerung, Rückkopplung, Interrupter     | B     |
| 15 min | Vorführung                                                          | A + B |
| 5 min  | Fazit: Soll/Ist-Vergleich, Probleme beim Bau, Ausblick              | A + B |
| 10 min | Rückfragen                                                          | A + B |

Falls nur ca. 60 min erwartet werden: Teil A und Teil B je auf 15–18 min kürzen. Die Vorführung bleibt.

---

## 3. Durchführung der Vorführung (von sicher nach spektakulär)

| Stufe | Was wird gezeigt                                                          | Aussage / Bezug                                  | Risiko |
|-------|---------------------------------------------------------------------------|--------------------------------------------------|--------|
| 1     | Oszilloskop: Gate-Signale aller vier MOSFETs, gegenphasig, ohne Überlappung | Teil B (Brücke, GDT, Treiber)                    | gering |
| 2     | Resonanzfrequenz: berechnet (JavaTC, ca. 250 kHz) gegen gemessen         | Teil A (LC-Schwingkreis), Soll/Ist-Vergleich     | gering |
| 3     | Leuchtstoffröhre oder Glimmlampe leuchtet drahtlos in der Nähe der Spule | Teil A (Felder, Energieübertragung)              | mittel |
| 4     | Funken bei niedriger Busspannung, kurze Pulse                            | Teil A (Hochspannung, Durchschlag)               | mittel |
| 5     | Musik über den Arduino-Interrupter (falls es klappt)                     | Teil B (Interrupter, Pulsfolge = Ton)            | mittel |

Plan B: Ein vorher aufgenommenes Video eines gelungenen Laufs bereithalten (falls Hochspannung oder Technik spinnt).

### Ablauf vor Ort

1. Vorab aufbauen und testen (mindestens 30 min vor Beginn), Raum und Abstände festlegen.
2. Reihenfolge der Netzteile: erst Logik (5 V und 12 V), dann 48-V-Bus. Ausschalten umgekehrt.
3. Bus stufenweise hochfahren (z. B. 12, 24, 36, 48 V), jeweils mit kurzen Pulsen (100–300 µs, 50–100 Hz).
4. Pro Stufe kurz Temperaturen prüfen, Läufe nur wenige Sekunden.
5. Danach Bus-Kondensator entladen lassen (Bleeder) und mit Multimeter prüfen.

### Rollen bei der Vorführung

- Person A: erklärt die Physik zu jeder Stufe, bedient das Oszilloskop bei Resonanzmessungen.
- Person B: bedient Netzteile, Not-Aus und Arduino, achtet auf Sicherheit.
- Ein Not-Aus ist immer in Reichweite einer Person.

---

## 4. Sicherheit und Genehmigung

- Vorher mit Fachlehrkraft (und ggf. Schulleitung) klären, ob und wo die Spule betrieben werden darf.
- Hochfrequenz-Hochspannung: HF-Verbrennungen, Störung von Elektronik, Brandgefahr.
- Abstand zum Publikum, nichts Leitendes oder Brennbares in der Nähe, Handys und Laptops aus dem Funkenbereich.
- Personen mit Herzschrittmacher oder ICD vorher abfragen: mindestens 5 m Abstand.
- Nie allein arbeiten. Sekundärspule immer kurz und dick erden. Nichts Geerdetes anfassen, während der Coil läuft.
- Funken nur mit isoliertem Griff abnehmen.
- Feuerlöscher oder Löschdecke in Reichweite.
- Kurze Läufe wegen Funkstörungen (EMVG).

---

## 5. Bau- und Zeitplan (Zeiten anpassen, Termin ergänzen)

GFS-Termin: ____________

| Phase | Aufgabe                                                         | Wer   | Hinweis                                              |
|-------|-----------------------------------------------------------------|-------|------------------------------------------------------|
| 1     | Bauteile bestellen, Datenblätter prüfen (siehe Abschnitt 6)    | A + B | Lieferzeiten einplanen                               |
| 2     | Sekundärspule wickeln und lackieren                             | A     | ca. 1,5 h Wickeln, danach Lack mit Trocknungszeit    |
| 3     | Primärspule, Toroid, Grundplatte                                | A     | Kopplung einstellbar machen                          |
| 4     | GDT wickeln und mit Signalgenerator testen                      | B     | 5 verdrillte Drähte, 8 Windungen                     |
| 5     | Feedback-Stromtrafo wickeln                                     | B     | 50 Windungen                                         |
| 6     | Logikplatine (74HC14, UCC27424DR auf Adapter, UVLO, LWL)        | B     | SMD-Treiber: Lötzeit einplanen                       |
| 7     | Brücke mit Kühlkörper aufbauen                                  | B     | Drains vom Kühlkörper isolieren, kurze Leitungen     |
| 8     | Arduino-Interrupter und Sketch                                  | B     | Limits: max. 1500 µs und 15 % Tastverhältnis         |
| 9     | Inbetriebnahme in Stufen (README Abschnitt 6.2)                 | A + B | gemeinsam, mit Oszilloskop                           |
| 10    | Messwerte sammeln, Folien erstellen, Probelauf der GFS          | A + B | mindestens 2–3 Wochen Puffer vor dem Termin          |

Fehlersuche und Probleme sind Inhalt der GFS ("Was ging schief, wie haben wir es gefunden?").

---

## 6. Vor dem Bau prüfen (im README noch ungeprüft)

- [ ] **TOTX173 / TORX173**: Polarität im Datenblatt (Licht = High oder Low?). Sonst droht Dauerbetrieb der Brücke.
- [ ] **TVS am Gate**: P4SMAJ16A klemmt erst an der Grenze des Gates (±20 V). Besser P4SMAJ12A/13A oder Z-Dioden 15 V.
- [ ] **Phasenlage des Feedbacks**: Der Strom soll der Spannung leicht nacheilen, sonst schaltet die Body-Diode hart und die MOSFETs sterben.
- [ ] **Kondensatoren (KEMET X7R)**: Nennspannung prüfen (C-Brücke mindestens 100 V).
- [ ] **Bus-Kondensator 10 mF / 70 V**: nur bis 55 V verwenden, ggf. vorher formieren.
- [ ] Alle Datenblattwerte aus dem README gegen die Hersteller-Datenblätter abgleichen.

---

## 7. Checkliste für die GFS selbst

- [ ] Folien für Teil A und Teil B fertig, gemeinsamer Einstieg und Fazit
- [ ] Messwerte: Resonanzfrequenz (Soll/Ist), Gate-Signale, Primärstrom, Funkenlänge bei verschiedenen Spannungen
- [ ] Backup-Video eines gelungenen Laufs
- [ ] Genehmigung für die Vorführung liegt vor
- [ ] Not-Aus, Feuerlöscher, Schutzabstand, Erdung geprüft
- [ ] Beide können den jeweils anderen Teil in zwei Sätzen erklären
- [ ] Probelauf der gesamten Präsentation inklusive Zeitmessung

---

## 8. Quellen

- Projekt-README: https://github.com/JuKimpfler/FullBridge_LowVoltage_STTC
- Originaldesign SSTC 2: https://www.loneoceans.com/labs/sstc2/
- UCC27424 Datenblatt: https://www.ti.com/product/UCC27424
- JavaTC (Spulenrechner): http://www.classictesla.com/java/javatc/javatc.html
