## Reflection

Lab 2 visar ganska tydligt att OT säkerhet är annorlunda än vanlig IT säkerhet. Det handlar inte bara om datorer och data, utan om riktiga fysiska processer som vattennivåer och ventiler. Små fel kan direkt påverka verkligheten.

En första skillnad jag märkte är att OT system ofta saknar saker som är standard i IT, som inloggning, kryptering och skydd för kommandon. I labben kunde man till exempel skriva direkt till PLC:ns register via Modbus utan lösenord. I IT världen hade det varit helt otänkbart. En annan skillnad är att allt måste fungera i realtid. Systemet kan inte “pausa” för att kontrollera säkerhet, eftersom processen hela tiden måste köras. En tredje skillnad är att vissa saker inte ses som farliga i normal drift, till exempel läsningar (FC3), vilket gör att en angripare kan gömma sig i “vanlig trafik”.

Den del jag lärde mig mest av var Del 3 med Suricata och larm. Det kändes också mest verkligt eftersom det liknar hur säkerhetsövervakning fungerar i verkligheten, där man sitter och tittar på larm och försöker förstå vad som är normalt och vad som är en attack. Del 2 om nätverkssegmentering var också viktig, men kändes mer som en förenklad modell. Del 4 kändes mest realistisk när det gäller vad som händer vid en riktig incident, eftersom man fick se hur snabbt något kan gå fel och hur man måste agera direkt.

Om jag skulle säkra en riktig vattenrenings PLC skulle jag börja med tre saker. Först skulle jag dela upp nätverket så att IT och OT är helt separerade, och bara låta trafik gå genom en kontrollerad punkt (en jump server). För det andra skulle jag se till att bara vissa godkända datorer kan skicka kommandon till PLC:n, så att inte vem som helst kan skriva värden. För det tredje skulle jag sätta upp övervakning som larmar direkt när något ovanligt händer, så man snabbt kan upptäcka attacker.

Det svåraste att skydda i det här systemet är att Modbus inte har något inbyggt skydd alls. Om någon redan är inne i rätt nätverk kan de nästan göra vad de vill med PLC:n. Det gör att säkerheten helt bygger på nätverksgränser och övervakning. Problemet är att om någon lyckas ta sig förbi de skydden, finns det väldigt lite kvar som stoppar dem.

## side note
Hej Erkan, det är din favorit ellev har :)

Jag skriver relfectionen har också (Fillen liger i del 4) för att det kändes lite ber bones att ha en readme fil utan något att läsa, plus var skulle jag kunna sätta denna söta killen någonstans:
![Alt-text](assets/bild.png)

Men det var allt, hoppas att du tycker om vad jag har skrivit och vad jag har gjort, och oavsett när har en bra dag/kvell/natt
