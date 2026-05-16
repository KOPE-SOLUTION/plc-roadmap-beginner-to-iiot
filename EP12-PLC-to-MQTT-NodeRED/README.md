# EP12 — PLC to MQTT / Node-RED

This episode introduces Industrial IoT communication using Siemens S7-1200 PLC.

The goal is to send PLC machine data to Node-RED and publish it to an MQTT Broker.

![Thumbnail](Thumbnail-ep12.png)

---

## System Architecture

```mermaid
flowchart LR

    A[PLC S7-1200] --> B[DB100 Communication Data]
    B --> C[Node-RED]
    C --> D[MQTT Broker]
    D --> E[Dashboard / Subscriber]
```

---

## Main Goal

In this EP, we will build the first Industrial IoT data flow: `PLC → Node-RED → MQTT Broker`

The PLC still controls the machine.

Node-RED reads PLC data and publishes it as MQTT messages.

---

## Installation References

This EP assumes that MQTT Broker and Node-RED are already installed.

Installation tutorials are provided separately.

MQTT Broker Installation Video : https://youtu.be/bqaUgJ9S1r4?si=shDTQIy2qeHcs2Lk

Node-RED Installation Video : https://youtu.be/Uv2iJd1kcRI?si=D-uoV6qVMWoPAJWk

---

## Connection Information

> Do not commit real credentials to a public repository.

## MQTT Broker

| Item           | Value                |
| -------------- | -------------------- |
| Host           | `<MQTT_BROKER_HOST>` |
| Port           | `<MQTT_BROKER_PORT>` |
| Authentication | Enabled              |
| Username       | `<MQTT_USERNAME>`    |
| Password       | `<MQTT_PASSWORD>`    |


## Node-RED

| Item           | Value                                    |
| -------------- | ---------------------------------------- |
| URL            | `http://<NODE_RED_HOST>:<NODE_RED_PORT>` |
| Authentication | Enabled                                  |
| Username       | `<NODE_RED_USERNAME>`                    |
| Password       | `<NODE_RED_PASSWORD>`                    |


---

## DB100 Communication Data

Use the same DB100 from EP11.

| Tag                  | Type | Address      | Description        |
| -------------------- | ---- | ------------ | ------------------ |
| PC_Start_Command     | Bool | DB100.DBX0.0 | Start Command      |
| PC_Stop_Command      | Bool | DB100.DBX0.1 | Stop Command       |
| PC_Reset_Command     | Bool | DB100.DBX0.2 | Reset Command      |
| PLC_Run_Status       | Bool | DB100.DBX0.3 | Machine Running    |
| PLC_Alarm_Status     | Bool | DB100.DBX0.4 | Alarm Active       |
| PLC_Emergency_Status | Bool | DB100.DBX0.5 | Emergency Active   |
| PLC_Ready_Status     | Bool | DB100.DBX0.6 | Machine Ready      |
| PLC_Production_Count | Int  | DB100.DBW2   | Production Counter |

---

## Step-by-Step Lab

### Step 1 — Prepare PLC Data

Make sure EP11 is working correctly.

Open Watch Table in TIA Portal and verify:

| Data                 | Expected                                |
| -------------------- | --------------------------------------- |
| PLC_Run_Status       | Changes when machine runs               |
| PLC_Alarm_Status     | TRUE when alarm active                  |
| PLC_Emergency_Status | TRUE when emergency pressed             |
| PLC_Ready_Status     | TRUE when machine ready                 |
| PLC_Production_Count | Increases when production count changes |

![step-1-1](step-1-tia-watch-1.png)

![step-1-2](step-1-tia-protection-1.png)

![step-1-3](step-1-tia-protection-2.png)

![step-1-4](step-1-tia-db100-1.png)

![step-1-5](step-1-tia-db100-2.png)

### Step 2 — Open Node-RED

Open Node-RED in a web browser: `http://<NODE_RED_HOST>:<NODE_RED_PORT>`

Login using your own Node-RED credential.

![step-2](step-2-nodered-login-page.png)

### Step 3 — Install Required Node-RED Nodes

In Node-RED: `Menu → Manage palette → Install`

Install a Siemens S7 communication node, for example: `node-red-contrib-s7`

You may also use other S7-compatible nodes depending on your environment.

![step-3](step-3-manage-palette.png)

### Step 4 — Create S7 PLC Connection

Create a PLC connection in Node-RED.

Example settings:

| Setting        | Value              |
| -------------- | ------------------ |
| PLC IP Address | `<PLC_IP_ADDRESS>` |
| Rack           | `0`                |
| Slot           | `1`                |
| Cycle Time     | `1000 ms`          |

> For Siemens S7-1200, Rack 0 / Slot 1 is commonly used.

![step-4-1](step-4-plc-ip-mode-name.png)

![step-4-2](step-4-plc-address.png)

### Step 5 — Read PLC Data from DB100

Create S7 variables for DB100.

Example:

| Variable  | Address    |
| --------- | ---------- |
| run       | DB100,X0.3 |
| alarm     | DB100,X0.4 |
| emergency | DB100,X0.5 |
| ready     | DB100,X0.6 |
| count     | DB100,INT2 |


> Address format may vary depending on the Node-RED S7 node used.

![step-5](step-5-plc-variable.png)

### Step 6 — Convert PLC Data to JSON

Use a Function node to create an MQTT payload.

Example:

```js
const now = new Date();

const thailandTime = new Date(now.getTime() + (7 * 60 * 60 * 1000))
    .toISOString()
    .replace("Z", "+07:00");

msg.topic = "factory/plc/s7-1200/status";

msg.payload = {
    machine: "S7-1200",
    timestamp: thailandTime,
    timezone: "Asia/Bangkok",
    run: msg.payload.run,
    alarm: msg.payload.alarm,
    emergency: msg.payload.emergency,
    ready: msg.payload.ready,
    count: msg.payload.count
};

return msg;
```

![step-6](step-6-function-node.png)

### Step 7 — Configure MQTT Broker

Add an MQTT Out node.

MQTT broker settings:

| Setting        | Value                |
| -------------- | -------------------- |
| Server         | `<MQTT_BROKER_HOST>` |
| Port           | `<MQTT_BROKER_PORT>` |
| Authentication | Enabled              |

Use your own MQTT credential.

### Step 8 — Publish MQTT Data

Example MQTT topic: `factory/plc/s7-1200/status`

Example MQTT payload:
```js
{
  "run": true,
  "alarm": false,
  "emergency": false,
  "ready": true,
  "count": 12
}
```

![step-8](step-8-mqtt-explorer.png)

### Step 9 — Test MQTT Output

Use one of these methods:
- MQTT Explorer
- Node-RED MQTT In node
- Mosquitto subscriber
- Any MQTT client

<br>

Subscribe to: `factory/plc/s7-1200/status`

<br>

Expected result:
- Press Start → run = true
- Press Stop → run = false
- Press Emergency → emergency = true, alarm = true
- Press Reset → alarm = false, count = 0

### Step 10 — Optional: Send Command from MQTT to PLC

![Part 2](Thumbnail-ep12-2.png)

MQTT can also send commands back to PLC through Node-RED.

Example command topics:

```sh
factory/plc/s7-1200/cmd/start
factory/plc/s7-1200/cmd/stop
factory/plc/s7-1200/cmd/reset
```

<br>

Command flow: `MQTT → Node-RED → DB100 PC Command Bit → PLC Logic`

<br>

Example mapping:

| MQTT Command | PLC DB Address |
| ------------ | -------------- |
| Start        | DB100.DBX0.0   |
| Stop         | DB100.DBX0.1   |
| Reset        | DB100.DBX0.2   |

<br>

Important:
- MQTT command should only request an action.
- PLC safety logic must decide whether the action is allowed.


![step-1](step-10-1-mqtt-command.png)

![step-2](step-10-2-check-cmd.png)

![step-3](step-10-3-start.png)

![step-4](step-10-3-stop.png)

![step-5](step-10-3-reset.png)

![step-6](step-10-4-plc-variable.png)

![step-7](step-10-4-plc-start.png)

![step-8](step-10-4-plc-stop.png)

![step-9](step-10-4-plc-reset.png)


---

## Important Industrial Concept

The PLC still owns:
- Machine Logic
- Safety Logic
- Emergency Handling
- Interlock Logic

<br>

Node-RED and MQTT are used for:
- Monitoring
- Data exchange
- Dashboard preparation
- Logging
- Remote visualization

Safety should NOT depend only on MQTT communication.

---

## Learning Outcome

After this EP, you should understand:
- How PLC data can be exposed through DB100
- How Node-RED reads PLC data
- How Node-RED publishes MQTT messages
- How MQTT can distribute PLC data
- How this becomes the foundation of IIoT

