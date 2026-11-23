# Rozhodnutia pri návrhu

[English](decisions.md) · **Slovenčina**

Každý bod: rozhodnutie, prečo, a čo sme zamietli. Nové rozhodnutia pribúdajú na koniec. Pri rozdiele platí anglická verzia.

## D1. Portovať firmvér FM-1, nepísať nový syntetizátor

- **Rozhodnutie:** pod existujúci firmvér FM-1 (najprv Felucca) pridať hardvérovú vrstvu pre ESP32-P4 a spustiť ho bez úprav.
- **Prečo:** Felucca 1.7.0 presunula hardvérovú vrstvu FM-1 do `felucca-bsp`; `fm1-base` už má host simulátor
  a emulátor v prehliadači. FM1VST ukazuje, že to funguje: firmvéry bežia bez úprav za abstrakciou panela.
- **Zamietnuté:** emulácia CPU FM-1 (JieLi AC791N, 2× pi32v2 @ 240 MHz) na P4 cez fm1-emu. Na 360 MHz RISC-V nebeží v reálnom čase.

## D2. Licencie: GPL-3.0 pre softvér, CERN-OHL-S-2.0 pre hardvér

- **Prečo:** Felucca je GPL-3.0-only, takže spojený firmvér musí byť GPL-3.0. Hardvér dostane obdobnú „recipročnú“ hardvérovú licenciu.

## D3. Hlavná doska: Waveshare ESP32-P4-NANO (nahradené D18)

> Nahradená doskou ESP32-P4-WIFI6-Touch-LCD-4.3 so vstavaným displejom. Ponechané kvôli histórii.

- **Prečo:** ESP32-P4 + ESP32-C6, 32 MB PSRAM, microSD, MIPI-DSI, USB 2.0 HS OTG na porte Type-A (USB host pre ovládač),
  kolíkové lišty pre nosnú dosku.
- **Zamietnuté:** M5Stack Tab5 (tablet), ESP32-P4 Knob (jeden enkóder, starší koprocesor),
  Waveshare ESP32-P4-WIFI6-Touch-LCD-4B (720×720 = presné 3× zväčšenie, ale len 5 voľných GPIO a mono ES8311).

## D4. Displej: MIPI-DSI 480×800 dotykový (veľkosť zmenená na 4.3" v D18)

- **Prečo:** displej FM-1 má 240×240. Pri presnom 2× zväčšení zaberie 480×480 a ostane pás 480×320 na ďalšie zobrazenia.
  Dotyk umožní nastavenia (napr. MIDI kanály) bez ďalších ovládacích prvkov.

## D5. Ovládací panel = externý USB-MIDI ovládač, mapovaný cez CSV

- **Rozhodnutie:** klávesy, enkódery ani tlačidlá FM-1 sa na module hardvérovo nekopírujú. USB-MIDI ovládač sa zapojí
  do USB-A hostu na P4. CSV profil mapuje MIDI správy na udalosti panela FM-1 (klávesy, enkódery, tlačidlá).
- **Referenčný ovládač:** M-VAVE SMK-25 II: 25 kláves s velocity (FM-1 má 27, zvyšok rieši posun oktávy),
  8 nekonečných 360° enkóderov (= 7 enkóderov FM-1 + MASTER), 16 RGB padov (= 14 tlačidiel FM-1 + 2 rezerva).
- **Prečo:** mechanické klávesy a pady s velocity sú najťažšia a najdrahšia časť; hotový ovládač je odolný a lacný.
  Modul ostane malý.
- **Pravidlá mapovania:**
  - Noty použité ako klávesy panela FM-1 a noty hrané ako hudba sa musia oddeliť (vlastný MIDI kanál alebo prepínací pad),
    inak sa zvuk spustí dvakrát.
  - Enkódery posielajúce absolútne 0–127 sa prevedú na relatívne kroky; skutočný relatívny režim je lepší.
- **Nezdokumentované, zmeria sa MIDI monitorom:** či enkódery SMK-25 II posielajú relatívne hodnoty a či sa dá farba
  padov nastaviť z hostu cez MIDI.
- **Zamietnuté:**
  - Zabudovať P4 do lacnej MIDI klaviatúry (prepojenie jej matice je viac práce ako PCB; zvyčajne potenciometre, bez ovládania LED).
  - Mriežky NeoTrellis / NeoKey (modul príliš široký; bez velocity; silikón bubnovanie neprežije).
  - Novation Launchkey MK3 ako referencia: má potenciometre s dorazom (nekonečné enkódery prišli až v MK4).
- **Pozor pri kúpe:** názov „SMK-25“ sa používa pre tri rôzne produkty. Sedí len **SMK-25 II** (16 padov, 8 enkóderov).
  SMK-25 / MVA-25 má 8 padov; pôvodný SMK-25 mini má 1 gombík.

## D6. Zvuk: moduly PCM5102A + PCM1808, nie ES8388

| | DAC SNR / THD+N | ADC SNR / THD+N |
|---|---|---|
| ES8388 | 96 dB / −83 dB | 95 dB / −85 dB |
| PCM5102A + PCM1808 | 112 dB / −93 dB | 99 dB / −93 dB |

- **Prečo:** lepšie parametre, žiadny I2C ovládač (nastavenie pinmi), jedna I2S zbernica s P4 ako zdrojom hodín, takže
  vstup a výstup sú synchrónne na vzorku. Oba existujú ako moduly s kolíkovou lištou, nosná doska tak ostane len THT.
  ES8388 je v puzdre QFN-28, ručne sa spájkovať nedá.
- **Cena za to:** PCM5102A nemá slúchadlový zosilňovač, preto D7. PCM1808 potrebuje MCLK (256/384/512 fs) z P4.
- **Riziko pre open source:** klony modulov majú rôzne rozloženie pinov. v1 cieli na jeden konkrétny modul, zdokumentovaný
  fotkou a pinoutom. Neskoršia revízia môže čipy osadiť priamo (SMT osadenie v JLC) kvôli reprodukovateľnosti.

## D7. Slúchadlový výstup: NJM4556AD

- **Prečo:** DIP-8, 70 mA na kanál, osvedčená voľba pre slúchadlá. V pätici, dá sa nahradiť (napr. 4580).
- **Obmedzenia:** výrobca ho vedie ako „Not For New Designs“, ale je bežne dostupný. Z jednoduchých 5 V je výstup
  asi 0.8 Vrms: na 32 Ω slúchadlá nahlas, na 250 Ω a viac slabo. Menič ±5 V sa dá doplniť neskôr.
- Hlasitosť je digitálna (MASTER enkóder cez mapovanie), bez potenciometra.

## D8. Konektory: všetky 3.5 mm, THT

Line out, line in, slúchadlá, MIDI IN, MIDI OUT: 5× 3.5 mm TRS jack do DPS, na jednej hrane dosky v jednej výške,
aby lícovali s 3D tlačeným predným panelom.

## D9. MIDI: TRS IN aj OUT na nosnej doske

- **Prečo:** FM-1 má len TRS MIDI IN (plus USB a BLE MIDI). OUT pridá posielanie hodín a nôt do iných zariadení.
- IN: optočlen H11L1 (galvanické oddelenie, MIDI štandard). OUT: sériové odpory podľa 3.3 V MIDI špecifikácie.
- Na trhu existuje TRS Type A aj Type B; prepojka pri každom jacku volí zapojenie. Predvolené: Type A (FM-1, MMA RP-054).
- Felucca sama hardvérový MIDI OUT neposiela; smerovanie na UART sa pridá v P4 hardvérovej vrstve.

## D10. Ochrana vstupov a výstupov („ktokoľvek tam pichne čokoľvek“)

| Port | Ochrana |
|---|---|
| Line in | oddeľovací kondenzátor, útlm 0 / −12 dB prepojkou, sériový odpor, ochranné diódy 1N4148. PCM1808 má plný rozsah 3 Vpp (≈1 Vrms), profesionálny signál +4 dBu a Eurorack úrovne potrebujú útlm. |
| Line out | sériové odpory 100 Ω (odolné proti skratu) |
| Slúchadlá | sériové odpory ~10 Ω na výstupoch NJM4556 |
| MIDI IN | galvanické oddelenie optočlenom |
| MIDI OUT | sériové odpory |

## D11. Ladenie a záložné ovládanie

- Enkóder EC11 s tlačidlom a jedna RGB stavová LED, priamo na GPIO (bez modulu).
- Použitie: podržanie pri štarte = bezpečný režim / reset nastavení; ovládanie menu, ak zlyhá dotyk.
- Skutočné ladenie je sériový log cez USB-C UART na hlavnej doske. LED ukáže stav, keď nefunguje displej
  (boot OK, chýba audio, chýba MIDI zariadenie, chyba displeja).
- Nastavenia a mapovania sú CSV súbory na microSD, upraviteľné na PC, keď zlyhá všetko ostatné.

## D12. Nosná doska: len THT, vyrobená v továrni na DPS

- Žiadna univerzálna doska. Návrh v KiCade, objednávka v čínskej továrni na DPS, ručné spájkovanie. SMD len vtedy,
  ak neskoršia revízia použije osadenie vo výrobe.
- Doska je za 3D tlačeným predným panelom; KiCad exportuje STEP, aby sa panel modeloval podľa skutočnej dosky.

## D13. Napájanie: len USB-C 5 V

Vo v1 bez batérie.

## D14. Etapa 1 je prototyp na kábloch z hotových modulov

- **Rozhodnutie:** pred akýmkoľvek PCB rozbehať dosku ESP32-P4 so 4.3" displejom + PCM5102A + PCM1808 + EC11 + RGB LED + SMK-25 II
  na káblikoch. Pozri [stage1-prototype.sk.md](stage1-prototype.sk.md).
- **Prečo:** lacno overí displej, hodiny zvuku, USB host a ovládanie; nosná doska potom stavia na tom, čo funguje.
- **Pridelenie pinov:** [pinmap.csv](pinmap.csv) je jediný zdroj pravdy pre firmvér aj KiCad.

## D15. Stavová LED: RGB LED so spoločnou katódou na troch PWM pinoch, nie WS2812

- **Prečo:** hlavná doska má len LED napájania. WS2812 potrebuje napájanie ≥3.5 V a dátovú úroveň nad 3.3 V logikou;
  obyčajná RGB LED ide priamo z 3.3 V GPIO, aj ako THT diel na nosnej doske.

## D16. Displej 4-DSI-TOUCH-A potvrdený pre ESP32-P4 (nahradené D18)

> Samostatný 4" panel sa už nepoužíva; 4.3" panel je súčasťou dosky z D18. Ponechané kvôli histórii.

- Waveshare BSP `esp32_p4_platform` uvádza 4-DSI-TOUCH-A (ST7701, `esp_lcd_st7701`) ako otestovaný.
- Panel je natívne na výšku 480×800: obraz FM-1 240×240 v 2× zaplní horných 480×480 a pod ním ostane 480×320,
  bez akejkoľvek rotácie.
- Napájanie: 5 V, ≥180 mA.

## D17. Etapa 1 bez breadboardu: prispájkované vývody s Dupont konektormi

- **Rozhodnutie:** žiadny breadboard. Moduly idú na lištu hlavnej dosky káblikmi samica–samica. RGB LED s odpormi
  a enkóder dostanú prispájkované vývody zakončené Dupont samicami; LED sedí ako jedno puzdro 2×2 na pinoch 31–34 lišty J3, enkóder ako jedno na pinoch 37–40.
- **Prečo:** breadboard sa pri prenášaní a otrasoch rozpája a pri hľadaní chýb pridáva neistý kontakt.
  Spájkované vývody vydržia prenos aj testovanie.
- **Nosná doska:** LED, odpory a enkóder sú tam priamo na PCB, používateľ nič ručne nezapája.

## D18. Hlavná doska: Waveshare ESP32-P4-WIFI6-Touch-LCD-4.3 (WS-33874)

- **Rozhodnutie:** nahrádza P4-NANO a samostatný 4" panel (D3, D16). Doska ESP32-P4 pripevnená na zadnej strane
  4.3" dotykového panelu 480×800 (ST7701/GT911) so 40-pinovou lištou (J3). Nosná doska sa nasadí na J3 zozadu.
- **Prečo:**
  - jeden diel namiesto dvoch, asi o 19 € lacnejšie (rlx: 36,78 € oproti 21,19 € + 34,32 €);
  - žiadny DSI kábel ani samostatné napájanie displeja;
  - rovnaké rozlíšenie aj radič displeja, obraz FM-1 240×240 v 2× stále zaplní 480×480;
  - nosná doska sa len zasunie: staviteľ nerieši, čo kam zapojiť;
  - Ethernet nepotrebujeme.
- **GPIO:** na J3 je 17 voľných GPIO (3, 4, 5, 21, 22, 28–32, 46–52); návrh potrebuje 13 (I2S 5, enkóder 3, LED 3,
  MIDI 2). Pridelenie v [pinmap.csv](pinmap.csv). Na doske a mimo J3: ES8311/ES7210 na GPIO9–13, zosilňovač
  GPIO53, LCD/podsvietenie/reset dotyku GPIO23/26/27/33, microSD GPIO39–44.
- **USB host:** OTG Type-C nedáva von 5 V (jeho VBUS len napája dosku cez ideálnu diódu, AO3401 + MMDT3906).
  Dátové linky HS OTG sú aj na pinoch 25/27 lišty J3; nosná doska má USB-A host port s VBUS z vlastného napájania.
  SMK-25 II je full-speed zariadenie, vedenie cez lištu nie je kritické. Etapa 1 použije hub s napájaním.
- **Napájanie (D13 bez zmeny):** nosná doska privádza 5 V na piny 1/3 lišty J3 (VCC_5V). Každý zdroj 5 V na doske
  vstupuje do VCC_5V cez vlastnú ideálnu diódu, takže USB-UART môže ostať pripojený na nahrávanie bez spätného napájania.
- **Otvorený bod:** medzi VCC_5V a napájaním systému je tlačidlový vypínač (ECJ23001). Či sa doska po privedení 5 V
  zapne sama, sa zistí pri prvom zapnutí; návrh nosnej dosky od toho závisí.
- **Nepoužíva sa:** reproduktor a zosilňovač na doske, mikrofóny, konektor kamery, konektor batérie (v1 bez batérie).
- **Zdroj:** schéma Waveshare Rev1.2 a audit hardvéru BSP v
  [waveshareteam/ESP32-P4-WIFI6-Touch-LCD-4.3](https://github.com/waveshareteam/ESP32-P4-WIFI6-Touch-LCD-4.3).

## D19. Ladiaci enkóder a stavová LED: na presahu nosnej dosky, vpravo od displeja

- **Rozhodnutie:** nosná doska presahuje za pravý okraj displeja. Enkóder EC11 je osadený na prednej strane presahu
  a jeho hriadeľ prechádza predným panelom; RGB LED je priamo nad ním.
- **Prečo:** nosná doska je za displejom, takže všetko na jej zadnej strane mieri od používateľa. Sklenený okraj
  široký 10 mm ovládacie prvky neunesie. Presah drží všetko na jednej zasúvacej doske.
- **Mechanika:** zadná strana hlavného PCB je 7.15 mm za čelom skla a J3 pridá 5.1 mm, takže plocha nosnej dosky je
  asi 15 mm za čelom; EC11 s 20 mm hriadeľom prejde panelom. Presné hodnoty dá STEP export z KiCadu.
- **Zadná strana:** audio jacky, TRS MIDI, USB-A host a USB-C napájanie ostávajú na zadnej strane nosnej dosky.
