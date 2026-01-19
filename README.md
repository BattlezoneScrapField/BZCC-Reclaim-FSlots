## How Mods Can Reclaim F7–F10 Slots from Players

This document presents a design concept outlining a pseudo-solution for reclaiming the F7–F10 slots from players.

A common complaint is that commanders lose the ability to build up to 30 additional units because players occupy the F slots.

While this solution is not perfect, it requires minimal changes to the BZ2 engine and allows players to play without the hindrance of losing up to four F slots.

**This solution avoids the use of the following features:**
- **DefaultTeams** — The setting that automatically allies Teams 1–6 and 5–10.
- `Ally(int teamA, int teamB)` — DLL/Lua function used to ally two teams.
- `UnAlly(int teamA, int teamB)` — DLL/Lua function used to un-ally two teams.
## Considerations

- This approach requires a small number of additional engine-level functions.
- While not a perfect solution, it avoids rewriting the core F-slot logic.
- Each strategy map would need to be updated to disable `DefaultTeams`.
## Engine Requirements

For this approach to work, ScriptUtils.h needs a way to instruct AI units to ignore other units so they do not fire on “allied” ships.

To accomplish this, I propose adding the following engine-level functions for DLL use:

`// Forces AI units to ignore all objects on a specified team. AIIgnoreTeam(int TeamNum)  // Removes the ignored team so AI units can engage them again. AIUnIgnoreTeam(int TeamNum)  // Forces a specific AI unit to ignore a specific object. AIIgnoreHim(Handle me, Handle him)  // Removes a specific ignored object from an AI unit. AIUnIgnoreHim(Handle me, Handle him)`

These functions are critical to the solution, as they provide a mechanism for AI units to ignore both specific handles and entire teams.
## Configuring Alliances

A custom shell can be created with sliders for each team, allowing the host to configure alliances manually. BlackDragon has noted that the VSR FFA maps already implement similar functionality, which we can emulate.

Instead of players selecting “Join Team 1” or “Join Team 2,” the host would use a new slider-based configuration screen to define alliances for each player.

An example layout for players might look like this:  
<img width="796" height="682" alt="Pasted image 20260119202317" src="https://github.com/user-attachments/assets/f9d8a607-7404-46d6-bb66-384598914315" />

## DLL-Based “Fake” Alliances

A DLL can determine whether actions such as sniping or shots impacting vehicles should be accepted by the engine. Using this capability, we can store arrays of teams that should be treated as “allies” and respond accordingly to events such as nav placement or AI targeting.

An example structure:

```lua
local Mission = { 	
	m_Team1 = {1, 2, 3, 4, 5}, 	
	m_Team2 = {6, 7, 8, 9, 10} 
}
```
## How Will Navs Work?

By default, enemies cannot see navs. To work around this, the DLL can automatically build a nav for each allied team whenever one is placed.

For example, if Team 2 places a nav near a pool, the DLL would create equivalent navs for all allied teams.

One consideration here is performance. Since additional objects are being created to ensure visibility for each allied player, there is a small performance cost. However, this impact should be minimal.

## How Will Players Interact with These Alliances?

We can inject libraries into the DLL to render a custom UI overlay on top of BZCC using DirectX 11.

This same approach is already planned for use in the VSR custom spectator UI.

The UI can track various gameplay elements and allow player interactions by leveraging the ScriptUtils library.
### Giving Units

When a player presses a designated button, a UI can be displayed that allows them to transfer selected units to their allied players.
### Player Health and Ammo

A custom UI overlay can display allied players’ health and ammo, replacing the missing information previously provided by the F-slot UI.

We can also track customizable key events that allow players to quickly pass units to their respective allies.

Below is an example of how the UI overlay might appear while running BZCC:  
<img width="3822" height="2079" alt="Pasted image 20260119202341" src="https://github.com/user-attachments/assets/6b516db6-2955-47c4-a9c3-2cc75d05c1eb" />
