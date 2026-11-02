# MueveT LoRaWAN Field Data — Tunja, Colombia

Raw field records from four test campaigns of the LoRaWAN/LTE telematic system for the urban collective public transport system (TPCU) of Tunja, Colombia. The system retrofits existing buses, seats and bus stops with low-cost IoT modules (i-Bus, i-Silla, i-Paradero) and delivers the data to the MueveT web platform.

## Repository structure

```
data/
├── test1_campus/
│   └── test1_campus_uptc_ttn.json
├── test2_urban_routes/
│   ├── test2_routes_A-B_ttn.json
│   └── test2_routes_C-D_ttn.json
├── test3_emulated_route/
│   ├── test3_route_3.72km_DR1_ttn.json
│   ├── test3_route_3.72km_DR2_ttn.json
│   └── test3_route_3.72km_DR4_ttn.json
└── test4_extended_route/
    ├── test4_route_6.53km_bus_lorawan_ttn.json
    ├── test4_route_6.53km_bus-stop_lorawan_ttn.json
    ├── test4_route_6.53km_bus_lte_mqtt.json
    └── test4_gateway_locations.kml
```

Files ending in `_ttn.json` are event exports from the LoRaWAN network server (The Things Stack Cloud). The file ending in `_lte_mqtt.json` is an export of the MQTT records stored by the MueveT platform (MongoDB). The `.kml` file contains the locations of the four gateways used in Test 4.

## Test campaigns

| Test | Date | Objective | Devices (network-server ID) | Payload | Radio configuration |
|---|---|---|---|---|---|
| 1 · Campus | 2025-03-20 | First link tests and radial coverage estimation with and without line of sight | LILYGO LoRa32 (`lylgobus`) | 3 B, two pseudo-random integers (`D1`, `D2`) | ADR enabled, SF7–SF10 / 125 kHz |
| 2 · Urban routes | 2025-03-27 | Mobile nodes with ADR along four predefined routes (A–D) | LILYGO LoRa32 (`lylgobus`, `lylgoparadero`) | 3 B, two pseudo-random integers (`D1`, `D2`) | ADR enabled, SF7–SF10 / 125 kHz |
| 3 · Emulated route (3.72 km) | 2025-04-09 | First real payload in a vehicle; comparison of three fixed data rates, one run each | i-Bus + i-Silla in a private vehicle (`ibus` for DR1, `bus2` for DR2 and DR4) | 15 B, bus, seat, occupancy, ZX sensor, position, altitude, speed, PIR count | ADR disabled: DR1 (SF9/125 kHz), DR2 (SF8/125 kHz), DR4 (SF8/500 kHz) |
| 4 · Extended route (6.53 km) | 2026-03-04 | Integrated test: final payload, simultaneous LoRaWAN and LTE, fixed bus-stop node, MueveT platform | i-Bus via LoRaWAN (`pru2otab`), i-Bus via LTE (`lt4modhoy`), i-Paradero (`pru2otap`) | Bus: 38 B. Bus stop: 31 B. LTE: JSON over MQTT | DR2 fixed (SF8/125 kHz, CR 4/5) |

The LoRaWAN tests used the US915 regional plan, sub-band 2.

## Data format

### Network-server exports (`*_ttn.json`)

Each file is a JSON array of network-server events. Uplinks are the events named `as.up.data.forward`; join procedures appear as `as.up.join.forward`. The most relevant fields of an uplink are:

| Field | Meaning |
|---|---|
| `data.end_device_ids.device_id` | Device identifier |
| `data.received_at` | Reception time at the application server (UTC) |
| `data.uplink_message.f_cnt` | Uplink frame counter (resets after each new OTAA join session) |
| `data.uplink_message.frm_payload` | Application payload (Base64) |
| `data.uplink_message.decoded_payload` | Payload decoded by the network-server payload formatter |
| `data.uplink_message.rx_metadata[]` | One entry per gateway that received the frame: `gateway_ids`, `rssi`, `snr`, `received_at` |
| `data.uplink_message.settings.data_rate.lora` | Spreading factor, bandwidth and coding rate |
| `data.uplink_message.settings.frequency` | Uplink channel frequency (Hz) |
| `data.uplink_message.consumed_airtime` | Time on air of the frame |

Some `as.up.data.forward` events contain only the event header (no `data` object), as exported by the console; they correspond to uplinks that were forwarded but whose content was not retained in the export.

### Decoded payload fields

| Test | Fields |
|---|---|
| 1 and 2 | `D1`, `D2`: pseudo-random integers (test frames, no sensor data) |
| 3 | `BUS_ID`, `iSilla` (seat ID), `ZX` (proximity sensor reading), `Oc` (seat occupancy 0/1), `Lat`, `Lon` (two decimals), `Alt` (m), `Speed` (km/h), `Pasajeros` (PIR counter) |
| 4 · bus | `seat_id`, `occupancy` (0/1), `voltage` (V), `current_mA` (mA), `latitude`, `longitude` (five decimals), `altitude` (m), `speed` (km/h), `person_count`, `rpi_timestamp` (local time, UTC−5) |
| 4 · bus stop | `temperature` (°C), `humidity` (%), `co2`, `uv_index`, `noise_level`, `pm25`, `station_timestamp` (local time, UTC−5) |

Each LoRaWAN frame of the bus carries the state of one seat, in rotation.

### LTE/MQTT export (`test4_route_6.53km_bus_lte_mqtt.json`)

A JSON array of the records stored by MueveT. The records of device `lt4modhoy` correspond to the i-Bus transmitting over LTE during Test 4. Main fields:

| Field | Meaning |
|---|---|
| `device_id` | Device identifier |
| `payload` | Bus telemetry, including `vehicle_occupancy` with the state of all seats |
| `payload.rpi_timestamp` | Device timestamp (UTC) |
| `created_at` | Time the record was stored in the platform (UTC) |
| `latency_ms` | Latency computed by the platform between the device timestamp and the reception of the record (ms) |
| `log.uplink_message.decoded_payload` | Message as received over MQTT |

The `radio` and `health_score` fields of the LTE records are placeholders generated by the platform and do not represent radio measurements. The file also contains 28 records of device `pru2otab` dated 2026-04-02; they are not part of Test 4.

## Notes

- Timestamps in the network-server exports are in UTC. Local time in Tunja is UTC−5.
- Packets sent can be obtained from the `f_cnt` range of each device session; packets received are the `as.up.data.forward` events with content.
- Device and gateway identifiers (DevEUI, gateway EUI, device address) are included as exported. The files contain no network or application keys.

## License

This dataset is released under the Creative Commons Attribution 4.0 International License (CC BY 4.0). See `LICENSE`.

## Acknowledgements

Funded by the Ministry of Science, Technology and Innovation of Colombia (Minciencias), call 890 of 2020, project 1109-890-82201, contract CT ICETEX 2023-0788, executed by Universidad Pedagógica y Tecnológica de Colombia (UPTC) through the GIDPOT and TelemaTICs research groups.
