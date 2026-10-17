# Research Brief: The Interoception Node
2026-10-16 · R&D scout · verified against Arduino docs unless marked *(unverified)*

## 1. Hardware reality check — "UNO Q" is real, and it changes the design
The brief assumed "UNO Q" was a typo for UNO R4 WiFi. It is not: **Arduino UNO Q** (launched ~late 2025) is a dual-processor board — a Qualcomm Dragonwing **QRB2210** (quad Cortex-A53, 2.0 GHz) running **full Debian Linux**, plus an **STM32U585** MCU (Cortex-M33, sketches via Zephyr) behind classic UNO headers. It has Wi-Fi 5 dual-band + BT 5.1, a **Qwiic/Modulino I²C connector**, USB-C, microSD, 2 GB/4 GB variants. (Arduino docs say QRB2210/STM32U585; some retail copy says "QC-6490" — final silicon *(unverified)*. Design is silicon-agnostic.) Design for what you own; ignore the R4 WiFi fallback unless a unit dies.

This beats the R4 WiFi for us: the Linux side runs Python, systemd, and git natively — the node can sense (STM32/GPIO), decide (Python), and speak HTTPS with no host PC involved. Caveats: young product; Debian means more failure surface than a bare MCU; long-uptime reliability *(unverified)*. Mitigate with systemd `Restart=always` watchdogs.

**Sensors it can host** (classic UNO pinout — check 5 V/3.3 V levels per datasheet): PIR motion (HC-SR501, digital in), LDR or BH1750 for lux (I²C), SHT31/DHT22 temp+humidity via Qwiic, sound *level* via MAX4466 mic into an ADC pin (RMS-derived dB only — never samples). **Power:** USB-C wall wart; small UPS hat optional *(unverified availability)*. **Networking:** Wi-Fi → HTTPS out; no inbound ports.

**Second unit:** keep it as the **dev/staging node** on the bench. Solo-dev discipline: node 1 runs 24/7 in the room; firmware/threshold changes are tested on node 2 before touching production. (A bedroom second-room node is the v2 upgrade path; a mailbox sensor wastes a Linux SBC.)

## 2. Bridge pattern — MCU → host → git
- **(a) USB-serial daemon on WSL2 laptop.** Works via usbipd-win, but quirks are real: bootloader re-enumeration breaks attach mid-flash, `/dev/ttyACM0` permission issues, attach dies on USB stack resets. Fatal flaw: laptop asleep → zero data and zero history. Reject.
- **(b) Wi-Fi + MQTT broker (mosquitto) on the laptop.** Standard, but adds a broker to maintain and *still* dies with the laptop. Overkill for one node, one consumer. Reject for now.
- **(c) Wi-Fi → HTTPS directly to our existing Cloudflare Worker (the vault), which commits to the agent repo. Recommended.** The node POSTs signed state; the Worker writes via GitHub API. Survives laptop sleep (data keeps flowing into git history), zero laptop dependencies, and staleness is *structurally* visible: if commits stop, the file's timestamp ages. Solo-dev cost is one Worker route (~50 lines).

**Failure modes visible, not silent:** the agent never trusts the file alone — it computes `age = now - sample_ts`; `age > 20 min` ⇒ stale regardless of content. Hourly heartbeat commits distinguish "quiet room, node alive" from "node dead" — without them, 90 min of no-motion is indistinguishable from a dead node.

## 3. Data shape
One small JSON committed at `body/room.json`, on change-threshold only (motion edge, lux bucket shift, ΔT > 0.5 °C) plus the hourly heartbeat — not every tick:

```json
{"schema":1,"node":"uno-q-1","sample_ts":"2026-10-16T23:40:00Z",
 "motion":false,"minutes_since_motion":92,"lux":0.4,
 "temp_c":21.3,"rh":44,"sound_db":31,"occupancy_guess":"likely_asleep"}
```

`occupancy_guess` is a labeled heuristic, never ground truth. The repo carries a **manifest** (`body/manifest.json`): capabilities dictionary, sensor list, thresholds, commit policy — the agent reads its own body.

**Privacy:** presence inference reveals sleep/absence patterns — repo stays **private**, no mirrors. No audio content ever leaves the device; only derived dB. Commit history itself is occupancy metadata: acceptable inside the private agent repo, disclosed to Casey.

## 4. Effort and the one-weekend build
Estimate: **~14–20 h** — firmware ~150 lines, Worker route ~50 lines, manifest, tuning.

- **Sat AM:** dev node (unit 2): PIR + SHT31 + LDR wired; STM32 sketch emits sensor lines via the Arduino Bridge RPC.
- **Sat PM:** Python service on Linux side: threshold logic, heartbeat, HTTPS POST to Worker with shared secret.
- **Sat eve:** Worker route → GitHub commit; `body/room.json` + manifest land in repo; staleness check added to agent's context loader.
- **Sun AM:** deploy unit 1 in the room; tune thresholds (PIR sensitivity, lux buckets).
- **Sun PM:** kill-switch test (unplug node, confirm agent reports stale, not silent); write manifest; decide v2 (second room).

**Do NOT build yet: no actuators.** No relays, lights, speakers, locks, camera (MIPI-CSI port exists — explicitly out of scope). The council already downgraded "reflex node" to interoception-only because actuation trust milestones were not met; that decision stands until the agent reliably *reads* the room across weeks of commits. No OTA fleet tooling, no Home Assistant, no ML presence models — heuristics first.

## 5. Why interoception beats reflex-actuation right now
This agent's bottleneck isn't lacking effectors — it's lacking a body-schema: it pings about flaky CI at 3 am because its world contains only code and logs. Sensing the room is the cheapest possible upgrade to that world-model, and it flows through the agent's native sensory channel — commits — so presence, light, and quiet hours arrive the way CI results do, queryable by the same machinery. Actuation without interoception is exactly the failure that broke trust; every future actuator must be *gated on* interoceptive context ("human asleep ⇒ don't act"), so the sense-layer is the prerequisite, not the luxury. It is also reversible and harmless when wrong — a misread room costs one deferred notification.
