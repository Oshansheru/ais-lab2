# Lab 2 — Del 3: Detektion med Suricata

## Sammanfattning

I denna del av laborationen implementerades en IDS lösning med Suricata för att övervaka Modbus trafik i OT miljön. Suricata placerades på OT bryggan och konfigurerades för att analysera trafik riktad mot PLC:n på port 502. Målet var att upptäcka både otillåtna writes och andra misstänkta aktiviteter mot den industriella processen.

Laborationen fokuserade på att förstå hur signaturbaserad detektion fungerar i OT miljöer samt hur Suricata regler kan användas för att identifiera Modbus baserade attacker.

## Verifiering av IDS miljön

Initialt verifierades att Suricata startade korrekt och att IDS:en faktiskt lyssnade på OT bryggan. Genom att kontrollera containerloggarna kunde det bekräftas att Modbus parsern var aktiv, att reglerna laddats korrekt och att Suricata analyserade trafik på `br-lab3-ot`.

Loggarna visade bland annat att fyra regler hade laddats framgångsrikt och att applikationslagerinspektion för Modbus var aktiverad.

## Befintliga detektionsregler

Miljön levererades med fyra grundläggande Suricata regler som detekterade olika typer av Modbus aktivitet.

Tre av reglerna övervakade Modbus write funktioner:

* FC5 — Force Single Coil
* FC6 — Write Single Register
* FC16 — Write Multiple Registers

Den fjärde regeln larmade på nya TCP sessioner mot port 502, vilket gjorde det möjligt att upptäcka nya anslutningar mot PLC:n.

Eftersom den legitima HMI:n endast använde FC3 reads i normal drift fanns det inga legitima writes i miljön. Alla write operationer kunde därför betraktas som misstänkta aktiviteter.

## Regler — översikt
| SID     | Typ        | Trigger | Allvarlighet |
|---------|-----------|---------|--------------|
| 2000005 | FC5       | Force Single Coil | High (priority 2) |
| 2000006 | FC6       | Write Single Register | High (priority 1) |
| 2000016 | FC16      | Write Multiple Registers | Critical (priority 1) |
| 2001000 | TCP SYN   | Ny session :502 | Medium |
| 1000100 | FC8 (min) | Diagnostics — potential DoS | Critical |
| 1000101 | FC6 (min) | Setpoint > 8000 (unsafe) | Critical |
| 1000200 | FC8 (min) | TCP scan against OT zone | Critical |

## Genomförda attacker och detektion

För att testa IDS reglerna användes färdiga attackscript från jump servern i DMZ zonen. Ett av testen bestod av en FC6 attack där PLC:ns setpoint ändrades till värdet `9999`.

Attacken genomfördes med:

```bash id="k5n3a8"
docker exec lab3-jump python3 /scripts/attack-fc6.py 9999
```

När attacken skickades genererade Suricata två separata alerts i `fast.log`.

Den första alerten kom från regeln för nya TCP sessioner mot port 502 och den andra från regeln som detekterade själva FC6 write kommandot.

Detta gav en tydlig tidslinje över både anslutningen till PLC:n och den efterföljande manipulationen av registervärdet.

## Verifiering av samtliga regler

Därefter kördes `verify-detection.sh`, vilket automatiskt triggar flera attacker och kontrollerar att samtliga Suricata regler fyrar korrekt.

Verifieringen bekräftade att följande signaturer fungerade:

* FC5 — Force Single Coil
* FC6 — Write Single Register
* FC16 — Write Multiple Registers
* Ny TCP session mot port 502

Resultatet visade att samtliga fyra detektioner fungerade korrekt och att Suricata framgångsrikt kunde identifiera write trafik riktad mot PLC:n.

## Egna regler

Efter att grundreglerna verifierats skapades två egna Suricata regler.

Den första regeln detekterade Modbus funktion FC8 (Diagnostics). Funktionen används för diagnostik och kan i vissa fall missbrukas för DoS liknande beteenden mot PLC:n. Regeln genererade därför ett kritiskt larm vid all FC8 trafik.

Den andra regeln övervakade FC6 writes mot registeradress `0` och larmade när setpoint värdet översteg `8000`, vilket definierades som ett osäkert processvärde i den simulerade miljön.

Den tredje egna regeln användes för att upptäcka möjlig nätverksrekognosering mot OT zonen. Regeln triggar ett larm om fem eller fler TCP SYN försök sker från samma källa inom fem sekunder mot OT nätet, vilket kan indikera portskanning eller tidiga steg i en attack.

Regeln kompletterar de Modbus specifika detektionerna genom att även fånga mer generell misstänkt scanning av OT miljön.

På så sätt kunde IDS:en inte bara upptäcka att writes skedde, utan även analysera själva innehållet i Modbus trafiken och avgöra om värdet låg utanför säkra processgränser.

## Regeluppdatering och testning

Efter att de egna reglerna lagts till i `rules/ot.rules` laddades Suricata reglerna om utan att hela containern behövde startas om.

De nya reglerna testades sedan genom att:

* manuellt skicka FC8 diagnostiktrafik
* genomföra en FC6 write med värdet `9999`

Därefter verifierades att de nya signatur ID:na `1000100` och `1000101` dök upp i `fast.log`.

## Begränsningar

Lösningen detekterar främst write baserade attacker mot PLC:n. Reads baserad reconnaissance genererar fortfarande ingen alert eftersom miljön saknar regler för normal lästrafik.

IDS:en kan heller inte skilja mellan legitim och illegitim write trafik om en angripare komprometterar ett redan betrott system, exempelvis HMI:n eller jump servern. Trafiken kan då se legitim ut ur nätverksperspektiv.

Övervakningen sker dessutom endast på okrypterad Modbus trafik. Om kommunikationen istället skulle ske över krypterade protokoll skulle ytterligare lösningar krävas för inspektion.

## Slutsats

Laborationen visade hur Suricata kan användas för att implementera grundläggande OT övervakning och detektion av Modbus baserade attacker.

Genom både färdiga och egenskrivna regler kunde IDS:en identifiera:

* nya anslutningar mot PLC:n
* Modbus write kommandon
* diagnostikfunktioner
* osäkra processvärden

Resultatet blev en enkel men fungerande detektionsmiljö där attacker mot PLC:n kunde observeras i realtid via Suricatas loggar.
