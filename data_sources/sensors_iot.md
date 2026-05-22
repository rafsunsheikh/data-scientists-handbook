# Sensors and IoT

> **TL;RT** — Sensor and IoT data is characterized by high-frequency time series from physical devices, often with irregular sampling, clock drift, calibration drift, and connectivity gaps. Data flows from devices through protocols (MQTT, OPC-UA, Modbus) to brokers and storage (time-series databases, partitioned Parquet). The data science challenges are domain-specific: time synchronization, tag-name normalization, calibration management, edge processing, and the OT/IT boundary that governs how data moves from factory floor to analytics. Understanding the physical meaning of each sensor reading is as important as the statistical methods applied to it.

## 1. Sensor types and parameters

### 1.1 Industrial sensors

| Type | Measures | Typical rate | Notes |
|---|---|---|---|
| **Temperature** | °C, °F, K | 1 Hz – 10 Hz | Thermocouple, RTD, infrared; slow response |
| **Pressure** | psi, bar, Pa | 10 Hz – 1 kHz | Piezoelectric, strain gauge; sensitive to shock |
| **Vibration** | g, mm/s, m/s² | 1 kHz – 50 kHz | Accelerometers; needs FFT for analysis |
| **Flow** | L/min, m³/h, GPM | 1 Hz – 10 Hz | Electromagnetic, ultrasonic, Coriolis |
| **Level** | mm, %, m | 1 Hz | Tank level, ultrasonic or radar |
| **Humidity** | %RH | 0.1 Hz – 1 Hz | Capacitive; slow response, condensation risk |
| **Current / voltage** | A, V, W, VA | 1 kHz – 10 kHz | For motor monitoring, power quality |
| **Rotation** | RPM, degrees | 10 Hz – 10 kHz | Encoders, tachometers |
| **Force / torque** | N, Nm | 100 Hz – 10 kHz | Strain gauges, piezoelectric |
| **Gas / chemical** | ppm, %LEL | 0.1 Hz – 1 Hz | Catalytic, electrochemical, PID |

### 1.2 Environmental / weather sensors

| Type | Measures | Typical rate |
|---|---|---|
| **Anemometer** | Wind speed | 1 Hz |
| **Wind vane** | Wind direction | 1 Hz |
| **Pyranometer** | Solar radiation | 1 Hz – 1 Hz |
| **Rain gauge** | Precipitation | Event-driven |
| **Barometer** | Atmospheric pressure | 1 Hz |
| **Visibility** | METAR visibility | 1 min |

### 1.3 Wearable / health sensors

| Type | Measures | Typical rate | Notes |
|---|---|---|---|
| **Accelerometer** | 3-axis acceleration | 25 – 200 Hz | Motion, step counting |
| **Gyroscope** | 3-axis angular velocity | 25 – 200 Hz | Orientation |
| **PPG** | Blood flow (optical) | 25 – 100 Hz | Heart rate, SpO2 |
| **ECG / EKG** | Heart electrical activity | 250 – 1000 Hz | Medical-grade |
| **GPS** | Lat/lon/alt | 1 – 10 Hz | Location, speed |
| **Barometer** | Altitude change | 1 Hz | Step counting, floor detection |
| **Ambient light** | Lux | 1 Hz | Screen brightness |
| **Skin temperature** | °C | 0.1 – 1 Hz | Wellness, fever detection |

### 1.4 Smart home sensors

| Type | Measures | Protocol | Rate |
|---|---|---|---|
| **Door/window** | Open/closed | Zigbee, Z-Wave, Matter | Event-driven |
| **Motion** | PIR detection | Zigbee, Z-Wave, Matter | Event-driven |
| **Smart plug** | Power consumption | Wi-Fi, Zigbee | 1 Hz |
| **Thermostat** | Temperature, humidity | Wi-Fi, Zigbee | 1 Hz |
| **Smart lock** | Lock/unlock | Bluetooth, Wi-Fi | Event-driven |

## 2. Communication protocols

### 2.1 MQTT (Message Queuing Telemetry Transport)

Lightweight publish/subscribe protocol designed for constrained devices.

**Core concepts:**

- **Topic hierarchy:** `plant1/line3/sensor42/temperature` — hierarchical, wildcard subscriptions (`plant1/+/sensor+/temperature`).
- **QoS levels:** 0 (fire and forget), 1 (at-least-once), 2 (exactly-once).
- **Retain flag:** Last message with retain=true is stored by broker for new subscribers.
- **Last Will and Testament (LWT):** Broker publishes a message when a client disconnects unexpectedly.
- **Payload:** Binary or text (often JSON, but can be binary for efficiency).
- **Sparkplug B:** Standardized MQTT payload for industrial IoT — defines namespace, metrics, and properties.

**Python:**

```python
import paho.mqtt.client as mqtt

def on_message(client, userdata, msg):
    import json
    data = json.loads(msg.payload.decode())
    print(f"{msg.topic}: {data}")

client = mqtt.Client()
client.on_message = on_message
client.connect("mqtt-broker", 1883, 60)
client.subscribe("plant1/line3/sensor+/temperature")
client.loop_forever()
```

### 2.2 OPC-UA (Open Platform Communications Unified Architecture)

Modern industrial communication standard — platform-independent, secure, type-safe.

**Core concepts:**

- **Address space:** Hierarchical namespace of nodes (sensors, devices, alarms).
- **Nodes:** Each node has a node ID, type, attributes (value, timestamp, quality), and methods.
- **Pub/sub and client/server:** Both models supported.
- **Security:** Built-in encryption, authentication, authorization.
- **Discovery:** Servers advertise themselves on the network.
- **History access:** Read historical values from the server.

**Python:**

```python
from asyncua import Client

async def read_sensor():
    client = Client("opc.tcp://plc:4840")
    async with client:
        node = client.get_node("ns=2;s=Line3.Pump1.Temperature")
        value = await node.get_value()
        timestamp = await node.get_timestamp()
        print(f"{value} at {timestamp}")
```

### 2.3 Modbus

Legacy but ubiquitous industrial protocol.

| Variant | Transport | Notes |
|---|---|---|
| **Modbus RTU** | Serial (RS-232/RS-485) | Binary, master-slave |
| **Modbus TCP** | Ethernet (TCP) | Binary over TCP, widely used |
| **Modbus ASCII** | Serial | Text-based, slower |

**Registers:**

- **Coils:** Write-only digital outputs.
- **Discrete inputs:** Read-only digital inputs.
- **Input registers:** Read-only analog inputs (sensor values).
- **Holding registers:** Read/write analog settings (setpoints).

**Python:**

```python
import pymodbus.client

client = pymodbus.client.ModbusTcpClient("plc-ip", port=502)
result = client.read_input_registers(address=0, count=10, slave=1)
values = result.registers
client.close()
```

### 2.4 Other protocols

| Protocol | Use | Notes |
|---|---|---|
| **BACnet** | Building automation | HVAC, lighting, access control |
| **Zigbee** | Smart home / building | Low-power, mesh network, 2.4 GHz |
| **Z-Wave** | Smart home | Sub-GHz, mesh, lower interference |
| **Matter** | Smart home | New standard (2023), over Wi-Fi/Thread/Ethernet |
| **CAN bus** | Automotive / marine | Controller Area Network, multi-master |
| **NMEA 0183** | Marine / GPS | Text-based, sentence format |
| **DLMS/COSEM** | Smart meters | Electricity, gas, water meters |

## 3. Edge processing

Not all processing happens in the cloud. Edge devices (gateways, controllers) do preprocessing.

### 3.1 What happens at the edge

- **Filtering:** Remove noise (moving average, median filter).
- **Feature extraction:** Compute FFT, RMS, peak-to-peak, kurtosis from raw vibration.
- **Aggregation:** Average, min, max, count over 1-minute windows.
- **Anomaly detection:** Simple threshold-based alerts.
- **Compression:** Delta encoding, dead-banding (only send when value changes > threshold).
- **Local storage:** Buffer during network outage, sync when restored.

### 3.2 Why it matters for data scientists

The data you receive from the edge is **not raw** — it's already processed. You need to know:

- What aggregation was applied (mean of 1 min? max of 5 min?).
- What filtering was applied (low-pass, moving average?).
- What was dropped (values within dead-band range).
- What features were extracted (FFT bins, RMS, etc.).

Always document edge processing in your data dictionary.

## 4. Time synchronization

### 4.1 Why it matters

Sensors across a fleet must agree on time for:

- Correlating events across devices.
- Computing time-sensitive features (rate of change, cross-correlation).
- Aligning with external data (weather, grid load).

### 4.2 Protocols

| Protocol | Accuracy | Use case |
|---|---|---|
| **NTP (Network Time Protocol)** | 10–50 ms | General-purpose, internet |
| **PTP (Precision Time Protocol, IEEE 1588)** | Sub-microsecond | Industrial automation, power grid |
| **GPS timing** | Nanosecond | Base station for PTP |
| **Internal clock** | Drifts seconds/day | Low-cost devices without network |

### 4.3 Clock drift

Low-cost RTC (real-time clock) crystals drift. A sensor that thinks it's 10 seconds ahead of reality will produce misaligned data.

**Detection:**

- Compare sensor timestamps with server time on connection.
- Look for jumps or gradual drift in the timestamp residual.

**Correction:**

- Apply offset correction (server time − sensor time).
- Apply drift correction (slope adjustment over time).
- Flag periods with large corrections.

## 5. Calibration and drift

### 5.1 Calibration

Sensors drift from their calibrated value over time due to:

- **Aging:** Component degradation.
- **Environment:** Temperature, humidity, vibration.
- **Contamination:** Dust, corrosion, chemical buildup.
- **Mechanical wear:** Bearings, seals, moving parts.

**Calibration records should track:**

- Date of last calibration.
- Calibration standard used (traceable to national standard).
- Before/after values.
- Adjustment made.
- Next calibration due date.

### 5.2 Impact on data science

- A sensor that drifted 5% over 6 months introduces a **trend that is not real**.
- Models trained on drifted data will fail when calibrated data is reintroduced.
- Always flag data periods affected by known sensor issues.

**Detection methods:**

- **Cross-sensor comparison:** Redundant sensors should agree within tolerance.
- **Physical plausibility:** Temperature can't change by 50°C in 1 second.
- **Model residual analysis:** If a physics-based model consistently over-predicts, the sensor may have drifted.

## 6. Missing data patterns

### 6.1 Causes

| Cause | Pattern | Fix |
|---|---|---|
| **Network outage** | Long gap (minutes to hours) | Impute or flag |
| **Device reboot** | Short gap (seconds) | Forward-fill if safe |
| **Sensor failure** | Flat line, error code, NaN | Flag as bad; replace sensor |
| **Maintenance** | Planned gap (hours to days) | Document; exclude from analysis |
| **Power loss** | Gap + restart artifacts | Flag; check for data loss on restart |
| **Buffer full** | Dropped records (non-contiguous gaps) | Hard to detect; check count expectations |

### 6.2 Quality flags

Industrial data should include a quality flag:

```json
{
    "timestamp": "2024-01-01T12:00:00Z",
    "value": 25.3,
    "quality": "GOOD",  // GOOD, BAD, UNCERTAIN, CALIBRATION_IN_PROGRESS
    "sensor_id": "TEMP_LINE3_PUMP1",
    "units": "celsius"
}
```

## 7. Storage

### 7.1 Time-series databases

See [`databases.md`](databases.md#16-time-series-databases) for details.

| Database | Best for |
|---|---|
| **InfluxDB** | IoT, monitoring, time-range queries |
| **TimescaleDB** | SQL analytics on time-series |
| **TDengine** | Large-scale IoT deployments |
| **QuestDB** | Real-time analytics, SQL + ILP |

### 7.2 Partitioned Parquet on object storage

For large-scale IoT analytics:

```
s3://bucket/sensor_data/
  sensor_id=TEMP_LINE3_PUMP1/
    year=2024/month=01/day=15/hour=00.parquet
    year=2024/month=01/day=15/hour=01.parquet
  sensor_id=VIB_LINE3_FAN1/
    year=2024/month=01/day=15/hour=00.parquet
```

**Advantages:**

- Predicate pushdown (read only specific sensors, specific times).
- Cheap storage on S3/GCS.
- Process with Spark, DuckDB, or Polars.

**Disadvantages:**

- Not designed for high-frequency writes (use a TSDB or stream for ingestion).
- No native time-series operations (resampling, downsampling).

## 8. Privacy and ethics

### 8.1 Location data

GPS from wearables or fleet vehicles can identify individuals. GDPR considers location data personal.

### 8.2 Biometric data

Heart rate, ECG, skin temperature from wearables may be health data (HIPAA in US, MDRR in EU).

### 8.3 Workplace surveillance

Employee wearables raise privacy concerns. Union agreements and local laws may restrict monitoring.

### 8.4 Smart home data

Voice assistants, smart TVs, and cameras collect intimate details of home life. Data sharing with third parties requires explicit consent.

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `paho-mqtt` | MQTT client |
| `asyncua` | OPC-UA client/server |
| `pymodbus` | Modbus TCP client/server |
| `zigpy` | Zigbee protocol stack |
| `gpsd` / `pynmea2` | GPS/NMEA parsing |
| `influxdb-client` | InfluxDB client |
| `timescale-python` | TimescaleDB utilities |
| `tsfresh` | Time-series feature extraction |
| `tslearn` | Time-series machine learning |
| `scipy.signal` | Filtering, FFT, spectral analysis |
| `pyarrow` | Parquet I/O for partitioned sensor data |
| `duckdb` | SQL on sensor data files |

## 10. References

- MQTT Specification (OASIS Standard) — https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/
- OPC-UA Specification (IEC 62541) — https://opcfoundation.org/about/opc-technologies/opc-ua/
- Kleppmann, M. *Designing Data-Intensive Applications*. O'Reilly (2017). (Ch. 11: The Education of a Data Engineer — IoT section)
- Montgomery, D. C. *Introduction to Statistical Quality Control*. (Sensor calibration, SPC)
- Randall, R. B. *Vibration-based Condition Monitoring*. (Sensor physics for vibration)
- Sparkplug B Specification (Eclipse Phoenix) — https://www.eclipse.org/phoenix/sparkplug/
