# Identifierad attackyta — Chas OT-range

## Protokoll-nivå
1. **Modbus saknar autentisering**

Modbus har ingen inbyggd autentisering. Alla system som kan nå PLC:n på OT-nätet kan läsa eller skriva register direkt. Det betyder att vem som helst på nätverket kan ändra exempelvis HR0 (Setpoint) dar finns ingen användarkonton verifiering, ingen sessions-token används och ingen kontroll finns för att säkerställa vem som skickade kommandot

2. **Modbus är okrypterat**

All kommunikation skickas i klartext som gör att en angripare som kan sniffa trafiken kan, läsa registervärden, se function codes, analysera kommunikationen och injicera egna Modbus-paket. Detta gör protokollet sårbart för exempelvis MITM-attacker och trafikavlyssning.

3. **Läsningar genererar inga larm**

I normalläget läser HMI:n kontinuerligt data från PLC:n via: FC1, FC2 ,FC3 och FC4
Problemet är att en angripare kan göra samma sak utan att sticka ut från normal trafik, Det innebär att reconnaissance/enumering är svårt att upptäcka, IDS-systemet reagerar främst på writes och att en angripare kan kartlägga systemet relativt tyst.

## Nätverksnivå
4. **Dual-homed attacker-host.**

   Attackmaskinen är ansluten till både IT-nätet (10.0.10.99) och OT-nätet (10.0.50.99). Detta bryter segmenteringen mellan IT och OT och innebär att en kompromettering på IT-sidan potentiellt kan ge direkt åtkomst till PLC:n i OT-miljön. Designen representerar ett vanligt problem där bristfällig segmentering skapar en direkt väg mellan administrativa system och industriella kontrollsystem.

5. **Inget skrivskydd mellan IT och OT.**

   Det finns ingen brandvägg, DMZ eller data-diod mellan IT- och OT-segmenten. Trafiken passerar endast via en enkel bridge utan filtrering eller begränsning av Modbus-trafik. Det innebär att nätverksseparationen i praktiken är mycket svag.

## Tillämpningsnivå
6. **OpenPLC standardlösenord.** 

   OpenPLC använder standardinloggningen openplc/openplc, vilket är offentligt kända credentials. En angripare som får åtkomst till webbgränssnittet skulle potentiellt kunna stoppa PLC:n, ändra konfiguration eller ladda upp ny PLC-kod. I labbmiljön är vissa delar instruktörslåsta, men grundproblemet finns fortfarande kvar.

7. **HMI:n saknar autentisering till PLC:n.**

   Node-RED skickar inga credentials när den kommunicerar med PLC:n. Detta beror på att Modbus saknar stöd för autentisering. Resultatet blir att PLC:n inte kan avgöra om kommunikationen kommer från den legitima HMI:n eller från en angripare på nätverket.

## Övergripande
8. **Ingen loggning av writes på applikationsnivån.** 
   
   Suricata kan logga nätverkstrafik och upptäcka vissa writes, men PLC:n själv sparar ingen tydlig audit-logg. Det går därför inte enkelt att avgöra vem som ändrade ett register, när ändringen skedde eller från vilken klient kommandot skickades. Detta försvårar både incidenthantering och forensisk analys.

9. **Inget incident response-flöde.** 
   
   Larm från IDS visas endast i ett webbgränssnitt och det finns ingen automatisk eskalering, notifiering eller integration mot exempelvis ticketing-system eller SOC-processer. Systemet kan därför upptäcka vissa attacker, men saknar funktioner för att hantera dem effektivt efter att de identifierats.

## Vad jag rangordnar högst (med motivering)
| # | Risk            | Sannolikhet | Konsekvens | Motivering |
|---|----------------|-------------|------------|------------|
| 4 | Dual-homing    | Hög         | Hög        | Direkt vägen in |
| 1 | Saknad authn   | Hög         | Hög        | Protokollet är så |
| 6 | Default creds  | Hög         | Hög        | Kända offentligt |
| 2 | Klartext       | Medel       | Medel      | Kräver MITM-position |
| 8 | Ingen audit-log| Medel       | Medel      | Postmortem-problem |
