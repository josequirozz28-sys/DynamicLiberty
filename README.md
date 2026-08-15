🌆 Dynamic Liberty

GTA IV Complete Edition — Steam Deck-First Trainer

CLEO Redux + ImGuiRedux | Controller-First | Open Source

Created & Project Directed by VivenzoV

Dynamic Liberty is an open-source trainer and dynamic-world simulation project for Grand Theft Auto IV: Complete Edition, designed from the beginning around Steam Deck and controller gameplay.

Instead of using the traditional ScriptHook-based trainer architecture, Dynamic Liberty is built around CLEO Redux + ImGuiRedux and uses GTA IV’s native scripting functionality to create a lightweight, controller-friendly sandbox that is gradually evolving into a more reactive Liberty City simulation.

The project is being developed openly so other GTA IV players and programmers can test it, improve it, fork it, fix it, and discover new things GTA IV can still do.

Built on an original Steam Deck.
Tested through actual gameplay.
Open sourced for the GTA IV community.

Current Development Build

v6.1.4 — Aftermath + EMS Response Alpha

⸻

🎮 CONTROLS

Dynamic Liberty is designed so normal trainer operation does not require a keyboard.

Steam Deck / Xbox Control	Action
LB + RB	Open / Close Dynamic Liberty
D-Pad Up / Down	Navigate
A	Select
B	Back

The controller interface is a primary feature of Dynamic Liberty rather than an afterthought.

⸻

💻 PRIMARY TEST HARDWARE

Dynamic Liberty is currently developed and tested on:

✅ Original Steam Deck — 256 GB LCD Model

* Original-generation LCD Steam Deck
* 256 GB storage configuration
* SteamOS
* Proton
* Built-in Steam Deck controls
* GTA IV: Complete Edition

Dynamic Liberty was not developed exclusively on newer Steam Deck OLED hardware.

The original LCD Steam Deck is the project’s primary development and testing machine.

⸻

📦 REQUIREMENTS

Grand Theft Auto IV: Complete Edition

The current development target is:

Grand Theft Auto IV: Complete Edition — Steam

⸻

CLEO Redux

Dynamic Liberty is a JavaScript trainer running through CLEO Redux.

GTA IV is a 32-bit title, so use the appropriate CLEO Redux build for GTA IV.

Official project:

https://github.com/cleolibrary/CLEO-Redux

⸻

ImGuiRedux

Dynamic Liberty uses ImGuiRedux to render its graphical interface.

The current Steam Deck build uses:

ImGuiReduxWin32.cleo

Without ImGuiRedux, the Dynamic Liberty graphical menu will not function.

⸻

Dynamic Liberty

Download the latest Dynamic Liberty .js release.

Current development build:

DynamicLibertyTrainer_v6_1_4_AFTERMATH_EMS_RESPONSE_ALPHA.js

If the downloaded release is provided as a .txt source file, rename it to:

.js

before placing it in the CLEO scripts location.

The trainer must run as JavaScript.

⸻

🛠️ STEAM DECK INSTALLATION

Dynamic Liberty is currently alpha software.

Backing up your GTA IV installation before installing mods is recommended.

1 — Install GTA IV

Install:

Grand Theft Auto IV: Complete Edition

through Steam.

Launch GTA IV normally at least once.

⸻

2 — Enter Steam Deck Desktop Mode

Press:

STEAM → Power → Switch to Desktop

Open Steam in Desktop Mode.

Find GTA IV.

Go to:

Properties → Installed Files → Browse

This opens the GTA IV installation directory.

⸻

3 — Install CLEO Redux

Download the GTA IV-compatible 32-bit CLEO Redux release from the official CLEO Redux project.

Install CLEO Redux according to its official GTA IV installation instructions.

Verify that CLEO Redux loads correctly before installing large numbers of additional scripts.

⸻

4 — Install ImGuiRedux

Install the GTA IV-compatible ImGuiRedux component.

Dynamic Liberty currently uses:

ImGuiReduxWin32.cleo

Make sure ImGuiRedux is placed where your CLEO Redux installation can load it.

⸻

5 — Install Dynamic Liberty

Download the latest Dynamic Liberty trainer source.

For v6.1.4:

DynamicLibertyTrainer_v6_1_4_AFTERMATH_EMS_RESPONSE_ALPHA.txt

Rename it to:

DynamicLibertyTrainer_v6_1_4_AFTERMATH_EMS_RESPONSE_ALPHA.js

Place the script inside the CLEO Redux scripts location used by your GTA IV installation.

The important part is that the final file ends in:

.js

not:

.txt

⸻

6 — Launch GTA IV

Return to Gaming Mode.

Start GTA IV normally through Steam.

Load into the game.

Once Niko is controllable, press:

LB + RB

Dynamic Liberty should appear.

Use:

D-Pad — Navigate

A — Select

B — Back

LB + RB — Close Trainer

⸻

🚀 CURRENT FEATURES

Dynamic Liberty is designed as more than a basic cheat list.

The project is gradually moving toward a larger reactive-world architecture where police, gangs, emergency services, AI actors and future ambient events can interact instead of existing as isolated trainer buttons.

⸻

🪖 Tactical Squad

Create and control a bodyguard squad.

Current systems include:

* Recruit bodyguards
* Multiple squad members
* Follow / regroup
* Hold position
* Guard position
* Combat commands
* Explicit hostile targeting
* Attack selected active ped
* Weapon loadouts
* Accuracy controls
* Fire-rate controls
* Cover behavior
* Health controls
* Invincibility options
* Vehicle boarding
* Drive-by support
* Vehicle combat behavior
* Squad presets
* Experimental GTA IV AI behaviors
* Dismiss / release squad

Recent builds improved squad combat so guards can explicitly target Dynamic Liberty-managed enemies rather than relying only on GTA IV’s generic hated-target behavior.

⸻

🔥 City War 2.0

Turn Liberty City into a managed NPC combat sandbox.

Current presets include:

* 3v3 Skirmish
* 5v5 City War
* 8v8 Large City War

Current systems include:

* Dedicated Team A / Team B fighters
* Curated gang character pools
* Albanian / Russian faction-style model pools
* SMG-equipped fighters
* AR-equipped fighters
* Mixed weapon groups
* Adjustable accuracy
* Adjustable fire rate
* Cover behavior
* Player-neutral war
* Carjackers
* Aggressive driving
* Dynamic target reassignment
* Managed actor lifecycle
* Spawn budgeting
* Startup rollback
* Visible-despawn protection
* Police response
* Graceful event ending
* Aftermath
* EMS response

Random civilians are no longer intentionally turned into riflemen for serious City War encounters.

The objective is to use GTA IV’s existing AI systems to create organic chaos instead of simply spawning explosions everywhere.

⸻

🚔 PHYSICAL POLICE RESPONSE

City War now includes scripted police-response units that physically arrive at the battle.

Rather than spawning officers directly in the middle of the fight, Dynamic Liberty stages police vehicles on road nodes away from the immediate scene.

Police response features include:

* Physical cruiser arrival
* Sirens
* Driver + passenger police crews
* Officers exiting vehicles
* Explicit gang-fighter targeting
* Reinforcement delays
* Response diagnostics
* Visibility-aware cleanup

Current balance:

City War	Maximum Scripted Police Response
3v3	1 cruiser
5v5	2 cruisers
8v8	3 cruisers

The third police response unit was added in v6.1.4 to better balance the larger 8v8 battles.

⸻

🚑 CITY WAR AFTERMATH + EMS

v6.1.4 introduces an expanded City War lifecycle:

OFF → STARTING → ACTIVE → ENDING → AFTERMATH → OFF

Instead of immediately deleting an active battle, Dynamic Liberty can now end the war more naturally.

During ENDING:

* Gang reinforcements stop
* New carjacker assignments stop
* Existing cops remain engaged
* No new police units are dispatched
* Remaining gang fighters are allowed to finish the active encounter

Once the gang threat has been eliminated, Dynamic Liberty enters:

AFTERMATH

If Dynamic Liberty police officers were killed or seriously injured, an ambulance can respond to the scene.

EMS features include:

* Police-only casualty tracking
* Ambulance staged away from the immediate scene
* Physical ambulance arrival
* Siren response
* Two paramedics
* Individual casualty assignment
* Maximum of two treatment attempts per aftermath
* GTA IV native REVIVE_INJURED_PED experimentation
* Paramedic return-to-ambulance behavior
* Visibility-aware cleanup

EMS ONLY TARGETS DYNAMIC LIBERTY POLICE CASUALTIES

The system does not intentionally treat:

* Gang fighters
* Random civilians
* Bodyguards
* Aggressive Peds
* Niko

The exact visual behavior of revived officers remains experimental and is still being tested on GTA IV Complete Edition hardware.

⸻

👤 Player Lab

Player experimentation options including:

* Health
* Armor
* Invincibility
* Visibility
* Position freezing
* Gravity experiments
* Windscreen ejection behavior
* Pedestrian-ignore behavior

⸻

🚗 Vehicle Spawner

Spawn GTA IV vehicles through controller navigation.

Categories include:

* Sports / Super
* Muscle / Classics
* Sedans
* SUVs / Vans
* Trucks / Utility
* Taxis / Service
* Emergency
* Motorcycles
* Boats
* Helicopters

⸻

🔧 Vehicle Tools & Vehicle Lab

Current vehicle experiments include:

* Repair vehicle
* Clean vehicle
* Damage controls
* Visible damage
* Vehicle proofs
* Collision resistance
* Watertight vehicles
* Vehicle visibility
* Vehicle freezing
* Upright / flip recovery
* Door locking
* Forward-speed experiments
* Traction manipulation
* Headlight intensity

⸻

🏁 Sleeper ECU

Dynamic Liberty includes an experimental runtime vehicle-performance system called:

Sleeper ECU

Instead of permanently rewriting handling.dat, the trainer can apply additional runtime vehicle force that tapers as vehicle speed increases.

The goal is to experiment with making ordinary Liberty City vehicles feel more powerful without permanently modifying their handling files.

⸻

🌎 World Control

Manipulate Liberty City’s environment.

Current options include:

* Traffic density
* Pedestrian density
* Aggressive pedestrians
* Armed pedestrians
* Game speed
* Train behavior
* Wind
* Mad drivers

⸻

🚨 Police

Police-related controls include:

* Wanted levels
* Police helicopter behavior
* Global police-system controls
* Physical City War police response
* Up to three scripted response cruisers
* Police-response diagnostics
* Police casualty tracking
* Restricted-island access assistance

⸻

🕐 Time / Clock

Choose any hour of the GTA IV day.

Includes:

* 00:00 through 23:00
* Freeze clock

⸻

📦 Sandbox

Experimental sandbox tools include:

* Money drops
* Money scatter
* Object spawning
* Object placement
* Attach objects to vehicles
* Cargo experimentation
* Object position adjustment
* Object rotation
* Island access mode

This section is intended to continue growing.

⸻

🎬 SCENARIOS

The older WOW MODES menu has been cleaned up and renamed:

SCENARIOS

The goal now is for each preset to represent an actual gameplay scenario instead of simply combining random world settings.

Current visible scenarios include:

* Purge Hour
* Night Siege
* Slow-Mo Mayhem
* City War 5v5 + Police + EMS
* Reset World

Older experimental presets such as Movie City, Ghost City, Fight Club City, Rush Hour Hell, Manhunt, Blackout Run, Swarm City, Convoy Escape and Cash Chaos may still exist in the source for research or future reuse, but are no longer exposed as primary scenarios.

⸻

👁️ IMMERSION & ENTITY LIFECYCLE

One of Dynamic Liberty’s current development rules is:

If the player can see it, don’t automatically despawn it.

Earlier development builds exposed situations where actors or vehicles could be returned to GTA IV’s ownership while still visible.

Modern Dynamic Liberty builds use visibility checks to delay automatic cleanup.

This applies to systems including:

* City War fighters
* Escaping actors
* Getaway vehicles
* Police units
* Police vehicles
* EMS units
* Ambulances

The goal is to make background entity management less noticeable to the player.

⸻

🧩 TESTED MOD COMPATIBILITY

Dynamic Liberty’s real development installation currently runs alongside:

✅ FusionFix

Dynamic Liberty has been tested while FusionFix is installed and active.

FusionFix is:

OPTIONAL

It is not required by Dynamic Liberty.

SteamOS/Proton users should follow FusionFix’s own Linux/Wine/Proton installation instructions.

⸻

✅ First Degree 154 Vehicle Addon Pack

Dynamic Liberty has also been tested with the:

GTA IV 154 Vehicle Addon Pack / First Degree Vehicle Pack

The development Steam Deck has been tested running:

Dynamic Liberty + CLEO Redux + ImGuiRedux + FusionFix + the 154 Vehicle Addon Pack

together.

The 154 Vehicle Pack is:

OPTIONAL

Dynamic Liberty does not contain or redistribute the vehicle pack.

All credit for that project belongs to its original creator(s).

⸻

🧪 CURRENT COMPATIBILITY MATRIX

Configuration	Status
Original 256 GB LCD Steam Deck	✅ Primary development hardware
GTA IV Complete Edition	✅ Tested
Steam version	✅ Tested
SteamOS	✅ Tested
Proton	✅ Tested
Steam Deck controller	✅ Primary input
CLEO Redux	✅ Required / Tested
ImGuiReduxWin32	✅ Required / Tested
FusionFix	✅ Tested alongside Dynamic Liberty
First Degree 154 Vehicle Pack	✅ Tested alongside Dynamic Liberty
Steam Deck OLED	🧪 Community testing wanted
Windows PC	🧪 Community testing wanted
Other Linux distributions	🧪 Community testing wanted

⸻

⚠️ LARGE MOD PACKS

GTA IV is an older 32-bit game.

Large vehicle packs, texture modifications and multiple script systems can place additional pressure on GTA IV’s engine, entity pools, tasks and memory.

If Dynamic Liberty works normally on a relatively clean installation but crashes after installing several large modifications, test the mods individually before assuming Dynamic Liberty is responsible.

One of the goals of this project is to remain relatively lightweight and avoid unnecessary persistent entities.

Dynamic Liberty increasingly tracks actor and vehicle ownership specifically to avoid uncontrolled entity growth.

⸻

🧪 CURRENT ALPHA / KNOWN ISSUES

Dynamic Liberty remains experimental.

Current areas still being tested include:

* EMS officer revival / get-up behavior
* Paramedic treatment presentation
* Tactical Squad vehicle firing
* Heavy 8v8 + bodyguards + 3 police cruisers
* Police / EMS pathfinding through blocked intersections
* Rare spawn placement around bridges, docks and multilevel geometry
* Global resource budgeting across all Dynamic Liberty systems
* Some experimental native behaviors
* ImGui Debug underlay / scrolling visual artifact

Hardware behavior always takes priority over assumptions made from documentation.

⸻

🐛 REPORTING BUGS

Please use GitHub Issues when reporting problems.

A useful report should include:

* Steam Deck or PC
* Steam Deck LCD/OLED if applicable
* GTA IV version
* SteamOS version
* Proton version
* CLEO Redux version
* ImGuiRedux version
* FusionFix version if installed
* Major additional mods
* Dynamic Liberty version
* Feature being used
* Expected behavior
* Actual behavior
* Whether the problem happens repeatedly

For crashes, explain exactly what you were doing immediately before GTA IV closed.

For City War, Police Response or EMS issues, include any status/failure information displayed by Dynamic Liberty.

Good reports help everybody.

⸻

🧠 WHERE DYNAMIC LIBERTY IS GOING

Dynamic Liberty started as a trainer.

It is gradually becoming something larger.

The long-term simulation system is currently referred to as:

Liberty Pulse

The idea is to let Liberty City react to what happens inside it.

Possible future systems include:

* Neighborhood heat
* Violence memory
* Gang pressure
* Police pressure
* Civilian fear
* Dynamic gang pursuits
* Police escalation
* Witness behavior
* Robberies
* Drive-bys
* Arrest outcomes
* Emergency-service response
* Crime-scene behavior
* Tow-truck response
* Traffic rerouting
* Ambient event chains

The design philosophy is:

Don’t script the whole story.

Create the pressure and let GTA IV’s AI, physics and world systems decide how the story ends.

⸻

🤝 COMMUNITY DEVELOPMENT

Dynamic Liberty is open source on purpose.

This isn’t meant to be a trainer that gets abandoned as one person’s private project.

If you’re an experienced GTA IV modder and see something that can be improved:

Improve it.

If you know a safer native:

Tell us.

If you find inefficient code:

Optimize it.

If you discover some obscure RAGE/GTA IV behavior:

Turn it into something fun.

If you want to build your own version:

Fork the project.

Pull requests, forks, native research, bug fixes, documentation, translations and new features are welcome.

Please preserve original project attribution according to the project’s license.

The goal is to give the GTA IV community another foundation to build from.

⸻

🤖 AI-ASSISTED DEVELOPMENT

Dynamic Liberty has an unusual development history.

The project was started by VivenzoV, a GTA player without a traditional GTA scripting/mod-development background.

Modern AI systems were used as a collaborative development team.

AI Development Team

ChatGPT — OpenAI

Claude — Anthropic

Gemini — Google

Different AI systems have been used for:

* Code generation
* Code review
* GTA IV native research
* Debugging
* Architecture discussion
* Feature development
* Challenging assumptions made by other AI systems

VivenzoV handled:

* Original concept
* Project direction
* Feature decisions
* Real-world hardware testing
* Bug discovery
* Gameplay testing
* Determining which changes actually worked

When documentation, generated code and actual game behavior disagreed:

The Steam Deck got the final vote.

⸻

🏆 PROJECT ORIGIN

Dynamic Liberty started with a simple problem:

“Why isn’t there a GTA IV trainer that feels like it was actually designed for my Steam Deck?”

That turned into an experiment:

Could a regular GTA IV player use CLEO Redux, ImGuiRedux, modern AI development tools and real hardware testing to build a controller-first trainer specifically around GTA IV Complete Edition on Steam Deck?

Dynamic Liberty is the result.

The first functional versions were designed, debugged and tested directly on an:

Original 256 GB LCD Steam Deck

Based on the public projects currently known to us, Dynamic Liberty appears to be the first publicly documented GTA IV Complete Edition trainer specifically designed around a:

Steam Deck-first + CLEO Redux JavaScript + ImGuiRedux architecture.

This does NOT mean Dynamic Liberty invented GTA IV trainers, CLEO Redux, ImGui trainers, controller trainers or GTA IV modding.

It describes the specific combination of architecture and Steam Deck-first design used by this project.

If an earlier publicly available GTA IV project using the same approach is discovered, please open a GitHub Issue with a link.

It will be documented and credited.

Open source should preserve history.

⸻

❤️ CREDITS

Creator & Project Director

VivenzoV

Original concept, project direction, feature design, hardware testing and release management.

⸻

AI Development Team

ChatGPT — OpenAI

Development, debugging, architecture, research and code review.

Claude — Anthropic

Independent auditing, GTA IV native verification and code review.

Gemini — Google

Independent review, feature development and GTA IV native experimentation.

⸻

Technology & Community

Special thanks to the people behind:

* CLEO Redux
* ImGuiRedux
* FusionFix
* Sanny Builder
* GTA IV native documentation
* GTAForums
* GTAMods
* GTA IV modding community

First Degree 154 Vehicle Addon Pack

Full credit belongs to the project’s respective creator(s) and contributors.

Dynamic Liberty does not contain or redistribute their assets.

⸻

Rockstar Games

Grand Theft Auto IV and all related trademarks and game assets belong to their respective owners.

Dynamic Liberty is an unofficial community modification.

It is not affiliated with or endorsed by Rockstar Games or Take-Two Interactive.

⸻

📜 OPEN SOURCE

Dynamic Liberty is released under the:

MIT License

Forking and modification are encouraged.

Please preserve required copyright and license notices when redistributing modified versions according to the MIT License.

See:

LICENSE

for the full license text.

⸻

📈 CURRENT DEVELOPMENT LINE

Recent major builds include:

v6.1.0

City War 2.0

v6.1.2

Core Stability Hotfix

v6.1.3

Immersion + Physical Police Response

v6.1.4

City War Aftermath + Police-Only EMS + Third Police Response Unit

⸻

🌆 ONE MORE THING

Liberty City is still one of the most interesting open worlds ever created.

Dynamic Liberty exists because there’s still more we can make it do.

Use it.

Test it.

Break it.

Fix it.

Improve it.

Share it.

Welcome back to Liberty City.

— VivenzoV
