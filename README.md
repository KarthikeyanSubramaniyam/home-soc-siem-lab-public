# Home SOC & SIEM Lab

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%204.9-blue)
![Docker](https://img.shields.io/badge/Deploy-Docker%20Compose-2496ED)
![MITRE ATT&CK](https://img.shields.io/badge/Mapped%20to-MITRE%20ATT%26CK-red)
![Detections](https://img.shields.io/badge/Custom%20detections-8-success)
![License](https://img.shields.io/badge/License-MIT-green)

A self-hosted Security Operations Center built on **Wazuh** (SIEM + XDR) and
deployed with Docker Compose. It collects telemetry from Windows and Linux
endpoints, detects adversary behaviour with custom rules mapped to
**MITRE ATT&CK**, validates every detection with safe test telemetry, and
documents incident response end to end.

![Architecture](docs/architecture.svg)

---

## Highlights

| Area                  | What's included                                                                     |
| --------------------- | ----------------------------------------------------------------------------------- |
| **SIEM deployment**   | Wazuh manager, indexer and dashboard as a reproducible single-node Docker stack     |
| **Log collection**    | Sysmon, Windows Security and PowerShell logs, Linux `auth.log`, auditd, FIM, syslog |
| **Detection content** | 8 custom detections (13 rules) and a custom decoder, all mapped to ATT&CK           |
| **Validation**        | `wazuh-logtest`, benign telemetry generators and Atomic Red Team guidance           |
| **Incident response** | NIST-based IR playbook and a worked investigation, from alert to lessons learned    |

## Detections

| #  | Detection                                         | Rule IDs       | MITRE ATT&CK     | Severity   |
| -- | ------------------------------------------------- | -------------- | ---------------- | ---------- |
| 01 | SSH brute force followed by successful login      | 100101, 100102 | T1110.001        | High       |
| 02 | Windows failed-logon burst followed by success    | 100110–100112  | T1110.001        | High       |
| 03 | Suspicious PowerShell (encoded / download cradle) | 100120         | T1059.001, T1027 | High       |
| 04 | New account added to Administrators               | 100130, 100131 | T1136.001, T1098 | High       |
| 05 | Security event log cleared                        | 100140         | T1070.001        | High       |
| 06 | Credential dumping / offensive tooling            | 100150         | T1003.001        | Critical   |
| 07 | Sensitive Linux file tampering (FIM)              | 100160         | T1098, T1053.003 | High       |
| 08 | Honeypot interaction (custom decoder)             | 100200, 100201 | T1595            | Low–Medium |

Each detection has a write-up covering its logic, test procedure, triage steps
and known evasions in [`detections/`](detections/). Coverage and gaps are mapped
in [`detections/mitre-attack-matrix.md`](detections/mitre-attack-matrix.md).

## Quick start

**Requirements:** Docker with Compose v2 and about 8 GB of free RAM.

```bash
git clone https://github.com/KarthikeyanSubramaniyam/home-soc-siem-lab.git
cd home-soc-siem-lab

cp .env.example .env     # set a strong, unique value for every password
./scripts/setup.sh       # hashes passwords, generates TLS certs, starts the stack
```

Open **https://localhost**, accept the self-signed certificate, and log in as
`admin` with your `INDEXER_PASSWORD`. Then enroll an agent and validate the
detections by following [`docs/setup-guide.md`](docs/setup-guide.md).

```bash
./scripts/health-check.sh   # containers, indexer health, recent alerts
```

## How it works

1. **Collect:** agents on each endpoint ship logs to the Wazuh manager over TLS.
2. **Detect:** the manager decodes raw logs into fields, and the rules engine
   correlates them into alerts tagged with a severity and an ATT&CK technique.
3. **Store:** alerts are indexed in the Wazuh indexer (OpenSearch).
4. **Investigate:** analysts triage and investigate alerts in the dashboard.

More detail: [`docs/architecture.md`](docs/architecture.md).

## Validating detections

Three tiers, safest first. See [`testing/README.md`](testing/README.md).

| Tier | Method           | Risk                                                    |
| ---- | ---------------- | ------------------------------------------------------- |
| 1    | `wazuh-logtest`  | None: replays a log line through the decoders and rules |
| 2    | Benign telemetry | Minimal: harmless real events against your own host     |
| 3    | Atomic Red Team  | Controlled: isolated, snapshotted lab VM only           |

## Repository layout

```text
home-soc-siem-lab/
├── detections/          Detection write-ups and the ATT&CK coverage map
├── docker/              Wazuh single-node stack, cert generator, service config
├── docs/                Architecture, setup guide, walkthrough, screenshots
├── incident-response/   IR playbook template and a worked investigation
├── scripts/             Setup, health check, indexer password hashing
├── testing/             Benign telemetry generator and Atomic Red Team notes
└── wazuh-config/        Custom rules, decoder, agent configs, Sysmon notes
```

## Roadmap

- [ ] Network sensor (Suricata or Zeek) for C2, lateral movement and exfiltration
- [ ] Second endpoint to make lateral-movement detections meaningful
- [ ] Selected Sigma rules converted to Wazuh
- [ ] Active response (auto-block brute-force sources) after a soak period
- [ ] ATT&CK Navigator coverage layer

## Ethical use

This lab is for defensive security learning on systems you own. The test
scripts perform only benign actions, and the project contains no malware or
exploits. Run attack-simulation tools such as Atomic Red Team only inside an
isolated lab you control, with snapshots. Never test against systems or
networks you do not have explicit permission to test.

## License

Released under the MIT License. See [LICENSE](LICENSE).
