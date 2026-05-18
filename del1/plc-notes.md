# PLC — water_treatment.st (sammanfattning)

## Adresskarta
| Tag    | Modbus           | Variabel       | Init |
|--------|------------------|----------------|------|
| %QW0   | HR0              | Setpoint       | 5000 |
| %QW1   | HR1              | TankLevel      | 3000 |
| %QW2   | HR2              | PumpSpeed      | 0    |
| %QW3   | HR3              | ChlorineLevel  | 600  |

## Logik (sammanfattning på 3 rader)
1. If tanklevel is smaller then setpoint, then calculate delta and set vaule for delta by Setpoint-Tanklevel then set value for PumpSpeed by delta/2. If pumpSpeed is grater then 8000 set PumpSpeed value to 8000. Set tankLevel value by Curent tanklevel value + PumpSpeed value/50, continue until TankLevel value is grater or equle to SetPoint value. Every 5 cycles, set cycle to 0 and set TankLevel value to tanklevel - 20.   
2. Every 30 cycles set cyucle to 0 and if ChlorineLevels Is grater then 50 then set chlorineLevel value to chlorinelevel -5. If chlorinelevel is smaller then 300 then dosingvalve equals 1 and set chlorinelevels to chlorinlevel +2 if otherwise dosingvalve is equal to 0.
3. If tanklevel value is grater then Setpoint value + 2000 then Alarm value is equal to 1, else alarm value is equal to 0

## Säkerhetsobservationer (kort)
- Setpoint är ett vanligt holding register — ingen extra skrivskydd
- Larmet är reaktivt, inte preventivt — det går först EFTER att TankLevel skjutit över
- If somebody would set setpoint value to grater nummber then the capacity of the tank the larm would never go off unless somehow adjusted.
- There is no alarm for if PumpSpeed becoumse too grate of a value.
