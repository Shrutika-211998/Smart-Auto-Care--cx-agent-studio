# Smart AutoCare — CX Agent Studio (Agent Builder) Export

This repository contains the **exported configuration** for **Autocare Agent** (App ID: `0bbc00a8-d03d-4e2c-8f07-1d5d7e8c057f`) built in **Google Cloud Conversational Experiences (CES) / Agent Builder (CX Agent Studio)**.

Project console link (your environment):
- https://ces.cloud.google.com/projects/genaiguruyoutube/locations/us/apps/0bbc00a8-d03d-4e2c-8f07-1d5d7e8c057f

## What this agent does
**Autocare Agent** is a professional automotive customer-service assistant for **Smart AutoCare**. It helps users with:

1. **Locate a Service Center** (by city)
2. **View Available Services**
3. **Book a Service Appointment**
4. **Cancel / Reschedule a Booking** (cancel supported in current flow)
5. **Provide Feedback**

The agent is designed to:
- Respond politely and professionally
- Stay strictly within car-service support scope (booking, service centers, service status/warranty, feedback)
- Avoid hallucinations: **uses tools/APIs for real data**
- Protect user privacy and follow guardrails

## Architecture (multi-agent)
The app is composed of a **Root (Orchestrator) agent** plus three specialist agents.

### Architecture diagram (Mermaid)

```mermaid
flowchart TB
  %% User entry
  U[User] -->|Message| R[Root Agent\nMain Orchestrator]

  %% Root routing
  R -->|Service center / services / warranty / status| S[service_agent\nService Assistant]
  R -->|Book / cancel / reschedule| B[booking_agent\nBooking Assistant]
  R -->|Feedback / complaint / rating| F[feedback_agent\nFeedback Assistant]

  %% Toolsets (APIs)
  subgraph APIs[Backend APIs (OpenAPI Toolsets)]
    GS[get_services toolset\n/get_service_center\n/get_services]
    BK[booking_service toolset\n/get_available_slots\n/book_service\n/cancel_booking]
  end

  %% Connections to APIs
  S -->|getServiceCenter, get_services| GS
  B -->|getAvailableSlots, bookService, cancelBooking| BK

  %% Feedback tool (function)
  subgraph LocalTools[Local / Function Tools]
    CF[collect_feedback\n(python_function)]
  end
  F -->|collect_feedback| CF

  %% Guardrails + Global Instructions
  subgraph Controls[Controls]
    GI[global_instruction.txt\nScope + privacy + no fabrication]
    GR[Guardrails\nSafety + Prompt]
  end

  R -. governed by .-> GI
  S -. governed by .-> GI
  B -. governed by .-> GI
  F -. governed by .-> GI

  R -. enforced by .-> GR
  S -. enforced by .-> GR
  B -. enforced by .-> GR
  F -. enforced by .-> GR
```

The app is composed of a **Root (Orchestrator) agent** plus three specialist agents:

- **Root agent** (`agents/Root_agent/`)
  - Shows the welcome menu for new sessions
  - Routes requests to the correct specialist
  - Does *not* perform business logic

- **service_agent** (`agents/service_agent/`)
  - Service center lookup
  - Available services
  - (Service status/warranty is referenced in description; implement/enable via tools as needed)

- **booking_agent** (`agents/booking_agent/`)
  - Booking flow
  - Cancellation flow

- **feedback_agent** (`agents/feedback_agent/`)
  - Captures rating + comments
  - Submits feedback

## Tools & APIs
This export includes **OpenAPI toolsets** that connect the agent to a backend service.

Backend base URL (configured in `environment.json`):
- `https://smart-autocare-backend-759503671462.europe-west1.run.app`

### Service toolset (`toolsets/get_services/open_api_toolset/open_api_schema.yaml`)
- `POST /get_service_center` → `getServiceCenter` (lookup center by city)
- `GET /get_services` → `get_services` (list services)

### Booking toolset (`toolsets/booking_service/open_api_toolset/open_api_schema.yaml`)
- `POST /get_available_slots` → `getAvailableSlots`
- `POST /book_service` → `bookService`
- `POST /cancel_booking` → `cancelBooking`

> Note: The agent flows rely on these toolsets for real data. If the backend URL changes, update `environment.json`.

## Logging
`app.json` enables logging and supports BigQuery export.

Current BigQuery export configuration in `environment.json`:
- **project**: `genaiguruyoutube`
- **dataset**: `autocare_logs`

Ensure the dataset exists and the runtime/service account has permissions to write.

## Repository contents
- `README.md` — this documentation
- `exported_app_Autocare Agent.zip` — the CX Agent Studio export package

After unzipping, the main app config is under:
- `exported_app/Autocare_Agent/`
  - `app.json` — top-level app definition
  - `global_instruction.txt` — global behavior constraints
  - `environment.json` — backend URLs + logging env vars
  - `agents/` — multi-agent routing + taskflows
  - `toolsets/` — OpenAPI toolsets (schemas)
  - `tools/` — function tools (python) included in export (may be sample/local tools)
  - `guardrails/` — safety/prompt guardrails

## How to use / import
Typical workflow:
1. Open your Agent Builder / CX Agent Studio app in CES.
2. Use **Import** (or update) with the exported package contents.
3. Verify:
   - Toolset base URL(s) in `environment.json`
   - Guardrails enabled
   - Logging destination (BigQuery) configured
4. Test with **Preview agent**.

## Customization checklist
- Update the **welcome message/menu** in `agents/Root_agent/instruction.txt`.
- Add/extend service flows (e.g., warranty/service status) by wiring additional API endpoints/tools.
- Update timezone if needed (currently set in `app.json`).
- Replace any sample/local python tools with production integrations if you use them.

## License
Add a license if you plan to distribute this publicly.
