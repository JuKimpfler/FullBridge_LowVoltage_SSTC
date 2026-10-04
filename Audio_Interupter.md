Hier ist die kompakte Gesamtübersicht für dein Audio-Interrupter-System mit Arduino, Optokoppler (TOTX173), Gate-Treiber (UCC27424) und H-Brücke.
------------------------------
## 1. Die Verkabelung (Hardware-Setup)## Arduino & Eingänge

* Audio-Eingang (Pin A0):
* Audio-Signalquelle $\rightarrow$ 10 µF Kondensator (Minuspol zum Arduino) $\rightarrow$ Pin A0.
   * 2,5V Bias: Pin A0 über einen 10 kΩ Widerstand an +5V und einen 10 kΩ Widerstand an GND hängen.
* Poti 1 (Pulsbreite / On-Time) (Pin A1): Äußere Pins an +5V und GND. Mittlerer Pin (Schleifer) an A1.
* Poti 2 (Grundfrequenz) (Pin A2): Äußere Pins an +5V und GND. Mittlerer Pin (Schleifer) an A2.

## Optische Übertragungsstrecke

* Sender (TOTX173 am Arduino):
* Pin 1 (Input) $\rightarrow$ Arduino Pin 3
   * Pin 2 (GND) $\rightarrow$ Arduino GND
   * Pin 3 (VCC) $\rightarrow$ Arduino +5V (setze einen 0,1 µF Kondensator direkt zwischen VCC und GND)
   * Indikator-LED: Arduino Pin 4 $\rightarrow$ Vorwiderstand (z. B. 220 Ω) $\rightarrow$ Anode LED $\rightarrow$ Kathode an GND.
* Empfänger (TORX173 an der SSTC-Treiberplatine):
* Pin 1 (Output) $\rightarrow$ Führt das saubere +5V Signal zum UCC27424 (Enable-Pin).
   * Pin 2 (GND) $\rightarrow$ SSTC-Treiber-GND.
   * Pin 3 (VCC) $\rightarrow$ Stabile +5V Versorgung der Treiberplatine.

------------------------------
## 2. Der finale Steuerungscode
Dieser Code arbeitet mit zwei Hardware-Interrupts (Timer 1): ISR A schaltet die Brücke ein, ISR B garantiert das harte Abschalten nach Ablauf der eingestellten Mikrosekunden (pulseWidthTicks).

#define TOTX_PIN 3       // Signal zum TOTX173 (0V / 5V für UCC27424 Enable)#define LED_PIN 4        // Kontroll-LED (analog zu Pin 6 im ATtiny-Plan)#define AUDIO_PIN A0     // Audio (zentriert über 2,5V Bias)#define POTI_PW_PIN A1   // Poti für Impulsbreite (On-Time Schutz)#define POTI_FREQ_PIN A2 // Poti für Grundfrequenz
volatile uint16_t pulseWidthTicks = 160; // Startwert On-Time (~10 µs)
void setup() {
  pinMode(TOTX_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  // ADC auf hohe Geschwindigkeit setzen (Prescaler 32)
  ADCSRA |= (1 << ADPS2) | (1 << ADPS0); 
  ADCSRA &= ~(1 << ADPS1);
  ADMUX = (1 << REFS0) | (1 << ADLAR); // 8-Bit Modus, Referenz VCC

  // Timer 1 im CTC-Modus (Clear Timer on Compare) konfigurieren
  cli();
  TCCR1A = 0; TCCR1B = 0; TCNT1 = 0;
  
  TCCR1B |= (1 << WGM12) | (1 << CS11); // Prescaler = 8 (1 Tick = 0,5 Mikrosekunden)
  TIMSK1 |= (1 << OCIE1A) | (1 << OCIE1B); // Beide Interrupts aktivieren
  sei();
}
// ISR A: Puls-Start (Frequenz)ISR(TIMER1_COMPA_vect) {
  digitalWrite(TOTX_PIN, HIGH); // +5V Pegel (sichert UVLO am UCC)
  digitalWrite(LED_PIN, HIGH);
  OCR1B = pulseWidthTicks;      // Abschaltzeitpunkt laden
}
// ISR B: Puls-Ende (Hardware-Sicherheit für die H-Brücke)ISR(TIMER1_COMPB_vect) {
  digitalWrite(TOTX_PIN, LOW);  // Brücke hart abschalten
  digitalWrite(LED_PIN, LOW);
}
void loop() {
  // 1. Poti für Impulsbreite (On-Time) auslesen (Kanal A1)
  ADMUX = (1 << REFS0) | (1 << ADLAR) | 1;
  ADCSRA |= (1 << ADSC); while (ADCSRA & (1 << ADSC));
  uint8_t pwVal = ADCH;
  // Regelt die On-Time von ca. 10 µs (20 Ticks) bis 150 µs (300 Ticks)
  pulseWidthTicks = map(pwVal, 0, 255, 20, 300);

  // 2. Poti für Frequenz auslesen (Kanal A2)
  ADMUX = (1 << REFS0) | (1 << ADLAR) | 2;
  ADCSRA |= (1 << ADSC); while (ADCSRA & (1 << ADSC));
  uint8_t freqVal = ADCH;

  // 3. Audiosignal einlesen (Kanal A0) und Frequenz modulieren (PFM)
  ADMUX = (1 << REFS0) | (1 << ADLAR) | 0;
  ADCSRA |= (1 << ADSC); while (ADCSRA & (1 << ADSC));
  int16_t audioVal = ADCH - 128; // AC-Signal zentrieren

  // Poti bestimmt Basis-Frequenz, Audio lenkt sie aus
  int16_t finalFreqVal = freqVal + (audioVal / 2); 
  if (finalFreqVal < 10) finalFreqVal = 10; // Frequenz-Untergrenze

  // Umrechnen in Timer-Periodendauer (Bereich ca. 50 Hz bis 1 kHz)
  uint16_t ocrValue = map(finalFreqVal, 0, 255, 40000, 2000); 
  
  // Plausibilitätsprüfung: Ausschaltzeitpunkt muss vor dem nächsten Puls liegen
  if (ocrValue > pulseWidthTicks + 50) {
    OCR1A = ocrValue;
  }

  delay(5);
}

------------------------------
## 3. Wichtiges für die Inbetriebnahme (Sicherheits-Checkliste)

* UVLO-Sicherheit: Durch die Verwendung der digitalWrite-Befehle zieht der Arduino den Pin starr auf GND oder starr auf +5V. Am Ende des TORX173 kommt ein perfekt sauberes Rechtecksignal an, sodass die Unterspannungsüberwachung (UVLO) des UCC27424 niemals in einen undefinierten Zustand gerät.
* Invertierung prüfen: Dieser Code schaltet bei HIGH die Spule ein. Nutzt du die invertierende Version des Treibers (z.B. UCC27423 oder UCC27424 an invertierten Eingängen), müssen HIGH und LOW in den beiden ISR-Blöcken getauscht werden.
* Der erste Test: Drehe das Frequenz-Poti in die Mitte und das Pulsbreiten-Poti ganz nach links (minimale On-Time). Schalte die H-Brücke zuerst nur mit einer sicheren Kleinspannung (z. B. 12V bis 24V aus einem Labornetzteil) ein und prüfe, ob die FETs kühl bleiben, bevor du Netzspannung anlegst.

Wenn du möchtest, können wir noch kurz klären:

* Welchen genauen Suffix hat dein UCC27424 (z. B. P, D, oder eine invertierte Variante)?
* Welche Resonanzfrequenz hat deine SSTC-Sekundärspule ungefähr?

Dann lässt sich die maximale Pulsbreite im Code noch exakter auf deine MOSFETs/IGBTs abstimmen.

