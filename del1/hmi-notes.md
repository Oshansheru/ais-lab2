# HMI — Node-RED Modbus-klient

## Anslutning
- Host: 10.0.50.10
- Port: 502 (Modbus TCP)
- Unit ID: 1

## Polling
- FC3 (Read Holding Registers)
- Adress 0, Quantity 4 (läser HR0–HR3 i en bulk-fråga)
- Rate: 1 sekund

## Mapping i HMI
HR0 → Setpoint gauge
HR1 → TankLevel gauge + trend
HR2 → PumpSpeed gauge
HR3 → ChlorineLevel gauge
