---
name: deontic-scenario-generator
description: >
  Generate a single driving scenario (OpenDRIVE + OpenSCENARIO) from a natural language description
  using the Deontic MCP server. Use this skill whenever a user wants to quickly generate a driving
  scenario from a prompt, create OpenDRIVE/OpenSCENARIO files for a specific driving situation, or
  test a single scenario requirement. Triggers include: "generate a scenario", "create a driving
  scenario", "generate OpenDRIVE", "generate OpenSCENARIO", "simulate this driving situation", or
  any request to produce a driving scenario from a text description. Also triggers when the user
  provides a driving situation description and wants scenario files generated, or when they mention
  Deontic scenario generation. Use this skill for single-scenario generation from a prompt; for
  multi-scenario batch generation with attribute suggestion workflows, use the
  deontic-scenario-generator skill instead.
compatibility: "Requires Deontic MCP server connection"
---

# Deontic Scenario Generation

Generate a complete driving scenario (OpenDRIVE + OpenSCENARIO) from a natural language description using the Deontic MCP server.

## Overview

This skill takes a scenario prompt (and optional constraints) and produces:
- A road network description
- A scene description
- Downloadable OpenDRIVE (.xodr) and OpenSCENARIO (.xosc) files

---

## Workflow

### Step 1 — Gather Inputs

You need two pieces of information:

- **Scenario prompt** (required): A natural language description of the driving scenario to generate (e.g., "A vehicle cuts in front of the ego car on a two-lane highway").
- **Additional constraints** (optional): Extra conditions like weather, speed limits, road type, number of lanes, etc. (e.g., "rainy conditions, 50 km/h speed limit").

If the user has not provided a scenario prompt, ask for one before proceeding.

---

### Step 2 — Generate the Scenario

Call `Deontic:generate_scenario` with:
- `scenario_prompt`: the scenario prompt
- `scenario_constraints`: the additional constraints (if any; omit or pass empty string if none)

---

### Step 3 — Display Results

From the response, present the following to the user:

1. **Road Network Description**: Show the `road_network_description` field from the response.
2. **Scene Description**: Show the `scene_description` field from the response.

Format these clearly with headers so the user can review the generated scenario.

---

### Step 4 — Provide File Downloads

The response contains download URLs for two files:
- `open_drive_download_url` — the OpenDRIVE (.xodr) file
- `open_scenario_download_url` — the OpenSCENARIO (.xosc) file

Present the download links to the user.

---

### Step 5 — Suggest Follow-up Tasks

After presenting the scenario, suggest these follow-up actions:

> **What would you like to do next?**
> - 🎬 **Create a video** for this scenario to visualize the simulation
> - 💬 **Give feedback** on the scenario to refine or adjust it
> - 🔄 **Generate another scenario** with a different prompt or constraints

If the user wants to create a video, use `Deontic:create_video_from_OpenX_blob_names` with the blob names extracted from the download URLs (the filename portion of the URL, without the path prefix).

If the user wants to give feedback, use `Deontic:update_scenario` with the scenario ID from the original response and the user's feedback text.

---

## Error Handling

- If `generate_scenario` fails, inform the user of the error and suggest rephrasing the prompt or adjusting constraints.
- If a file download fails, note the failure but still display the road network and scene descriptions. Provide the raw download URL so the user can try manually.

---

## Example Interaction

**User**: Generate a scenario where a pedestrian crosses the road at an unmarked crossing while the ego vehicle is approaching at 40 km/h.

**Assistant**:
1. Calls `Deontic:generate_scenario` with:
   - `scenario_prompt`: "A pedestrian crosses the road at an unmarked crossing while the ego vehicle is approaching at 40 km/h"
2. Displays the road network description and scene description
3. Downloads and presents the .xodr and .xosc files
4. Suggests follow-up tasks (video, feedback, new scenario)
