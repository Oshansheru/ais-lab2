# Lab 2 — Del 2: IT/OT-nätverkssegmentering

## Sammanfattning

I denna del av laborationen implementerades nätverkssegmentering enligt Purdue-modellen genom att separera miljön i tre Docker-nätverk: IT, DMZ och OT. Syftet var att säkerställa att IT-zonen inte kunde kommunicera direkt med OT-zonen och att all legitim åtkomst till PLC:n istället gick via en jump-server i DMZ-segmentet.

Miljön bestod av flera containrar med olika roller. `attacker` representerade en klient i IT-zonen, `mock-plc` representerade OT-miljön och `jump-server` fungerade som en bastion-host mellan zonerna. En `historian` placerades i DMZ för att kunna läsa produktionsdata från PLC:n utan att exponera OT-nätet direkt mot IT-sidan.

## Identifierad brist

Vid den initiala verifieringen med `verify.sh` upptäcktes att `attacker`-containern kunde nå `mock-plc:502` direkt. Detta bröt mot grundregeln i Purdue-modellen att IT aldrig ska kommunicera direkt med OT.

Problemet berodde på att `attacker` var ansluten till både IT- och OT-nätverket i `docker-compose.yml`. Containern var alltså dual-homed och hade därmed direkt nätverksåtkomst till PLC:n.

Den initiala verifieringen visade därför:

```text
✗ BROKEN attacker (IT) → mock-plc:502 (OT) (expected BLOCK, got ALLOW)
```

Samtidigt fungerade de legitima kommunikationsvägarna redan korrekt.

## Genomförd åtgärd

För att åtgärda segmenteringsbristen togs OT-nätverket bort från `attacker`-containerns `networks:`-block i `docker-compose.yml`.

Före ändringen var containern ansluten till både:

```yaml
networks:
  it:
  ot:
```

Efter ändringen återstod endast:

```yaml
networks:
  it:
```

När miljön därefter deployades om med:

```bash
docker compose up -d
```

byggde Docker om containerns nätverksanslutningar och attacker-containern förlorade sin direkta route till OT-segmentet.

## Resulterande regelmatris

| Från     | Till         | Förväntat | Faktiskt |
|----------|--------------|-----------|----------|
| attacker | mock-plc:502 | BLOCK     | BLOCK ✓  |
| attacker | jump-srv:22  | ALLOW     | ALLOW ✓  |
| jump-srv | mock-plc:502 | ALLOW     | ALLOW ✓  |
| historian| mock-plc:502 | ALLOW     | ALLOW ✓  |
| historian| internet     | ALLOW     | ALLOW ✓  |
| mock-plc | internet     | BLOCK     | BLOCK ✓  |

## Varför detta fungerade

Docker-nätverk fungerar som separata L2-segment. Om en container inte är ansluten till ett specifikt nätverk får den inget interface eller någon route till det nätverket. Trafiken blockeras därför redan av Docker-runtimen innan några iptables-regler ens behöver användas.

Detta gav effektiv zone-isolation mellan IT och OT enligt Purdue-principerna.

## Verifiering

Efter ändringen kördes `verify.sh` igen och samtliga kontroller blev godkända.

Den illegitima vägen mellan IT och OT blockerades:

```text
✓ OK attacker (IT) → mock-plc:502 (OT) (expected BLOCK, got BLOCK)
```

Samtidigt fortsatte de legitima kommunikationsvägarna att fungera:

* attacker → jump-server:22
* jump-server → mock-plc:502
* historian → mock-plc:502
* historian → internet

Resultatet visade att segmenteringen nu fungerade korrekt utan att påverka nödvändiga flöden mellan zonerna.

## Jump-serverns roll

Jump-servern placerades i DMZ-zonen och fungerade som den enda tillåtna vägen från IT in till OT. Detta följer bastion-mönstret som används i många industriella miljöer.

Genom att all administrativ åtkomst passerar jump-servern skapas en central kontrollpunkt där autentisering, loggning och eventuell MFA kan implementeras. Om en klient i IT-zonen komprometteras kan angriparen inte längre nå PLC:n direkt utan måste först ta sig igenom jump-servern.

Detta minskar attackytan och förbättrar möjligheten till övervakning och audit-loggning.

## Begränsningar

Segmenteringen skyddar dock inte mot alla typer av attacker. Om jump-servern komprometteras finns fortfarande en möjlig väg in i OT-miljön. Lösningen innehåller heller ingen IDS eller avancerad filtrering av OT-intern trafik.

Laborationen fokuserar främst på grundläggande zonseparering enligt Purdue-modellen och inte fullständig industriell säkerhetsarkitektur.

## Slutsats

Efter korrigeringen uppfyllde miljön de viktigaste segmenteringskraven enligt Purdue-modellen:

* IT kunde inte längre kommunicera direkt med OT
* OT hade ingen direkt internetåtkomst
* all legitim åtkomst till PLC:n gick via jump-servern i DMZ

Verifieringen med `verify.sh` bekräftade att samtliga nätverksregler fungerade som förväntat.
