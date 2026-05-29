SP = 5000
TL = 5000
PS = 0
CL = 300 - 315
FastLog = 10

ICS-CERT Incidentrapport — Lab 2 Del 4
1. Sammanfattning
Rapportdatum: 2026-05-29
Rapportör: [Adam konopa]
Miljö: Lab 2 Del 3 sandbox (lokal OT-simulering)
Incidenttyp: Obehörig manipulation av industriell process via Modbus TCP från komprometterad jump-server
Allvarlighetsgrad: Hög
Påverkade system: Mock PLC (172.31.50.10:502), HMI-process
Status: Hanterad och återställd

En angripare som fått åtkomst till jump-servern genom stulna SSH-uppgifter använde den som fotfäste för att utföra rekognosering och därefter skriva värden direkt till PLC:n via Modbus. Attacken upptäcktes av Suricata IDS och hanterades genom containment, forensisk insamling och återställning via historian.

2. Tidslinje (UTC)
Tid	Händelse
T+0	Angripare genomför nmap-skanning mot OT-nätet via jump-server
T+15s	FC16 bulk-write ändrar flera register (SID 2000016)
T+25s	FC6-write maskerar setpoint till 5500 (SID 2000006)
T+1m	IDS-larm observeras i Suricata fast.log
T+1m30s	Triage: angripar-IP identifieras som jump-server (172.31.50.99)
T+2m	Nödåterställning av setpoint till 5000
T+2m15s	Jump-server isoleras från OT-nätet
T+5m	Evidence-insamling slutförd
T+8m	Process återställd via historian
T+12m	Jump-server återbyggd från ren image
3. Detektion

Incidenten detekterades av Suricata IDS placerad på OT-bryggan (br-lab3-ot). IDS:en genererade flera larm kopplade till både nätverksrekognosering och Modbus-write-operationer.

Följande signaturer var centrala:

SID 2001000 — TCP scanning / nya sessioner mot OT-zonen
SID 2000016 — FC16 bulk-write (registermanipulation)
SID 2000006 — FC6 single register write (setpoint manipulation)

Larmen genererades i nära realtid och gav en tydlig kedja från rekognosering till faktisk påverkan av processvärden.

4. Påverkan
Processintegritet: Setpoint ändrades till 9999 och senare maskerat till 5500. Ingen fysisk skada uppstod i simuleringen, men i verklig miljö hade detta kunnat orsaka överfyllnad eller processinstabilitet.
Tillgänglighet: Jump-server isolerades temporärt (~8 minuter).
Konfidentialitet: Begränsad insyn i processdata via rekognosering (FC3 reads detekterades ej).
Integritet: PLC-register (HR0–HR3 och coils) manipulerades men återställdes via historian.
5. Root cause

Den primära orsaken till incidenten var komprometterad åtkomst till jump-servern, vilket möjliggjorde lateral rörelse in i OT-nätet.

Bidragande faktorer:

Avsaknad av MFA på bastion/jump-server
Brist på SSH-session-logging och audit trails
Ingen autentisering i Modbus-protokollet
Otillräcklig separering mellan åtkomstlager och processlager
6. Containment-strategi

Den valda åtgärden var att koppla bort jump-servern från OT-nätet:

docker network disconnect lab2-del3-sandbox_ot lab3-jump

Denna metod valdes eftersom den stoppade angriparens åtkomst utan att förstöra forensiska bevis på systemet. HMI och historian lämnades opåverkade för att bibehålla processövervakning.

Alternativa åtgärder som avvisades:

Stängning av container (förlust av forensiska data)
Nedstängning av OT-nät (påverkade hela processen och övervakning)
7. Forensiska begränsningar

Flera viktiga begränsningar identifierades:

Ingen full PCAP-loggning av nätverkstrafik (endast Suricata alerts)
Ingen autentisering eller audit-logg på PLC-sidan (protokollbegränsning)
Begränsad SSH-loggning i jump-servern i denna miljö

Dessa brister begränsar möjligheten till fullständig rekonstruktion av attacken.

8. Rekommendationer
Inför MFA på jump-server för att förhindra credential-baserad åtkomst
Aktivera SSH session recording (PAM eller liknande lösning)
Implementera IP-allowlisting för administrativa åtkomster
Inför Modbus write-policy (endast betrodda system får skriva till PLC)
Utöka IDS med recon-detektion (t.ex. port scanning, SID 1000200)
Aktivera full PCAP-insamling i OT-DMZ för forensik
Implementera process-säkerhetsinterlocks direkt i PLC-logik
9. Lessons learned

Incidenten visar att detektering i sig inte är tillräckligt utan måste kombineras med förebyggande kontroller. Suricata kunde identifiera hela attackkedjan, men kunde inte stoppa den innan påverkan uppstod.

Containment genom nätverksisolering (“soft kill”) var effektiv i denna miljö, men i en verklig OT-miljö hade tiden till åtgärd varit kritisk. Detta pekar på behovet av automatiserad respons och striktare åtkomstkontroller i tidiga steg av attackkedjan.

10. Bevis (evidens)

Följande artefakter samlades in:

fast.log — Suricata alerts
eve.json — strukturerade IDS-events
jump-history.txt — angriparens kommandohistorik
plc-state.txt — PLC-register vid incident
timeline.txt — tidslinje extraherad från loggar
ot-network.json + compose-state.txt — systemstatus vid incident
