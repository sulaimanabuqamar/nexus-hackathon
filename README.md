# ERgent

**Nexus 26 · Track 3 Systems & Informatics · ER Crisis Command Center**
American University of Sharjah, 24 October 2026

One live view of every bed, patient, ambulance, nurse and supply in an emergency department, with a bedside phone and a watch that update it by tap, voice or photo.

## Run it

Open `index.html` in a browser, or visit the Vercel deployment. There is no build step and nothing to install: it is one HTML file with inline CSS and JavaScript.

Use the tabs at the top to switch between the three views:

- **Command center**: the wall or laptop screen.
- **Bedside phone**: on a narrow screen this view opens automatically, and `#mobile` in the URL also opens it. If no command center answers, the phone starts one on the same device, so a phone on its own still runs the full demo.
- **Watch**: `#watch` in the URL opens it.

Simulation speed is in the toolbar: from real time (1 simulated minute = 60 s) and 1 min = 20 s up to 4 simulated minutes per second.

Roles (toolbar): **Charge nurse** sees everything; **ER physician** sees patients, live vital signs and clinical alerts only; **Bed manager** sees capacity, flow and supplies with no clinical data.

Demo sign-in on the phone view (two factors, PIN + authenticator code):
- Staff member: any (try Dr. Hamad, or Paramedic Tariq for ambulance pre-alerts)
- PIN: `2468`
- 2FA code: the six digits shown in the "Demo authenticator" box

**About the data:** every patient and number is simulated, and 1 real second is 1 simulated minute. The learned-pattern statistics come from a synthetic year of 40,266 visits in a Riayati-style record format. Hospital and supplier names are fictional.

## What it does

### Command center
The command center has seven pages in a sidebar: **Floor** (the bed map), **Alerts**, **Arrivals**, **Resources**, **Staff**, **Insights** and **System**. Floor buttons on the left switch between hospital levels; only the Emergency floor is live in the demo.

- **Headline numbers**: occupancy, inbound ambulances and time until the ED is full.
- **Floor map**: hover a bed for a quick summary, click it for the full record.
- **Bed board**: Resus, Acute, Fast track and Overflow beds, each showing ESI (with the CTAS equivalent), live heart rate and SpO₂ from bedside monitors, and its predicted free time. Clicking a bed shows a live ECG trace, vital signs with LOINC codes, and the FHIR R4 bundle.
- **Inbound ambulances**: ETA, en-route vitals, mechanism and interventions. Critical patients get a bed held 7 minutes before arrival.
- **Routing**: if this ED can't take a patient, the pre-alert is forwarded to the next capable hospital.
- **Smart tickets**: ranked P1–P4 by severity, acuity and age. Each one names its owner, the action, the SBAR detail and an acknowledge deadline. Tickets page every person and team needed and escalate to the on-call consultant when not acknowledged.
- **Next best actions**: one-click decisions such as opening overflow beds, calling in staff, moving low-acuity patients, diverting ambulances, rebalancing nurses or requesting bank nurses.
- **Learned patterns**: arrivals by hour, night crash share, and length of stay by age and complaint. These drive the bed-free predictions and the staffing forecast.
- **Staff**: shift tracking with handover summaries (SBAR), and nurse workload weighted by acuity with a fatigue factor.
- **Connected devices**: ECG, blood pressure, pulse oximeter, capnography, temperature, ventilators and pumps. A silent device opens a ticket.
- **Next availability**: predicted free time for each bed zone, CT, X-ray, ultrasound and ventilators.
- **Inventory**: burn rates and stock forecasts that include what inbound casualties will need. Purchase orders go to suppliers before stock runs out.
- **Data pathway**: every source shown live or silent, with a message log.
- **Run a case** menu:
  - mass casualty
  - stroke pre-alert
  - severe burn rerouted
  - patient deteriorates
  - monitor probe falls off
  - O-neg running low
  - night crash peak
  - bed-feed outage

### Bedside phone and watch
- The phone takes typed or spoken updates, or a photo of a monitor. Claude turns them into coded updates. Risky changes such as discharges ask for confirmation first.
- Tap a bed for quick actions.
- Tickets arrive on the phone and watch with one-tap acknowledge, a help call and dictated notes.
- **Paramedic mode**: EMTs describe symptoms and get the routing decision back.

## Security
- **Sign-in is two factors, no more**: PIN, checked against a PBKDF2-SHA256 hash with 100k iterations, plus a TOTP authenticator code (RFC 6238). The code is verified by the command center before a device is trusted.
- **Unlock after idle is one factor**: Face ID / Touch ID (WebAuthn, optional, turned on from the phone) or the PIN. On the claude.ai preview the browser frame blocks WebAuthn, so a labelled stand-in appears.
- **Signed updates**: every phone or watch update is signed with a device-bound ECDSA P-256 key. The command center checks the signature, the device, a 90-second time window and a one-time nonce, which rejects forged, tampered and replayed messages.
- **Least-privilege roles**, enforced at the command center. A nurse cannot discharge, and a bed manager sees no clinical data unless they use break-glass, which requires a reason and expires.
- **Device control**: auto-lock on idle, and a lost device can be revoked in one click.
- **Audit log** of every view and change.
- **Minimum necessary data**: shared screens show initials and pseudonymous IDs only.
- **Security headers** in `vercel.json`: CSP, HSTS, frame denial, no referrer and a restricted permissions policy.
- **"Run attack test"** button: shows four attack types being blocked live.

## Standards

| Standard | Where it is used |
|---|---|
| HL7 FHIR R4 | Encounter, Location, Observation, Device and SupplyRequest resources, visible in every drawer |
| HL7 v2.x | ADT A01/A02/A03/A04 bed movements and table 0116 bed status codes |
| IEEE 11073-10201 | Bedside device gateway; each Observation references its Device |
| SNOMED CT | Coded chief complaints; the AI may only choose from the coded list |
| ICD-10-CM / ICD-11 | ICD-10-CM on every Encounter; ICD-11 through terminology-service mapping |
| LOINC | 8867-4, 59408-5, 8480-6, 9279-1, 8310-5 and 9269-2 vital signs |
| ESI / CTAS | ESI drives zones, wait targets, ticket priority and nurse load; the CTAS equivalent is shown alongside |
| ISO 9241-210 | Role-based design for charge nurse, physician, bed manager and paramedic |
| WCAG 2.2 | Never colour alone, visible focus, large touch targets, light and dark themes, reduced motion |
| HIPAA Security Rule, ISO/IEC 27001, NIST SP 800-66 | Access control, audit, integrity, transmission security, automatic logoff |
| UAE Federal Law No. 2 of 2019 | Article 13 keeps health data inside the UAE; Article 20 sets 25-year retention |
| ADHICS, UAE PDPL, Riayati / NABIDH / Malaffi | Local security standard, data-protection scope and integration with the national record |

## Hosting notes

**On claude.ai:** the published artifact syncs the phone, watch and command center across devices and uses Claude for voice and photo reading.

**On Vercel:** the static deployment runs everything on one device. The phone and watch views sync with the command center across browser tabs, and voice and photo parsing fall back to on-device rules. Cross-device sync and Claude parsing need a small backend, which is not included in this repo.

In production the command center, database and message broker would be hospital-hosted in the UAE, as Law No. 2 of 2019 requires.
