# 🌆 Dynamic Liberty

### GTA IV Complete Edition — Steam Deck-First Trainer
### CLEO Redux + ImGuiRedux | Controller-First | Open Source

**Created & Project Directed by VivenzoV**

Dynamic Liberty is an open-source trainer for **Grand Theft Auto IV: Complete Edition**, designed from the beginning around **Steam Deck and controller gameplay**.

Instead of using the traditional ScriptHook-based trainer architecture, Dynamic Liberty is built around **CLEO Redux + ImGuiRedux** and uses GTA IV's native scripting functionality to create a lightweight, controller-friendly sandbox.

The project is being developed openly so other GTA IV players and programmers can test it, improve it, fork it, fix it, and discover new things GTA IV can still do.

> Built on an original Steam Deck.
> Tested through actual gameplay.
> Open sourced for the GTA IV community.

---

# 🎮 CONTROLS

Dynamic Liberty is designed so normal trainer operation does not require a keyboard.

| Steam Deck / Xbox Control | Action |
|---|---|
| **LB + RB** | Open / Close Dynamic Liberty |
| **D-Pad Up / Down** | Navigate |
| **A** | Select |
| **B** | Back |

The controller interface is a primary feature of Dynamic Liberty rather than an afterthought.

---

# 💻 PRIMARY TEST HARDWARE

Dynamic Liberty is currently developed and tested on:

### ✅ Original Steam Deck — 256 GB LCD Model

- Original-generation LCD Steam Deck
- 256 GB storage configuration
- SteamOS
- Proton
- Built-in Steam Deck controls
- GTA IV: Complete Edition

Dynamic Liberty was not developed exclusively on newer Steam Deck OLED hardware.

The original LCD Steam Deck is the project's primary development and testing machine.

---

# 📦 REQUIREMENTS

## Grand Theft Auto IV: Complete Edition

The current development target is:

**Grand Theft Auto IV: Complete Edition — Steam**

---

## CLEO Redux

Dynamic Liberty is a JavaScript trainer running through **CLEO Redux**.

GTA IV is a 32-bit title, so use the appropriate CLEO Redux build for GTA IV.

Official project:

https://github.com/cleolibrary/CLEO-Redux

---

## ImGuiRedux

Dynamic Liberty uses **ImGuiRedux** to render its graphical interface.

The current Steam Deck build uses:

`ImGuiReduxWin32.cleo`

Without ImGuiRedux, the Dynamic Liberty graphical menu will not function.

---

## Dynamic Liberty

Download the latest Dynamic Liberty `.js` release.

Example:

`DynamicLibertyTrainer.js`

The trainer must remain a JavaScript file.

Do not leave it as:

`DynamicLibertyTrainer.txt`

---

# 🛠️ STEAM DECK INSTALLATION

Dynamic Liberty is currently beta software.

Backing up your GTA IV installation before installing mods is recommended.

## 1 — Install GTA IV

Install:

**Grand Theft Auto IV: Complete Edition**

through Steam.

Launch GTA IV normally at least once.

---

## 2 — Enter Steam Deck Desktop Mode

Press:

**STEAM → Power → Switch to Desktop**

Open Steam in Desktop Mode.

Find GTA IV.

Go to:

**Properties → Installed Files → Browse**

This opens the GTA IV installation directory.

---

## 3 — Install CLEO Redux

Download the GTA IV-compatible **32-bit CLEO Redux** release from the official CLEO Redux project.

Install CLEO Redux according to its official GTA IV installation instructions.

Verify that CLEO Redux loads correctly before installing large numbers of additional scripts.

---

## 4 — Install ImGuiRedux

Install the GTA IV-compatible ImGuiRedux component.

Dynamic Liberty currently uses:

`ImGuiReduxWin32.cleo`

Make sure ImGuiRedux is placed where your CLEO Redux installation can load it.

---

## 5 — Install Dynamic Liberty

Download:

`DynamicLibertyTrainer.js`

Place the script inside the CLEO Redux scripts location used by your GTA IV installation.

The important part is that the file ends in:

`.js`

not:

`.txt`

---

## 6 — Launch GTA IV

Return to Gaming Mode.

Start GTA IV normally through Steam.

Load into the game.

Once Niko is controllable, press:

### **LB + RB**

Dynamic Liberty should appear.

Use:

**D-Pad** — Navigate

**A** — Select

**B** — Back

**LB + RB** — Close Trainer

---

# 🚀 CURRENT FEATURES

Dynamic Liberty is designed as more than a basic cheat list.

## 🪖 Tactical Squad

Create and control a bodyguard squad.

Current systems include:

- Recruit bodyguards
- Multiple squad members
- Follow / regroup
- Hold position
- Guard position
- Combat commands
- Weapon loadouts
- Accuracy controls
- Fire-rate controls
- Cover behavior
- Health controls
- Invincibility options
- Vehicle boarding
- Drive-by support
- Vehicle combat behavior
- Squad presets
- Experimental GTA IV AI behaviors
- Dismiss / release squad

---

# 🔥 City War Lab

Turn Liberty City into a dynamic NPC combat sandbox.

Features include:

- Street skirmishes
- Larger street wars
- Full war mode
- AR-equipped fighters
- SMG-equipped fighters
- Mixed weapon groups
- Adjustable accuracy
- Adjustable fire rate
- Cover behavior
- Player-neutral war
- Carjackers
- Aggressive driving
- Incendiary effects
- Dynamic target reassignment
- Mad-driver behavior

The objective is to use GTA IV's existing AI systems to create organic chaos rather than simply spawning explosions everywhere.

---

# 👤 Player Lab

Player experimentation options including:

- Health
- Armor
- Invincibility
- Visibility
- Position freezing
- Gravity experiments
- Windscreen ejection behavior
- Pedestrian-ignore behavior

---

# 🚗 Vehicle Spawner

Spawn GTA IV vehicles through controller navigation.

Categories include:

- Sports / Super
- Muscle / Classics
- Sedans
- SUVs / Vans
- Trucks / Utility
- Taxis / Service
- Emergency
- Motorcycles
- Boats
- Helicopters

---

# 🔧 Vehicle Tools & Vehicle Lab

Current vehicle experiments include:

- Repair vehicle
- Clean vehicle
- Damage controls
- Visible damage
- Vehicle proofs
- Collision resistance
- Watertight vehicles
- Vehicle visibility
- Vehicle freezing
- Upright / flip recovery
- Door locking
- Forward-speed experiments
- Traction manipulation
- Headlight intensity

---

# 🏁 Sleeper ECU

Dynamic Liberty includes an experimental runtime vehicle-performance system called:

### Sleeper ECU

Instead of permanently rewriting `handling.dat`, the trainer can apply additional runtime vehicle force that tapers as vehicle speed increases.

The goal is to experiment with making ordinary Liberty City vehicles feel more powerful without permanently modifying their handling files.

---

# 🌎 World Control

Manipulate Liberty City's environment.

Current options include:

- Traffic density
- Pedestrian density
- Aggressive pedestrians
- Armed pedestrians
- Game speed
- Train behavior
- Wind
- Mad drivers

---

# 🚨 Police

Police-related controls include:

- Wanted levels
- Police helicopter behavior
- Police response experimentation
- Restricted-island access assistance

---

# 🕐 Time / Clock

Choose any hour of the GTA IV day.

Includes:

- 00:00 through 23:00
- Freeze clock

---

# 📦 Sandbox

Experimental sandbox tools include:

- Money drops
- Money scatter
- Object spawning
- Object placement
- Attach objects to vehicles
- Cargo experimentation
- Object position adjustment
- Object rotation
- Island access mode

This section is intended to continue growing.

---

# 🎬 WOW MODES

Dynamic Liberty includes presets that combine multiple world systems.

Current examples include:

- Movie City
- Ghost City
- Purge Hour
- Night Siege
- Fight Club City
- Rush Hour Hell
- Slow-Mo Mayhem
- Manhunt
- Blackout Run
- Swarm City
- Convoy Escape
- Cash Chaos

These presets manipulate combinations of traffic, pedestrians, time, police, AI behavior and game speed.

---

# 🧩 TESTED MOD COMPATIBILITY

Dynamic Liberty's real development installation currently runs alongside:

## ✅ FusionFix

Dynamic Liberty has been tested while **FusionFix** is installed and active.

FusionFix is:

### OPTIONAL

It is not required by Dynamic Liberty.

SteamOS/Proton users should follow FusionFix's own Linux/Wine/Proton installation instructions.

---

## ✅ First Degree 154 Vehicle Addon Pack

Dynamic Liberty has also been tested with the:

### GTA IV 154 Vehicle Addon Pack / First Degree Vehicle Pack

The development Steam Deck is currently running:

**Dynamic Liberty + CLEO Redux + ImGuiRedux + FusionFix + the 154 Vehicle Addon Pack**

together.

The 154 Vehicle Pack is:

### OPTIONAL

Dynamic Liberty does not contain or redistribute the vehicle pack.

All credit for that project belongs to its original creator(s).

---

# 🧪 CURRENT COMPATIBILITY MATRIX

| Configuration | Status |
|---|---|
| Original 256 GB LCD Steam Deck | ✅ Primary development hardware |
| GTA IV Complete Edition | ✅ Tested |
| Steam version | ✅ Tested |
| SteamOS | ✅ Tested |
| Proton | ✅ Tested |
| Steam Deck controller | ✅ Primary input |
| CLEO Redux | ✅ Required / Tested |
| ImGuiReduxWin32 | ✅ Required / Tested |
| FusionFix | ✅ Tested alongside Dynamic Liberty |
| First Degree 154 Vehicle Pack | ✅ Tested alongside Dynamic Liberty |
| Steam Deck OLED | 🧪 Community testing wanted |
| Windows PC | 🧪 Community testing wanted |
| Other Linux distributions | 🧪 Community testing wanted |

---

# ⚠️ LARGE MOD PACKS

GTA IV is an older 32-bit game.

Large vehicle packs, texture modifications and multiple script systems can place additional pressure on GTA IV's engine and memory.

If Dynamic Liberty works normally on a relatively clean installation but crashes after installing several large modifications, test the mods individually before assuming Dynamic Liberty is responsible.

One of the goals of this project is to remain relatively lightweight and avoid unnecessary persistent entities.

---

# 🐛 REPORTING BUGS

Please use **GitHub Issues** when reporting problems.

A useful report should include:

- Steam Deck or PC
- Steam Deck LCD/OLED if applicable
- GTA IV version
- SteamOS version
- Proton version
- CLEO Redux version
- ImGuiRedux version
- FusionFix version if installed
- Major additional mods
- Dynamic Liberty version
- Feature being used
- Expected behavior
- Actual behavior
- Whether the problem happens repeatedly

For crashes, explain exactly what you were doing immediately before GTA IV closed.

Good reports help everybody.

---

# 🤝 COMMUNITY DEVELOPMENT

### Dynamic Liberty is open source on purpose.

This isn't meant to be a trainer that gets abandoned as one person's private project.

If you're an experienced GTA IV modder and see something that can be improved:

**Improve it.**

If you know a safer native:

**Tell us.**

If you find inefficient code:

**Optimize it.**

If you discover some obscure RAGE/GTA IV behavior:

**Turn it into something fun.**

If you want to build your own version:

**Fork the project.**

Pull requests, forks, native research, bug fixes, documentation, translations and new features are welcome.

Please preserve original project attribution when redistributing modified versions.

The goal is to give the GTA IV community another foundation to build from.

---

# 🤖 AI-ASSISTED DEVELOPMENT

Dynamic Liberty has an unusual development history.

The project was started by **VivenzoV**, a GTA player without a traditional GTA scripting/mod-development background.

Modern AI systems were used as a collaborative development team.

### AI Development Team

**ChatGPT — OpenAI**

**Claude — Anthropic**

**Gemini — Google**

Different AI systems were used for:

- Code generation
- Code review
- GTA IV native research
- Debugging
- Architecture discussion
- Feature development
- Challenging assumptions made by other AI systems

VivenzoV handled:

- Original concept
- Project direction
- Feature decisions
- Real-world hardware testing
- Bug discovery
- Gameplay testing
- Determining which changes actually worked

When documentation, generated code and actual game behavior disagreed:

### The Steam Deck got the final vote.

---

# 🏆 PROJECT ORIGIN

Dynamic Liberty started with a simple problem:

### "Why isn't there a GTA IV trainer that feels like it was actually designed for my Steam Deck?"

That turned into an experiment:

Could a regular GTA IV player use **CLEO Redux, ImGuiRedux, modern AI development tools and real hardware testing** to build a controller-first trainer specifically around GTA IV Complete Edition on Steam Deck?

Dynamic Liberty is the result.

The first functional versions were designed, debugged and tested directly on an:

### Original 256 GB LCD Steam Deck

To our knowledge, Dynamic Liberty is the first publicly released GTA IV Complete Edition trainer specifically designed around a:

### Steam Deck-first + CLEO Redux + ImGuiRedux architecture.

This does NOT mean Dynamic Liberty invented GTA IV trainers, CLEO Redux, ImGui trainers or GTA IV modding.

It describes the specific architecture and Steam Deck-first design approach used by this project.

If an earlier publicly available GTA IV project using the same approach is discovered, please open a GitHub Issue with a link.

It will be documented and credited.

Open source should preserve history.

---

# ❤️ CREDITS

## Creator & Project Director

### VivenzoV

Original concept, project direction, feature design, hardware testing and release management.

---

## AI Development Team

### ChatGPT — OpenAI
Development, debugging, architecture, research and code review.

### Claude — Anthropic
Independent auditing, GTA IV native verification and code review.

### Gemini — Google
Independent review, feature development and GTA IV native experimentation.

---

## Technology & Community

Special thanks to the people behind:

- CLEO Redux
- ImGuiRedux
- FusionFix
- Sanny Builder
- GTA IV native documentation
- GTAForums
- GTA IV modding community

### First Degree 154 Vehicle Addon Pack

Full credit belongs to the project's respective creator(s) and contributors.

Dynamic Liberty does not contain or redistribute their assets.

---

## Rockstar Games

Grand Theft Auto IV and all related trademarks and game assets belong to their respective owners.

Dynamic Liberty is an unofficial community modification.

It is not affiliated with or endorsed by Rockstar Games or Take-Two Interactive.

---

# 📜 OPEN SOURCE

Dynamic Liberty is intended to remain an open-source community project.

A formal open-source license should accompany the repository.

Forking and modification are encouraged.

Please preserve the original Dynamic Liberty / VivenzoV attribution when redistributing derivative versions according to the project's license.

---

# 🌆 ONE MORE THING

Liberty City is still one of the most interesting open worlds ever created.

Dynamic Liberty exists because there's still more we can make it do.

**Use it.**

**Test it.**

**Break it.**

**Fix it.**

**Improve it.**

**Share it.**

### Welcome back to Liberty City.

— **VivenzoV**
