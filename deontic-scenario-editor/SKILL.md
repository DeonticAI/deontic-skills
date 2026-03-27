---
name: deontic-scenario-editor
description: >
  Edit and refine an existing autonomous driving scenario (OpenDRIVE + OpenSCENARIO) using
  natural language feedback, via the Deontic MCP server. Use this skill whenever a user has
  already generated a scenario and now wants to modify it — for example: "add foggy conditions",
  "add a right driving lane", "add another car 50 meters in front of Ego", "make it rain",
  "change the speed limit to 60 km/h", "remove the pedestrian", "add an intersection", or any
  other refinement request. Triggers on feedback phrases like "add", "remove", "change", "adjust",
  "make it", "update the scenario", or any natural language that describes a modification to an
  existing scenario. Also triggers when the user says something looks wrong and asks to fix it.
  Do NOT use this skill if no scenario has been generated yet — use deontic-scenario-generator instead.
compatibility: "Requires Deontic MCP server connection"
---

# Deontic Scenario Editor

Refine an existing autonomous driving scenario using natural language feedback. Calls the Deontic
MCP server to apply changes and returns updated OpenDRIVE (.xodr) and OpenSCENARIO (.xosc) files.

---

## Feedback Categories

This skill handles all types of scenario feedback, including:

- **Environment conditions**: fog, rain, snow, time of day, lighting, weather visibility
- **Road/lane changes**: add/remove lanes, change road type, add intersections, roundabouts, ramps
- **Actor changes**: add, move, or remove vehicles, pedestrians, cyclists, or other agents; adjust
  their speed, starting position, or behavior
- **Ego vehicle**: change speed, route, starting position, or maneuvers

If the feedback is ambiguous (e.g., "make it more realistic"), ask a short clarifying question
before proceeding.

---

## Workflow

### Step 1 — Identify the Scenario

Retrieve the **scenario ID** from the current conversation context. This was returned when the
scenario was originally generated (from `Deontic:generate_open_scenario` as `scenario_id` or
equivalent). If no scenario ID is available in context, ask the user to provide it or to
regenerate the scenario first.

### Step 2 — Interpret the Feedback

Parse the user's natural language feedback into a clear, concise update instruction. You don't
need to transform it — pass it as-is or lightly cleaned up. Examples:

| User says | Pass to MCP as |
|---|---|
| "add foggy conditions" | "add foggy conditions" |
| "add a right driving lane" | "add a right driving lane" |
| "add another car 50 meters in front of Ego" | "add another car 50 meters in front of Ego" |
| "make it nighttime" | "change time of day to night" |

If the user gives multiple changes in one message (e.g., "add fog and a pedestrian"), pass them
together in one update call.

### Step 3 — Call `Deontic:update_scenario`

Call the MCP tool with:
- `scenario_id`: the ID from Step 1
- `feedback`: the feedback string from Step 2

### Step 4 — Display Results

After a successful update, present the following clearly:

#### ✅ What Changed
Summarize the change(s) that were applied, in plain language. Base this on the feedback and the
updated descriptions returned by the MCP response.

#### 🗺️ Updated Scenario Summary
Show both:
- **Road Network Description** — from `road_network_description` in the response
- **Scene Description** — from `scene_description` in the response

#### 📁 Download Updated Files
Provide download links for:
- OpenDRIVE file (`open_drive_download_url`)
- OpenSCENARIO file (`open_scenario_download_url`)

Label them clearly so the user knows which is which.

### Step 5 — Invite Further Feedback

End with a short prompt inviting the user to keep refining:

> Anything else you'd like to adjust? You can keep describing changes and I'll update the scenario.

---

## Error Handling

- If `update_scenario` fails, tell the user what went wrong and suggest rephrasing the feedback
  or breaking it into smaller changes.
- If the scenario ID is missing, prompt the user to share it or regenerate the scenario.
- If the MCP returns no download URLs, still show the descriptions and note the file issue.

---

## Example Interaction

**User**: Add foggy conditions and a car parked on the right shoulder.

**Assistant**:
1. Retrieves `scenario_id` from context
2. Calls `Deontic:update_scenario` with feedback `"add foggy conditions and a car parked on the right shoulder"`
3. Displays:
   - ✅ **What Changed**: Added fog weather conditions and a parked vehicle on the right shoulder
   - 🗺️ Road network and scene descriptions
   - 📁 Download links for updated .xodr and .xosc files
4. Asks if the user wants further changes
