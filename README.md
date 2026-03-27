# Deontic Skills

Equip your agent to generate, update and visualize driving scenarios in OpenDRIVE and OpenSCENARIO format with Deontic's MCP server.

## Installation

1. Download the skills:
- Clone the repository: `git clone https://github.com/DeonticAI/deontic-skills`
- Or download ZIP from Releases

2. Install in Claude:
- Open Claude.ai > Settings > skills
- Click "Upload skill"
- Select the zipped skill folder

3. Enable the skill:
- Toggle on the relevant skill
- Ensure the Deontic MCP server is connected

4. Test:
- Ask Claude: "Create a scenario with a car driving on a one-lane road."

## Skills

### deontic-scenario-generator
- Triggers when the user asks to create a driving scenario. Examples: "a car makes a lane change on a highway", "a car adapts its speed to a slower vehicle in the same lane", "a car meets an obstacle in the same lane", etc.
- Creates the OpenDRIVE and OpenSCENARIO files for this scenario and displays the download links.

### deontic-odd-scenario-generator
- Triggers when the user asks to create a set of scenarios for a specific Operational Design Domain (ODD).
- The workflow proceeds in two steps:
  1. Suggests the attributes for each scenario, and asks the user to approve or correct these.
  2. Creates the OpenDRIVE and OpenSCENARIO files for each scenario and displays the download links.

### deontic-scenario-for-opendrive
- Triggers when the user wants to create a scenario for an existing OpenDRIVE file.
- Uploads the OpenDRIVE file to Deontic, creates a matching OpenSCENARIO file and displays the download link.

### deontic-scenario-editor
- Triggers when a user gives feedback on a previously created scenario. Feedback can target the road (e.g. "add another right driving lane"), the scene (e.g. "add a slower car 50m in front of Ego") or the weather conditions (e.g. "add thick fog").
- Edits the scenario based on user feedback.

### deontic-video-creator
- Triggers when the user asks for a video of a previously created scenario.
- Creates the video and shows a download link.

