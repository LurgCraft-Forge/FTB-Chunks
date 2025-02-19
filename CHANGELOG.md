# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1802.3.17]

### Fixed

* Some significant client-side memory management work has been done
  * Addressed some conditions which could lead to client-side memory starvation when the game has been running for a while
  * Specifically, well-explored worlds where the player is either moving around the world a lot, or viewing the world with high map zoom-out
  * Periodically, least-recently accessed region data is released from RAM, requiring reload from disk on the next access. Every 300 seconds by default; can be tuned in client config.
  * When the large map screen is closed, least-recently accessed regions are released from RAM, down to 32 loaded regions by default; also tunable in client config.
  * Map zoom-out is limited where the ratio of the number of known (explored) regions to available JVM memory is poor. Limiting zoom-out reduces the number of regions which need to be loaded in memory at a given moment. This can be disabled in client config if you prefer.
  * New client config settings are available in the "Memory Usage" section of the client config; tuning them is a trade-off between RAM usage and disk activity. However, even when tuned toward lower RAM usage, the level of disk activity should not be a major concern.

## [1802.3.16]

### Fixed
- Fixed an NPE which can occur in conjunction with some mods' fake player objects

## [1802.3.15]

### Fixed
* Fixed a problem on Fabric (Forge not affected) with block break permissions in protected chunks

## [1802.3.14]

### Added
- New team property "Non-living Entity Attack Mode", used when left-clicking non-living entities like item frames or paintings
- The `max_idle_days_before_unforce` and `max_idle_days_before_unclaim` server config settings can now be floating-point values
- Fade distances for waypoint beacons and waypoint dots can now be configured independently
  - Previously in-game dot icons for waypoints never faded out, regardless of proximity
  - New client-side setting "Waypoint dot fade distance", default 1 block away
  - "Waypoint fade distance" is now "Waypoint beacon fade distance" (default still 12 blocks)

### Fixed
- Fixed a block placement dupe issue on Fabric
  - On Fabric, there is now only a single "Block Interaction and Edit" team property, since it isn't possible to reliably distinguish between right-clicking a block to use vs. right-clicking a block to place another block
  - Forge functionality is unchanged
- Re-sync player's held item when a block placement fails due to protected claims
- Fixed another chunkloading issue on Forge where stale tickets weren't always being cleaned up on server restart

## [1802.3.13]

### Added
- The handling of Fake Player access to protected chunks has been improved
  - New team property "Allow Fake Players by Player ID", default true. This is the secure way to allow fake player access to your claims, but it depends on mods actually giving their fake players the ID of the real player deploying that fake player. Examples of mods which do this properly are the Mekanism Digital Miner, PneumaticCraft: Repressurized Drones, and Modular Routers (with an installed Security Upgrade).
  - Team property "Allow Fake Players" is now "Allow All Fake Players", and is false by default (you may wish to review this setting for your team). Beware: setting this to true treats ALL fake players as team allies, including those from blocks/entities owned by potentially hostile players!
  - New team property "Allied Fake Player Names/IDs": you can add names or IDs of known fake players. Beware: adding names or IDs to this list treats these fake players as team allies, even from blocks/entities owned by potentially hostile players!
- Right-clicking corpse entities from the "Corpse" mod is now permitted in any claimed chunk
  - the `corpse:corpse` entity type is added to the `ftbchunks:entity_interact_whitelist` entity type tag
- Sharestones from the Waystones mod, and all waystones from the Fabric Waystones mods can now be interacted with in claimed chunks
  - added some more entries to the `ftbchunks:block_interact_whitelist` block tag
- Added new server-side config setting `max_idle_days_before_unforce`
  - If no member of a team logs in for this many days, any force-loaded chunks owned by the team will become un-forceloaded
  - Default is 0, meaning no un-forceloading will be done; server admins should set this to a value suiting their server
- Added new protection team property: "Non-living Entity Attack Mode"
  - Allows for protection from left-clicking of non-living entities like Item Frames and Paintings in your base
  - Does *not* prevent living entities from being attacked

### Fixes
- Fixed NPE when a block break event is received with bad level data in it
- Fixed player head icons in the previous dimension not disappearing from the map or minimap when you change dimension
  - Only players on long-range tracking (i.e. outside normal vanilla entity tracking range)
- Fixed chunkloading issue on Forge where some forceloaded tickets weren't being cleared when offline chunkloading is disabled for a team
  - Causing chunks to sometimes stay loaded when they should not be

## [1802.3.12]

### Fixes
- Fix force-loaded chunks not always ticking entities & block entities in those chunks

## [1802.3.11]

### Added
- Gamestage support for map and minimap usage
  - New `require_game_stage` server setting, default false. If true, players must have the `ftbchunks_mapping` stage to view the map or minimap
  - Requires KubeJS (Forge or Fabric) and/or Gamestages (Forge only) to be installed

### Fixes
- Fixed FTB Chunks keybinds getting picked up where they shouldn't be, e.g. when chat is open (bug introduced in 1802.3.10)

## [1802.3.10]

### Added
* Server settings are now editable via a new button in the map GUI (bottom right), for players with permission level >= 2

### Fixes
* Fixed a server CME crash under some circumstances when players disconnect
* Fixed keybinds used by FTB Chunks sometimes getting "stuck" if other keys pressed at the same time

## [1802.3.9]

### Fixes
* Fixed rare crash on player login when FTB Ranks is also installed

## [1802.3.8]

### Added

* Square minimap option (see "Square Minimap" in client options, default false)
* Added ability for teams to specify new visibility settings in Team settings
  * "Location Visibility" determines how player heads are visible to players in other teams on the map (default: Allies)
  * "Claim Visibility" determines how chunk claims are visible to other teams on the map (default: Public)
  * This means player heads will only be visible to team-mates and allies by default; if you're not in a party team but want to be visible on the map, set your "Location Visibility" to "Public" in team settings.
* Player heads can now be tracked on the map at any range, not just inside the default entity tracking range
  * The same visibility restrictions apply as above, via "Location Visibility"
  * Added server-side config item `long_range_tracker_interval` which controls how frequently long-range tracking data is sent by the server to clients; default is every 20 ticks (for players who are moving). 
  * Set this to 0 to disable long-tracking entirely
  * Be careful about setting this to very low (non-zero) values; it can cause extra server and network load, especially on very busy servers
* Added entity interaction protection as a Team setting
  * Controls the ability for non-team-members to interact with e.g. Armor Stands, Item Frames and other entities
  * Support for entity interaction whitelisting via the `ftbchunks:entity_interact_whitelist` entity type tag
* Added hotkey 'C' to quickly switch between large map GUI and chunk claiming GUI
* Added a hotkey (not bound by default) to quickly add a waypoint at the player's current position
* Added a client configurable distance whereby death waypoints are automatically removed if the player gets close enough
  * Default of 0 means death waypoints are not auto-removed
* Added a waypoint manager GUI, accessible via a button on the large map GUI, or via hotkey (unbound by default)
  * Can be used to view all of your waypoints, sorted by dimension and distance, toggle visibility, change color & label, and delete them
* Added server configurable timeout whereby a team's chunk claims are automatically released if no one in the team logs in
  * Default of 0 days means no automatic claim loss
  * Intended to prevent claims owned by teams who no longer play hanging around forever
  * See `max_idle_days_before_unclaim` in server config file
* Beneficial splash and lingering potions may be used by any player in any chunk, regardless of claim protection
* A player's original chunk claims are now remembered when they join a team and returned to them if they leave the team
  * Prevents claim stealing by maliciously inviting then kicking a player from a team
  * This doesn't apply retrospectively; claims made before this release of the mod are not remembered in this way
* Added ability to temporarily force-load a chunk via chunk claiming GUI
  * Can use the mouse wheel on force-loaded chunks owned by your team to adjust a force-load time for the chunk
  * Chunk will be automatically unforced within 10 minutes of this time expiring (but kept claimed, of course)
* The way team claim and forceload limits is calculated has been significantly reworked
  * Previously the team owner's limits were used, leading to confusion if a team member with higher limits joined
  * Now there is a server configurable `party_limit_mode` to control this
  * LARGEST (default): use the limits of the team member with the largest limits
  * SUM: team limit is the sum of all team members' limits
  * OWNER: use the owner limits only (old behaviour)
  * AVERAGE: use an average of all members' limits
  * Note that limits can't be fully calculated until players actually log in
  * Also added `hard_team_claim_limit` and `hard_team_force_limit` server configurables; if set to a non-zero value, these hard limits apply regardless of calculated team limits
* Integration with the Waystones mod has been added back

### Fixed
* Fixed fake death markers sometimes being created (generally when some other mod cancelled a player death)
* Fixed forced chunks sometimes staying loaded when they shouldn't (e.g. if all players in a team have logged out and offline force-loading is disabled)
* Fixed forced chunks sometimes not getting loaded when they should (e.g. player logs in when offline-forceloading is disabled)
* Fixed team data not getting properly saved to disk when a player leaves a party team
* Fixed deleted party team data not getting purged from disk
* Fixed Ghast fireballs being able to grief protected chunks
  * A more fundamental problem here is that not all explosions sources can be determined, and Ghast fireballs fall into this category. Previously such explosions were not protected against but are now.
  * Added a server-side config item `protect_unknown_explosions` (default true) to control this behaviour
* Waypoint beacons now render correctly below Y level 0 (down to the world's min height, e.g. -64 for overworld)