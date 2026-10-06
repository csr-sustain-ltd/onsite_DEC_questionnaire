# Small Building BMS: Project Plan and Claude Prompt Pack

2 October 2026 · Ali (CSR Sustain)

## How to use this plan with Claude

Work through the plan one step at a time: one Claude conversation per step, using the ready-made prompts in each phase. Expect roughly 9–10 months part-time to a paid pilot. That timing is an assumption for one person working with Claude, so adjust it once Phase 0 is done.

1. **Create a Claude Project** called "Small Building BMS". Paste the master prompt (below) into the Project instructions, and upload this plan as Project knowledge, so every conversation starts with the full context.
2. **Use Claude Code for building software** (Phases 2–6). It works directly in your code folder, runs tests and fixes errors. Use normal Claude chat for research, specifications, documents and business work.
3. **Start each step in a new conversation.** Paste the step prompt, attach any outputs from earlier steps, and let Claude ask clarifying questions before it starts.
4. **Save everything to one GitHub repository**: code, specifications, decisions and test results. Claude's conversations don't remember each other, but the repository does.
5. **Check the "Done when" criteria** before moving on. If something fails, paste the exact error or result back to Claude rather than describing it.
6. **Record every significant decision** in the decision log at the end of this plan, with the date and the reason.
7. **Never let Claude-written control code touch real plant** until it has passed the bench tests in Phase 7.

## Product vision

Build a low-cost, standardised BMS that small and mid-size buildings can afford: the "Model T" of building controls. Like Ford, the price comes down through standardisation, not by cutting corners on safety.

The product monitors and controls a building's heating, cooling, ventilation and hot water. It runs automatic fault detection in the background, and reports energy use and savings every month. It is sold as a fixed hardware kit plus a monthly subscription, installed in hours rather than weeks.

**Design principles**

- **Standard templates, not bespoke engineering.** Every boiler, heat pump, split AC unit or fan is set up from a template with fixed points, logic and screens.
- **Certified off-the-shelf hardware.** No custom electronics in version 1.
- **Wireless-first sensors.** Use battery room sensors and meter pulse readers to cut cabling cost.
- **Self-commissioning.** A guided setup wizard on a phone or laptop replaces specialist commissioning.
- **Essentials only.** Schedules, setpoints, optimum start, frost protection, alerts, fault detection and energy reports.
- **Safety stays in the equipment.** Our system only enables, disables and sends setpoints. Manufacturer safety controls are never bypassed.
- **Fail-safe by default.** If the internet or the controller fails, plant falls back to its own controls or frost protection.
- **Local first.** Schedules run on the site controller, so the building keeps working if the cloud is down.
- **Read before write.** Every new site starts in monitoring-only mode. Control is switched on only after the point mapping has been checked.

**Unfair advantage:** CSR Sustain already visits these buildings for EPCs, MEES, DECs and energy audits. That gives a trusted route to customers, and a compliance reason to buy as well as an energy saving.

## Scope of the first product (MVP)

Version 1 targets small offices, schools, surgeries and retail units of roughly 200–2,000 m² (an assumption to confirm in Phase 0), heated by gas boilers and cooled by split or small VRF air conditioning.

| Area | In scope for version 1 | Out of scope for version 1 |
| --- | --- | --- |
| Heating | 1–3 gas boilers (enable, status, fault, flow temperature), 1–4 heating zones or circuits, pumps | Large central plant, district heating, CHP |
| Cooling | Split and small VRF units through the manufacturer's interface or Modbus gateway | Chillers, cooling towers |
| Heat pumps | Air-source heat pumps with a Modbus or manufacturer interface | Ground-source and hybrid systems |
| Ventilation | Extract fans, simple supply/extract units (on/off, speed) | Full AHUs with heat recovery and dampers |
| Hot water | Calorifier or cylinder temperature and time control | Legionella management system (record only) |
| Sensors | Wireless room temperature, humidity and CO2; outside air temperature | Occupancy cameras |
| Metering | Electricity and gas pulse or Modbus meters | Half-hourly supplier data import (add later) |
| Control | Schedules, setpoints, optimum start and stop, frost protection, holidays, overrides with timeout | Complex PID tuning per loop, demand response |
| Software | Live dashboard, auto-generated graphics, alarms, fault detection, email/Teams/SMS alerts, monthly energy report | Custom drawn graphics, third-party BMS integration |
| Users | Building manager (web and mobile), installer setup wizard, your support view across all sites | Tenant apps, public API |
| Lighting, fire, access | None | All (separate safety-critical systems) |

## System architecture

The edge controller runs schedules and fail-safe logic on site, so the building keeps working if the internet drops. The cloud stores data, runs fault detection, sends alerts and serves the app.

```
ON SITE
  Plant equipment (boilers, AC, heat pumps, fans; relays, 0-10 V, Modbus, BACnet)
  Wireless sensors (room temperature, humidity, CO2, outside air)
  Meters (electricity and gas; pulse or Modbus)
        |
        v
  EDGE CONTROLLER (on site)
  Local schedules and setpoints, fail-safe and frost protection,
  7-day data buffer, signed updates
        |
        |  MQTT over TLS, outbound only: readings up, commands down
        v
CLOUD PLATFORM
  Ingest and commands (validates data, audits changes)
        -> Time-series database (TimescaleDB / PostgreSQL)
              -> FDD worker (template rules, runs 24/7) -> Alerts (email, Teams, SMS)
              -> API and login (roles, customer isolation)
              -> Reports (monthly energy and savings)
        |
        v
  Web and mobile app (BMS-style screens, setup wizard)
        |
        v
USERS: Building managers | Installers | Our support team
```

The site connection is outbound only, so nothing on the internet can reach into the building's network. The same link carries readings up and checked commands down.

## Phases and timeline

Nine phases take you from idea to first paying sites in about 40 weeks part-time. The pilot in Phase 7 is the gate that matters: it proves safety, installation time and savings.

| Phase | Weeks (assumption) |
| --- | --- |
| 0 Validate the idea | 1–4 |
| 1 Specification and hardware | 3–6 |
| 2 Development setup and simulator | 5–8 |
| 3 Edge controller | 7–14 |
| 4 Cloud platform | 9–16 |
| 5 Web and mobile app | 12–20 |
| 6 FDD, alerts and reports | 14–20 |
| **7 Bench test and pilot (critical gate)** | **20–36** |
| 8 Commercial launch | 30–40 |

Phases overlap on purpose: research and specification run alongside early software work. Don't start Phase 7 until Phases 3–6 pass their Done when checks against the simulator.

## Master context prompt

Paste this into the Claude Project instructions once. If you aren't using a Project, paste it at the start of every new conversation, above the step prompt.

```
You are my technical co-founder and senior engineer for a new product: a low-cost, standardised building management system (BMS) for small and mid-size non-domestic buildings in the UK. Think of it as the "Model T" of building controls: affordable through standardisation.

ABOUT ME
- I am Ali, a building energy and performance specialist at CSR Sustain (Manchester). I do non-domestic EPCs, MEES, DECs, TM45, Part L and energy audits, and I investigate FDD alerts on commercial plant.
- I understand HVAC plant and controls well. I am not a professional software developer, so explain how to install, run and test everything on Windows, step by step.

THE PRODUCT
- Target: small offices, schools, surgeries and retail units (about 200 to 2,000 m2) with gas boilers, radiators, split or small VRF AC, small air-source heat pumps, extract fans and hot water cylinders.
- Hardware: certified off-the-shelf only (edge controller, I/O modules, wireless sensors, Modbus/BACnet gateways). No custom electronics.
- Edge controller on site runs local schedules, fail-safe logic and data buffering, and talks to equipment via relays, 0-10 V, Modbus RTU/TCP, BACnet/IP and wireless sensors.
- Cloud platform: MQTT over TLS for ingest, TimescaleDB (PostgreSQL) for data, a Python FastAPI backend, a fault detection and diagnostics (FDD) worker, an alerts service (email, Teams, SMS), and a React web app with auto-generated BMS-style graphics.
- Equipment is defined by standard templates (points, control logic, FDD rules, graphics). Adding a site means picking templates and mapping points, never bespoke engineering.

NON-NEGOTIABLE PRINCIPLES
1. Safety stays in the equipment. Our system only enables, disables and sends setpoints; it never bypasses manufacturer safety controls.
2. Fail-safe: if the cloud, network or controller fails, plant must revert to its own controls or frost protection.
3. Local first: schedules and fail-safe logic run on the edge controller, not in the cloud.
4. Read before write: every new site starts monitoring-only; control is enabled per point after checks.
5. Security by design: outbound-only connections from site, encrypted transport, no default passwords, least privilege, signed updates.
6. Keep it simple: version 1 has only the essentials. Push back if I ask for complexity that does not serve small buildings.

HOW TO WORK WITH ME
- Use British English, metric units and UK standards.
- Before starting a task, restate the goal, list your assumptions and ask me up to three questions if anything essential is missing.
- Work in small steps that I can test. For code: produce complete files, tests and a README section on how to run them. Do not leave placeholders without telling me.
- Clearly separate facts, assumptions and recommendations. Never invent standards, regulations, product specifications or prices; say when something needs checking.
- When there are real choices, compare two or three options with pros and cons, then recommend one.
- Keep everything consistent with our repository structure and earlier decisions. If a new decision changes an earlier one, say so and tell me to update the decision log.
- End each step with: what was done, how to test it, what is still open, and the suggested next step.

REPOSITORY (monorepo on GitHub)
/docs (specs, decisions, test reports)  /templates (equipment templates)  /edge (controller software)  /cloud (API, ingest, FDD, alerts)  /web (React app)  /simulator (virtual building)  /infra (Docker, deployment)  /tests
```

## Step-by-step plan and prompts

Each phase lists its goal, the prompts to send in order, what you should have at the end, and the check before moving on. Send the master prompt first (or use the Project), then the step prompt.

### Phase 0: Validate the idea (weeks 1–4)

**Goal:** prove that small-building owners have the problem and would pay to solve it, before building anything.

**Step 0.1: Competitor and market research** (Claude chat with web search on)

```
Research the market for low-cost building management and controls for small and mid-size non-domestic buildings in the UK (roughly 200 to 2,000 m2). Find the main competitors (start-ups, manufacturer "light" ranges, smart thermostat and IoT providers), what they offer, how they are priced and installed, and their weaknesses. Also check the current UK regulatory drivers for building automation (Approved Document L requirements, MEES, EPC treatment of controls) with links to official sources. Give me a comparison table, a list of gaps we could own, and the three biggest threats to our idea. Cite every source.
```

**Step 0.2: Customer interview kit**

```
Create a customer discovery interview guide for owners, facilities managers and business managers of small non-domestic buildings (offices, schools, surgeries, retail). I will run 10 to 15 interviews with existing CSR Sustain clients. Include: a short introduction script, 12 to 15 open questions about their heating/cooling/ventilation, current controls, energy costs, pain points, who decides on spending, what they would pay and how (upfront vs monthly), and a simple scoring sheet to record answers. Avoid leading questions. Also create a one-page site checklist I can fill in during EPC visits to record what plant and controls each building has.
```

**Step 0.3: Analyse the interviews**

```
Here are my notes from [number] customer interviews and site checklists (attached). Analyse them: most common building types and plant, top pain points ranked by frequency, willingness to pay, buying process, and objections. Then recommend the single best "Model T" target: building type, plant list and price range for version 1. Be honest if the evidence is weak.
```

**Deliverables:** competitor report, interview kit, interview notes, target customer and plant definition.

**Done when:** at least 5 organisations say they would pilot it, and the version 1 target is written down.

### Phase 1: Specification and hardware selection (weeks 3–6)

**Goal:** a written specification and a costed hardware kit that meets the target price.

**Step 1.1: Product requirements specification**

```
Using our target customer and plant list (attached), write the product requirements specification for version 1. Include: user types and their goals, user stories, functional requirements (monitoring, schedules, setpoints, optimum start, frost protection, holidays, overrides, alarms, FDD, reports), non-functional requirements (reliability, offline behaviour, data retention, security, performance), and an explicit out-of-scope list. Number every requirement so we can trace it later. Save as /docs/requirements.md.
```

**Step 1.2: Equipment templates**

```
Design the standard equipment template format and write the first templates: gas boiler, heating zone/circuit, circulation pump, split AC unit, small air-source heat pump, extract fan, hot water cylinder, room sensor, utility meter. For each: points (name, type, unit, read or write, typical range), control logic in plain English, safe default and fail-safe state, FDD rules with thresholds, and what the graphic should show. Use a machine-readable format (YAML) with a JSON schema to validate it. Save under /templates.
```

**Step 1.3: Hardware shortlist and bill of materials**

```
Shortlist certified off-the-shelf hardware for the site kit: edge controller (industrial Linux device), digital/analogue I/O, Modbus and BACnet interfaces, wireless room sensors and gateway (compare LoRaWAN, Zigbee and others for small buildings), meter pulse readers, 4G backup option, and enclosure/power. For each: two or three options, key specs, certifications (UKCA/CE), UK suppliers and indicative prices with links, and clearly mark any price you could not verify. Then build a bill of materials for a typical small office kit and a cost model in Excel with install labour hours as an input.
```

**Deliverables:** /docs/requirements.md, /templates, hardware comparison, bill of materials and cost model.

**Done when:** the kit cost plus installation fits the price range found in Phase 0, and you have ordered one development kit.

### Phase 2: Development setup and building simulator (weeks 5–8)

**Goal:** a working development environment and a virtual small building, so everything can be built and tested without a real site.

**Step 2.1: Repository and environment** (Claude Code)

```
Set up our monorepo using the structure in the master prompt. Create the Docker Compose development stack (MQTT broker with TLS, TimescaleDB, a placeholder FastAPI service, a placeholder React app), a Makefile or PowerShell scripts for common tasks, linting and formatting, a GitHub Actions workflow that runs tests, and a README that tells me exactly how to install prerequisites on Windows and start everything. Verify that it all starts and the tests pass.
```

**Step 2.2: Virtual building simulator** (Claude Code)

```
Build /simulator: a physics-based simulation of a small office (about 800 m2) with 2 gas boilers, 3 heating zones with radiators, 4 split AC units, 2 extract fans, a hot water cylinder, an electricity meter and a gas meter. Use a simple thermal model per zone driven by outside temperature (use real UK weather data), occupancy and internal gains. Expose the equipment as real protocols: Modbus TCP for the boilers and AC units, BACnet/IP for at least one device, and MQTT for wireless sensors, exactly as real hardware would. Add a fault injection API (stuck valve, boiler lockout, sensor drift, frozen sensor, fan running out of hours, AC fighting heating) so we can test fault detection. Include tests and a short guide.
```

**Deliverables:** running development stack, simulator with protocol interfaces and fault injection.

**Done when:** you can start the stack and the simulator with one command, and see live simulated readings arriving in the MQTT broker.

### Phase 3: Edge controller software (weeks 7–14)

**Goal:** software on the site controller that reads equipment, runs schedules locally, fails safe and sends data to the cloud.

**Step 3.1: Edge architecture design**

```
Design the edge controller software before we write it. Cover: process structure, protocol drivers (Modbus RTU/TCP, BACnet/IP, MQTT wireless sensors, digital/analogue I/O), how templates are loaded and mapped to physical points, local schedule and control engine, fail-safe state machine (cloud lost, sensor lost, controller restart, power loss), local data buffer for outages, secure provisioning and certificates, remote configuration, and signed over-the-air updates. Include sequence diagrams for start-up, normal operation and loss of connection. Save as /docs/edge-architecture.md and list open decisions for me.
```

**Step 3.2: Drivers and point mapping** (Claude Code)

```
Implement the edge protocol drivers and the point-mapping layer described in /docs/edge-architecture.md. Read-only first: poll Modbus and BACnet points, subscribe to MQTT sensors, apply scaling and units from the templates, validate ranges, and publish to the cloud MQTT broker in our agreed message format with timestamps and quality flags. Test against /simulator. Include a command-line tool that scans a network for BACnet devices and Modbus registers to help installers.
```

**Step 3.3: Local control, schedules and fail-safe** (Claude Code)

```
Implement the local control engine: weekly schedules with exceptions and holidays, setpoints, optimum start and stop (self-learning from heat-up rates), frost protection, temporary overrides with automatic timeout, and the fail-safe state machine. Writes to equipment must be per-point enabled (read before write), rate-limited, logged and reversible. Write thorough tests, including simulated cloud outage, sensor failure and controller restart, and prove with the simulator that the building stays safe in each case.
```

**Step 3.4: Buffering, provisioning and updates** (Claude Code)

```
Add store-and-forward buffering for at least 7 days of outage, secure device provisioning (unique certificates per device, no default passwords), a local diagnostics web page for installers on the LAN only, health reporting (CPU, memory, disk, connectivity), and signed over-the-air updates with automatic rollback. Document the security model in /docs/security.md.
```

**Deliverables:** edge software, installer scan tool, test suite, edge and security documentation.

**Done when:** the edge software runs the simulated building for 7 days unattended, survives forced outages and restarts, and every fail-safe test passes.

### Phase 4: Cloud platform (weeks 9–16)

**Goal:** a secure, multi-customer cloud that stores data, sends commands and serves the web app.

**Step 4.1: Data model and database** (Claude Code)

```
Design and implement the cloud data model in TimescaleDB: organisations, sites, buildings, floors, zones, equipment (from templates), points, time-series readings, commands and their audit trail, schedules, alarms and acknowledgements, users and roles. Include data retention and downsampling (raw 15-minute for 2 years, hourly beyond), migrations, seed data for the simulated building, and tests. Explain the design in /docs/data-model.md.
```

**Step 4.2: Ingest and command services** (Claude Code)

```
Build the ingest service (MQTT to TimescaleDB, with validation, deduplication and late-data handling) and the command service (cloud to edge, with acknowledgements, timeouts and a full audit log of who changed what and when). Handle at least 200 sites with 300 points each at 1-minute resolution on a single small server, and load test it.
```

**Step 4.3: API, authentication and roles** (Claude Code)

```
Build the FastAPI backend: secure login (email plus multi-factor authentication), roles (our support staff, customer admin, building manager, read-only, installer), multi-tenant isolation so one customer can never see another's data, and REST endpoints for sites, equipment, live values, trends, schedules, setpoints, alarms and reports. Generate OpenAPI docs and write tests that prove tenant isolation.
```

**Deliverables:** database, ingest and command services, API with authentication.

**Done when:** simulated data flows end to end into the database, commands round-trip to the edge, and the tenant isolation and load tests pass.

### Phase 5: Web and mobile app (weeks 12–20)

**Goal:** a simple, BMS-style app that building managers can use without training, and a setup wizard for installers.

**Step 5.1: Design system and screens**

```
Design the user experience for building managers of small sites who are not controls experts. Pages: site overview, auto-generated plant graphics from templates (boilers, heating zones, AC, fans, hot water), floor/zone view, schedules and holidays, setpoints and overrides, alarms, trends, energy and savings report, settings. Also an installer setup wizard (add site, add equipment from templates, scan and map points, test each point, switch from monitoring to control). Produce wireframes and a written design system (colours, typography, status colours, accessibility). Keep it simpler than a large-building BMS.
```

**Step 5.2: Build the app** (Claude Code)

```
Build the React web app from the design in /docs/ux.md against our API: responsive for phone and desktop, live updates, auto-generated graphics from the equipment templates, schedule editor, setpoint and override controls with confirmation and automatic timeout, alarm list with acknowledgement, trend viewer and energy report. Every control action must show the current value, the requested value and the confirmed result from the edge.
```

**Step 5.3: Installer setup wizard** (Claude Code)

```
Build the installer setup wizard: create site, pick equipment templates, run the network scan from the edge, map points with suggestions, test each input and output with the installer watching on site, record photos and notes, and require a signed checklist before control is enabled. Produce a commissioning certificate PDF at the end.
```

**Deliverables:** web app, installer wizard, commissioning certificate.

**Done when:** someone who has never seen the product can set up the simulated building with the wizard in under 2 hours, and a test building manager can change a schedule unaided.

### Phase 6: Fault detection, alerts and energy reports (weeks 14–20)

**Goal:** the background intelligence that finds waste and faults, tells the right person, and proves savings.

**Step 6.1: FDD engine** (Claude Code)

```
Build the FDD worker that runs continuously in the cloud against the template FDD rules. Requirements: rules defined in the templates (not hard-coded), persistence before alarming (e.g. 1 hour) and recovery logic, schedule awareness (occupied vs unoccupied), suppression when equipment is in a known state (e.g. boiler off in summer), and a confidence or severity level per alarm. Port these rules from our prototype: boiler firing with no heat call, boiler above heating lockout temperature, both boilers at low load, short cycling, enabled but not firing, heating and cooling at the same time, plant running out of hours, zone too hot or cold, valve open with fan off, frozen or out-of-range sensors. Test every rule against fault injection in /simulator and report detection rate and false alarms.
```

**Step 6.2: Alerts and escalation** (Claude Code)

```
Build the alerts service: email, Microsoft Teams and SMS notifications with per-user preferences, quiet hours, a daily digest option, escalation if an alarm is not acknowledged, and duplicate suppression. Include a plain-English explanation and the suggested action in every alert, written for a non-expert building manager.
```

**Step 6.3: Energy and savings report** (Claude Code)

```
Build the automatic monthly report: energy use by fuel, weather-normalised comparison with the baseline (degree-day regression), estimated savings in kWh, GBP and kgCO2e with the method and carbon factors clearly stated, top faults and their estimated cost, schedule compliance, and recommended actions. Produce it as a branded PDF emailed to the customer, and as a page in the app.
```

**Deliverables:** FDD worker, alerts service, monthly report.

**Done when:** in the simulator, at least 90% of injected faults are detected with fewer than 1 false alarm per site per week, and a sample monthly report reads clearly to a non-expert.

### Phase 7: Bench test and pilot building (weeks 20–36)

**Goal:** prove the real hardware and software work safely in a real building and deliver measurable savings.

**Step 7.1: Bench test rig**

```
Design a bench test rig and test plan for the real hardware kit before any site installation: edge controller, I/O, a real Modbus device, a BACnet device, wireless sensors, relays driving indicator lamps instead of plant, and a network switch. List every test (each point type, every fail-safe scenario, power cut, network loss, update rollback, security checks) with pass criteria, and produce a test record template. Then help me analyse my results.
```

**Step 7.2: Pilot site plan**

```
Help me plan the pilot installation at [pilot building] (site survey attached). Produce: a site survey checklist, wiring and interface schedule for the electrician, risk assessment and method statement inputs, the client agreement summary (monitoring-only period, then control), a rollback plan, the baseline measurement plan (at least 4 weeks monitoring-only before control), and a weekly pilot review template.
```

**Step 7.3: Pilot review and fixes**

```
Here is the data and the issue log from week [n] of the pilot (attached). Identify false alarms, missed faults, control problems, usability issues and anything unsafe. Propose fixes in priority order, and update templates, rules and documentation accordingly.
```

**Deliverables:** bench test record, pilot installation pack, weekly pilot reviews, measured savings.

**Done when:** the pilot runs 8 or more weeks in control mode with no safety incidents, the client is satisfied, and savings are measured against the baseline.

### Phase 8: Commercial launch (weeks 30–40)

**Goal:** turn the pilot into a product people can buy, install and get support for.

**Step 8.1: Pricing and business model**

```
Using the pilot costs (hardware, installation hours, cloud costs, support time) and measured savings (attached), build a pricing model: kit price, installation price, monthly subscription tiers, payback for a typical customer, our gross margin, and break-even number of sites. Compare upfront, subscription-only and hardware-as-a-service options. Build it in Excel with assumptions clearly separated.
```

**Step 8.2: Installer and support operations**

```
Create the operating documents: installer training guide and competency checklist, installation manual, commissioning procedure, customer onboarding guide, support process with response times, fault escalation, and a knowledge base of common problems. Keep them practical and short.
```

**Step 8.3: Sales materials**

```
Write the sales materials for small-building owners: a one-page product sheet, a case study from the pilot, a savings calculator brief, a short email sequence for existing CSR Sustain clients, and a script for raising it during EPC and audit visits. No exaggerated claims: every number must come from the pilot or be labelled as an estimate.
```

**Deliverables:** pricing model, operating documents, sales materials, first paying customers.

**Done when:** at least 3 paying sites are live, and the contracts, insurance and support process are in place.

## Safety, compliance and legal checklist

Complete these before the pilot goes into control mode. Items marked "check" need confirming with a qualified professional or the official source; this plan is not legal advice.

**Safety and installation**

- [ ] Manufacturer safety controls (boiler lockouts, flame safeguards, high-limit stats, AC protections) are never bypassed or overridden.
- [ ] Fail-safe states are defined, tested and documented for every equipment template.
- [ ] Frost protection works with the cloud disconnected.
- [ ] Electrical work is carried out and certified by a qualified electrician to BS 7671.
- [ ] Interfaces use the manufacturer's approved control inputs (volt-free enable contacts, 0–10 V, Modbus or BACnet), and these are confirmed with the manufacturer or installer.
- [ ] Risk assessment and method statement are prepared for each installation.

**Hardware and product**

- [ ] All hardware is certified off-the-shelf (UKCA/CE marked) and used within its intended scope.
- [ ] Check whether the Product Security and Telecommunications Infrastructure (PSTI) regime applies to anything you supply.
- [ ] Security follows recognised guidance (NCSC guidance; ETSI EN 303 645 as a reference): unique credentials, no default passwords, encrypted links, signed updates, vulnerability reporting route.

**Data and contracts**

- [ ] UK GDPR: record what personal data is held (user accounts; occupancy patterns can be personal data), write a privacy notice, and check whether ICO registration is needed.
- [ ] Customer terms: data ownership, service levels, liability limits, the monitoring-only period, and the customer's responsibility for plant maintenance. Check with a solicitor.
- [ ] Written permission from the building owner, and agreement from any existing controls or maintenance contractor.
- [ ] Insurance: professional indemnity, public liability and product liability, with cover for remote control of plant. Check with a broker.

**Business set-up**

- [ ] Decide whether this is run inside CSR Sustain or as a separate company, and agree ownership of the product and code with CSR Sustain's directors early.
- [ ] Agree who on site can override the system, and how the customer turns control off.

## Risk register

The biggest risk is installation cost, not software: it decides whether small buildings can afford the product. Risks are listed in priority order.

| Risk | Why it matters | Mitigation | Check by |
| --- | --- | --- | --- |
| Installation labour too costly | Kills the low price for small buildings | Wireless sensors, pre-wired kit, setup wizard, measure real hours in the pilot | Phase 7 |
| Unsafe control action | Cold building, damaged plant, liability | Safety stays in equipment, fail-safe states, read before write, bench tests | Phase 3 and 7 |
| Equipment too varied to standardise | Each site becomes bespoke, costs rise | Narrow version 1 target, refuse sites outside the template list | Phase 0 |
| Weak demand or long sales cycle | No revenue to fund development | Validate with interviews, sell to existing CSR Sustain clients first | Phase 0 |
| Strong competitors already serve the gap | Hard to win on price alone | Compete on compliance link (EPC, MEES), local support and simplicity | Phase 0 |
| Cybersecurity breach | Plant controlled by an attacker, reputational damage | Outbound-only links, certificates, signed updates, external security review | Phase 3 and 4 |
| Support load from many small customers | Margin eaten by phone calls | Clear alerts, self-service app, knowledge base, remote diagnostics | Phase 8 |
| Code built by one person with AI | Hard to maintain or hand over | Tests, documentation, decision log, code review by a developer before launch | Every phase |
| Ownership disputes with employer | Product rights unclear | Agree in writing with CSR Sustain early | Phase 0 |

## Tools, accounts and decision log

You need these before Phase 2. Costs are not listed because they change; Claude can price them for you in Step 1.3.

| Item | Purpose | Needed from |
| --- | --- | --- |
| Claude plan with Projects and Claude Code | Research, documents and coding | Phase 0 |
| GitHub account (private repository) | Single source of truth for code and documents | Phase 1 |
| Windows laptop with admin rights, Python, Node.js, Docker Desktop, VS Code | Running the development stack and simulator | Phase 2 |
| Cloud account (Azure or AWS) | Hosting from Phase 4 onwards | Phase 4 |
| Domain name and business email | App address and alert emails | Phase 4 |
| Development hardware kit | Edge controller, I/O, sensors, gateway for the bench | Phase 3 |
| Bench test parts | A real Modbus device, a BACnet device, lamps, switch, enclosure | Phase 7 |
| Electrician and installer | Pilot installation | Phase 7 |
| Solicitor and insurance broker | Terms, liability and insurance | Before the pilot goes into control |

**Decision log** (add a row for every significant decision)

| Date | Decision | Reason | Phase |
| --- | --- | --- | --- |
| 02/10/2026 | Build an independent platform; no dependence on PEAK | Own the data source and the product | 0 |
| | | | |
