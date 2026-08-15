// Dynamic Liberty Trainer v6.1.4 AFTERMATH + EMS RESPONSE ALPHA
// GTA IV Complete Edition + CLEO Redux (x86)
// Steam Deck controller-native trainer
// Requires: ImGuiReduxWin32.cleo
//
// Controls:
// LB + RB = open / close
// D-pad Up/Down = move
// A = select
// B = back
//
// Uses GTA IV native names from the Sanny Builder GTA IV definition set.
// No memory editing is used in this build.

log("Dynamic Liberty Trainer v6.1.4 AFTERMATH + EMS RESPONSE ALPHA loaded");

// ------------------------------------------------------------
// CONTROLLER
// These are the button IDs already proven on the user's setup.
// ------------------------------------------------------------
const PAD = 0;
const BTN = {
    LB: 4,
    RB: 6,
    UP: 8,
    DOWN: 9,
    A: 16,
    B: 17
};

// ------------------------------------------------------------
// STATE
// ------------------------------------------------------------
let menuOpen = false;
let comboLatch = false;
let menuStack = [];
let selected = 0;
let lastRender = 0;
let controlLocked = false;

let frozenTime = false;
let frozenHour = 12;
let lastFreezeUpdate = 0;

let bodyguards = [];
const MAX_BODYGUARDS = 10;
let bodyguardModel = 0;
let lastSquadUpdate = 0;
let squadCombatMode = false;

// v6.1.3 explicit squad targeting. GTA IV's hated-target task only works when
// useful relationship hostility already exists; Dynamic Liberty now keeps its own
// combat target state for known hostile actors instead of assuming that table.
let squadTargetRecords = [];
let squadForcedTarget = 0;
const SQUAD_TARGET_HOLD_MS = 6000;

// GTA IV native Group-based Tactical Squad state.
let squadGroup = 0;
let squadSpacing = 2.0;
let squadFormation = 0;
let squadWeapon = 15; // M4
let squadAccuracy = 72;
let squadShootRate = 100;
let squadHealth = 500;

let squadInvincible = false;
let squadHeadshotImmune = false;
let squadUseCover = true;
let squadDrivebys = true;
let squadUseCarsCombat = true;
let squadLeaveCarCombat = false;
let squadMoveWhenInjured = true;
let squadCoveringFire = true;
let squadClearLosOnly = false;
let squadSignalAfterKill = false;
let squadDropsWeapons = false;
let squadStayInCarWhenJacked = true;
let squadCantBeDraggedOut = true;
let squadCanBeKnockedOffBike = false;
let squadDrunk = false;
let squadBlindRage = false;

const SQUAD_WEAPONS = [
    { label: "UNARMED", id: 0 },
    { label: "BASEBALL BAT", id: 1 },
    { label: "POOL CUE", id: 2 },
    { label: "KNIFE", id: 3 },
    { label: "PISTOL", id: 7 },
    { label: "DEAGLE", id: 9 },
    { label: "SHOTGUN", id: 10 },
    { label: "BARETTA", id: 11 },
    { label: "MICRO UZI", id: 12 },
    { label: "MP5", id: 13 },
    { label: "AK47", id: 14 },
    { label: "M4", id: 15 },
    { label: "SNIPER RIFLE", id: 16 },
    { label: "M40A1", id: 17 },
    { label: "RPG", id: 18 },
    { label: "FLAMETHROWER", id: 19 },
    { label: "MINIGUN", id: 20 }
];

let aggressiveMode = 0; // 0 off, 1 fists, 2 armed
let lastAggroPulse = 0;
let aggroPeds = [];

let trainsEnabled = true;
let policeHelisEnabled = true;

// ------------------------------------------------------------
// SANDBOX STATE
// ------------------------------------------------------------
let accessMode = false;
let placedObjects = [];
let activePlacedObject = 0;
let objectOffset = { x: 0.0, y: -2.2, z: 0.7, rx: 0.0, ry: 0.0, rz: 0.0 };

// ------------------------------------------------------------
// V5.1 PLAYER / VEHICLE / WORLD LAB STATE
// ------------------------------------------------------------
let playerInvincible = false;
let playerInvisible = false;
let playerFrozen = false;
let playerWindscreenEject = false;
let everyoneIgnorePlayer = false;
let savedPlayerPos = null;

let carDamageEnabled = true;
let carVisibleDamageEnabled = true;
let carProofsEnabled = false;
let carStrongEnabled = false;
let carWatertightEnabled = false;
let carInvisible = false;
let carFrozen = false;

let madDriversEnabled = false;

// ------------------------------------------------------------
// V5.3 NATIVE RESCUE / WEATHER / POLICE / PED SPAWNER STATE
// ------------------------------------------------------------
let policeDisabled = false;
let previousCreateRandomCops = true;
let lastPoliceDisabledEnforce = 0;
let lastPoliceAreaClear = 0;

let forcedWeather = -1;

let spawnedPeds = [];
let lastSpawnedPed = 0;
const MAX_SPAWNED_PEDS = 24;

let hazardLightsEnabled = false;
let interiorLightEnabled = false;
let strongAxlesEnabled = false;
let randomBoatsEnabled = true;

// One empty ImGui frame is submitted only when a close needs to clear the
// previous draw buffer. We do not create empty ImGui frames forever.
let imguiClearPending = false;

// ------------------------------------------------------------
// V6.0 SANDBOX TOOLKIT STATE
// ------------------------------------------------------------
let activePed = 0;
let activePedInvincible = false;
let activePedDrunk = false;
let activePedBleeding = false;
let activePedCover = true;
let activePedCarsCombat = true;
let activePedAccuracy = 50;
let activePedShootRate = 100;

let selectedVehicleMode = 0; // 0=current, 1=nearest, 2=last spawned
let lastSpawnedVehicle = 0;
let selectedVehicleHydraulics = false;
let selectedVehicleSkids = false;
let selectedVehicleOnlyPlayerDamage = false;
let allIndicatorsEnabled = false;

// Research note: legacy GTA IV native dumps expose SET_CAR_TRACTION, but no
// verified script-native setter for handling.dat fDriveBias was found. v6.0
// therefore does NOT fake a live FWD/RWD/AWD slider.

// ------------------------------------------------------------
// V5.2 ACCESS / CITY WAR / INCENDIARY STATE
// ------------------------------------------------------------
let previousMaxWantedLevel = 6;
let lastAccessEnforce = 0;

let streetWarLimit = 0; // total managed fighters; split evenly into Team A / Team B
let streetWarPeds = [];
let streetWarTeamA = [];
let streetWarTeamB = [];
let streetWarCarjackers = false;
let streetWarPlayerNeutral = true;
let streetWarWeaponMode = 2; // 0=SMG, 1=AR, 2=mixed (A favors SMG, B favors AR)
let streetWarAccuracy = 50;
let streetWarShootRate = 110;
let streetWarHealth = 400;
let streetWarArmour = 50;
let streetWarUseCover = true;
let streetWarDriveSpeed = 25.0;
let streetWarFireAmmo = false;
let streetWarReinforcements = true;
let streetWarInitialFill = false;
let streetWarSpawnSerial = 0;
let lastStreetWarPulse = 0;
let lastStreetWarRetarget = 0;
let lastCarjackPulse = 0;
let carjackAssignments = [];

// v6.1.2 City War startup transaction.
// A requested war does not become ACTIVE until at least one fighter exists on
// BOTH teams. Failed starts roll back instead of leaving a ghost war state.
let streetWarState = "OFF"; // OFF, STARTING, ACTIVE, ENDING, AFTERMATH
let streetWarStartPulses = 0;
const STREET_WAR_START_MAX_PULSES = 12;
let streetWarSpawnFailures = 0;
let lastStreetWarSpawnFailure = "NONE";

// Reusable managed-actor registry. City War is the first production user;
// Liberty Pulse / Director events can reuse the same ownership + lifecycle model later.
let managedActors = [];
const MAX_MANAGED_ACTORS = 24;

// v6.1.1 actor-lifecycle stabilization.
// A successful carjacker stays script-owned briefly as an ESCAPING actor so the
// getaway is visible before ownership is handed back to GTA IV.
let lastManagedActorPulse = 0;
const MANAGED_ACTOR_PULSE_MS = 500;
const ESCAPER_HANDOFF_MS = 45000;
const ESCAPER_RELEASE_DISTANCE = 180.0;

// Lifetime spawn budget for one City War session. The active managed-actor cap
// limits simultaneous ownership; this second budget prevents an endless
// reinforcement battle from creating unlimited replacement fighters over time.
const CITY_WAR_TOTAL_SPAWN_BUDGET = 48;
let cityWarTotalSpawns = 0;
let cityWarBudgetExhaustedNotified = false;

// v6.1.4 City War police response. 3v3 gets one cruiser, 5v5 gets two,
// and 8v8 can escalate to three. Units physically drive to the incident,
// deploy, then receive explicit gang-fighter targets.
let warPoliceResponseEnabled = true;
let warPoliceUnits = [];
let lastWarPolicePulse = 0;
let warPoliceNextDispatchAt = 0;
let warPoliceSpawnFailures = 0;
let warPoliceLastFailure = "NONE";
const WAR_POLICE_PULSE_MS = 600;
const WAR_POLICE_INITIAL_DELAY_MS = 9000;
const WAR_POLICE_REINFORCEMENT_DELAY_MS = 12000;
const WAR_POLICE_MAX_UNITS = 3;
const WAR_POLICE_ARRIVAL_DISTANCE = 22.0;
const WAR_POLICE_ENROUTE_TIMEOUT_MS = 16000;
const WAR_POLICE_DEPLOY_DELAY_MS = 1400;
const WAR_POLICE_DRIVE_SPEED = 22.0;
const WAR_POLICE_TARGET_HOLD_MS = 6000;

// v6.1.4 graceful ending / aftermath. Ending stops gang reinforcements but lets
// the current fight resolve naturally. EMS is only eligible after the gang
// threat has cleared, and it only treats cops created by warPoliceUnits.
let streetWarEndRequested = false;
let streetWarEndRequestedAt = 0;
let streetWarAftermathStartedAt = 0;
const STREET_WAR_AFTERMATH_MIN_MS = 12000;
const STREET_WAR_ENDING_RETREAT_MS = 25000;

let warEmsEnabled = true;
let warEmsUnit = null;
let warEmsDispatchAt = 0;
let lastWarEmsPulse = 0;
let warEmsSpawnFailures = 0;
let warEmsLastFailure = "NONE";
let warEmsTreatments = 0;
let warEmsTreatedPolice = [];
let warEmsSceneComplete = false;
const WAR_EMS_PULSE_MS = 600;
const WAR_EMS_INITIAL_DELAY_MS = 5000;
const WAR_EMS_ARRIVAL_DISTANCE = 20.0;
const WAR_EMS_ENROUTE_TIMEOUT_MS = 18000;
const WAR_EMS_DEPLOY_DELAY_MS = 1400;
const WAR_EMS_APPROACH_TIMEOUT_MS = 5500;
const WAR_EMS_AID_TIME_MS = 2600;
const WAR_EMS_RETURN_TIMEOUT_MS = 9000;
const WAR_EMS_DRIVE_SPEED = 18.0;
const WAR_EMS_MAX_TREATMENTS = 2;
const WAR_EMS_MAX_SPAWN_RETRIES = 4;

let incendiaryHits = false;
let incendiaryChance = 100;
let incendiaryPeds = [];
let incendiaryCars = [];
let lastIncendiaryPulse = 0;

const WAR_WEAPONS = {
    smg: [12, 13],       // Micro Uzi, MP5
    ar: [14, 15],        // AK47, M4
    mixed: [12, 13, 14, 15]
};

// Curated GTA IV faction models. Runtime validation is mandatory; if a team's
// models are unavailable, City War startup fails/rolls back rather than turning
// a random civilian/hobo into a rifleman.
const WAR_TEAM_MODELS = {
    0: ["M_Y_GALB_LO_01", "M_Y_GALB_LO_02", "M_Y_GALB_LO_03", "M_Y_GALB_LO_04"],
    1: ["M_Y_GRUS_LO_01", "M_Y_GRUS_LO_02", "M_Y_GRUS_HI_02"]
};

// Known GTA IV object-model names used as safe starter cargo presets.
// Each is validated with IS_MODEL_IN_CDIMAGE before creation.
const CARGO_OBJECTS = [
    { label: "CARDBOARD BOX", model: "sexcardboardbox12" },
    { label: "BOX / PROP 09", model: "sexbox09" },
    { label: "BOX / PROP 79", model: "sexbox79" },
    { label: "BOX / PROP 80", model: "sexbox80" },
    { label: "BOX / PROP 81", model: "sexbox81" },
    { label: "BOX / PROP 82", model: "sexbox82" }
];

const MAX_VISIBLE = 9;

// ------------------------------------------------------------
// SAFE NATIVE WRAPPER
// ------------------------------------------------------------
function n(name, ...args) {
    try {
        return native(name, ...args);
    } catch (e) {
        log("Native failed:", name, e);
        showTextBox("DYNAMIC TRAINER\nERROR: " + name);
        return null;
    }
}

function btnPressed(id) {
    return !!n("IS_BUTTON_PRESSED", PAD, id);
}

function btnJust(id) {
    return !!n("IS_BUTTON_JUST_PRESSED", PAD, id);
}

function playerId() {
    return n("GET_PLAYER_ID");
}

function playerPed() {
    const p = playerId();
    if (p === null) return 0;
    return n("GET_PLAYER_CHAR", p) || 0;
}

function playerPos() {
    const ped = playerPed();
    if (!ped) return null;
    return n("GET_CHAR_COORDINATES", ped);
}


// ------------------------------------------------------------
// MENU INPUT LOCK
// While the trainer is open, GTA IV's own gameplay controls are
// disabled. Raw IS_BUTTON_* reads remain available to CLEO so the
// trainer can still read the Steam Deck controller.
// ------------------------------------------------------------
function lockPlayerControl() {
    if (controlLocked) return;

    const p = playerId();
    if (p === null) return;

    n("SET_PLAYER_CONTROL", p, false);
    n("DISABLE_PAUSE_MENU", true);
    controlLocked = true;
}

function unlockPlayerControl() {
    if (!controlLocked) return;

    const p = playerId();
    if (p !== null) {
        n("SET_PLAYER_CONTROL", p, true);
    }

    n("DISABLE_PAUSE_MENU", false);
    controlLocked = false;
}

function notify(text) {
    showTextBox("DYNAMIC TRAINER\n" + text);
}

// ------------------------------------------------------------
// MODEL HELPERS
// ------------------------------------------------------------
function hash(name) {
    return n("GET_HASH_KEY", name);
}

function loadModel(modelHash, timeoutMs = 4000) {
    if (!modelHash) return false;

    if (!n("IS_MODEL_IN_CDIMAGE", modelHash)) {
        return false;
    }

    n("REQUEST_MODEL", modelHash);

    let waited = 0;
    while (!n("HAS_MODEL_LOADED", modelHash) && waited < timeoutMs) {
        wait(50);
        waited += 50;
    }

    return !!n("HAS_MODEL_LOADED", modelHash);
}

function releaseModel(modelHash) {
    if (modelHash) n("MARK_MODEL_AS_NO_LONGER_NEEDED", modelHash);
}

// ------------------------------------------------------------
// VEHICLE SPAWNER
// ------------------------------------------------------------
function spawnVehicle(modelName) {
    const ped = playerPed();
    const pos = playerPos();
    if (!ped || !pos) {
        notify("PLAYER NOT READY");
        return;
    }

    const model = hash(modelName);
    if (!model || !n("IS_MODEL_IN_CDIMAGE", model)) {
        notify(modelName + "\nNOT AVAILABLE IN THIS EPISODE");
        return;
    }

    if (!loadModel(model)) {
        notify("FAILED TO LOAD " + modelName);
        return;
    }

    const heading = n("GET_CHAR_HEADING", ped) || 0;
    const r = heading * Math.PI / 180.0;

    // Spawn a few metres in front and slightly to the side.
    const sx = pos.x + Math.sin(r) * 5.0 + Math.cos(r) * 2.0;
    const sy = pos.y + Math.cos(r) * 5.0 - Math.sin(r) * 2.0;
    const sz = pos.z + 0.5;

    const car = n("CREATE_CAR", model, sx, sy, sz);

    if (car) {
        lastSpawnedVehicle = car;
        n("SET_CAR_HEADING", car, heading);
        notify(modelName + " SPAWNED");
    } else {
        notify("SPAWN FAILED: " + modelName);
    }

    releaseModel(model);
}

function currentCar() {
    const ped = playerPed();
    if (!ped || !n("IS_CHAR_IN_ANY_CAR", ped)) return 0;
    return n("GET_CAR_CHAR_IS_USING", ped) || 0;
}

function fixCurrentCar() {
    const car = currentCar();
    if (!car) {
        notify("GET IN A VEHICLE FIRST");
        return;
    }
    n("FIX_CAR", car);
    n("SET_CAR_HEALTH", car, 1000);
    notify("VEHICLE REPAIRED");
}

function cleanCurrentCar() {
    const car = currentCar();
    if (!car) {
        notify("GET IN A VEHICLE FIRST");
        return;
    }

    // GTA IV exposes a direct dirt-level setter. 0.0 = clean baseline.
    n("SET_VEHICLE_DIRT_LEVEL", car, 0.0);
    notify("VEHICLE DIRT: 0.0 / CLEAN");
}

// ------------------------------------------------------------
// TIME
// ------------------------------------------------------------
function setHour(h) {
    frozenHour = h;
    n("SET_TIME_OF_DAY", h, 0);
    notify("TIME SET: " + String(h).padStart(2, "0") + ":00");
}

function toggleFreezeTime() {
    frozenTime = !frozenTime;
    if (frozenTime) {
        const t = n("GET_TIME_OF_DAY");
        if (t && typeof t.hour !== "undefined") frozenHour = t.hour;
        n("SET_TIME_OF_DAY", frozenHour, 0);
    }
    notify("FREEZE CLOCK: " + (frozenTime ? "ON" : "OFF"));
}

// ------------------------------------------------------------
// BODYGUARD / RECRUIT
// ------------------------------------------------------------
function validGuard(ped) {
    return ped && !!n("DOES_CHAR_EXIST", ped) && !n("IS_CHAR_DEAD", ped);
}

function pruneBodyguards() {
    bodyguards = bodyguards.filter(validGuard);
}

function bodyguardCount() {
    pruneBodyguards();
    return bodyguards.length;
}

function squadGroupExists() {
    return squadGroup && !!n("DOES_GROUP_EXIST", squadGroup);
}

function ensureSquadGroup() {
    // Safer GTA IV route: use Niko's existing player group instead of creating
    // a second group and trying to make the player its leader.
    const p = playerId();
    if (p === null) return false;

    const group = n("GET_PLAYER_GROUP", p) || 0;
    if (!group || !n("DOES_GROUP_EXIST", group)) {
        squadGroup = 0;
        return false;
    }

    squadGroup = group;
    n("SET_GROUP_FORMATION", squadGroup, squadFormation);
    n("SET_GROUP_FORMATION_SPACING", squadGroup, squadSpacing);
    n("SET_GROUP_SEPARATION_RANGE", squadGroup, 80.0);
    return true;
}

function forEachGuard(fn) {
    pruneBodyguards();
    for (const guard of bodyguards) {
        if (validGuard(guard)) fn(guard);
    }
}

function applyGuardLoadout(ped) {
    n("REMOVE_ALL_CHAR_WEAPONS", ped);
    if (squadWeapon !== 0) {
        n("GIVE_WEAPON_TO_CHAR", ped, squadWeapon, 999, true);
        n("SET_CURRENT_CHAR_WEAPON", ped, squadWeapon, true);
    }
}

// Minimal spawn profile. Keep experimental AI flags OUT of the creation path.
// The player can enable those later from the Tactical Squad menus.
function applyGuardSpawnBaseline(ped) {
    n("SET_CHAR_MAX_HEALTH", ped, squadHealth);
    n("SET_CHAR_HEALTH", ped, squadHealth);
    n("ADD_ARMOUR_TO_CHAR", ped, 100);
    n("SET_CHAR_ACCURACY", ped, squadAccuracy);
    n("SET_CHAR_WILL_USE_COVER", ped, squadUseCover);
    n("SET_CHAR_WILL_DO_DRIVEBYS", ped, squadDrivebys);
    n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", ped, squadUseCarsCombat);
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, true);
    applyGuardLoadout(ped);
}

function applyGuardBehavior(ped) {
    // Full profile is only applied when the user chooses profile/toggle actions.
    n("SET_CHAR_MAX_HEALTH", ped, squadHealth);
    n("SET_CHAR_HEALTH", ped, squadHealth);
    n("ADD_ARMOUR_TO_CHAR", ped, 100);

    n("SET_CHAR_ACCURACY", ped, squadAccuracy);
    n("SET_CHAR_SHOOT_RATE", ped, squadShootRate);

    n("SET_CHAR_WILL_USE_COVER", ped, squadUseCover);
    n("SET_CHAR_PROVIDE_COVERING_FIRE", ped, squadCoveringFire);
    n("SET_CHAR_WILL_ONLY_FIRE_WITH_CLEAR_LOS", ped, squadClearLosOnly);

    n("SET_CHAR_WILL_DO_DRIVEBYS", ped, squadDrivebys);
    n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", ped, squadUseCarsCombat);
    n("SET_CHAR_WILL_LEAVE_CAR_IN_COMBAT", ped, squadLeaveCarCombat);
    n("SET_CHAR_WILL_MOVE_WHEN_INJURED", ped, squadMoveWhenInjured);

    n("SET_CHAR_SIGNAL_AFTER_KILL", ped, squadSignalAfterKill);
    n("SET_CHAR_DROPS_WEAPONS_WHEN_DEAD", ped, squadDropsWeapons);

    n("SET_CHAR_INVINCIBLE", ped, squadInvincible);
    n("SET_CHAR_SUFFERS_CRITICAL_HITS", ped, !squadHeadshotImmune);

    n("SET_CHAR_STAY_IN_CAR_WHEN_JACKED", ped, squadStayInCarWhenJacked);
    n("SET_CHAR_CANT_BE_DRAGGED_OUT", ped, squadCantBeDraggedOut);
    n("SET_CHAR_CAN_BE_KNOCKED_OFF_BIKE", ped, squadCanBeKnockedOffBike);

    n("SET_PED_IS_DRUNK", ped, squadDrunk);
    n("SET_PED_IS_BLIND_RAGING", ped, squadBlindRage);

    n("SET_CHAR_NEVER_LEAVES_GROUP", ped, false);
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, true);

    applyGuardLoadout(ped);
}

function configureBodyguard(ped) {
    n("SET_CHAR_RANDOM_COMPONENT_VARIATION", ped);
    applyGuardSpawnBaseline(ped);

    if (ensureSquadGroup()) {
        n("SET_GROUP_MEMBER", squadGroup, ped);
        return true;
    }
    return false;
}

function spawnOneBodyguard(slotIndex) {
    if (bodyguardCount() >= MAX_BODYGUARDS) return false;

    const me = playerPed();
    const pos = playerPos();
    if (!me || !pos) return false;

    if (!bodyguardModel) {
        bodyguardModel = hash("M_Y_GAFR_LO_01");
    }

    if (!loadModel(bodyguardModel)) {
        notify("BODYGUARD MODEL FAILED");
        return false;
    }

    const angle = (slotIndex / MAX_BODYGUARDS) * Math.PI * 2.0;
    const radius = 2.5 + (slotIndex % 2) * 1.0;
    const x = pos.x + Math.cos(angle) * radius;
    const y = pos.y + Math.sin(angle) * radius;

    const guard = n("CREATE_CHAR", 1, bodyguardModel, x, y, pos.z);
    if (!guard || !n("DOES_CHAR_EXIST", guard)) {
        notify("BODYGUARD CREATE FAILED");
        return false;
    }

    bodyguards.push(guard);
    const grouped = configureBodyguard(guard);

    // Fallback manual follow only if the player's GTA IV group was unavailable.
    if (!grouped) {
        const side = ((bodyguards.length % 3) - 1) * 1.2;
        const back = -1.5 - Math.floor(bodyguards.length / 3) * 0.7;
        n("TASK_GOTO_CHAR_OFFSET", guard, me, -1, side, back);
    }

    return true;
}

function recruitBodyguards(amount) {
    pruneBodyguards();

    let wanted = Math.min(amount, MAX_BODYGUARDS - bodyguards.length);
    if (wanted <= 0) {
        notify("SQUAD FULL: 10/10");
        return;
    }

    let added = 0;
    for (let i = 0; i < wanted; i++) {
        if (spawnOneBodyguard(bodyguards.length + i)) added++;
        wait(40);
    }

    releaseModel(bodyguardModel);
    notify("SQUAD: " + bodyguardCount() + "/10");
}

function healBodyguards() {
    pruneBodyguards();
    if (!bodyguards.length) {
        notify("NO ACTIVE BODYGUARDS");
        return;
    }

    for (const guard of bodyguards) {
        n("SET_CHAR_HEALTH", guard, 500);
        n("ADD_ARMOUR_TO_CHAR", guard, 100);
    }

    notify("SQUAD HEALED");
}

function squadEnterCar() {
    pruneBodyguards();
    if (!bodyguards.length) {
        notify("NO ACTIVE BODYGUARDS");
        return;
    }

    const car = currentCar();
    if (!car) {
        notify("GET IN A VEHICLE FIRST");
        return;
    }

    let maxPassengers = n("GET_MAXIMUM_NUMBER_OF_PASSENGERS", car);
    if (maxPassengers === null || typeof maxPassengers !== "number") {
        maxPassengers = 3;
    }

    let guardIndex = 0;

    for (let seat = 0; seat < maxPassengers && guardIndex < bodyguards.length; seat++) {
        const free = n("IS_CAR_PASSENGER_SEAT_FREE", car, seat);
        if (free) {
            const guard = bodyguards[guardIndex++];
            n("SET_CHAR_WILL_DO_DRIVEBYS", guard, true);
            n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", guard, true);
            n("TASK_ENTER_CAR_AS_PASSENGER", guard, car, 12000, seat);
        }
    }

    notify("SQUAD BOARDING: " + guardIndex);
}

function squadTargetRecord(guard) {
    for (const rec of squadTargetRecords) if (rec.guard === guard) return rec;
    const rec = { guard, target: 0, lastTaskAt: 0 };
    squadTargetRecords.push(rec);
    return rec;
}

function clearSquadTargetRecords() {
    squadTargetRecords = [];
    squadForcedTarget = 0;
}

function validKnownSquadHostile(ped) {
    if (!ped || ped === playerPed() || bodyguards.includes(ped) || isWarPolicePed(ped)) return false;
    return !!n("DOES_CHAR_EXIST", ped) && !n("IS_CHAR_DEAD", ped);
}

function knownSquadHostiles() {
    const result = [];
    const seen = {};
    const add = ped => {
        if (!validKnownSquadHostile(ped)) return;
        const key = String(ped);
        if (seen[key]) return;
        seen[key] = true;
        result.push(ped);
    };

    if (squadForcedTarget) add(squadForcedTarget);
    for (const ped of streetWarPeds) add(ped);
    for (const ped of aggroPeds) add(ped);
    return result;
}

function pickSquadCombatTarget(guard) {
    if (!validGuard(guard)) return 0;
    if (squadForcedTarget && validKnownSquadHostile(squadForcedTarget)) return squadForcedTarget;

    const q = n("GET_CHAR_COORDINATES", guard);
    const candidates = knownSquadHostiles();
    if (!q || !candidates.length) return 0;

    const ranked = [];
    for (const ped of candidates) {
        const p = n("GET_CHAR_COORDINATES", ped);
        if (!p) continue;
        const dx = p.x - q.x;
        const dy = p.y - q.y;
        const d2 = dx * dx + dy * dy;
        if (d2 <= 14400.0) ranked.push({ ped, d2 }); // 120m
    }
    ranked.sort((a, b) => a.d2 - b.d2);
    if (!ranked.length) return 0;

    // Spread a squad over a few nearby hostiles instead of ten guards always
    // dumping fire into the exact same actor.
    const guardSlot = Math.max(0, bodyguards.indexOf(guard));
    return ranked[guardSlot % Math.min(ranked.length, 4)].ped;
}

function assignSquadCombatTarget(guard, force = false) {
    if (!validGuard(guard)) return false;
    const rec = squadTargetRecord(guard);
    const now = Date.now();
    const targetGood = rec.target && validKnownSquadHostile(rec.target);

    if (!force && targetGood) {
        const status = n("GET_SCRIPT_TASK_STATUS", guard, 91); // Combat task
        if (status !== 7 && status !== null && now - rec.lastTaskAt < SQUAD_TARGET_HOLD_MS) return true;
    }

    const target = pickSquadCombatTarget(guard);
    if (!target) {
        rec.target = 0;
        // Keep GTA IV's native hated-target behavior as a low-frequency fallback
        // for ordinary mission/world enemies that Dynamic Liberty does not own.
        // Explicit DL targets always take priority when available.
        if (force || now - rec.lastTaskAt >= SQUAD_TARGET_HOLD_MS) {
            rec.lastTaskAt = now;
            n("TASK_COMBAT_HATED_TARGETS_AROUND_CHAR", guard, 100.0);
        }
        return false;
    }

    rec.target = target;
    rec.lastTaskAt = now;
    n("SET_CHAR_WILL_DO_DRIVEBYS", guard, squadDrivebys);
    n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", guard, squadUseCarsCombat);
    n("TASK_COMBAT", guard, target);
    return true;
}

function squadAttackActivePed() {
    if (!activePed || !validKnownSquadHostile(activePed)) {
        notify("SELECT A VALID ACTIVE PED FIRST");
        return;
    }
    squadForcedTarget = activePed;
    squadCombatMode = true;
    forEachGuard(g => assignSquadCombatTarget(g, true));
    notify("SQUAD TARGET: ACTIVE PED");
}

function squadCombatNearby() {
    pruneBodyguards();
    if (!bodyguards.length) {
        notify("NO ACTIVE BODYGUARDS");
        return;
    }

    squadCombatMode = true;
    squadForcedTarget = 0;
    squadTargetRecords = [];

    let assigned = 0;
    for (const guard of bodyguards) if (assignSquadCombatTarget(guard, true)) assigned++;

    notify(assigned > 0 ? "SQUAD COMBAT: ON / TARGETS " + assigned :
        "SQUAD COMBAT: ON / NO KNOWN HOSTILES YET");
}

function squadStandDown() {
    pruneBodyguards();
    squadCombatMode = false;
    clearSquadTargetRecords();

    const me = playerPed();
    let idx = 0;

    for (const guard of bodyguards) {
        n("CLEAR_CHAR_TASKS", guard);
        if (me) {
            const side = ((idx % 3) - 1) * 1.2;
            const back = -1.5 - Math.floor(idx / 3) * 0.7;
            n("TASK_GOTO_CHAR_OFFSET", guard, me, -1, side, back);
        }
        idx++;
    }

    notify("SQUAD COMBAT: OFF");
}

function dismissBodyguards() {
    pruneBodyguards();

    for (const guard of bodyguards) {
        n("REMOVE_CHAR_FROM_GROUP", guard);
        n("CLEAR_CHAR_TASKS", guard);
        n("TASK_WANDER_STANDARD", guard);
        n("MARK_CHAR_AS_NO_LONGER_NEEDED", guard);
    }

    bodyguards = [];
    squadCombatMode = false;
    clearSquadTargetRecords();

    // Never REMOVE_GROUP here: squadGroup is Niko's own player group in v5.1.
    squadGroup = 0;

    notify("TACTICAL SQUAD DISMISSED");
}

function updateBodyguardSquad() {
    const now = Date.now();
    if (now - lastSquadUpdate < 1400) return;
    lastSquadUpdate = now;

    pruneBodyguards();
    squadTargetRecords = squadTargetRecords.filter(r => validGuard(r.guard));
    if (!bodyguards.length) return;

    if (!squadGroupExists()) {
        squadGroup = 0;
        if (ensureSquadGroup()) {
            forEachGuard(g => n("SET_GROUP_MEMBER", squadGroup, g));
        }
    }

    forEachGuard(guard => {
        n("SET_CHAR_WILL_DO_DRIVEBYS", guard, squadDrivebys);
        n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", guard, squadUseCarsCombat);
        if (squadCombatMode) assignSquadCombatTarget(guard, false);
    });
}


// ------------------------------------------------------------
// TACTICAL SQUAD CONTROL
// ------------------------------------------------------------
function setSquadWeapon(id, label) {
    squadWeapon = id;
    forEachGuard(applyGuardLoadout);
    notify("SQUAD WEAPON: " + label);
}

function setSquadAccuracy(value) {
    squadAccuracy = value;
    forEachGuard(g => n("SET_CHAR_ACCURACY", g, value));
    notify("SQUAD ACCURACY: " + value);
}

function setSquadShootRate(value) {
    squadShootRate = value;
    forEachGuard(g => n("SET_CHAR_SHOOT_RATE", g, value));
    notify("SQUAD SHOOT RATE: " + value);
}

function setSquadHealth(value) {
    squadHealth = value;
    forEachGuard(g => {
        n("SET_CHAR_MAX_HEALTH", g, value);
        n("SET_CHAR_HEALTH", g, value);
    });
    notify("SQUAD HEALTH: " + value);
}

function setSquadSpacing(value) {
    squadSpacing = value;
    if (ensureSquadGroup()) n("SET_GROUP_FORMATION_SPACING", squadGroup, value);
    notify("FORMATION SPACING: " + value + "m");
}

function setSquadFormation(value) {
    squadFormation = value;
    if (ensureSquadGroup()) n("SET_GROUP_FORMATION", squadGroup, value);
    notify("FORMATION ID: " + value);
}

function toggleSquadFlag(key, nativeName, label, nativeValueFn = v => v) {
    const next = !globalThis[key];
    globalThis[key] = next;
    forEachGuard(g => n(nativeName, g, nativeValueFn(next)));
    notify(label + ": " + (next ? "ON" : "OFF"));
}

// Explicit toggles avoid depending on reflective global assignment in CLEO's JS host.
function toggleInvincible() {
    squadInvincible = !squadInvincible;
    forEachGuard(g => n("SET_CHAR_INVINCIBLE", g, squadInvincible));
    notify("INVINCIBLE: " + (squadInvincible ? "ON" : "OFF"));
}
function toggleHeadshotImmune() {
    squadHeadshotImmune = !squadHeadshotImmune;
    forEachGuard(g => n("SET_CHAR_SUFFERS_CRITICAL_HITS", g, !squadHeadshotImmune));
    notify("HEADSHOT IMMUNE: " + (squadHeadshotImmune ? "ON" : "OFF"));
}
function toggleUseCover() {
    squadUseCover = !squadUseCover;
    forEachGuard(g => n("SET_CHAR_WILL_USE_COVER", g, squadUseCover));
    notify("USE COVER: " + (squadUseCover ? "ON" : "OFF"));
}
function toggleCoveringFire() {
    squadCoveringFire = !squadCoveringFire;
    forEachGuard(g => n("SET_CHAR_PROVIDE_COVERING_FIRE", g, squadCoveringFire));
    notify("COVERING FIRE: " + (squadCoveringFire ? "ON" : "OFF"));
}
function toggleClearLos() {
    squadClearLosOnly = !squadClearLosOnly;
    forEachGuard(g => n("SET_CHAR_WILL_ONLY_FIRE_WITH_CLEAR_LOS", g, squadClearLosOnly));
    notify("CLEAR LOS ONLY: " + (squadClearLosOnly ? "ON" : "OFF"));
}
function toggleDrivebys() {
    squadDrivebys = !squadDrivebys;
    forEachGuard(g => n("SET_CHAR_WILL_DO_DRIVEBYS", g, squadDrivebys));
    notify("DRIVE-BYS: " + (squadDrivebys ? "ON" : "OFF"));
}
function toggleCarsCombat() {
    squadUseCarsCombat = !squadUseCarsCombat;
    forEachGuard(g => n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", g, squadUseCarsCombat));
    notify("USE CARS IN COMBAT: " + (squadUseCarsCombat ? "ON" : "OFF"));
}
function toggleLeaveCarCombat() {
    squadLeaveCarCombat = !squadLeaveCarCombat;
    forEachGuard(g => n("SET_CHAR_WILL_LEAVE_CAR_IN_COMBAT", g, squadLeaveCarCombat));
    notify("LEAVE CAR IN COMBAT: " + (squadLeaveCarCombat ? "ON" : "OFF"));
}
function toggleMoveInjured() {
    squadMoveWhenInjured = !squadMoveWhenInjured;
    forEachGuard(g => n("SET_CHAR_WILL_MOVE_WHEN_INJURED", g, squadMoveWhenInjured));
    notify("MOVE WHEN INJURED: " + (squadMoveWhenInjured ? "ON" : "OFF"));
}
function toggleSignalKill() {
    squadSignalAfterKill = !squadSignalAfterKill;
    forEachGuard(g => n("SET_CHAR_SIGNAL_AFTER_KILL", g, squadSignalAfterKill));
    notify("SIGNAL AFTER KILL: " + (squadSignalAfterKill ? "ON" : "OFF"));
}
function toggleDropsWeapons() {
    squadDropsWeapons = !squadDropsWeapons;
    forEachGuard(g => n("SET_CHAR_DROPS_WEAPONS_WHEN_DEAD", g, squadDropsWeapons));
    notify("DROP WEAPONS ON DEATH: " + (squadDropsWeapons ? "ON" : "OFF"));
}
function toggleStayCarJacked() {
    squadStayInCarWhenJacked = !squadStayInCarWhenJacked;
    forEachGuard(g => n("SET_CHAR_STAY_IN_CAR_WHEN_JACKED", g, squadStayInCarWhenJacked));
    notify("STAY IN CAR WHEN JACKED: " + (squadStayInCarWhenJacked ? "ON" : "OFF"));
}
function toggleCantDragged() {
    squadCantBeDraggedOut = !squadCantBeDraggedOut;
    forEachGuard(g => n("SET_CHAR_CANT_BE_DRAGGED_OUT", g, squadCantBeDraggedOut));
    notify("CANT BE DRAGGED OUT: " + (squadCantBeDraggedOut ? "ON" : "OFF"));
}
function toggleBikeKnockoff() {
    squadCanBeKnockedOffBike = !squadCanBeKnockedOffBike;
    forEachGuard(g => n("SET_CHAR_CAN_BE_KNOCKED_OFF_BIKE", g, squadCanBeKnockedOffBike));
    notify("CAN BE KNOCKED OFF BIKE: " + (squadCanBeKnockedOffBike ? "ON" : "OFF"));
}
function toggleDrunk() {
    squadDrunk = !squadDrunk;
    forEachGuard(g => n("SET_PED_IS_DRUNK", g, squadDrunk));
    notify("DRUNK AI: " + (squadDrunk ? "ON" : "OFF"));
}
function toggleBlindRage() {
    squadBlindRage = !squadBlindRage;
    forEachGuard(g => n("SET_PED_IS_BLIND_RAGING", g, squadBlindRage));
    notify("BLIND RAGE: " + (squadBlindRage ? "ON" : "OFF"));
}

function squadHoldPosition() {
    pruneBodyguards();
    squadCombatMode = false;
    forEachGuard(g => n("TASK_STAND_STILL", g, 600000));
    notify("SQUAD HOLDING POSITION");
}

function squadGuardPosition() {
    pruneBodyguards();
    squadCombatMode = false;
    // Unknown middle parameters are kept conservative at 0.0.
    forEachGuard(g => n("TASK_GUARD_CURRENT_POSITION", g, 0.0, 0.0, 600000));
    notify("SQUAD GUARDING CURRENT POSITION");
}

function squadRegroup() {
    pruneBodyguards();
    squadCombatMode = false;
    clearSquadTargetRecords();

    if (ensureSquadGroup()) {
        forEachGuard(g => {
            n("CLEAR_CHAR_TASKS", g);
            n("SET_GROUP_MEMBER", squadGroup, g);
        });
        n("SET_GROUP_FORMATION", squadGroup, squadFormation);
        n("SET_GROUP_FORMATION_SPACING", squadGroup, squadSpacing);
        n("SET_GROUP_SEPARATION_RANGE", squadGroup, 80.0);
    } else {
        const me = playerPed();
        let idx = 0;
        forEachGuard(g => {
            n("CLEAR_CHAR_TASKS", g);
            const side = ((idx % 3) - 1) * 1.2;
            const back = -1.5 - Math.floor(idx / 3) * 0.7;
            n("TASK_GOTO_CHAR_OFFSET", g, me, -1, side, back);
            idx++;
        });
    }
    notify("SQUAD REGROUP");
}

function reapplySquadProfile() {
    forEachGuard(applyGuardBehavior);
    notify("SQUAD PROFILE REAPPLIED");
}

function presetRifleTeam() {
    squadWeapon = 15;
    squadAccuracy = 72;
    squadShootRate = 100;
    squadUseCover = true;
    squadCoveringFire = true;
    squadClearLosOnly = false;
    squadInvincible = false;
    squadHeadshotImmune = false;
    forEachGuard(applyGuardBehavior);
    notify("ROLE PRESET: RIFLE TEAM");
}

function presetTankTeam() {
    squadWeapon = 15;
    squadHealth = 1200;
    squadAccuracy = 65;
    squadInvincible = false;
    squadHeadshotImmune = true;
    squadUseCover = true;
    forEachGuard(applyGuardBehavior);
    notify("ROLE PRESET: TANK TEAM");
}

function presetDrivebyCrew() {
    squadWeapon = 13;
    squadAccuracy = 68;
    squadDrivebys = true;
    squadUseCarsCombat = true;
    squadLeaveCarCombat = false;
    squadStayInCarWhenJacked = true;
    squadCantBeDraggedOut = true;
    forEachGuard(applyGuardBehavior);
    notify("ROLE PRESET: DRIVE-BY CREW");
}

function presetChaosCrew() {
    squadWeapon = 20;
    squadAccuracy = 45;
    squadShootRate = 130;
    squadUseCover = false;
    squadBlindRage = true;
    squadInvincible = false;
    forEachGuard(applyGuardBehavior);
    notify("ROLE PRESET: CHAOS CREW");
}


// ------------------------------------------------------------
// AGGRESSIVE PEDS
// This deliberately changes individual nearby ambient peds rather than
// permanently rewriting the whole relationship table.
// ------------------------------------------------------------
function isTrackedAggro(ped) {
    for (let i = 0; i < aggroPeds.length; i++) {
        if (aggroPeds[i] === ped) return true;
    }
    return false;
}

function clearAggroPeds() {
    for (let i = 0; i < aggroPeds.length; i++) {
        const ped = aggroPeds[i];
        if (ped && n("DOES_CHAR_EXIST", ped)) {
            n("CLEAR_CHAR_TASKS", ped);
            n("TASK_WANDER_STANDARD", ped);
            n("MARK_CHAR_AS_NO_LONGER_NEEDED", ped);
        }
    }
    aggroPeds = [];
}

function setAggressiveMode(mode) {
    aggressiveMode = mode;
    if (mode === 0) clearAggroPeds();
    notify(
        mode === 0 ? "AGGRESSIVE PEDS: OFF" :
        mode === 1 ? "AGGRESSIVE PEDS: FISTS" :
                     "AGGRESSIVE PEDS: ARMED"
    );
}

function effectivePlayerIgnore() {
    // Global player-ignore is now owned ONLY by the explicit WORLD/PLAYER toggle.
    // City War neutrality is local event policy and must never suppress unrelated
    // Dynamic Liberty systems such as Aggressive Peds or WOW modes.
    return everyoneIgnorePlayer;
}

function applyPlayerIgnorePolicy() {
    const p = playerId();
    if (p === null) return;

    const ignore = effectivePlayerIgnore();
    n("SET_EVERYONE_IGNORE_PLAYER", p, ignore);
    n("SET_PLAYER_CAN_BE_HASSLED_BY_GANGS", p, !ignore);

    // Access Mode and Police Disabled independently require police ignore.
    n("SET_POLICE_IGNORE_PLAYER", p, ignore || accessMode || policeDisabled);
}

function pulseAggressivePeds() {
    if (!aggressiveMode || effectivePlayerIgnore()) return;

    const me = playerPed();
    const pos = playerPos();
    if (!me || !pos) return;

    const ped = n(
        "GET_RANDOM_CHAR_IN_AREA_OFFSET_NO_SAVE",
        pos.x - 35.0, pos.y - 35.0, pos.z - 8.0,
        70.0, 70.0, 16.0
    );

    if (!ped || ped === me || bodyguards.includes(ped) || isWarPolicePed(ped)) return;
    if (!n("DOES_CHAR_EXIST", ped) || n("IS_CHAR_DEAD", ped)) return;
    if (isTrackedAggro(ped)) return;

    n("SET_CHAR_AS_MISSION_CHAR", ped);

    if (aggressiveMode === 2) {
        const weapons = [7, 10, 12, 14]; // pistol, shotgun, Micro Uzi, AK47
        const w = weapons[Math.floor(Math.random() * weapons.length)];
        n("GIVE_WEAPON_TO_CHAR", ped, w, 250, true);
        n("SET_CHAR_ACCURACY", ped, 35 + Math.floor(Math.random() * 35));
    }

    n("TASK_COMBAT", ped, me);
    aggroPeds.push(ped);

    // Keep the script from owning too many ambient peds.
    if (aggroPeds.length > 8) {
        const oldPed = aggroPeds.shift();
        if (oldPed && n("DOES_CHAR_EXIST", oldPed)) {
            n("MARK_CHAR_AS_NO_LONGER_NEEDED", oldPed);
        }
    }
}

// ------------------------------------------------------------
// CITY WAR 2.0 / MANAGED ACTOR ENGINE
// v6.1+ no longer drafts random ambient pedestrians into the war. Fighters are
// purpose-created, owned by Dynamic Liberty, split into two teams, and given
// persistent enemy-team combat targets. Ambient civilians remain civilians.
// ------------------------------------------------------------
function managedActorIndex(ped) {
    for (let i = 0; i < managedActors.length; i++) {
        if (managedActors[i].ped === ped) return i;
    }
    return -1;
}

function managedActorRecord(ped) {
    const i = managedActorIndex(ped);
    return i >= 0 ? managedActors[i] : null;
}

function registerManagedActor(ped, owner, team = -1, role = "ACTOR") {
    if (!ped || managedActorIndex(ped) >= 0) return managedActorRecord(ped);
    const rec = {
        ped,
        owner,
        team,
        role,
        state: "ACTIVE",
        createdAt: Date.now(),
        target: 0,
        lastTaskAt: 0,
        vehicle: 0,
        releaseAfter: 0,
        handoffReason: ""
    };
    managedActors.push(rec);
    return rec;
}

function unregisterManagedActor(ped) {
    managedActors = managedActors.filter(a => a.ped !== ped);
}

function charOnScreenSafe(ped) {
    return !!(ped && n("DOES_CHAR_EXIST", ped) && n("IS_CHAR_ON_SCREEN", ped));
}

function carOnScreenSafe(car) {
    return !!(car && n("DOES_VEHICLE_EXIST", car) && n("IS_CAR_ON_SCREEN", car));
}

function managedRecordOnScreen(rec) {
    if (!rec) return false;
    return charOnScreenSafe(rec.ped) || carOnScreenSafe(rec.vehicle);
}

function releaseTrackedManagedVehicle(rec) {
    if (!rec || !rec.vehicle) return;
    const car = rec.vehicle;
    rec.vehicle = 0;
    if (n("DOES_VEHICLE_EXIST", car)) {
        n("MARK_CAR_AS_NO_LONGER_NEEDED", car);
    }
}

function releaseManagedActor(ped, keepCurrentTask = false) {
    if (!ped) return;

    // Save the record before unregistering so tracked handoff vehicles can also
    // be returned to GTA IV ownership.
    const rec = managedActorRecord(ped);
    unregisterManagedActor(ped);

    releaseTrackedManagedVehicle(rec);

    if (!n("DOES_CHAR_EXIST", ped)) return;

    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, false);
    if (!keepCurrentTask && !n("IS_CHAR_DEAD", ped)) {
        n("CLEAR_CHAR_TASKS", ped);
        n("TASK_WANDER_STANDARD", ped);
    }
    n("MARK_CHAR_AS_NO_LONGER_NEEDED", ped);
}

function managedActorDistanceFromPlayer(ped) {
    const ppos = playerPos();
    if (!ppos || !ped || !n("DOES_CHAR_EXIST", ped)) return -1.0;
    const q = n("GET_CHAR_COORDINATES", ped);
    if (!q || typeof q.x !== "number") return -1.0;
    const dx = q.x - ppos.x;
    const dy = q.y - ppos.y;
    return Math.sqrt(dx * dx + dy * dy);
}

function beginManagedActorEscape(ped, vehicle = 0, reason = "ESCAPE") {
    const rec = managedActorRecord(ped);
    if (!rec || !n("DOES_CHAR_EXIST", ped)) return false;

    rec.state = "ESCAPING";
    rec.role = "ESCAPER";
    rec.target = 0;
    rec.lastTaskAt = Date.now();
    rec.releaseAfter = rec.lastTaskAt + ESCAPER_HANDOFF_MS;
    rec.handoffReason = reason;
    rec.vehicle = vehicle || 0;

    // Escapers keep their current drive/task, but stop being insulated from
    // ordinary world events. They remain mission-owned only until handoff.
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, false);

    if (rec.vehicle && n("DOES_VEHICLE_EXIST", rec.vehicle)) {
        // Hold the getaway car through the short handoff window too. It is
        // explicitly released together with the actor later.
        n("SET_CAR_AS_MISSION_CAR", rec.vehicle);
    }
    return true;
}

function pulseManagedActors() {
    if (!managedActors.length) return;

    const now = Date.now();
    const snapshot = managedActors.slice();

    for (const rec of snapshot) {
        const ped = rec.ped;

        if (!ped || !n("DOES_CHAR_EXIST", ped)) {
            // If only the tracked getaway car remains, do not relinquish it while
            // the player can literally see it. This prevents visible pop-outs.
            if (carOnScreenSafe(rec.vehicle)) continue;
            releaseTrackedManagedVehicle(rec);
            unregisterManagedActor(ped);
            continue;
        }

        if (n("IS_CHAR_DEAD", ped)) {
            if (managedRecordOnScreen(rec)) continue;
            releaseManagedActor(ped, true);
            continue;
        }

        if (rec.state !== "ESCAPING") continue;

        const distance = managedActorDistanceFromPlayer(ped);
        const timedOut = rec.releaseAfter > 0 && now >= rec.releaseAfter;
        const farEnough = distance >= ESCAPER_RELEASE_DISTANCE;

        // Timeout/distance makes an escaper ELIGIBLE for handoff. Visibility is
        // the final gate: background cleanup must never happen on camera.
        if ((timedOut || farEnough) && !managedRecordOnScreen(rec)) {
            releaseManagedActor(ped, true);
        }
    }
}

function managedActorStatus() {
    const escaping = managedActors.filter(a => a.state === "ESCAPING").length;
    notify(
        "ACTOR ENGINE" +
        "\nMANAGED: " + managedActors.length + "/" + MAX_MANAGED_ACTORS +
        "\nESCAPING: " + escaping +
        "\nWAR SPAWNS: " + cityWarTotalSpawns + "/" + CITY_WAR_TOTAL_SPAWN_BUDGET
    );
}

function warPedValid(ped) {
    const me = playerPed();
    return ped && ped !== me && !bodyguards.includes(ped) &&
        !!n("DOES_CHAR_EXIST", ped) && !n("IS_CHAR_DEAD", ped);
}

function warTeamForPed(ped) {
    if (streetWarTeamA.includes(ped)) return 0;
    if (streetWarTeamB.includes(ped)) return 1;
    const rec = managedActorRecord(ped);
    return rec && rec.owner === "CITY_WAR" ? rec.team : -1;
}

function warTeamArray(team) {
    return team === 0 ? streetWarTeamA : streetWarTeamB;
}

function enemyWarTeamArray(team) {
    return team === 0 ? streetWarTeamB : streetWarTeamA;
}

function warDesiredTeamSize(team) {
    if (streetWarLimit <= 0) return 0;
    return team === 0 ? Math.ceil(streetWarLimit / 2) : Math.floor(streetWarLimit / 2);
}

function livingWarFighters() {
    return streetWarPeds.filter(p => warPedValid(p));
}

function livingWarFighterCount() {
    return livingWarFighters().length;
}

function warWeaponPoolForTeam(team) {
    if (streetWarWeaponMode === 0) return WAR_WEAPONS.smg;
    if (streetWarWeaponMode === 1) return WAR_WEAPONS.ar;
    // Mixed mode gives each side a recognizable bias while still using only
    // the already-proven GTA IV weapon IDs from previous Dynamic Liberty builds.
    return team === 0 ? WAR_WEAPONS.smg : WAR_WEAPONS.ar;
}

function equipWarPed(ped) {
    if (!warPedValid(ped)) return;
    const team = warTeamForPed(ped);
    const pool = warWeaponPoolForTeam(team < 0 ? 0 : team);
    const weapon = pool[Math.floor(Math.random() * pool.length)];

    n("REMOVE_ALL_CHAR_WEAPONS", ped);
    n("GIVE_WEAPON_TO_CHAR", ped, weapon, 900, true);
    n("SET_CURRENT_CHAR_WEAPON", ped, weapon, true);
    n("SET_CHAR_ACCURACY", ped, streetWarAccuracy);
    n("SET_CHAR_SHOOT_RATE", ped, streetWarShootRate);
}

function applyWarFighterProfile(ped) {
    if (!warPedValid(ped)) return;

    n("SET_CHAR_MAX_HEALTH", ped, streetWarHealth);
    n("SET_CHAR_HEALTH", ped, streetWarHealth);
    if (streetWarArmour > 0) n("ADD_ARMOUR_TO_CHAR", ped, streetWarArmour);

    n("SET_CHAR_WILL_USE_COVER", ped, streetWarUseCover);
    n("SET_CHAR_PROVIDE_COVERING_FIRE", ped, true);
    n("SET_CHAR_WILL_ONLY_FIRE_WITH_CLEAR_LOS", ped, false);
    n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", ped, true);
    n("SET_CHAR_WILL_LEAVE_CAR_IN_COMBAT", ped, false);
    n("SET_CHAR_WILL_MOVE_WHEN_INJURED", ped, true);
    n("SET_CHAR_WILL_DO_DRIVEBYS", ped, true);
    n("SET_CHAR_DROPS_WEAPONS_WHEN_DEAD", ped, true);

    // This is the key City War 2.0 difference: fighters do not abandon their
    // assigned combat task because a nearby civilian screams, a car crashes,
    // or another temporary world event fires.
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, true);
    equipWarPed(ped);
}

function recordWarSpawnFailure(reason) {
    streetWarSpawnFailures++;
    lastStreetWarSpawnFailure = reason || "UNKNOWN";
    log("CITY WAR SPAWN FAIL:", lastStreetWarSpawnFailure,
        "state=", streetWarState,
        "A=", streetWarTeamA.length,
        "B=", streetWarTeamB.length,
        "managed=", managedActors.length);
}

function hiddenSpawnBasis(distance, side = 0.0) {
    const pos = playerPos();
    const me = playerPed();
    if (!pos || !me) return null;

    let dirX = 0.0;
    let dirY = 0.0;
    const cam = n("GET_GAME_CAM") || 0;
    const camPos = cam ? n("GET_CAM_POS", cam) : null;

    if (camPos && typeof camPos.x === "number") {
        const dx = camPos.x - pos.x;
        const dy = camPos.y - pos.y;
        const mag = Math.sqrt(dx * dx + dy * dy);
        if (mag > 0.5) {
            // From player toward camera = normally behind the current view.
            dirX = dx / mag;
            dirY = dy / mag;
        }
    }

    if (dirX === 0.0 && dirY === 0.0) {
        const heading = n("GET_CHAR_HEADING", me) || 0.0;
        const r = heading * Math.PI / 180.0;
        dirX = -Math.sin(r);
        dirY = -Math.cos(r);
    }

    const sideX = -dirY;
    const sideY = dirX;
    return {
        x: pos.x + dirX * distance + sideX * side,
        y: pos.y + dirY * distance + sideY * side,
        z: pos.z + 0.20
    };
}

function warSpawnPoint(team) {
    const distance = 28.0 + Math.random() * 10.0;
    const sideBase = 8.0 + Math.random() * 5.0;
    const raw = hiddenSpawnBasis(distance, team === 0 ? -sideBase : sideBase);
    if (!raw) return null;

    // Fail-open safe-position improvement. If CLEO/GTA IV cannot return a sane
    // safe nav position, the proven raw placement remains the fallback.
    const safe = n("GET_SAFE_POSITION_FOR_CHAR", raw.x, raw.y, raw.z, true);
    if (safe && typeof safe.pSafeX === "number") {
        const dx = safe.pSafeX - raw.x;
        const dy = safe.pSafeY - raw.y;
        const dz = safe.pSafeZ - raw.z;
        if ((dx * dx + dy * dy) <= 400.0 && Math.abs(dz) <= 6.0) {
            return { x: safe.pSafeX, y: safe.pSafeY, z: safe.pSafeZ + 0.10 };
        }
    }
    return raw;
}

function createCuratedWarPed(team, p) {
    const pool = WAR_TEAM_MODELS[team] || [];
    if (!pool.length) return 0;
    const start = streetWarSpawnSerial % pool.length;

    for (let i = 0; i < pool.length; i++) {
        const name = pool[(start + i) % pool.length];
        const model = hash(name);
        if (!model || !n("IS_MODEL_IN_CDIMAGE", model)) continue;
        if (!loadModel(model, 2200)) continue;

        const ped = n("CREATE_CHAR", 1, model, p.x, p.y, p.z) || 0;
        releaseModel(model);
        if (ped && n("DOES_CHAR_EXIST", ped)) return ped;
    }
    return 0;
}

function spawnManagedWarPed(team) {
    if (streetWarLimit <= 0) return 0;
    if (managedActors.length >= MAX_MANAGED_ACTORS) {
        recordWarSpawnFailure("MANAGED ACTOR CAP");
        return 0;
    }
    if (cityWarTotalSpawns >= CITY_WAR_TOTAL_SPAWN_BUDGET) {
        if (!cityWarBudgetExhaustedNotified) {
            cityWarBudgetExhaustedNotified = true;
            notify("CITY WAR SPAWN BUDGET EXHAUSTED\nREINFORCEMENTS STOPPED");
        }
        streetWarInitialFill = false;
        return 0;
    }

    const p = warSpawnPoint(team);
    if (!p) {
        recordWarSpawnFailure("PLAYER/SPAWN POINT NOT READY");
        return 0;
    }

    // Serious City War actors use curated faction models. Random civilians are
    // no longer promoted into riflemen, which removes the "hobo with an AR" look.
    const ped = createCuratedWarPed(team, p);
    if (!ped || !n("DOES_CHAR_EXIST", ped)) {
        recordWarSpawnFailure(team === 0 ? "TEAM A MODEL/CREATE FAILED" : "TEAM B MODEL/CREATE FAILED");
        return 0;
    }

    n("SET_CHAR_AS_MISSION_CHAR", ped);
    n("SET_CHAR_RANDOM_COMPONENT_VARIATION", ped);

    const rec = registerManagedActor(ped, "CITY_WAR", team, "FIGHTER");
    if (!rec) {
        recordWarSpawnFailure("MANAGED ACTOR REGISTER FAILED");
        n("MARK_CHAR_AS_NO_LONGER_NEEDED", ped);
        return 0;
    }

    streetWarPeds.push(ped);
    warTeamArray(team).push(ped);
    streetWarSpawnSerial++;
    cityWarTotalSpawns++;

    applyWarFighterProfile(ped);
    return ped;
}

function randomWarTargetForTeam(team, excludePed = 0) {
    const candidates = enemyWarTeamArray(team).filter(p =>
        p !== excludePed && warPedValid(p)
    );
    if (!candidates.length) return 0;
    return candidates[Math.floor(Math.random() * candidates.length)];
}

function randomWarTarget(excludePed) {
    const team = warTeamForPed(excludePed);
    if (team < 0) return 0;
    return randomWarTargetForTeam(team, excludePed);
}

function isWarCarjackAssigned(ped) {
    for (const a of carjackAssignments) {
        if (a.ped === ped) return true;
    }
    return false;
}

function assignWarTarget(ped, force = false) {
    if (!warPedValid(ped) || n("IS_CHAR_IN_ANY_CAR", ped)) return false;
    if (!force && isWarCarjackAssigned(ped)) return false;

    const rec = managedActorRecord(ped);
    const team = warTeamForPed(ped);
    if (!rec || team < 0) return false;

    const now = Date.now();
    const currentTargetGood = rec.target && warPedValid(rec.target) &&
        warTeamForPed(rec.target) !== team;

    if (!force && currentTargetGood) {
        const status = n("GET_SCRIPT_TASK_STATUS", ped, 91); // TaskId Combat
        // Let GTA IV's combat AI work. Only refresh a still-valid target after
        // several seconds so cover / movement behavior is not hammered every tick.
        if (status !== 7 && status !== null && now - rec.lastTaskAt < 6000) return true;
    }

    const target = randomWarTargetForTeam(team, ped);
    if (!target) {
        rec.target = 0;
        return false;
    }

    rec.target = target;
    rec.lastTaskAt = now;
    n("TASK_COMBAT", ped, target);
    return true;
}

function removeWarCombatReferences(ped) {
    streetWarPeds = streetWarPeds.filter(p => p !== ped);
    streetWarTeamA = streetWarTeamA.filter(p => p !== ped);
    streetWarTeamB = streetWarTeamB.filter(p => p !== ped);
}

function removeWarPedReferences(ped) {
    removeWarCombatReferences(ped);
    carjackAssignments = carjackAssignments.filter(a => a.ped !== ped);
}

function releaseWarPed(ped, keepCurrentTask = false) {
    removeWarPedReferences(ped);
    releaseManagedActor(ped, keepCurrentTask);
}

function handoffWarCarjackerToEscape(ped, car) {
    // Leave the infantry/team pool immediately so reinforcements can replace
    // this fighter, but keep the actor managed during the visible getaway.
    removeWarCombatReferences(ped);
    if (!beginManagedActorEscape(ped, car, "WAR_CARJACK")) {
        releaseManagedActor(ped, true);
        return false;
    }
    return true;
}

function pruneStreetWarPeds() {
    const ppos = playerPos();
    const snapshot = streetWarPeds.slice();

    for (const ped of snapshot) {
        if (!ped || !n("DOES_CHAR_EXIST", ped)) {
            releaseWarPed(ped, true);
            continue;
        }

        if (n("IS_CHAR_DEAD", ped)) {
            // Dead fighters are still part of the visible scene. Never hand them
            // back to GTA IV while the player is looking at the body.
            const rec = managedActorRecord(ped);
            if (charOnScreenSafe(ped) || managedRecordOnScreen(rec)) continue;
            releaseWarPed(ped, true);
            continue;
        }

        if (!warPedValid(ped)) {
            releaseWarPed(ped, true);
            continue;
        }

        if (ppos) {
            const q = n("GET_CHAR_COORDINATES", ped);
            if (q && typeof q.x === "number") {
                const dx = q.x - ppos.x;
                const dy = q.y - ppos.y;
                if ((dx * dx + dy * dy) > 22500.0 && !charOnScreenSafe(ped)) { // >150m + off-screen
                    releaseWarPed(ped, false);
                }
            }
        }
    }
}

function clearStreetWar(silent = false) {
    clearWarEmsResponse(true);
    clearWarPoliceResponse(true);
    const snapshot = streetWarPeds.slice();
    for (const ped of snapshot) releaseWarPed(ped, false);

    // Defensive cleanup in case an actor was registered but lost from a team array.
    const leftovers = managedActors.filter(a => a.owner === "CITY_WAR").map(a => a.ped);
    for (const ped of leftovers) releaseManagedActor(ped, false);

    streetWarPeds = [];
    streetWarTeamA = [];
    streetWarTeamB = [];
    carjackAssignments = [];
    streetWarLimit = 0;
    streetWarInitialFill = false;
    streetWarState = "OFF";
    streetWarStartPulses = 0;
    streetWarSpawnFailures = 0;
    lastStreetWarSpawnFailure = "NONE";
    streetWarEndRequested = false;
    streetWarEndRequestedAt = 0;
    streetWarAftermathStartedAt = 0;
    cityWarTotalSpawns = 0;
    cityWarBudgetExhaustedNotified = false;
    warPoliceNextDispatchAt = 0;
    warEmsDispatchAt = 0;
    warEmsSpawnFailures = 0;
    warEmsLastFailure = "NONE";
    warEmsTreatments = 0;
    warEmsTreatedPolice = [];
    warEmsSceneComplete = false;
    applyPlayerIgnorePolicy();
    if (!silent) notify("CITY WAR 2.0: OFF");
}

function requestStreetWarEnd() {
    if (streetWarState === "OFF") {
        notify("NO ACTIVE CITY WAR");
        return;
    }
    if (streetWarState === "STARTING") {
        clearStreetWar();
        return;
    }
    if (streetWarState === "AFTERMATH") {
        notify("CITY WAR ALREADY IN AFTERMATH");
        return;
    }

    streetWarEndRequested = true;
    streetWarEndRequestedAt = Date.now();
    streetWarReinforcements = false;
    streetWarInitialFill = false;
    streetWarCarjackers = false;
    carjackAssignments = [];
    streetWarState = "ENDING";
    notify("CITY WAR ENDING" +
        "\nREINFORCEMENTS STOPPED" +
        "\nEMS WAITS FOR THREAT CLEAR");
}

function beginStreetWarAftermath(reason = "THREAT CLEARED") {
    if (streetWarState === "AFTERMATH" || streetWarState === "OFF") return;

    streetWarState = "AFTERMATH";
    streetWarEndRequested = false;
    streetWarInitialFill = false;
    streetWarCarjackers = false;
    carjackAssignments = [];
    streetWarAftermathStartedAt = Date.now();
    warPoliceNextDispatchAt = 0;

    // Surviving Dynamic Liberty cops secure the scene instead of continuing to
    // receive combat tasks after the last gang fighter is gone.
    for (const unit of warPoliceUnits) {
        unit.targets = {};
        for (const cop of unit.cops || []) {
            if (!cop || !n("DOES_CHAR_EXIST", cop) || n("IS_CHAR_DEAD", cop)) continue;
            n("CLEAR_CHAR_TASKS", cop);
            n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", cop, false);
            n("TASK_STAND_STILL", cop, 30000);
        }
    }

    const casualties = warPoliceCasualties();
    warEmsSceneComplete = casualties.length === 0;
    warEmsDispatchAt = (warEmsEnabled && !policeDisabled && casualties.length > 0) ?
        Date.now() + WAR_EMS_INITIAL_DELAY_MS : 0;

    notify("CITY WAR AFTERMATH" +
        "\n" + reason +
        "\nPOLICE CASUALTIES: " + casualties.length +
        (casualties.length && warEmsEnabled ? "\nEMS DISPATCH PENDING" : ""));
}

function finalizeStreetWarAftermathIfClear() {
    if (streetWarState !== "AFTERMATH") return;
    const now = Date.now();
    if (now - streetWarAftermathStartedAt < STREET_WAR_AFTERMATH_MIN_MS) return;
    if (!warEmsSceneComplete || warEmsUnit) return;

    pruneStreetWarPeds();
    clearWarPoliceResponse(false);

    // Visible bodies/units deliberately keep the aftermath alive. Once the
    // player looks away and GTA IV can safely own the scene again, finish.
    if (streetWarPeds.length || warPoliceUnits.length) return;

    streetWarTeamA = [];
    streetWarTeamB = [];
    streetWarLimit = 0;
    streetWarInitialFill = false;
    streetWarState = "OFF";
    streetWarStartPulses = 0;
    streetWarEndRequested = false;
    streetWarEndRequestedAt = 0;
    streetWarAftermathStartedAt = 0;
    carjackAssignments = [];
    cityWarTotalSpawns = 0;
    cityWarBudgetExhaustedNotified = false;
    warPoliceNextDispatchAt = 0;
    applyPlayerIgnorePolicy();
    notify("CITY WAR AFTERMATH COMPLETE");
}

function setStreetWar(limit, label) {
    if (limit <= 0) {
        clearStreetWar();
        return;
    }

    // Restart cleanly when changing battle size.
    if (streetWarLimit > 0 || streetWarState !== "OFF") clearStreetWar(true);

    // PREPARE: configure the requested event, but do not let City War seize
    // unrelated global ped behavior. The first pulse must prove both teams can spawn.
    streetWarLimit = limit;
    streetWarInitialFill = true;
    streetWarState = "STARTING";
    streetWarStartPulses = 0;
    streetWarSpawnFailures = 0;
    lastStreetWarSpawnFailure = "NONE";
    cityWarTotalSpawns = 0;
    cityWarBudgetExhaustedNotified = false;
    warPoliceNextDispatchAt = 0;
    warPoliceSpawnFailures = 0;
    warPoliceLastFailure = "NONE";
    streetWarEndRequested = false;
    streetWarEndRequestedAt = 0;
    streetWarAftermathStartedAt = 0;
    warEmsDispatchAt = 0;
    warEmsSpawnFailures = 0;
    warEmsLastFailure = "NONE";
    warEmsTreatments = 0;
    warEmsTreatedPolice = [];
    warEmsSceneComplete = false;

    applyPlayerIgnorePolicy();
    notify("CITY WAR 2.0 STARTING: " + label +
        " (" + warDesiredTeamSize(0) + "v" + warDesiredTeamSize(1) + ")");
}

function setWarWeaponMode(mode, label) {
    streetWarWeaponMode = mode;
    for (const ped of streetWarPeds) equipWarPed(ped);
    notify("WAR WEAPONS: " + label);
}

function setWarAccuracy(value) {
    streetWarAccuracy = value;
    for (const ped of streetWarPeds) if (warPedValid(ped)) n("SET_CHAR_ACCURACY", ped, value);
    notify("WAR ACCURACY: " + value);
}

function setWarShootRate(value) {
    streetWarShootRate = value;
    for (const ped of streetWarPeds) if (warPedValid(ped)) n("SET_CHAR_SHOOT_RATE", ped, value);
    notify("WAR FIRE RATE: " + value);
}

function setWarHealth(value) {
    streetWarHealth = value;
    for (const ped of streetWarPeds) {
        if (!warPedValid(ped)) continue;
        n("SET_CHAR_MAX_HEALTH", ped, value);
        n("SET_CHAR_HEALTH", ped, value);
    }
    notify("WAR FIGHTER HEALTH: " + value);
}

function toggleWarReinforcements() {
    streetWarReinforcements = !streetWarReinforcements;
    notify("WAR REINFORCEMENTS: " + (streetWarReinforcements ? "ON" : "OFF"));
}

function warTeamStatus() {
    pruneStreetWarPeds();
    const escaping = managedActors.filter(a => a.owner === "CITY_WAR" && a.state === "ESCAPING").length;
    notify("CITY WAR 2.0" +
        "\nSTATE: " + streetWarState +
        "\nTEAM A: " + streetWarTeamA.length + "/" + warDesiredTeamSize(0) +
        "\nTEAM B: " + streetWarTeamB.length + "/" + warDesiredTeamSize(1) +
        "\nESCAPING: " + escaping +
        "\nSPAWNS: " + cityWarTotalSpawns + "/" + CITY_WAR_TOTAL_SPAWN_BUDGET +
        "\nFAILS: " + streetWarSpawnFailures +
        "\nLAST: " + lastStreetWarSpawnFailure);
}

function toggleWarCover() {
    streetWarUseCover = !streetWarUseCover;
    for (const ped of streetWarPeds) if (warPedValid(ped)) n("SET_CHAR_WILL_USE_COVER", ped, streetWarUseCover);
    notify("WAR USE COVER: " + (streetWarUseCover ? "ON" : "OFF"));
}

function toggleWarPlayerNeutral() {
    streetWarPlayerNeutral = !streetWarPlayerNeutral;

    // Important: this toggle no longer calls SET_EVERYONE_IGNORE_PLAYER and no
    // longer shuts down Aggressive Peds. City War's explicit enemy-team targeting
    // remains isolated from the rest of the world simulation.
    notify("PLAYER NEUTRAL IN WAR: " + (streetWarPlayerNeutral ? "ON" : "OFF"));
}

function toggleWarCarjackers() {
    streetWarCarjackers = !streetWarCarjackers;
    if (!streetWarCarjackers) carjackAssignments = [];
    notify("WAR CARJACKERS: " + (streetWarCarjackers ? "ON" : "OFF"));
}

function setWarDriveSpeed(value) {
    streetWarDriveSpeed = value;
    notify("WAR DRIVE SPEED: " + value);
}

function toggleWarFireAmmo() {
    streetWarFireAmmo = !streetWarFireAmmo;
    notify("WAR FIRE AMMO: " + (streetWarFireAmmo ? "ON" : "OFF"));
}

function toggleIncendiaryHits() {
    incendiaryHits = !incendiaryHits;
    if (!incendiaryHits) {
        incendiaryPeds = [];
        incendiaryCars = [];
    }
    notify("PLAYER INCENDIARY HITS: " + (incendiaryHits ? "ON" : "OFF"));
}

function setIncendiaryChance(value) {
    incendiaryChance = value;
    notify("INCENDIARY CHANCE: " + value + "%");
}

function retreatRemainingWarFighters() {
    const me = playerPed();
    const remaining = livingWarFighters().slice();
    for (const ped of remaining) {
        if (!ped || !n("DOES_CHAR_EXIST", ped) || n("IS_CHAR_DEAD", ped)) continue;
        n("CLEAR_CHAR_TASKS", ped);
        n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", ped, false);
        if (me) n("TASK_SMART_FLEE_CHAR", ped, me, 180.0, 60000);
        removeWarPedReferences(ped);
        beginManagedActorEscape(ped, 0, "WAR END RETREAT");
    }
    return remaining.length;
}

function pulseStreetWar() {
    if (streetWarLimit <= 0 || streetWarState === "OFF") return;

    pruneStreetWarPeds();

    if (streetWarState === "AFTERMATH") {
        finalizeStreetWarAftermathIfClear();
        return;
    }

    const desiredA = warDesiredTeamSize(0);
    const desiredB = warDesiredTeamSize(1);
    const canFill = streetWarState === "STARTING" ||
        (streetWarState === "ACTIVE" && (streetWarInitialFill || streetWarReinforcements));

    // Add at most one fighter per team per pulse. ENDING never replenishes.
    if (canFill && streetWarTeamA.filter(p => warPedValid(p)).length < desiredA) spawnManagedWarPed(0);
    if (canFill && streetWarTeamB.filter(p => warPedValid(p)).length < desiredB) spawnManagedWarPed(1);

    // STARTUP TRANSACTION.
    if (streetWarState === "STARTING") {
        streetWarStartPulses++;

        if (streetWarTeamA.some(p => warPedValid(p)) && streetWarTeamB.some(p => warPedValid(p))) {
            streetWarState = "ACTIVE";
            streetWarStartPulses = 0;
            warPoliceNextDispatchAt = warPoliceResponseEnabled && !policeDisabled ?
                Date.now() + WAR_POLICE_INITIAL_DELAY_MS : 0;
            notify("CITY WAR 2.0 ACTIVE");
        } else if (streetWarStartPulses >= STREET_WAR_START_MAX_PULSES) {
            const failReason = lastStreetWarSpawnFailure;
            const failCount = streetWarSpawnFailures;
            clearStreetWar(true);
            notify("CITY WAR START FAILED / ROLLED BACK" +
                "\nFAILS: " + failCount +
                "\nLAST: " + failReason);
            return;
        }
    }

    if (streetWarTeamA.filter(p => warPedValid(p)).length >= desiredA &&
        streetWarTeamB.filter(p => warPedValid(p)).length >= desiredB) {
        streetWarInitialFill = false;
    }

    const now = Date.now();
    if ((streetWarState === "ACTIVE" || streetWarState === "ENDING") &&
        now - lastStreetWarRetarget >= 1200) {
        lastStreetWarRetarget = now;
        for (const ped of streetWarPeds) {
            if (!warPedValid(ped) || n("IS_CHAR_IN_ANY_CAR", ped)) continue;
            assignWarTarget(ped, false);
        }
    }

    let living = livingWarFighterCount();
    if (streetWarState === "ENDING" && living > 0 && streetWarEndRequestedAt > 0 &&
        now - streetWarEndRequestedAt >= STREET_WAR_ENDING_RETREAT_MS) {
        const retreated = retreatRemainingWarFighters();
        living = livingWarFighterCount();
        if (retreated > 0) notify("REMAINING GANGS RETREATING");
    }

    if (streetWarState === "ENDING" && living === 0) {
        beginStreetWarAftermath("THREAT CLEARED");
        return;
    }

    // A finite battle naturally reaches aftermath if replenishment is disabled,
    // or if the lifetime budget is exhausted and every remaining fighter falls.
    if (streetWarState === "ACTIVE" && living === 0 && !streetWarInitialFill &&
        (!streetWarReinforcements || cityWarTotalSpawns >= CITY_WAR_TOTAL_SPAWN_BUDGET)) {
        beginStreetWarAftermath("BATTLE RESOLVED");
    }
}

function isWarPolicePed(ped) {
    if (!ped) return false;
    for (const unit of warPoliceUnits) if (unit.cops && unit.cops.includes(ped)) return true;
    return false;
}

function warPoliceUnitVisible(unit) {
    if (!unit) return false;
    if (carOnScreenSafe(unit.car)) return true;
    for (const cop of unit.cops || []) if (charOnScreenSafe(cop)) return true;
    return false;
}

function recordWarPoliceFailure(reason) {
    warPoliceSpawnFailures++;
    warPoliceLastFailure = reason || "UNKNOWN";
    log("WAR POLICE SPAWN FAIL:", warPoliceLastFailure,
        "units=", warPoliceUnits.length,
        "warState=", streetWarState);
}

function warSceneCenter() {
    let x = 0.0, y = 0.0, z = 0.0, count = 0;
    for (const ped of streetWarPeds) {
        if (!warPedValid(ped)) continue;
        const p = n("GET_CHAR_COORDINATES", ped);
        if (!p) continue;
        x += p.x; y += p.y; z += p.z; count++;
    }
    if (count > 0) return { x: x / count, y: y / count, z: z / count };
    return playerPos();
}

function findWarPoliceRoadNode() {
    const attempts = [
        { d: 70.0, s: 12.0 }, { d: 78.0, s: -14.0 },
        { d: 88.0, s: 20.0 }, { d: 96.0, s: -22.0 }
    ];
    const ppos = playerPos();
    if (!ppos) return null;

    for (let i = 0; i < attempts.length; i++) {
        const stage = hiddenSpawnBasis(attempts[i].d, attempts[i].s);
        if (!stage) continue;
        let node = n("GET_CLOSEST_CAR_NODE_WITH_HEADING", stage.x, stage.y, stage.z);
        if (!node || typeof node.pResX !== "number") {
            node = n("GET_NTH_CLOSEST_CAR_NODE_WITH_HEADING", stage.x, stage.y, stage.z, 2);
        }
        if (!node || typeof node.pResX !== "number") continue;
        const dx = node.pResX - ppos.x;
        const dy = node.pResY - ppos.y;
        if ((dx * dx + dy * dy) < 2025.0) continue; // keep police creation >=45m away
        return node;
    }
    return null;
}

function currentPoliceVehicleModel() {
    let model = n("GET_CURRENT_BASIC_POLICE_CAR_MODEL") || 0;
    if (!model || !n("IS_MODEL_IN_CDIMAGE", model)) model = hash("POLICE");
    return model;
}

function currentPolicePedModel() {
    let model = n("GET_CURRENT_BASIC_COP_MODEL") || 0;
    if (!model || !n("IS_MODEL_IN_CDIMAGE", model)) model = hash("M_Y_COP");
    return model;
}

function configureWarPoliceCop(cop, weapon) {
    if (!cop || !n("DOES_CHAR_EXIST", cop)) return;
    n("SET_CHAR_AS_MISSION_CHAR", cop);
    n("SET_CHAR_RANDOM_COMPONENT_VARIATION", cop);
    n("SET_CHAR_MAX_HEALTH", cop, 400);
    n("SET_CHAR_HEALTH", cop, 400);
    n("ADD_ARMOUR_TO_CHAR", cop, 100);
    n("SET_CHAR_ACCURACY", cop, 55);
    n("SET_CHAR_SHOOT_RATE", cop, 100);
    n("SET_CHAR_WILL_USE_COVER", cop, true);
    n("SET_CHAR_WILL_MOVE_WHEN_INJURED", cop, true);
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", cop, true);
    n("REMOVE_ALL_CHAR_WEAPONS", cop);
    n("GIVE_WEAPON_TO_CHAR", cop, weapon, 600, true);
    n("SET_CURRENT_CHAR_WEAPON", cop, weapon, true);
}

function warPoliceDesiredUnits() {
    if (!warPoliceResponseEnabled || policeDisabled || streetWarState !== "ACTIVE") return 0;
    if (streetWarLimit <= 6) return 1;
    if (streetWarLimit <= 10) return 2;
    return WAR_POLICE_MAX_UNITS;
}

function createWarPoliceUnit() {
    const node = findWarPoliceRoadNode();
    if (!node) {
        recordWarPoliceFailure("NO ROAD NODE");
        return false;
    }

    const carModel = currentPoliceVehicleModel();
    const copModel = currentPolicePedModel();
    if (!carModel || !copModel) {
        recordWarPoliceFailure("POLICE MODEL UNAVAILABLE");
        return false;
    }
    if (!loadModel(carModel, 2500) || !loadModel(copModel, 2500)) {
        recordWarPoliceFailure("POLICE MODEL LOAD FAILED");
        releaseModel(carModel); releaseModel(copModel);
        return false;
    }

    const car = n("CREATE_CAR", carModel, node.pResX, node.pResY, node.pResZ) || 0;
    if (!car || !n("DOES_VEHICLE_EXIST", car)) {
        recordWarPoliceFailure("POLICE CAR CREATE FAILED");
        releaseModel(carModel); releaseModel(copModel);
        return false;
    }

    n("SET_CAR_AS_MISSION_CAR", car);
    n("SET_CAR_HEADING", car, node.pHeading || 0.0);
    n("SWITCH_CAR_SIREN", car, true);
    n("SET_SIREN_WITH_NO_DRIVER", car, true);

    const driver = n("CREATE_CHAR_INSIDE_CAR", car, 1, copModel) || 0;
    const passenger = n("CREATE_CHAR_AS_PASSENGER", car, 1, copModel, 0) || 0;
    releaseModel(carModel); releaseModel(copModel);

    const cops = [];
    if (driver && n("DOES_CHAR_EXIST", driver)) { configureWarPoliceCop(driver, 7); cops.push(driver); }
    if (passenger && n("DOES_CHAR_EXIST", passenger)) { configureWarPoliceCop(passenger, 13); cops.push(passenger); }
    if (!cops.length || !driver || !n("DOES_CHAR_EXIST", driver)) {
        recordWarPoliceFailure("POLICE CREW CREATE FAILED");
        for (const cop of cops) n("MARK_CHAR_AS_NO_LONGER_NEEDED", cop);
        n("MARK_CAR_AS_NO_LONGER_NEEDED", car);
        return false;
    }

    const now = Date.now();
    const unit = {
        car, driver, cops,
        state: "ENROUTE",
        createdAt: now,
        stateAt: now,
        lastDriveOrderAt: 0,
        targets: {}
    };
    warPoliceUnits.push(unit);
    taskWarPoliceDrive(unit, true);
    return true;
}

function taskWarPoliceDrive(unit, force = false) {
    if (!unit || unit.state !== "ENROUTE") return;
    if (!unit.car || !n("DOES_VEHICLE_EXIST", unit.car) || n("IS_CAR_DEAD", unit.car)) return;
    if (!unit.driver || !n("DOES_CHAR_EXIST", unit.driver) || n("IS_CHAR_DEAD", unit.driver)) return;
    const now = Date.now();
    if (!force && now - unit.lastDriveOrderAt < 3000) return;
    const target = warSceneCenter();
    if (!target) return;
    unit.lastDriveOrderAt = now;
    n("SWITCH_CAR_SIREN", unit.car, true);
    // Current Sanny GTA IV enums define CarMission.GoTo = 4 and
    // DrivingMode.AvoidCars = 2. Small stop/straight-line radii let the cruiser
    // approach the changing war center without constantly overshooting it.
    n("TASK_CAR_MISSION_COORS_TARGET", unit.driver, unit.car,
        target.x, target.y, target.z, 4, WAR_POLICE_DRIVE_SPEED, 2, 5, 10);
}

function warPoliceDistanceToScene(unit) {
    if (!unit || !unit.car || !n("DOES_VEHICLE_EXIST", unit.car)) return 9999.0;
    const c = n("GET_CAR_COORDINATES", unit.car);
    const t = warSceneCenter();
    if (!c || !t) return 9999.0;
    const dx = c.x - t.x;
    const dy = c.y - t.y;
    return Math.sqrt(dx * dx + dy * dy);
}

function deployWarPoliceUnit(unit) {
    if (!unit || unit.state !== "ENROUTE") return;
    unit.state = "DEPLOYING";
    unit.stateAt = Date.now();
    for (const cop of unit.cops) {
        if (cop && n("DOES_CHAR_EXIST", cop) && !n("IS_CHAR_DEAD", cop)) n("TASK_LEAVE_ANY_CAR", cop);
    }
}

function pickWarPoliceTarget(cop) {
    if (!cop || !n("DOES_CHAR_EXIST", cop)) return 0;
    const q = n("GET_CHAR_COORDINATES", cop);
    if (!q) return 0;
    let best = 0, bestD2 = 99999999.0;
    for (const ped of streetWarPeds) {
        if (!warPedValid(ped)) continue;
        const p = n("GET_CHAR_COORDINATES", ped);
        if (!p) continue;
        const dx = p.x - q.x, dy = p.y - q.y;
        const d2 = dx * dx + dy * dy;
        if (d2 < bestD2) { bestD2 = d2; best = ped; }
    }
    return best;
}

function assignWarPoliceTarget(unit, cop, force = false) {
    if (!unit || !cop || !n("DOES_CHAR_EXIST", cop) || n("IS_CHAR_DEAD", cop)) return false;
    const key = String(cop);
    const old = unit.targets[key] || { target: 0, lastTaskAt: 0 };
    const now = Date.now();
    if (!force && old.target && warPedValid(old.target)) {
        const status = n("GET_SCRIPT_TASK_STATUS", cop, 91);
        if (status !== 7 && status !== null && now - old.lastTaskAt < WAR_POLICE_TARGET_HOLD_MS) return true;
    }
    const target = pickWarPoliceTarget(cop);
    if (!target) { unit.targets[key] = { target: 0, lastTaskAt: now }; return false; }
    unit.targets[key] = { target, lastTaskAt: now };
    n("TASK_COMBAT", cop, target);
    return true;
}

function releaseWarPoliceUnit(unit, force = false) {
    if (!unit) return true;
    if (!force && warPoliceUnitVisible(unit)) return false;
    for (const cop of unit.cops || []) {
        if (!cop || !n("DOES_CHAR_EXIST", cop)) continue;
        n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", cop, false);
        if (!n("IS_CHAR_DEAD", cop)) {
            n("CLEAR_CHAR_TASKS", cop);
            n("TASK_WANDER_STANDARD", cop);
        }
        n("MARK_CHAR_AS_NO_LONGER_NEEDED", cop);
    }
    if (unit.car && n("DOES_VEHICLE_EXIST", unit.car)) {
        n("SWITCH_CAR_SIREN", unit.car, false);
        n("MARK_CAR_AS_NO_LONGER_NEEDED", unit.car);
    }
    return true;
}

function clearWarPoliceResponse(force = true) {
    const kept = [];
    for (const unit of warPoliceUnits) {
        if (!releaseWarPoliceUnit(unit, force)) kept.push(unit);
    }
    warPoliceUnits = kept;
    if (force || !warPoliceUnits.length) warPoliceNextDispatchAt = 0;
}

function toggleWarPoliceResponse() {
    warPoliceResponseEnabled = !warPoliceResponseEnabled;
    if (!warPoliceResponseEnabled) {
        // Do not pop a visible cruiser/officer out of existence. Existing visible
        // response units retire as soon as the player looks away.
        clearWarPoliceResponse(false);
    } else if (streetWarState === "ACTIVE") {
        warPoliceNextDispatchAt = Date.now() + WAR_POLICE_INITIAL_DELAY_MS;
    }
    notify("CITY WAR POLICE RESPONSE: " + (warPoliceResponseEnabled ? "ON" : "OFF"));
}

function warPoliceStatus() {
    let enroute = 0, deployed = 0;
    for (const unit of warPoliceUnits) {
        if (unit.state === "ENROUTE" || unit.state === "DEPLOYING") enroute++;
        else if (unit.state === "DEPLOYED") deployed++;
    }
    notify("WAR POLICE" +
        "\nENABLED: " + (warPoliceResponseEnabled ? "YES" : "NO") +
        "\nUNITS: " + warPoliceUnits.length + "/" + WAR_POLICE_MAX_UNITS +
        "\nENROUTE: " + enroute +
        "\nDEPLOYED: " + deployed +
        "\nCASUALTIES: " + warPoliceCasualties().length +
        "\nEMS: " + (warEmsEnabled ? "ON" : "OFF") +
        "\nSPAWN FAILS: " + warPoliceSpawnFailures +
        "\nLAST: " + warPoliceLastFailure);
}

function pulseWarPoliceResponse() {
    const now = Date.now();
    const combatPhase = streetWarState === "ACTIVE" || streetWarState === "ENDING";
    const preserveForAftermath = warEmsEnabled &&
        (streetWarState === "ACTIVE" || streetWarState === "ENDING" || streetWarState === "AFTERMATH");

    // Master police disable / full City War OFF still retires the scripted response.
    if ((!warPoliceResponseEnabled || policeDisabled || streetWarState === "OFF") && warPoliceUnits.length) {
        const retiring = [];
        for (const unit of warPoliceUnits) {
            if (!releaseWarPoliceUnit(unit, false)) retiring.push(unit);
        }
        warPoliceUnits = retiring;
        if (!warPoliceUnits.length) warPoliceNextDispatchAt = 0;
        return;
    }

    // During AFTERMATH, keep police scene ownership so EMS can address only these
    // known officers. No combat retasking and no new cruisers are allowed.
    if (streetWarState === "AFTERMATH") return;

    const kept = [];
    for (const unit of warPoliceUnits) {
        const carExists = unit.car && n("DOES_VEHICLE_EXIST", unit.car) && !n("IS_CAR_DEAD", unit.car);
        const livingCops = (unit.cops || []).filter(c => c && n("DOES_CHAR_EXIST", c) && !n("IS_CHAR_DEAD", c));

        if (!carExists && unit.state === "ENROUTE") deployWarPoliceUnit(unit);
        if (!livingCops.length) {
            // Dead Dynamic Liberty officers remain script-owned until the battle
            // reaches aftermath so the EMS system can find them reliably.
            if (preserveForAftermath && warPoliceCasualtiesForUnit(unit).length) {
                kept.push(unit);
                continue;
            }
            if (!releaseWarPoliceUnit(unit, false)) kept.push(unit);
            continue;
        }

        if (unit.state === "ENROUTE") {
            taskWarPoliceDrive(unit, false);
            const timedOut = now - unit.createdAt >= WAR_POLICE_ENROUTE_TIMEOUT_MS;
            if (warPoliceDistanceToScene(unit) <= WAR_POLICE_ARRIVAL_DISTANCE || timedOut) deployWarPoliceUnit(unit);
        } else if (unit.state === "DEPLOYING") {
            if (now - unit.stateAt >= WAR_POLICE_DEPLOY_DELAY_MS) {
                unit.state = "DEPLOYED";
                unit.stateAt = now;
                if (combatPhase) for (const cop of livingCops) assignWarPoliceTarget(unit, cop, true);
            }
        } else if (unit.state === "DEPLOYED" && combatPhase) {
            for (const cop of livingCops) assignWarPoliceTarget(unit, cop, false);
        }
        kept.push(unit);
    }
    warPoliceUnits = kept;

    // ENDING keeps current cops fighting but never dispatches fresh cruisers.
    if (!warPoliceResponseEnabled || policeDisabled || streetWarState !== "ACTIVE") return;
    const desired = warPoliceDesiredUnits();
    if (warPoliceUnits.length >= desired) return;
    if (!warPoliceNextDispatchAt) warPoliceNextDispatchAt = now + WAR_POLICE_INITIAL_DELAY_MS;
    if (now < warPoliceNextDispatchAt) return;

    if (createWarPoliceUnit()) {
        warPoliceNextDispatchAt = now + WAR_POLICE_REINFORCEMENT_DELAY_MS;
        notify("POLICE RESPONSE EN ROUTE: " + warPoliceUnits.length + "/" + desired);
    } else {
        // Optional response failure must never break the gang war.
        warPoliceNextDispatchAt = now + 5000;
    }
}

// ------------------------------------------------------------
// V6.1.4 CITY WAR AFTERMATH / EMS RESPONSE
// Production path intentionally uses the already-proven road-node + CREATE_CAR
// approach instead of depending on a conditional multi-output emergency-services
// native before that return shape is hardware-tested on GTA IV CE.
// ------------------------------------------------------------
function warPoliceCasualtiesForUnit(unit) {
    if (!unit) return [];
    return (unit.cops || []).filter(cop => {
        if (!cop || !n("DOES_CHAR_EXIST", cop)) return false;
        if (n("IS_CHAR_DEAD", cop)) return true;
        const injured = !!n("IS_CHAR_INJURED", cop);
        const health = n("GET_CHAR_HEALTH", cop);
        return injured || (typeof health === "number" && health > 0 && health < 220);
    });
}

function warPoliceCasualties() {
    const seen = {};
    const out = [];
    for (const unit of warPoliceUnits) {
        for (const cop of warPoliceCasualtiesForUnit(unit)) {
            const key = String(cop);
            if (seen[key]) continue;
            seen[key] = true;
            out.push(cop);
        }
    }
    // Dead officers first, then injured survivors.
    out.sort((a, b) => (n("IS_CHAR_DEAD", b) ? 1 : 0) - (n("IS_CHAR_DEAD", a) ? 1 : 0));
    return out;
}

function warEmsUnitVisible(unit) {
    if (!unit) return false;
    if (carOnScreenSafe(unit.car)) return true;
    for (const medic of unit.medics || []) if (charOnScreenSafe(medic)) return true;
    return false;
}

function recordWarEmsFailure(reason) {
    warEmsSpawnFailures++;
    warEmsLastFailure = reason || "UNKNOWN";
    log("WAR EMS FAIL:", warEmsLastFailure,
        "state=", streetWarState,
        "casualties=", warPoliceCasualties().length);
}

function findWarEmsRoadNode() {
    const attempts = [
        { d: 86.0, s: 16.0 }, { d: 96.0, s: -18.0 },
        { d: 106.0, s: 24.0 }, { d: 114.0, s: -26.0 }
    ];
    const ppos = playerPos();
    if (!ppos) return null;

    for (const a of attempts) {
        const stage = hiddenSpawnBasis(a.d, a.s);
        if (!stage) continue;
        let node = n("GET_CLOSEST_CAR_NODE_WITH_HEADING", stage.x, stage.y, stage.z);
        if (!node || typeof node.pResX !== "number") {
            node = n("GET_NTH_CLOSEST_CAR_NODE_WITH_HEADING", stage.x, stage.y, stage.z, 2);
        }
        if (!node || typeof node.pResX !== "number") continue;
        const dx = node.pResX - ppos.x, dy = node.pResY - ppos.y;
        if ((dx * dx + dy * dy) < 3025.0) continue; // >=55m from player
        return node;
    }
    return null;
}

function configureWarMedic(medic) {
    if (!medic || !n("DOES_CHAR_EXIST", medic)) return;
    n("SET_CHAR_AS_MISSION_CHAR", medic);
    n("SET_CHAR_RANDOM_COMPONENT_VARIATION", medic);
    n("SET_CHAR_MAX_HEALTH", medic, 300);
    n("SET_CHAR_HEALTH", medic, 300);
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", medic, true);
    n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", medic, false);
    n("REMOVE_ALL_CHAR_WEAPONS", medic);
}

function createWarEmsUnit() {
    if (warEmsUnit || streetWarState !== "AFTERMATH") return false;
    const casualties = warPoliceCasualties().filter(c => !warEmsTreatedPolice.includes(c));
    if (!casualties.length) {
        warEmsSceneComplete = true;
        return false;
    }

    const node = findWarEmsRoadNode();
    if (!node) { recordWarEmsFailure("NO EMS ROAD NODE"); return false; }

    const ambulanceModel = hash("AMBULANCE");
    const medicModel = hash("M_Y_PMEDIC");
    if (!ambulanceModel || !medicModel ||
        !n("IS_MODEL_IN_CDIMAGE", ambulanceModel) || !n("IS_MODEL_IN_CDIMAGE", medicModel)) {
        recordWarEmsFailure("EMS MODEL UNAVAILABLE");
        return false;
    }
    if (!loadModel(ambulanceModel, 2500) || !loadModel(medicModel, 2500)) {
        recordWarEmsFailure("EMS MODEL LOAD FAILED");
        releaseModel(ambulanceModel); releaseModel(medicModel);
        return false;
    }

    const car = n("CREATE_CAR", ambulanceModel, node.pResX, node.pResY, node.pResZ) || 0;
    if (!car || !n("DOES_VEHICLE_EXIST", car)) {
        recordWarEmsFailure("AMBULANCE CREATE FAILED");
        releaseModel(ambulanceModel); releaseModel(medicModel);
        return false;
    }
    n("SET_CAR_AS_MISSION_CAR", car);
    n("SET_CAR_HEADING", car, node.pHeading || 0.0);
    n("SWITCH_CAR_SIREN", car, true);
    n("SET_SIREN_WITH_NO_DRIVER", car, true);

    const driver = n("CREATE_CHAR_INSIDE_CAR", car, 1, medicModel) || 0;
    const passenger = n("CREATE_CHAR_AS_PASSENGER", car, 1, medicModel, 0) || 0;
    releaseModel(ambulanceModel); releaseModel(medicModel);

    const medics = [];
    if (driver && n("DOES_CHAR_EXIST", driver)) { configureWarMedic(driver); medics.push(driver); }
    if (passenger && n("DOES_CHAR_EXIST", passenger)) { configureWarMedic(passenger); medics.push(passenger); }
    if (!driver || !n("DOES_CHAR_EXIST", driver) || !medics.length) {
        recordWarEmsFailure("EMS CREW CREATE FAILED");
        for (const medic of medics) n("MARK_CHAR_AS_NO_LONGER_NEEDED", medic);
        n("MARK_CAR_AS_NO_LONGER_NEEDED", car);
        return false;
    }

    const now = Date.now();
    warEmsUnit = {
        car, driver, passenger, medics,
        state: "ENROUTE", createdAt: now, stateAt: now,
        lastDriveOrderAt: 0, treatments: [], returnOrdered: false
    };
    taskWarEmsDrive(warEmsUnit, true);
    notify("EMS RESPONSE EN ROUTE");
    return true;
}

function warEmsTargetPoint() {
    const casualties = warPoliceCasualties();
    let x = 0.0, y = 0.0, z = 0.0, count = 0;
    for (const cop of casualties) {
        if (!cop || !n("DOES_CHAR_EXIST", cop)) continue;
        const p = n("GET_CHAR_COORDINATES", cop);
        if (!p) continue;
        x += p.x; y += p.y; z += p.z; count++;
    }
    if (count) return { x: x / count, y: y / count, z: z / count };
    return warSceneCenter();
}

function taskWarEmsDrive(unit, force = false) {
    if (!unit || unit.state !== "ENROUTE") return;
    if (!unit.car || !n("DOES_VEHICLE_EXIST", unit.car) || n("IS_CAR_DEAD", unit.car)) return;
    if (!unit.driver || !n("DOES_CHAR_EXIST", unit.driver) || n("IS_CHAR_DEAD", unit.driver)) return;
    const now = Date.now();
    if (!force && now - unit.lastDriveOrderAt < 3000) return;
    const target = warEmsTargetPoint();
    if (!target) return;
    unit.lastDriveOrderAt = now;
    n("SWITCH_CAR_SIREN", unit.car, true);
    n("TASK_CAR_MISSION_COORS_TARGET", unit.driver, unit.car,
        target.x, target.y, target.z, 4, WAR_EMS_DRIVE_SPEED, 2, 6, 12);
}

function warEmsDistanceToScene(unit) {
    if (!unit || !unit.car || !n("DOES_VEHICLE_EXIST", unit.car)) return 9999.0;
    const c = n("GET_CAR_COORDINATES", unit.car);
    const t = warEmsTargetPoint();
    if (!c || !t) return 9999.0;
    const dx = c.x - t.x, dy = c.y - t.y;
    return Math.sqrt(dx * dx + dy * dy);
}

function deployWarEmsUnit(unit) {
    if (!unit || unit.state !== "ENROUTE") return;
    unit.state = "DEPLOYING";
    unit.stateAt = Date.now();
    for (const medic of unit.medics || []) {
        if (medic && n("DOES_CHAR_EXIST", medic) && !n("IS_CHAR_DEAD", medic)) n("TASK_LEAVE_ANY_CAR", medic);
    }
}

function assignWarEmsTreatments(unit) {
    if (!unit) return;
    const casualties = warPoliceCasualties().filter(c => !warEmsTreatedPolice.includes(c));
    const medics = (unit.medics || []).filter(m => m && n("DOES_CHAR_EXIST", m) && !n("IS_CHAR_DEAD", m));
    unit.treatments = [];

    const count = Math.min(WAR_EMS_MAX_TREATMENTS, casualties.length, medics.length);
    for (let i = 0; i < count; i++) {
        const medic = medics[i];
        const cop = casualties[i];
        n("CLEAR_CHAR_TASKS", medic);
        n("TASK_GOTO_CHAR_OFFSET", medic, cop, 10000, i === 0 ? -0.7 : 0.7, 0.8);
        unit.treatments.push({ medic, cop, state: "APPROACH", stateAt: Date.now() });
    }

    if (!unit.treatments.length) {
        warEmsSceneComplete = true;
        beginWarEmsReturn(unit);
    } else {
        unit.state = "TREATING";
        unit.stateAt = Date.now();
    }
}

function pedDistance(a, b) {
    if (!a || !b || !n("DOES_CHAR_EXIST", a) || !n("DOES_CHAR_EXIST", b)) return 9999.0;
    const pa = n("GET_CHAR_COORDINATES", a);
    const pb = n("GET_CHAR_COORDINATES", b);
    if (!pa || !pb) return 9999.0;
    const dx = pa.x - pb.x, dy = pa.y - pb.y;
    return Math.sqrt(dx * dx + dy * dy);
}

function performWarEmsAid(t) {
    if (!t || !t.cop || !n("DOES_CHAR_EXIST", t.cop)) return;
    const cop = t.cop;

    // REVIVE_INJURED_PED is a verified GTA IV native. The follow-up health/task
    // reset is intentionally conservative because the exact CE get-up transition
    // still needs hardware validation.
    if (n("IS_CHAR_DEAD", cop) || n("IS_CHAR_INJURED", cop)) {
        n("REVIVE_INJURED_PED", cop);
    }
    n("SET_CHAR_MAX_HEALTH", cop, 400);
    n("SET_CHAR_HEALTH", cop, 250);
    n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", cop, false);
    n("CLEAR_CHAR_TASKS", cop);
    n("TASK_STAND_STILL", cop, 12000);

    if (!warEmsTreatedPolice.includes(cop)) warEmsTreatedPolice.push(cop);
    warEmsTreatments++;
}

function beginWarEmsReturn(unit) {
    if (!unit || unit.state === "RETURNING" || unit.state === "LEAVING") return;
    unit.state = "RETURNING";
    unit.stateAt = Date.now();
    unit.returnOrdered = true;

    if (!unit.car || !n("DOES_VEHICLE_EXIST", unit.car)) {
        unit.state = "LEAVING";
        for (const medic of unit.medics || []) {
            if (medic && n("DOES_CHAR_EXIST", medic) && !n("IS_CHAR_DEAD", medic)) {
                n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", medic, false);
                n("TASK_WANDER_STANDARD", medic);
            }
        }
        return;
    }

    if (unit.driver && n("DOES_CHAR_EXIST", unit.driver) && !n("IS_CHAR_DEAD", unit.driver))
        n("TASK_ENTER_CAR_AS_DRIVER", unit.driver, unit.car, 12000);
    if (unit.passenger && n("DOES_CHAR_EXIST", unit.passenger) && !n("IS_CHAR_DEAD", unit.passenger))
        n("TASK_ENTER_CAR_AS_PASSENGER", unit.passenger, unit.car, 12000, 0);
}

function releaseWarEmsUnit(force = false) {
    const unit = warEmsUnit;
    if (!unit) return true;
    if (!force && warEmsUnitVisible(unit)) return false;

    for (const medic of unit.medics || []) {
        if (!medic || !n("DOES_CHAR_EXIST", medic)) continue;
        n("SET_BLOCKING_OF_NON_TEMPORARY_EVENTS", medic, false);
        if (!n("IS_CHAR_DEAD", medic)) {
            n("CLEAR_CHAR_TASKS", medic);
            n("TASK_WANDER_STANDARD", medic);
        }
        n("MARK_CHAR_AS_NO_LONGER_NEEDED", medic);
    }
    if (unit.car && n("DOES_VEHICLE_EXIST", unit.car)) {
        n("SWITCH_CAR_SIREN", unit.car, false);
        n("MARK_CAR_AS_NO_LONGER_NEEDED", unit.car);
    }
    warEmsUnit = null;
    return true;
}

function clearWarEmsResponse(force = true) {
    if (warEmsUnit) releaseWarEmsUnit(force);
    if (force) warEmsUnit = null;
    warEmsDispatchAt = 0;
}

function toggleWarEmsResponse() {
    warEmsEnabled = !warEmsEnabled;
    if (!warEmsEnabled) {
        warEmsDispatchAt = 0;
        clearWarEmsResponse(false);
        if (streetWarState === "AFTERMATH") warEmsSceneComplete = true;
    } else if (streetWarState === "AFTERMATH" && warPoliceCasualties().length) {
        warEmsSceneComplete = false;
        warEmsDispatchAt = Date.now() + WAR_EMS_INITIAL_DELAY_MS;
    }
    notify("CITY WAR EMS: " + (warEmsEnabled ? "ON" : "OFF"));
}

function warEmsStatus() {
    notify("WAR EMS" +
        "\nENABLED: " + (warEmsEnabled ? "YES" : "NO") +
        "\nSTATE: " + (warEmsUnit ? warEmsUnit.state : "IDLE") +
        "\nPOLICE CASUALTIES: " + warPoliceCasualties().length +
        "\nTREATED: " + warEmsTreatments +
        "\nFAILS: " + warEmsSpawnFailures +
        "\nLAST: " + warEmsLastFailure);
}

function pulseWarEmsResponse() {
    if (streetWarState !== "AFTERMATH") return;
    const now = Date.now();

    if (!warEmsEnabled || policeDisabled) {
        warEmsSceneComplete = true;
        clearWarEmsResponse(false);
        finalizeStreetWarAftermathIfClear();
        return;
    }

    const remaining = warPoliceCasualties().filter(c => !warEmsTreatedPolice.includes(c));
    if (!warEmsUnit) {
        if (!remaining.length || warEmsTreatments >= WAR_EMS_MAX_TREATMENTS) {
            warEmsSceneComplete = true;
            finalizeStreetWarAftermathIfClear();
            return;
        }
        if (warEmsSpawnFailures >= WAR_EMS_MAX_SPAWN_RETRIES) {
            warEmsSceneComplete = true;
            notify("EMS RESPONSE FAILED / AFTERMATH CONTINUES");
            finalizeStreetWarAftermathIfClear();
            return;
        }
        if (!warEmsDispatchAt) warEmsDispatchAt = now + WAR_EMS_INITIAL_DELAY_MS;
        if (now < warEmsDispatchAt) return;
        if (!createWarEmsUnit()) warEmsDispatchAt = now + 5000;
        return;
    }

    const unit = warEmsUnit;
    if (unit.state === "ENROUTE") {
        taskWarEmsDrive(unit, false);
        const timedOut = now - unit.createdAt >= WAR_EMS_ENROUTE_TIMEOUT_MS;
        if (warEmsDistanceToScene(unit) <= WAR_EMS_ARRIVAL_DISTANCE || timedOut) deployWarEmsUnit(unit);
    } else if (unit.state === "DEPLOYING") {
        if (now - unit.stateAt >= WAR_EMS_DEPLOY_DELAY_MS) assignWarEmsTreatments(unit);
    } else if (unit.state === "TREATING") {
        let allDone = true;
        for (const t of unit.treatments) {
            if (t.state === "DONE") continue;
            allDone = false;
            if (!t.medic || !n("DOES_CHAR_EXIST", t.medic) || n("IS_CHAR_DEAD", t.medic) ||
                !t.cop || !n("DOES_CHAR_EXIST", t.cop)) {
                t.state = "DONE";
                continue;
            }

            if (t.state === "APPROACH") {
                const closeEnough = pedDistance(t.medic, t.cop) <= 2.4;
                const timedOut = now - t.stateAt >= WAR_EMS_APPROACH_TIMEOUT_MS;
                if (closeEnough || timedOut) {
                    n("CLEAR_CHAR_TASKS", t.medic);
                    n("TASK_STAND_STILL", t.medic, WAR_EMS_AID_TIME_MS);
                    t.state = "AIDING";
                    t.stateAt = now;
                }
            } else if (t.state === "AIDING" && now - t.stateAt >= WAR_EMS_AID_TIME_MS) {
                performWarEmsAid(t);
                t.state = "DONE";
                t.stateAt = now;
            }
        }

        if (unit.treatments.length && unit.treatments.every(t => t.state === "DONE")) {
            warEmsSceneComplete = true;
            beginWarEmsReturn(unit);
        }
    } else if (unit.state === "RETURNING") {
        const driverReady = unit.driver && n("DOES_CHAR_EXIST", unit.driver) &&
            unit.car && n("DOES_VEHICLE_EXIST", unit.car) && n("IS_CHAR_IN_CAR", unit.driver, unit.car);
        if (driverReady) {
            n("SWITCH_CAR_SIREN", unit.car, false);
            n("TASK_CAR_DRIVE_WANDER", unit.driver, unit.car, WAR_EMS_DRIVE_SPEED, 2);
            unit.state = "LEAVING";
            unit.stateAt = now;
        } else if (now - unit.stateAt >= WAR_EMS_RETURN_TIMEOUT_MS) {
            unit.state = "LEAVING";
            unit.stateAt = now;
        }
    } else if (unit.state === "LEAVING") {
        if (!warEmsUnitVisible(unit) && now - unit.stateAt >= 5000) {
            releaseWarEmsUnit(false);
            finalizeStreetWarAftermathIfClear();
        }
    }
}

function pulseWarCarjackers() {
    if (!streetWarCarjackers || streetWarLimit <= 0 || !streetWarPeds.length) return;
    const now = Date.now();

    // Finish pending theft attempts first.
    const pending = [];
    for (const a of carjackAssignments) {
        if (!warPedValid(a.ped) || !a.car || n("IS_CAR_DEAD", a.car)) continue;

        if (n("IS_CHAR_IN_CAR", a.ped, a.car)) {
            n("TASK_CAR_DRIVE_WANDER", a.ped, a.car, streetWarDriveSpeed, 3); // PloughThrough
            n("SET_CHAR_WILL_DO_DRIVEBYS", a.ped, true);

            // v6.1.1 delayed handoff: the thief leaves the infantry slot now,
            // but stays mission-owned as an ESCAPING managed actor long enough
            // for the getaway to remain visible before GTA IV can cull it.
            handoffWarCarjackerToEscape(a.ped, a.car);
            continue;
        }

        if (now - a.started < 14000) pending.push(a);
    }
    carjackAssignments = pending;

    if (carjackAssignments.length >= 3) return;
    if (Math.random() > 0.55) return;

    const available = streetWarPeds.filter(p =>
        warPedValid(p) && !n("IS_CHAR_IN_ANY_CAR", p) && !isWarCarjackAssigned(p)
    );
    if (!available.length) return;
    const ped = available[Math.floor(Math.random() * available.length)];
    const pos = n("GET_CHAR_COORDINATES", ped);
    if (!pos) return;

    // GTA IV: model hash 0 searches all vehicle models (GTAMods native docs).
    const car = n("GET_RANDOM_CAR_IN_SPHERE_NO_SAVE", pos.x, pos.y, pos.z, 20.0, 0, true);
    if (!car || n("IS_CAR_DEAD", car)) return;
    if (car === currentCar()) return;

    n("TASK_ENTER_CAR_AS_DRIVER", ped, car, 12000);
    carjackAssignments.push({ ped, car, started: now });
}

function sampleIncendiaryCandidates() {
    const pos = playerPos();
    const me = playerPed();
    if (!pos || !me) return;

    const ped = n(
        "GET_RANDOM_CHAR_IN_AREA_OFFSET_NO_SAVE",
        pos.x - 40.0, pos.y - 40.0, pos.z - 8.0,
        80.0, 80.0, 16.0
    );
    if (ped && ped !== me && !bodyguards.includes(ped) && !incendiaryPeds.includes(ped)) {
        incendiaryPeds.push(ped);
        if (incendiaryPeds.length > 20) incendiaryPeds.shift();
    }

    const car = n("GET_RANDOM_CAR_IN_SPHERE_NO_SAVE", pos.x, pos.y, pos.z, 55.0, 0, true);
    if (car && !incendiaryCars.includes(car)) {
        incendiaryCars.push(car);
        if (incendiaryCars.length > 12) incendiaryCars.shift();
    }
}

function pulseIncendiaryHits() {
    const me = playerPed();
    if (!me) return;

    if (incendiaryHits) {
        sampleIncendiaryCandidates();

        incendiaryPeds = incendiaryPeds.filter(p => p && !!n("DOES_CHAR_EXIST", p) && !n("IS_CHAR_DEAD", p));
        for (const ped of incendiaryPeds) {
            if (n("HAS_CHAR_BEEN_DAMAGED_BY_CHAR", ped, me, true)) {
                if (!n("IS_CHAR_ON_FIRE", ped) && Math.random() * 100.0 < incendiaryChance) {
                    n("START_CHAR_FIRE", ped);
                }
                n("CLEAR_CHAR_LAST_DAMAGE_ENTITY", ped);
            }
        }

        incendiaryCars = incendiaryCars.filter(c => c && !n("IS_CAR_DEAD", c));
        for (const car of incendiaryCars) {
            if (n("HAS_CAR_BEEN_DAMAGED_BY_CHAR", car, me)) {
                if (!n("IS_CAR_ON_FIRE", car) && Math.random() * 100.0 < incendiaryChance) {
                    n("START_CAR_FIRE", car);
                }
                n("CLEAR_CAR_LAST_DAMAGE_ENTITY", car);
            }
        }
    }

    // Optional war-zone incendiary ammunition. We check weapon damage on
    // tracked fighters only, avoiding an O(n^2) attacker/victim scan.
    if (streetWarFireAmmo && streetWarLimit > 0) {
        for (const ped of streetWarPeds) {
            if (!warPedValid(ped) || n("IS_CHAR_ON_FIRE", ped)) continue;
            let hit = false;
            for (const w of WAR_WEAPONS.mixed) {
                if (n("HAS_CHAR_BEEN_DAMAGED_BY_WEAPON", ped, w)) {
                    hit = true;
                    break;
                }
            }
            if (hit) {
                n("START_CHAR_FIRE", ped);
                n("CLEAR_CHAR_LAST_WEAPON_DAMAGE", ped);
            }
        }
    }
}

function warZonePreset() {
    streetWarWeaponMode = 2;
    streetWarAccuracy = 50;
    streetWarShootRate = 125;
    streetWarHealth = 400;
    streetWarArmour = 50;
    streetWarUseCover = true;
    streetWarReinforcements = true;
    streetWarCarjackers = true;
    streetWarPlayerNeutral = true;
    streetWarDriveSpeed = 28.0;
    madDriversEnabled = true;
    n("SWITCH_MAD_DRIVERS", true);
    n("SET_CAR_DENSITY_MULTIPLIER", 1.25);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 1.25);
    n("SET_PED_DENSITY_MULTIPLIER", 1.65);
    setStreetWar(16, "WAR ZONE 8v8");
}

function forceWarRetarget() {
    for (const ped of streetWarPeds) assignWarTarget(ped, true);
    notify("WAR TARGETS REFRESHED");
}

function imguiDiagnostic() {
    notify("IMGUIREDUX ACTIVE\nDYNAMIC LIBERTY V6.1.2");
}


// ------------------------------------------------------------
// WORLD / POLICE / WOW MODES
// ------------------------------------------------------------
function trafficDensity(value, label) {
    n("SET_CAR_DENSITY_MULTIPLIER", value);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", value);
    notify("TRAFFIC: " + label);
}

function pedDensity(value, label) {
    n("SET_PED_DENSITY_MULTIPLIER", value);
    notify("PEDESTRIANS: " + label);
}

function setGameSpeed(value, label) {
    n("SET_TIME_SCALE", value);
    notify("WORLD SPEED: " + label);
}

function setWanted(level) {
    const p = playerId();
    n("ALTER_WANTED_LEVEL", p, level);
    n("APPLY_WANTED_LEVEL_CHANGE_NOW", p);
    notify("WANTED LEVEL: " + level);
}

function toggleTrains() {
    trainsEnabled = !trainsEnabled;
    n("SWITCH_RANDOM_TRAINS", trainsEnabled);
    notify("TRAINS: " + (trainsEnabled ? "ON" : "OFF"));
}

function togglePoliceHelis() {
    policeHelisEnabled = !policeHelisEnabled;
    n("SWITCH_POLICE_HELIS", policeDisabled ? false : policeHelisEnabled);
    notify("POLICE HELIS: " + (policeHelisEnabled ? "ON" : "OFF") +
        (policeDisabled ? " (MASTER POLICE OFF)" : ""));
}

// A hidden-feeling "movie city" preset: more life, normal game speed.
function movieCity() {
    n("SET_CAR_DENSITY_MULTIPLIER", 1.75);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 1.75);
    n("SET_PED_DENSITY_MULTIPLIER", 1.6);
    n("SWITCH_RANDOM_TRAINS", true);
    n("SET_TIME_SCALE", 1.0);
    notify("MOVIE CITY MODE");
}

// Empty streets without deleting anything already spawned.
function ghostCity() {
    n("SET_CAR_DENSITY_MULTIPLIER", 0.0);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.0);
    n("SET_PED_DENSITY_MULTIPLIER", 0.0);
    notify("GHOST CITY MODE");
}

// "Purge hour": midnight, lots of people, fewer cars, armed hostility.
function purgeHour() {
    frozenHour = 0;
    n("SET_TIME_OF_DAY", 0, 0);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.45);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.45);
    n("SET_PED_DENSITY_MULTIPLIER", 1.8);
    setAggressiveMode(2);
    notify("PURGE HOUR ENABLED");
}

function nightSiege() {
    const p = playerId();
    frozenHour = 2;
    n("SET_TIME_OF_DAY", 2, 0);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.75);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.75);
    n("SET_PED_DENSITY_MULTIPLIER", 1.6);
    n("SWITCH_POLICE_HELIS", true);
    policeHelisEnabled = true;
    setAggressiveMode(2);
    n("ALTER_WANTED_LEVEL", p, 4);
    n("APPLY_WANTED_LEVEL_CHANGE_NOW", p);
    notify("NIGHT SIEGE");
}

function fightClubCity() {
    const p = playerId();
    n("ALTER_WANTED_LEVEL", p, 0);
    n("APPLY_WANTED_LEVEL_CHANGE_NOW", p);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.35);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.35);
    n("SET_PED_DENSITY_MULTIPLIER", 2.0);
    setAggressiveMode(1);
    notify("FIGHT CLUB CITY");
}

function rushHourHell() {
    n("SET_TIME_OF_DAY", 17, 0);
    n("SET_CAR_DENSITY_MULTIPLIER", 2.35);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 2.35);
    n("SET_PED_DENSITY_MULTIPLIER", 1.35);
    n("SET_TIME_SCALE", 1.0);
    notify("RUSH HOUR HELL");
}

function slowMoMayhem() {
    n("SET_TIME_SCALE", 0.35);
    n("SET_CAR_DENSITY_MULTIPLIER", 1.25);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 1.25);
    n("SET_PED_DENSITY_MULTIPLIER", 1.5);
    setAggressiveMode(2);
    notify("SLOW-MO MAYHEM");
}

function manhuntMode() {
    const p = playerId();
    setAggressiveMode(0);
    n("SET_TIME_SCALE", 1.0);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.85);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.85);
    n("SET_PED_DENSITY_MULTIPLIER", 1.0);
    n("SWITCH_POLICE_HELIS", true);
    policeHelisEnabled = true;
    n("ALTER_WANTED_LEVEL", p, 6);
    n("APPLY_WANTED_LEVEL_CHANGE_NOW", p);
    notify("MANHUNT");
}

function normalWorld() {
    const p = playerId();

    clearStreetWar(true);
    clearSpawnedPeds();
    releaseWeatherControl();
    n("FORCE_WIND", 1.0);

    if (policeDisabled) setPoliceDisabled(false, false);
    streetWarCarjackers = false;
    streetWarFireAmmo = false;
    incendiaryHits = false;
    incendiaryPeds = [];
    incendiaryCars = [];

    if (accessMode) setAccessModeState(false, false);
    else n("SET_MAX_WANTED_LEVEL", previousMaxWantedLevel);

    everyoneIgnorePlayer = false;
    n("SET_EVERYONE_IGNORE_PLAYER", p, false);
    n("SET_PLAYER_CAN_BE_HASSLED_BY_GANGS", p, true);
    n("DONT_DISPATCH_COPS_FOR_PLAYER", p, false);
    n("SET_POLICE_IGNORE_PLAYER", p, false);

    n("SET_CAR_DENSITY_MULTIPLIER", 1.0);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 1.0);
    n("SET_PED_DENSITY_MULTIPLIER", 1.0);
    n("SET_TIME_SCALE", 1.0);
    n("SWITCH_RANDOM_TRAINS", true);
    n("SWITCH_POLICE_HELIS", true);
    n("SWITCH_MAD_DRIVERS", false);
    n("SET_CREATE_RANDOM_COPS", true);
    n("SWITCH_RANDOM_BOATS", true);
    trainsEnabled = true;
    policeHelisEnabled = true;
    madDriversEnabled = false;
    setAggressiveMode(0);
    notify("WORLD RESET TO NORMAL");
}


// ------------------------------------------------------------
// MONEY / ACCESS / OBJECT SANDBOX
// ------------------------------------------------------------
function dropMoney(amount) {
    const pos = playerPos();
    if (!pos) {
        notify("PLAYER NOT READY");
        return;
    }

    const ped = playerPed();
    const heading = ped ? (n("GET_CHAR_HEADING", ped) || 0) : 0;
    const r = heading * Math.PI / 180.0;
    const x = pos.x + Math.sin(r) * 1.6;
    const y = pos.y + Math.cos(r) * 1.6;
    const z = pos.z + 0.15;

    n("CREATE_MONEY_PICKUP", x, y, z, amount, true);
    notify("DROPPED $" + amount);
}

function moneyScatter(count = 12, each = 5000) {
    const pos = playerPos();
    if (!pos) return;

    for (let i = 0; i < count; i++) {
        const a = Math.random() * Math.PI * 2.0;
        const d = 1.5 + Math.random() * 5.0;
        const x = pos.x + Math.cos(a) * d;
        const y = pos.y + Math.sin(a) * d;
        const z = pos.z + 0.25;
        n("CREATE_MONEY_PICKUP", x, y, z, each, true);
        wait(25);
    }

    notify("MONEY SCATTER: $" + (count * each));
}

// This is deliberately called "Access Mode", not a fake story-unlock.
// It suppresses the police response so you can travel to restricted
// islands without the normal bridge-crossing wanted response.
function setAccessModeState(enabled, showNotice = true) {
    const p = playerId();
    if (p === null) return;

    if (enabled && !accessMode && !policeDisabled) {
        const max = n("GET_MAX_WANTED_LEVEL");
        if (typeof max === "number" && max >= 0) previousMaxWantedLevel = max;
    }

    accessMode = enabled;
    n("DONT_DISPATCH_COPS_FOR_PLAYER", p, accessMode || policeDisabled);

    if (accessMode || policeDisabled) {
        n("SET_MAX_WANTED_LEVEL", 0);
        n("CLEAR_WANTED_LEVEL", p);
    } else {
        n("SET_MAX_WANTED_LEVEL", previousMaxWantedLevel);
    }

    applyPlayerIgnorePolicy();

    if (showNotice) {
        notify("ISLAND ACCESS MODE: " + (accessMode ? "ON / NO WANTED" : "OFF"));
    }
}

function toggleAccessMode() {
    setAccessModeState(!accessMode, true);
}

function spawnPlaceableObject(modelName) {
    const pos = playerPos();
    if (!pos) return;

    const model = hash(modelName);
    if (!model || !n("IS_MODEL_IN_CDIMAGE", model)) {
        notify("OBJECT NOT AVAILABLE");
        return;
    }

    if (!loadModel(model)) {
        notify("OBJECT LOAD FAILED");
        return;
    }

    const obj = n("CREATE_OBJECT", model, pos.x, pos.y, pos.z + 0.5);
    if (!obj) {
        notify("OBJECT CREATE FAILED");
        releaseModel(model);
        return;
    }

    n("SET_OBJECT_COORDINATES", obj, pos.x, pos.y, pos.z + 0.5);
    activePlacedObject = obj;
    placedObjects.push(obj);
    releaseModel(model);

    notify("OBJECT SPAWNED");
}

function objectExists(obj) {
    return obj && !!n("DOES_OBJECT_EXIST", obj);
}

function attachActiveObjectToVehicle() {
    if (!objectExists(activePlacedObject)) {
        notify("SPAWN AN OBJECT FIRST");
        return;
    }

    const car = currentCar();
    if (!car) {
        notify("GET IN VEHICLE FIRST");
        return;
    }

    n(
        "ATTACH_OBJECT_TO_CAR",
        activePlacedObject,
        car,
        0,
        objectOffset.x,
        objectOffset.y,
        objectOffset.z,
        objectOffset.rx,
        objectOffset.ry,
        objectOffset.rz
    );

    notify("OBJECT ATTACHED");
}

function detachActiveObject() {
    if (!objectExists(activePlacedObject)) {
        notify("NO ACTIVE OBJECT");
        return;
    }

    n("DETACH_OBJECT", activePlacedObject, true);
    notify("OBJECT DETACHED");
}

function deleteActiveObject() {
    if (!objectExists(activePlacedObject)) {
        activePlacedObject = 0;
        notify("NO ACTIVE OBJECT");
        return;
    }

    n("DELETE_OBJECT", activePlacedObject);
    placedObjects = placedObjects.filter(o => o !== activePlacedObject);
    activePlacedObject = 0;
    notify("OBJECT DELETED");
}

function clearPlacedObjects() {
    for (const obj of placedObjects) {
        if (objectExists(obj)) n("DELETE_OBJECT", obj);
    }
    placedObjects = [];
    activePlacedObject = 0;
    notify("PLACED OBJECTS CLEARED");
}

function changeObjectOffset(axis, delta) {
    objectOffset[axis] += delta;

    const v = Math.round(objectOffset[axis] * 100) / 100;
    notify(axis.toUpperCase() + " = " + v);
}

function resetObjectOffset() {
    objectOffset = { x: 0.0, y: -2.2, z: 0.7, rx: 0.0, ry: 0.0, rz: 0.0 };
    notify("OBJECT OFFSET RESET");
}

// ------------------------------------------------------------
// BASE GTA IV VEHICLE LISTS
// Runtime validation with IS_MODEL_IN_CDIMAGE keeps unavailable names
// from crashing the trainer.
// ------------------------------------------------------------
const VEHICLES = {
    "Sports & Super": [
        "BANSHEE","COMET","COQUETTE","FELTZER","INFERNUS",
        "SULTANRS","SUPERGT","TURISMO"
    ],
    "Muscle & Classics": [
        "BUCCANEER","DUKES","FACTION","FORTUNE","MANANA","PEYOTE",
        "SABRE","SABRE2","SABREGT","STALION","VIGERO","VIGERO2",
        "VIRGO","VOODOO"
    ],
    "Sedans": [
        "ADMIRAL","CHAVOS","COGNOSCENTI","DF8","DILETTANTE","EMPEROR",
        "ESPERANTO","FEROCI","FUTO","HAKUMAI","INTRUDER","LOKUS",
        "MARBELLA","MERIT","ORACLE","PINNACLE","PMP600","PREMIER",
        "PRIMO","SCHAFTER","SENTINEL","SOLAIR","STRATUM","SULTAN",
        "VINCENT","WASHINGTON","WILLARD"
    ],
    "SUVs & Vans": [
        "BLISTA","CAVALCADE","HABANERO","HUNTLEY","LANDSTALKER",
        "MINIVAN","MOONBEAM","PATRIOT","PERENNIAL","RANCHER","REBLA",
        "BURRITO","PONY","SPEEDO"
    ],
    "Trucks & Utility": [
        "BENSON","BIFF","BOBCAT","BOXVILLE","CONTENDER","FLATBED",
        "PACKER","PHANTOM","RIPLEY","STEED","STOCKADE","TRASH","YANKEE",
        "AIRTUG","MRTASTY"
    ],
    "Taxis & Service": [
        "BUS","CABBY","TAXI","TAXI2","STRETCH","ROMERO"
    ],
    "Emergency": [
        "AMBULANCE","FIRETRUK","NOOSE","POLICE","POLICE2","POLPATRIOT"
    ],
    "Motorcycles": [
        "BOBBER","FAGGIO","HELLFURY","NRG900","PCJ","SANCHEZ","ZOMBIEB"
    ],
    "Boats": [
        "DINGHY","JETMAX","MARQUIS","PREDATOR","REEFER","SQUALO","TUG"
    ],
    "Helicopters": [
        "ANNIHILATOR","MAVERICK","POLMAV"
    ]
};


// ------------------------------------------------------------
// V5.1 PLAYER / TRAVEL / VEHICLE / WORLD LABS
// Source-verified GTA IV natives; hardware behavior is still being tested.
// ------------------------------------------------------------
function healPlayer() {
    const ped = playerPed();
    if (!ped) return;
    n("SET_CHAR_HEALTH", ped, 200);
    notify("NIKO HEALED");
}

function armourPlayer() {
    const ped = playerPed();
    if (!ped) return;
    n("ADD_ARMOUR_TO_CHAR", ped, 100);
    notify("ARMOUR +100");
}

function togglePlayerInvincible() {
    const ped = playerPed();
    if (!ped) return;
    playerInvincible = !playerInvincible;
    n("SET_CHAR_INVINCIBLE", ped, playerInvincible);
    notify("PLAYER INVINCIBLE: " + (playerInvincible ? "ON" : "OFF"));
}

function togglePlayerInvisible() {
    const ped = playerPed();
    if (!ped) return;
    playerInvisible = !playerInvisible;
    n("SET_CHAR_VISIBLE", ped, !playerInvisible);
    notify("PLAYER INVISIBLE: " + (playerInvisible ? "ON" : "OFF"));
}

function togglePlayerFrozen() {
    const ped = playerPed();
    if (!ped) return;
    playerFrozen = !playerFrozen;
    n("FREEZE_CHAR_POSITION", ped, playerFrozen);
    notify("PLAYER FREEZE: " + (playerFrozen ? "ON" : "OFF"));
}

function setPlayerGravity(value, label) {
    const ped = playerPed();
    if (!ped) return;
    n("SET_CHAR_GRAVITY", ped, value);
    notify("PLAYER GRAVITY: " + label);
}

function toggleWindscreenEjection() {
    const ped = playerPed();
    if (!ped) return;
    playerWindscreenEject = !playerWindscreenEject;
    n("SET_CHAR_WILL_FLY_THROUGH_WINDSCREEN", ped, playerWindscreenEject);
    notify("WINDSCREEN EJECTION: " + (playerWindscreenEject ? "ON" : "OFF"));
}

function toggleEveryoneIgnorePlayer() {
    everyoneIgnorePlayer = !everyoneIgnorePlayer;

    if (everyoneIgnorePlayer) {
        // Our old Aggressive Peds feature issues TASK_COMBAT directly at
        // Niko. A global ignore flag cannot cancel that scripted task.
        aggressiveMode = 0;
        clearAggroPeds();
    }

    applyPlayerIgnorePolicy();
    notify("EVERYONE IGNORE PLAYER: " + (everyoneIgnorePlayer ? "ON" : "OFF"));
}

function saveCurrentPosition() {
    const pos = playerPos();
    if (!pos) return;
    savedPlayerPos = { x: pos.x, y: pos.y, z: pos.z };
    notify("POSITION SAVED");
}

function teleportEntityTo(x, y, z) {
    const car = currentCar();
    if (car) {
        n("SET_CAR_COORDINATES", car, x, y, z);
        return;
    }
    const ped = playerPed();
    if (ped) n("SET_CHAR_COORDINATES", ped, x, y, z);
}

function returnToSavedPosition() {
    if (!savedPlayerPos) {
        notify("NO SAVED POSITION");
        return;
    }
    teleportEntityTo(savedPlayerPos.x, savedPlayerPos.y, savedPlayerPos.z);
    notify("RETURNED TO SAVED POSITION");
}

function nudgePlayer(dx, dy, dz, label) {
    const pos = playerPos();
    if (!pos) return;
    teleportEntityTo(pos.x + dx, pos.y + dy, pos.z + dz);
    notify("NUDGE: " + label);
}

// Researched exterior GTA IV teleport coordinates.
// The first 10 come from a GTAForums static-teleport example used by IV
// modders; Roman's and the Alderney/Niko house point come from a public
// GTA IV coordinate dataset. Keep these outdoors to reduce interior issues.
const TELEPORT_POINTS = [
    { label: "AIRPORT HELIPAD", x: 2242.19, y: 727.24, z: 5.91 },
    { label: "BROKER BEACH", x: 1084.53, y: -698.65, z: 14.70 },
    { label: "CHARGE ISLAND", x: 549.92, y: 854.03, z: 20.99 },
    { label: "HELITOURS", x: 380.20, y: -716.62, z: 4.69 },
    { label: "STAR JUNCTION", x: -204.06, y: 264.04, z: 15.02 },
    { label: "MIDDLE PARK", x: -235.67, y: 739.12, z: 7.26 },
    { label: "BASEBALL PARK", x: 708.00, y: 1919.57, z: 27.16 },
    { label: "HAPPINESS ISLAND", x: -556.62, y: -905.11, z: 4.99 },
    { label: "ALDERNEY PRISON", x: -1077.03, y: -466.85, z: 2.26 },
    { label: "SULTAN RS AREA", x: -951.60, y: 1869.55, z: 22.58 },
    { label: "ROMAN'S / HOVE BEACH", x: 1139.18, y: -11.99, z: 15.05 },
    { label: "ALDERNEY SAFEHOUSE AREA", x: -724.71, y: 1390.01, z: 13.68 }
];

function teleportToNamedPoint(point) {
    if (!point) return;

    // Ask GTA IV to stream the destination before moving the player/car.
    n("LOAD_SCENE", point.x, point.y, point.z);
    wait(50);

    const car = currentCar();
    teleportEntityTo(point.x, point.y, point.z);

    if (car) {
        wait(25);
        n("SET_CAR_ON_GROUND_PROPERLY", car);
    }

    notify("TELEPORT: " + point.label);
}

function withCurrentCar(fn) {
    const car = currentCar();
    if (!car) {
        notify("GET IN A VEHICLE FIRST");
        return;
    }
    fn(car);
}

function toggleCarDamage() {
    carDamageEnabled = !carDamageEnabled;
    withCurrentCar(car => n("SET_CAR_CAN_BE_DAMAGED", car, carDamageEnabled));
    notify("CAR DAMAGE: " + (carDamageEnabled ? "ON" : "OFF"));
}

function toggleCarVisibleDamage() {
    carVisibleDamageEnabled = !carVisibleDamageEnabled;
    withCurrentCar(car => n("SET_CAR_CAN_BE_VISIBLY_DAMAGED", car, carVisibleDamageEnabled));
    notify("VISIBLE DAMAGE: " + (carVisibleDamageEnabled ? "ON" : "OFF"));
}

function toggleCarProofs() {
    carProofsEnabled = !carProofsEnabled;
    withCurrentCar(car => n(
        "SET_CAR_PROOFS",
        car,
        carProofsEnabled,
        carProofsEnabled,
        carProofsEnabled,
        carProofsEnabled,
        carProofsEnabled
    ));
    notify("FULL CAR PROOFS: " + (carProofsEnabled ? "ON" : "OFF"));
}

function toggleCarStrong() {
    carStrongEnabled = !carStrongEnabled;
    withCurrentCar(car => n("SET_CAR_STRONG", car, carStrongEnabled));
    notify("STRONG COLLISIONS: " + (carStrongEnabled ? "ON" : "OFF"));
}

function toggleCarWatertight() {
    carWatertightEnabled = !carWatertightEnabled;
    withCurrentCar(car => n("SET_CAR_WATERTIGHT", car, carWatertightEnabled));
    notify("WATERTIGHT: " + (carWatertightEnabled ? "ON" : "OFF"));
}

function toggleCarInvisible() {
    carInvisible = !carInvisible;
    withCurrentCar(car => n("SET_CAR_VISIBLE", car, !carInvisible));
    notify("INVISIBLE VEHICLE: " + (carInvisible ? "ON" : "OFF"));
}

function toggleCarFrozen() {
    carFrozen = !carFrozen;
    withCurrentCar(car => n("FREEZE_CAR_POSITION", car, carFrozen));
    notify("VEHICLE FREEZE: " + (carFrozen ? "ON" : "OFF"));
}

function uprightCurrentCar() {
    withCurrentCar(car => n("SET_CAR_ON_GROUND_PROPERLY", car));
    notify("VEHICLE UPRIGHT");
}

function setDoorLock(status, label) {
    withCurrentCar(car => n("LOCK_CAR_DOORS", car, status));
    notify("DOORS: " + label);
}

function setForwardSpeed(speed, label) {
    withCurrentCar(car => n("SET_CAR_FORWARD_SPEED", car, speed));
    notify("FORWARD SPEED: " + label);
}

function setCarTraction(value, label) {
    withCurrentCar(car => n("SET_CAR_TRACTION", car, value));
    notify("TRACTION: " + label);
}

function setCarLightMultiplier(value, label) {
    withCurrentCar(car => n("SET_CAR_LIGHT_MULTIPLIER", car, value));
    notify("LIGHT MULTIPLIER: " + label);
}

function toggleMadDrivers() {
    madDriversEnabled = !madDriversEnabled;
    n("SWITCH_MAD_DRIVERS", madDriversEnabled);
    notify("MAD DRIVERS: " + (madDriversEnabled ? "ON" : "OFF"));
}

function setWind(value, label) {
    n("FORCE_WIND", value);
    notify("WIND: " + label);
}

// ------------------------------------------------------------
// V5.3 WEATHER
// GTA IV's FORCE_WEATHER/FORCE_WEATHER_NOW take integer weather IDs.
// ------------------------------------------------------------
const WEATHER_TYPES = [
    { id: 0, label: "EXTRA SUNNY" },
    { id: 1, label: "SUNNY" },
    { id: 2, label: "SUNNY / WINDY" },
    { id: 3, label: "CLOUDY" },
    { id: 4, label: "RAIN" },
    { id: 5, label: "DRIZZLE" },
    { id: 6, label: "FOGGY" },
    { id: 7, label: "STORM / HEAVY RAIN" }
];

function forceWeatherType(id, label) {
    forcedWeather = id;
    n("FORCE_WEATHER_NOW", id);
    n("FORCE_WEATHER", id);
    notify("WEATHER: " + label);
}

function releaseWeatherControl() {
    forcedWeather = -1;
    n("RELEASE_WEATHER");
    notify("WEATHER: GAME CONTROL");
}

function pickRandomWeather() {
    forcedWeather = -1;
    n("RELEASE_WEATHER");
    n("PICK_RANDOM_WEATHER");
    notify("WEATHER: RANDOM");
}

// ------------------------------------------------------------
// V5.3 POLICE MASTER CONTROL
// ------------------------------------------------------------
function clearNearbyCops(radius = 120.0, showNotice = true) {
    const pos = playerPos();
    if (!pos) return;
    n("CLEAR_AREA_OF_COPS", pos.x, pos.y, pos.z, radius);
    if (showNotice) notify("COPS CLEARED: " + radius + "m");
}

function setPoliceDisabled(enabled, showNotice = true) {
    const p = playerId();
    if (p === null) return;

    if (enabled && !policeDisabled) {
        const max = n("GET_MAX_WANTED_LEVEL");
        if (typeof max === "number" && max >= 0) previousMaxWantedLevel = max;

        const randomCops = n("GET_CREATE_RANDOM_COPS");
        if (typeof randomCops === "boolean") previousCreateRandomCops = randomCops;
    }

    policeDisabled = enabled;

    if (policeDisabled) {
        clearWarEmsResponse(true);
        clearWarPoliceResponse(true);
        if (streetWarState === "AFTERMATH") warEmsSceneComplete = true;
        n("SET_CREATE_RANDOM_COPS", false);
        n("DONT_DISPATCH_COPS_FOR_PLAYER", p, true);
        n("SET_MAX_WANTED_LEVEL", 0);
        n("CLEAR_WANTED_LEVEL", p);
        n("SWITCH_POLICE_HELIS", false);
        clearNearbyCops(140.0, false);
    } else {
        n("SET_CREATE_RANDOM_COPS", previousCreateRandomCops);
        n("DONT_DISPATCH_COPS_FOR_PLAYER", p, accessMode);

        if (accessMode) {
            n("SET_MAX_WANTED_LEVEL", 0);
            n("CLEAR_WANTED_LEVEL", p);
        } else {
            n("SET_MAX_WANTED_LEVEL", previousMaxWantedLevel);
        }

        n("SWITCH_POLICE_HELIS", policeHelisEnabled);
    }

    applyPlayerIgnorePolicy();

    if (showNotice) {
        notify("POLICE SYSTEM: " + (policeDisabled ? "DISABLED" : "ENABLED"));
    }
}

function togglePoliceDisabled() {
    setPoliceDisabled(!policeDisabled, true);
}

// ------------------------------------------------------------
// V5.3 RANDOM PED SPAWNER
// ------------------------------------------------------------
function pedSpawnPosition(distance = 3.0, side = 0.0) {
    const ped = playerPed();
    const pos = playerPos();
    if (!ped || !pos) return null;

    const heading = n("GET_CHAR_HEADING", ped) || 0.0;
    const r = heading * Math.PI / 180.0;

    return {
        x: pos.x + Math.sin(r) * distance + Math.cos(r) * side,
        y: pos.y + Math.cos(r) * distance - Math.sin(r) * side,
        z: pos.z + 0.20
    };
}

function pruneSpawnedPeds() {
    spawnedPeds = spawnedPeds.filter(p =>
        p && !!n("DOES_CHAR_EXIST", p) && !n("IS_CHAR_DEAD", p)
    );
}

function trackSpawnedPed(ped) {
    if (!ped) return;
    lastSpawnedPed = ped;
    spawnedPeds.push(ped);

    while (spawnedPeds.length > MAX_SPAWNED_PEDS) {
        const old = spawnedPeds.shift();
        if (old && n("DOES_CHAR_EXIST", old) && !bodyguards.includes(old)) {
            n("CLEAR_CHAR_TASKS", old);
            n("TASK_WANDER_STANDARD", old);
            n("MARK_CHAR_AS_NO_LONGER_NEEDED", old);
        }
    }
}

function createRandomPedNear(kind = "ANY", track = true) {
    const p = pedSpawnPosition(3.0 + Math.random() * 2.0, (Math.random() - 0.5) * 3.0);
    if (!p) return 0;

    let ped = 0;
    if (kind === "MALE") {
        ped = n("CREATE_RANDOM_MALE_CHAR", p.x, p.y, p.z) || 0;
    } else if (kind === "FEMALE") {
        ped = n("CREATE_RANDOM_FEMALE_CHAR", p.x, p.y, p.z) || 0;
    } else {
        ped = n("CREATE_RANDOM_CHAR", p.x, p.y, p.z) || 0;
    }

    if (!ped || !n("DOES_CHAR_EXIST", ped)) {
        notify("RANDOM PED CREATE FAILED");
        return 0;
    }

    n("SET_CHAR_AS_MISSION_CHAR", ped);
    n("SET_CHAR_RANDOM_COMPONENT_VARIATION", ped);

    lastSpawnedPed = ped;

    if (track) {
        n("TASK_WANDER_STANDARD", ped);
        trackSpawnedPed(ped);
    }

    return ped;
}

function spawnRandomPed(kind = "ANY") {
    const ped = createRandomPedNear(kind, true);
    if (ped) notify("SPAWNED RANDOM " + kind + " PED");
}

function spawnRandomPedGroup(count, kind = "ANY") {
    let created = 0;
    for (let i = 0; i < count; i++) {
        if (createRandomPedNear(kind, true)) created++;
        wait(60);
    }
    notify("SPAWNED PEDS: " + created + "/" + count);
}

function recruitRandomPedAsGuard() {
    if (bodyguardCount() >= MAX_BODYGUARDS) {
        notify("SQUAD FULL: 10/10");
        return;
    }

    const ped = createRandomPedNear("ANY", false);
    if (!ped) return;

    bodyguards.push(ped);
    const grouped = configureBodyguard(ped);
    if (!grouped) {
        const me = playerPed();
        if (me) n("TASK_GOTO_CHAR_OFFSET", ped, me, -1, 0.0, -2.0);
    }

    notify("RANDOM PED RECRUITED: " + bodyguardCount() + "/10");
}

function fillCurrentVehicleWithRandomPassengers() {
    const car = currentCar();
    if (!car) {
        notify("GET IN A VEHICLE FIRST");
        return;
    }

    let maxPassengers = n("GET_MAXIMUM_NUMBER_OF_PASSENGERS", car);
    if (typeof maxPassengers !== "number") maxPassengers = 3;

    let added = 0;
    for (let seat = 0; seat < maxPassengers; seat++) {
        if (!n("IS_CAR_PASSENGER_SEAT_FREE", car, seat)) continue;

        const ped = n("CREATE_RANDOM_CHAR_AS_PASSENGER", car, seat) || 0;
        if (ped && n("DOES_CHAR_EXIST", ped)) {
            n("SET_CHAR_AS_MISSION_CHAR", ped);
            trackSpawnedPed(ped);
            added++;
            wait(40);
        }
    }

    notify("RANDOM PASSENGERS: " + added);
}

function clearSpawnedPeds() {
    pruneSpawnedPeds();

    for (const ped of spawnedPeds) {
        if (!ped || !n("DOES_CHAR_EXIST", ped) || bodyguards.includes(ped)) continue;
        n("CLEAR_CHAR_TASKS", ped);
        n("TASK_WANDER_STANDARD", ped);
        n("MARK_CHAR_AS_NO_LONGER_NEEDED", ped);
    }

    spawnedPeds = [];
    lastSpawnedPed = 0;
    notify("SPAWNED PEDS RELEASED");
}

function panicLastSpawnedPed() {
    if (!lastSpawnedPed || !n("DOES_CHAR_EXIST", lastSpawnedPed)) {
        notify("NO SPAWNED PED");
        return;
    }
    n("PANIC_SCREAM", lastSpawnedPed);
    notify("LAST PED: PANIC SCREAM");
}

// ------------------------------------------------------------
// V5.3 NATIVE RESCUE LAB
// Small, verified GTA IV natives that conventional trainer menus often skip.
// ------------------------------------------------------------
function toggleHazardLights() {
    hazardLightsEnabled = !hazardLightsEnabled;
    withCurrentCar(car => n("SET_VEH_HAZARDLIGHTS", car, hazardLightsEnabled));
    notify("HAZARD LIGHTS: " + (hazardLightsEnabled ? "ON" : "OFF"));
}

function toggleInteriorLight() {
    interiorLightEnabled = !interiorLightEnabled;
    withCurrentCar(car => n("SET_VEH_INTERIORLIGHT", car, interiorLightEnabled));
    notify("INTERIOR LIGHT: " + (interiorLightEnabled ? "ON" : "OFF"));
}

function toggleStrongAxles() {
    strongAxlesEnabled = !strongAxlesEnabled;
    withCurrentCar(car => n("SET_VEH_HAS_STRONG_AXLES", car, strongAxlesEnabled));
    notify("STRONG AXLES: " + (strongAxlesEnabled ? "ON" : "OFF"));
}

function soundCurrentHorn() {
    withCurrentCar(car => n("SOUND_CAR_HORN", car, 1800));
    notify("HORN: 1.8 SEC");
}

function setHeliThrust(power) {
    const ped = playerPed();
    if (!ped || !n("IS_CHAR_IN_ANY_HELI", ped)) {
        notify("GET IN A HELICOPTER FIRST");
        return;
    }

    const heli = currentCar();
    if (!heli) return;

    n("ACTIVATE_HELI_SPEED_CHEAT", heli, power);
    notify("HELI EXTRA THRUST: " + power);
}

function toggleRandomBoats() {
    randomBoatsEnabled = !randomBoatsEnabled;
    n("SWITCH_RANDOM_BOATS", randomBoatsEnabled);
    notify("RANDOM BOATS: " + (randomBoatsEnabled ? "ON" : "OFF"));
}


// ------------------------------------------------------------
// V6.0 ACTIVE PED / PED CONTROL
// ------------------------------------------------------------
function validActivePed() {
    return activePed && !!n("DOES_CHAR_EXIST", activePed) && !n("IS_CHAR_DEAD", activePed);
}

function selectNearestPed() {
    const pos = playerPos();
    const me = playerPed();
    if (!pos || !me) return;

    const ped = n("GET_CLOSEST_CHAR", pos.x, pos.y, pos.z, 30.0, 1, 1) || 0;
    if (!ped || ped === me || !n("DOES_CHAR_EXIST", ped)) {
        activePed = 0;
        notify("ACTIVE PED: NONE FOUND");
        return;
    }
    activePed = ped;
    n("SET_CHAR_AS_MISSION_CHAR", activePed);
    notify("ACTIVE PED SELECTED");
}

function withActivePed(fn) {
    if (!validActivePed()) {
        activePed = 0;
        notify("SELECT ACTIVE PED FIRST");
        return;
    }
    fn(activePed);
}

function activePedFollowMe() {
    withActivePed(ped => {
        if (ensureSquadGroup()) {
            n("SET_GROUP_MEMBER", squadGroup, ped);
        } else {
            n("TASK_GOTO_CHAR_OFFSET", ped, playerPed(), -1, 1.2, -1.6);
        }
    });
    notify("ACTIVE PED: FOLLOW ME");
}

function activePedGuardMe() {
    withActivePed(ped => {
        n("SET_CHAR_WILL_USE_COVER", ped, true);
        n("SET_CHAR_WILL_USE_CARS_IN_COMBAT", ped, true);
        n("TASK_COMBAT_HATED_TARGETS_AROUND_CHAR", ped, 60.0);
    });
    notify("ACTIVE PED: GUARD / COMBAT NEARBY");
}

function activePedWander() { withActivePed(p => { n("CLEAR_CHAR_TASKS", p); n("TASK_WANDER_STANDARD", p); }); notify("ACTIVE PED: WANDER"); }
function activePedStand() { withActivePed(p => { n("CLEAR_CHAR_TASKS", p); n("TASK_STAND_STILL", p, 600000); }); notify("ACTIVE PED: STAND STILL"); }
function activePedFlee() { withActivePed(p => n("TASK_SMART_FLEE_CHAR", p, playerPed(), 120.0, 60000)); notify("ACTIVE PED: FLEE"); }
function activePedHandsUp() { withActivePed(p => n("TASK_HANDS_UP", p, 30000)); notify("ACTIVE PED: HANDS UP"); }
function activePedPanic() { withActivePed(p => n("PANIC_SCREAM", p)); notify("ACTIVE PED: PANIC SCREAM"); }

function activePedGetInMyCar() {
    const car = currentCar();
    if (!car) { notify("GET IN A VEHICLE FIRST"); return; }
    withActivePed(ped => {
        let max = n("GET_MAXIMUM_NUMBER_OF_PASSENGERS", car);
        if (typeof max !== "number") max = 3;
        for (let seat = 0; seat < max; seat++) {
            if (n("IS_CAR_PASSENGER_SEAT_FREE", car, seat)) {
                n("TASK_ENTER_CAR_AS_PASSENGER", ped, car, 12000, seat);
                return;
            }
        }
        notify("NO FREE PASSENGER SEAT");
    });
}
function activePedGetOut() { withActivePed(p => n("TASK_LEAVE_ANY_CAR", p)); notify("ACTIVE PED: EXIT VEHICLE"); }
function activePedAttackMe() { withActivePed(p => n("TASK_COMBAT", p, playerPed())); notify("ACTIVE PED: ATTACK PLAYER"); }
function activePedAttackNearbyPed() {
    withActivePed(p => {
        const q = n("GET_CHAR_COORDINATES", p);
        if (!q) return;
        const target = n("GET_RANDOM_CHAR_IN_AREA_OFFSET_NO_SAVE", q.x-18.0,q.y-18.0,q.z-6.0,36.0,36.0,12.0) || 0;
        if (target && target !== p && target !== playerPed() && n("DOES_CHAR_EXIST", target)) n("TASK_COMBAT", p, target);
        else notify("NO NEARBY PED TARGET");
    });
}
function activePedGiveWeapon(id,label) { withActivePed(p => { n("GIVE_WEAPON_TO_CHAR", p, id, 600, true); n("SET_CURRENT_CHAR_WEAPON", p, id, true); }); notify("ACTIVE PED WEAPON: "+label); }
function activePedRemoveWeapons() { withActivePed(p => n("REMOVE_ALL_CHAR_WEAPONS", p)); notify("ACTIVE PED: DISARMED"); }
function activePedHeal() { withActivePed(p => { n("SET_CHAR_MAX_HEALTH", p, 500); n("SET_CHAR_HEALTH", p, 500); n("ADD_ARMOUR_TO_CHAR", p, 100); }); notify("ACTIVE PED: HEALED"); }
function toggleActivePedInvincible() { activePedInvincible=!activePedInvincible; withActivePed(p=>n("SET_CHAR_INVINCIBLE",p,activePedInvincible)); notify("ACTIVE PED INVINCIBLE: "+(activePedInvincible?"ON":"OFF")); }
function activePedRagdoll() { withActivePed(p => n("SWITCH_PED_TO_RAGDOLL", p, 1000, 3500, true, true, true, true)); notify("ACTIVE PED: RAGDOLL"); }
function toggleActivePedDrunk() { activePedDrunk=!activePedDrunk; withActivePed(p=>n("SET_PED_IS_DRUNK",p,activePedDrunk)); notify("ACTIVE PED DRUNK: "+(activePedDrunk?"ON":"OFF")); }
function toggleActivePedBleed() { activePedBleeding=!activePedBleeding; withActivePed(p=>n("SET_CHAR_BLEEDING",p,activePedBleeding)); notify("ACTIVE PED BLEEDING: "+(activePedBleeding?"ON":"OFF")); }
function deleteActivePed() { withActivePed(p => { n("REMOVE_CHAR_FROM_GROUP", p); n("DELETE_CHAR", p); }); activePed=0; notify("ACTIVE PED DELETED"); }

// ------------------------------------------------------------
// V6.0 PED COMBAT LAB
// ------------------------------------------------------------
function setActivePedAccuracy(v) { activePedAccuracy=v; withActivePed(p=>n("SET_CHAR_ACCURACY",p,v)); notify("PED ACCURACY: "+v); }
function setActivePedShootRate(v) { activePedShootRate=v; withActivePed(p=>n("SET_CHAR_SHOOT_RATE",p,v)); notify("PED SHOOT RATE: "+v); }
function toggleActivePedCover() { activePedCover=!activePedCover; withActivePed(p=>n("SET_CHAR_WILL_USE_COVER",p,activePedCover)); notify("PED USE COVER: "+(activePedCover?"ON":"OFF")); }
function toggleActivePedCarsCombat() { activePedCarsCombat=!activePedCarsCombat; withActivePed(p=>n("SET_CHAR_WILL_USE_CARS_IN_COMBAT",p,activePedCarsCombat)); notify("PED CARS IN COMBAT: "+(activePedCarsCombat?"ON":"OFF")); }
function activePedCombatNearby() { withActivePed(p=>n("TASK_COMBAT_HATED_TARGETS_AROUND_CHAR",p,100.0)); notify("PED COMBAT NEARBY"); }
function activePedOnlyClearLos() { withActivePed(p=>n("SET_CHAR_WILL_ONLY_FIRE_WITH_CLEAR_LOS",p,true)); notify("PED CLEAR LOS ONLY"); }
function activePedAllowBlindFire() { withActivePed(p=>n("SET_CHAR_WILL_ONLY_FIRE_WITH_CLEAR_LOS",p,false)); notify("PED CLEAR LOS ONLY: OFF"); }

// ------------------------------------------------------------
// V6.0 VEHICLE SELECTOR / PHYSICS
// ------------------------------------------------------------
function nearestVehicle() {
    const pos = playerPos();
    if (!pos) return 0;
    return n("GET_CLOSEST_CAR", pos.x, pos.y, pos.z, 40.0, 0, 70) || 0;
}
function selectedVehicle() {
    if (selectedVehicleMode === 0) return currentCar();
    if (selectedVehicleMode === 1) return nearestVehicle();
    if (lastSpawnedVehicle && n("DOES_VEHICLE_EXIST", lastSpawnedVehicle) && !n("IS_CAR_DEAD", lastSpawnedVehicle)) return lastSpawnedVehicle;
    return 0;
}
function withSelectedVehicle(fn) { const car=selectedVehicle(); if(!car){notify("SELECTED VEHICLE NOT AVAILABLE");return;} fn(car); }
function setVehicleSelector(mode,label){selectedVehicleMode=mode;notify("VEHICLE TARGET: "+label);}
function setSelectedTraction(v){withSelectedVehicle(c=>n("SET_CAR_TRACTION",c,v));notify("LIVE TRACTION: "+v);}
function setSelectedEngineHealth(v){withSelectedVehicle(c=>n("SET_ENGINE_HEALTH",c,v));notify("ENGINE HEALTH: "+v);}
function setSelectedTankHealth(v){withSelectedVehicle(c=>n("SET_PETROL_TANK_HEALTH",c,v));notify("TANK HEALTH: "+v);}
function burstSelectedTyre(i){withSelectedVehicle(c=>n("BURST_CAR_TYRE",c,i));notify("TYRE "+i+" BURST");}
function fixSelectedTyre(i){withSelectedVehicle(c=>n("FIX_CAR_TYRE",c,i));notify("TYRE "+i+" FIXED");}
function fixAllSelectedTyres(){withSelectedVehicle(c=>{for(let i=0;i<4;i++)n("FIX_CAR_TYRE",c,i);});notify("TYRES 0-3 FIXED");}
function toggleSelectedHydraulics(){selectedVehicleHydraulics=!selectedVehicleHydraulics;withSelectedVehicle(c=>n("SET_CAR_HYDRAULICS",c,selectedVehicleHydraulics));notify("HYDRAULICS: "+(selectedVehicleHydraulics?"ON":"OFF"));}
function toggleSelectedSkids(){selectedVehicleSkids=!selectedVehicleSkids;withSelectedVehicle(c=>n("SET_CAR_ALWAYS_CREATE_SKIDS",c,selectedVehicleSkids));notify("ALWAYS CREATE SKIDS: "+(selectedVehicleSkids?"ON":"OFF"));}
function toggleSelectedOnlyPlayerDamage(){selectedVehicleOnlyPlayerDamage=!selectedVehicleOnlyPlayerDamage;withSelectedVehicle(c=>n("SET_CAR_ONLY_DAMAGED_BY_PLAYER",c,selectedVehicleOnlyPlayerDamage));notify("ONLY PLAYER DAMAGE: "+(selectedVehicleOnlyPlayerDamage?"ON":"OFF"));}
function selectedVehicleHeavy(on){withSelectedVehicle(c=>n("SET_CAR_HEAVY",c,on));notify("CAR HEAVY [EXP]: "+(on?"ON":"OFF"));}

// ------------------------------------------------------------
// V6.0 VEHICLE DOORS & LIGHTS
// ------------------------------------------------------------
function toggleAllIndicators(){allIndicatorsEnabled=!allIndicatorsEnabled;withSelectedVehicle(c=>n("SET_VEH_INDICATORLIGHTS",c,allIndicatorsEnabled));notify("ALL INDICATORS: "+(allIndicatorsEnabled?"ON":"OFF"));}
function selectedHazards(on){withSelectedVehicle(c=>n("SET_VEH_HAZARDLIGHTS",c,on));notify("HAZARDS: "+(on?"ON":"OFF"));}
function selectedInteriorLight(on){withSelectedVehicle(c=>n("SET_VEH_INTERIORLIGHT",c,on));notify("INTERIOR LIGHT: "+(on?"ON":"OFF"));}
function selectedForceLights(mode,label){withSelectedVehicle(c=>n("FORCE_CAR_LIGHTS",c,mode));notify("CAR LIGHTS: "+label);}
function selectedOpenDoor(i,label){withSelectedVehicle(c=>n("OPEN_CAR_DOOR",c,i));notify("OPEN: "+label);}
function selectedShutDoor(i,label){withSelectedVehicle(c=>n("SHUT_CAR_DOOR",c,i));notify("SHUT: "+label);}
function selectedCloseAllDoors(){withSelectedVehicle(c=>n("CLOSE_ALL_CAR_DOORS",c));notify("ALL DOORS CLOSED");}
function selectedPopBoot(){withSelectedVehicle(c=>n("POP_CAR_BOOT",c));notify("BOOT / TRUNK POPPED");}
function selectedAlarm(){withSelectedVehicle(c=>{n("SET_VEH_ALARM",c,true);n("TRIGGER_VEH_ALARM",c);});notify("VEHICLE ALARM TRIGGERED");}

// ------------------------------------------------------------
// V6.0 TRAFFIC CONTROL
// ------------------------------------------------------------
function setTrafficProfile(car,random,parked,ped,label){
    n("SET_CAR_DENSITY_MULTIPLIER",car);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER",random);
    n("SET_PARKED_CAR_DENSITY_MULTIPLIER",parked);
    n("SET_PED_DENSITY_MULTIPLIER",ped);
    notify("TRAFFIC PROFILE: "+label);
}
function trafficStop(){setTrafficProfile(0.0,0.0,0.0,0.4,"STOPPED");}
function trafficLow(){setTrafficProfile(0.35,0.35,0.5,0.7,"LOW");}
function trafficNormal(){setTrafficProfile(1.0,1.0,1.0,1.0,"NORMAL");}
function trafficHeavy(){setTrafficProfile(2.5,2.5,2.0,1.4,"HEAVY");}
function trafficInsane(){setTrafficProfile(5.0,5.0,3.0,1.8,"INSANE [EXP]");}
function parkedCars(v,label){n("SET_PARKED_CAR_DENSITY_MULTIPLIER",v);notify("PARKED CARS: "+label);}

// ------------------------------------------------------------
// V6.0 DIRECTOR MODE - composed scenarios
// ------------------------------------------------------------
function directorGangWar(){setStreetWar(16,"DIRECTOR GANG WAR 8v8"); streetWarCarjackers=true; streetWarPlayerNeutral=true;}
function directorPoliceSiege(){if(policeDisabled)setPoliceDisabled(false,false);setAggressiveMode(0);n("SET_CREATE_RANDOM_COPS",true);n("SWITCH_POLICE_HELIS",true);policeHelisEnabled=true;setWanted(6);setTrafficProfile(0.75,0.75,0.8,1.1,"POLICE SIEGE");}
function directorHighwayChaos(){n("SWITCH_MAD_DRIVERS",true);madDriversEnabled=true;setTrafficProfile(3.25,3.25,1.0,0.9,"HIGHWAY CHAOS");}
function directorNooseAssault(){if(policeDisabled)setPoliceDisabled(false,false);n("SET_CREATE_RANDOM_COPS",true);n("SWITCH_POLICE_HELIS",true);setWanted(6);notify("DIRECTOR: MAX POLICE RESPONSE");}
function directorRiot(){setAggressiveMode(2);setTrafficProfile(0.6,0.6,0.8,2.0,"RIOT");}
function directorCarjackingWave(){setStreetWar(16,"CARJACKING WAVE 8v8");streetWarCarjackers=true;streetWarDriveSpeed=32.0;}
function directorConvoyAttack(){setStreetWar(10,"ARMED ROAD AMBUSH 5v5");streetWarCarjackers=true;n("SWITCH_MAD_DRIVERS",true);setTrafficProfile(2.0,2.0,1.0,1.2,"ARMED ROAD AMBUSH");}
function directorBodyguardEscort(){recruitBodyguards(5);presetRifleTeam();squadRegroup();notify("DIRECTOR: BODYGUARD ESCORT");}
function directorStormShootout(){forceWeatherType(7,"STORM / HEAVY RAIN");n("FORCE_WIND",10.0);setStreetWar(12,"STORM SHOOTOUT 6v6");}
function directorCityPanic(){forceWeatherType(7,"STORM / HEAVY RAIN");n("FORCE_WIND",25.0);setTrafficProfile(0.45,0.45,0.5,1.8,"CITY PANIC");spawnRandomPedGroup(10,"ANY");for(const p of spawnedPeds){if(p&&n("DOES_CHAR_EXIST",p))n("PANIC_SCREAM",p);}setAggressiveMode(2);notify("DIRECTOR: CITY PANIC [EXP]");}
function directorReset(){normalWorld();dismissBodyguards();notify("DIRECTOR RESET / NORMAL WORLD");}

// ------------------------------------------------------------
// V6.0 NATIVE RESEARCH NR-xxx
// ------------------------------------------------------------
function nrHeadDamage(on){withActivePed(p=>n("SET_PED_FORCE_VISUALISE_HEAD_DAMAGE_FROM_BULLETS",p,on));notify("NR-301 HEAD DAMAGE VIS: "+(on?"ON":"OFF"));}
function nrStunReady(on){withActivePed(p=>n("SET_CHAR_READY_TO_BE_STUNNED",p,on));notify("NR-302 STUN READY: "+(on?"ON":"OFF"));}
function nrBlockCowerCover(on){withActivePed(p=>n("BLOCK_COWERING_IN_COVER",p,on));notify("NR-303 BLOCK COWER COVER: "+(on?"ON":"OFF"));}
function nrBlockPeekCover(on){withActivePed(p=>n("BLOCK_PEEKING_IN_COVER",p,on));notify("NR-304 BLOCK PEEK COVER: "+(on?"ON":"OFF"));}
function nrNoEvasiveDive(on){withActivePed(p=>n("SET_PED_DONT_DO_EVASIVE_DIVES",p,on));notify("NR-305 NO EVASIVE DIVES: "+(on?"ON":"OFF"));}
function nrMovementAnimsBlocked(on){withActivePed(p=>n("SET_CHAR_MOVEMENT_ANIMS_BLOCKED",p,on));notify("NR-306 MOVE ANIMS BLOCKED: "+(on?"ON":"OFF"));}
function nrLadders(on){withActivePed(p=>n("SET_PED_PATH_MAY_USE_LADDERS",p,on));notify("NR-307 MAY USE LADDERS: "+(on?"ON":"OFF"));}
function nrClimbovers(on){withActivePed(p=>n("SET_PED_PATH_MAY_USE_CLIMBOVERS",p,on));notify("NR-308 MAY CLIMB: "+(on?"ON":"OFF"));}
function nrDropFromHeight(on){withActivePed(p=>n("SET_PED_PATH_MAY_DROP_FROM_HEIGHT",p,on));notify("NR-309 MAY DROP HEIGHT: "+(on?"ON":"OFF"));}
function nrHeavyCar(on){selectedVehicleHeavy(on);}

// ------------------------------------------------------------
// MENU DEFINITION HELPERS
// ------------------------------------------------------------
function action(label, fn) {
    return { label, action: fn };
}

function submenu(label, getter) {
    return { label, submenu: getter };
}

function backItem() {
    return action("< BACK", () => goBack());
}


function moneyMenu() {
    return {
        title: "MONEY DROPS",
        items: [
            action("DROP $1,000", () => dropMoney(1000)),
            action("DROP $10,000", () => dropMoney(10000)),
            action("DROP $20,000", () => dropMoney(20000)),
            action("DROP $50,000", () => dropMoney(50000)),
            action("DROP $100,000", () => dropMoney(100000)),
            action("SCATTER $60,000", () => moneyScatter(12, 5000)),
            backItem()
        ]
    };
}

function cargoPresetMenu() {
    const items = CARGO_OBJECTS.map(entry =>
        action(entry.label, () => spawnPlaceableObject(entry.model))
    );
    items.push(backItem());
    return { title: "CARGO OBJECTS", items };
}

function objectOffsetMenu() {
    return {
        title: "OBJECT POSITION",
        items: [
            action("X -0.10", () => changeObjectOffset("x", -0.10)),
            action("X +0.10", () => changeObjectOffset("x", 0.10)),
            action("Y -0.10", () => changeObjectOffset("y", -0.10)),
            action("Y +0.10", () => changeObjectOffset("y", 0.10)),
            action("Z -0.10", () => changeObjectOffset("z", -0.10)),
            action("Z +0.10", () => changeObjectOffset("z", 0.10)),
            action("ROT Z -5", () => changeObjectOffset("rz", -5.0)),
            action("ROT Z +5", () => changeObjectOffset("rz", 5.0)),
            action("RESET OFFSET", resetObjectOffset),
            backItem()
        ]
    };
}

function objectPlacerMenu() {
    return {
        title: "OBJECT PLACER",
        items: [
            submenu("SPAWN CARGO OBJECT", cargoPresetMenu),
            submenu("POSITION / ROTATE", objectOffsetMenu),
            action("ATTACH TO CURRENT VEHICLE", attachActiveObjectToVehicle),
            action("DETACH ACTIVE OBJECT", detachActiveObject),
            action("DELETE ACTIVE OBJECT", deleteActiveObject),
            action("CLEAR ALL PLACED OBJECTS", clearPlacedObjects),
            backItem()
        ]
    };
}

function landmarkTeleportMenu() {
    const items = TELEPORT_POINTS.map(point =>
        action(point.label, () => teleportToNamedPoint(point))
    );
    items.push(backItem());
    return { title: "MAP TELEPORTS", items };
}

function weatherMenu() {
    const items = WEATHER_TYPES.map(w =>
        action((forcedWeather === w.id ? "* " : "") + w.label, () => forceWeatherType(w.id, w.label))
    );
    items.push(action("RANDOM WEATHER", pickRandomWeather));
    items.push(action("RELEASE TO GAME", releaseWeatherControl));
    items.push(backItem());
    return { title: "WEATHER", items };
}

function pedSpawnerMenu() {
    return {
        title: "PED SPAWNER",
        items: [
            action("SPAWN 1 RANDOM PED", () => spawnRandomPed("ANY")),
            action("SPAWN 1 RANDOM MALE", () => spawnRandomPed("MALE")),
            action("SPAWN 1 RANDOM FEMALE", () => spawnRandomPed("FEMALE")),
            action("SPAWN 5 RANDOM PEDS", () => spawnRandomPedGroup(5, "ANY")),
            action("SPAWN 10 RANDOM PEDS", () => spawnRandomPedGroup(10, "ANY")),
            action("RECRUIT RANDOM PED AS GUARD", recruitRandomPedAsGuard),
            action("FILL MY VEHICLE WITH PEDS", fillCurrentVehicleWithRandomPassengers),
            action("PANIC LAST SPAWNED PED", panicLastSpawnedPed),
            action("RELEASE ALL SPAWNED PEDS", clearSpawnedPeds),
            backItem()
        ]
    };
}

function nativeRescueMenu() {
    return {
        title: "NATIVE RESCUE LAB",
        items: [
            action("HAZARD LIGHTS: " + (hazardLightsEnabled ? "ON" : "OFF"), toggleHazardLights),
            action("INTERIOR LIGHT: " + (interiorLightEnabled ? "ON" : "OFF"), toggleInteriorLight),
            action("STRONG AXLES: " + (strongAxlesEnabled ? "ON" : "OFF"), toggleStrongAxles),
            action("HOLD HORN 1.8 SEC", soundCurrentHorn),
            action("HELI THRUST: OFF / 0", () => setHeliThrust(0)),
            action("HELI THRUST: +2", () => setHeliThrust(2)),
            action("HELI THRUST: +5", () => setHeliThrust(5)),
            action("RANDOM BOATS: " + (randomBoatsEnabled ? "ON" : "OFF"), toggleRandomBoats),
            action("PANIC LAST SPAWNED PED", panicLastSpawnedPed),
            backItem()
        ]
    };
}


function pedControlMenu() {
    return { title: "PED CONTROL", items: [
        action("SELECT NEAREST PED", selectNearestPed),
        action("FOLLOW ME", activePedFollowMe),
        action("GUARD / COMBAT NEARBY", activePedGuardMe),
        action("WANDER", activePedWander),
        action("STAND STILL", activePedStand),
        action("FLEE FROM ME", activePedFlee),
        action("HANDS UP", activePedHandsUp),
        action("PANIC SCREAM", activePedPanic),
        action("GET IN MY CAR", activePedGetInMyCar),
        action("GET OUT", activePedGetOut),
        action("ATTACK ME", activePedAttackMe),
        action("ATTACK NEARBY PED", activePedAttackNearbyPed),
        action("GIVE PISTOL",()=>activePedGiveWeapon(7,"PISTOL")),
        action("GIVE AK47",()=>activePedGiveWeapon(14,"AK47")),
        action("REMOVE WEAPONS",activePedRemoveWeapons),
        action("HEAL + ARMOUR",activePedHeal),
        action("INVINCIBLE: "+(activePedInvincible?"ON":"OFF"),toggleActivePedInvincible),
        action("RAGDOLL",activePedRagdoll),
        action("DRUNK: "+(activePedDrunk?"ON":"OFF"),toggleActivePedDrunk),
        action("BLEEDING: "+(activePedBleeding?"ON":"OFF"),toggleActivePedBleed),
        action("DELETE ACTIVE PED",deleteActivePed),
        backItem()
    ]};
}

function pedCombatLabMenu() {
    return { title: "PED COMBAT LAB", items: [
        action("ACCURACY 20",()=>setActivePedAccuracy(20)),
        action("ACCURACY 50",()=>setActivePedAccuracy(50)),
        action("ACCURACY 80",()=>setActivePedAccuracy(80)),
        action("ACCURACY 100",()=>setActivePedAccuracy(100)),
        action("SHOOT RATE 50",()=>setActivePedShootRate(50)),
        action("SHOOT RATE 100",()=>setActivePedShootRate(100)),
        action("SHOOT RATE 160",()=>setActivePedShootRate(160)),
        action("USE COVER: "+(activePedCover?"ON":"OFF"),toggleActivePedCover),
        action("USE CARS IN COMBAT: "+(activePedCarsCombat?"ON":"OFF"),toggleActivePedCarsCombat),
        action("COMBAT NEARBY",activePedCombatNearby),
        action("CLEAR LOS ONLY",activePedOnlyClearLos),
        action("ALLOW BLIND FIRE",activePedAllowBlindFire),
        backItem()
    ]};
}

function vehicleSelectorMenu() { return { title:"VEHICLE TARGET", items:[
    action((selectedVehicleMode===0?"* ":"")+"CURRENT VEHICLE",()=>setVehicleSelector(0,"CURRENT")),
    action((selectedVehicleMode===1?"* ":"")+"NEAREST VEHICLE",()=>setVehicleSelector(1,"NEAREST")),
    action((selectedVehicleMode===2?"* ":"")+"LAST SPAWNED VEHICLE",()=>setVehicleSelector(2,"LAST SPAWNED")),
    backItem()]}; }

function vehicleTyreMenu(){return {title:"TYRE CONTROL",items:[
    action("BURST TYRE 0",()=>burstSelectedTyre(0)),action("FIX TYRE 0",()=>fixSelectedTyre(0)),
    action("BURST TYRE 1",()=>burstSelectedTyre(1)),action("FIX TYRE 1",()=>fixSelectedTyre(1)),
    action("BURST TYRE 2",()=>burstSelectedTyre(2)),action("FIX TYRE 2",()=>fixSelectedTyre(2)),
    action("BURST TYRE 3",()=>burstSelectedTyre(3)),action("FIX TYRE 3",()=>fixSelectedTyre(3)),
    action("FIX TYRES 0-3",fixAllSelectedTyres),backItem()]};}

function vehiclePhysicsMenu(){return {title:"VEHICLE PHYSICS",items:[
    submenu("SELECT VEHICLE",vehicleSelectorMenu),
    action("TRACTION 0.50",()=>setSelectedTraction(0.5)),action("TRACTION 1.00",()=>setSelectedTraction(1.0)),action("TRACTION 1.50",()=>setSelectedTraction(1.5)),action("TRACTION 2.00",()=>setSelectedTraction(2.0)),
    action("ENGINE HEALTH 100",()=>setSelectedEngineHealth(100.0)),action("ENGINE HEALTH 1000",()=>setSelectedEngineHealth(1000.0)),
    action("TANK HEALTH 100",()=>setSelectedTankHealth(100.0)),action("TANK HEALTH 1000",()=>setSelectedTankHealth(1000.0)),
    submenu("TYRE CONTROL",vehicleTyreMenu),
    action("HYDRAULICS: "+(selectedVehicleHydraulics?"ON":"OFF"),toggleSelectedHydraulics),
    action("ALWAYS SKIDS: "+(selectedVehicleSkids?"ON":"OFF"),toggleSelectedSkids),
    action("ONLY PLAYER CAN DAMAGE: "+(selectedVehicleOnlyPlayerDamage?"ON":"OFF"),toggleSelectedOnlyPlayerDamage),
    action("CAR HEAVY ON [EXP]",()=>selectedVehicleHeavy(true)),action("CAR HEAVY OFF [EXP]",()=>selectedVehicleHeavy(false)),
    backItem()]};}

function vehicleDoorsLightsMenu(){return {title:"VEHICLE DOORS & LIGHTS",items:[
    submenu("SELECT VEHICLE",vehicleSelectorMenu),action("HAZARDS ON",()=>selectedHazards(true)),action("HAZARDS OFF",()=>selectedHazards(false)),action("ALL INDICATORS TOGGLE",toggleAllIndicators),
    action("INTERIOR LIGHT ON",()=>selectedInteriorLight(true)),action("INTERIOR LIGHT OFF",()=>selectedInteriorLight(false)),
    action("HEADLIGHTS GAME CONTROL",()=>selectedForceLights(0,"GAME")),action("HEADLIGHTS FORCE OFF",()=>selectedForceLights(1,"OFF")),action("HEADLIGHTS FORCE ON",()=>selectedForceLights(2,"ON")),
    action("OPEN FRONT LEFT / DOOR 0",()=>selectedOpenDoor(0,"DOOR 0")),action("OPEN FRONT RIGHT / DOOR 1",()=>selectedOpenDoor(1,"DOOR 1")),action("OPEN REAR LEFT / DOOR 2",()=>selectedOpenDoor(2,"DOOR 2")),action("OPEN REAR RIGHT / DOOR 3",()=>selectedOpenDoor(3,"DOOR 3")),
    action("POP BOOT / TRUNK",selectedPopBoot),action("CLOSE ALL DOORS",selectedCloseAllDoors),action("TRIGGER ALARM",selectedAlarm),backItem()]};}

function trafficControlMenu(){return {title:"TRAFFIC CONTROL",items:[
    action("STOP TRAFFIC",trafficStop),action("LOW TRAFFIC",trafficLow),action("NORMAL TRAFFIC",trafficNormal),action("HEAVY TRAFFIC",trafficHeavy),action("INSANE TRAFFIC [EXP]",trafficInsane),
    action("PARKED CARS OFF",()=>parkedCars(0.0,"OFF")),action("PARKED CARS NORMAL",()=>parkedCars(1.0,"NORMAL")),action("PARKED CARS HEAVY",()=>parkedCars(2.5,"HEAVY")),
    action("MAD DRIVERS: "+(madDriversEnabled?"ON":"OFF"),toggleMadDrivers),action("RANDOM BOATS: "+(randomBoatsEnabled?"ON":"OFF"),toggleRandomBoats),action("RANDOM TRAINS: "+(trainsEnabled?"ON":"OFF"),toggleTrains),backItem()]};}

function directorModeMenu(){return {title:"DIRECTOR MODE",items:[
    action("FULL STREET WAR",directorGangWar),action("POLICE SIEGE",directorPoliceSiege),action("HIGHWAY CHAOS",directorHighwayChaos),action("MAX POLICE RESPONSE",directorNooseAssault),action("ARMED RIOT VS PLAYER",directorRiot),action("CARJACKING WAVE",directorCarjackingWave),action("ARMED ROAD AMBUSH",directorConvoyAttack),action("BODYGUARD ESCORT",directorBodyguardEscort),action("STORM SHOOTOUT",directorStormShootout),action("CITY PANIC [EXP]",directorCityPanic),action("RESET DIRECTOR / WORLD",directorReset),backItem()]};}

function nativeResearchMenu(){return {title:"NATIVE RESEARCH / NR",items:[
    action("NR-301 HEAD DAMAGE VIS ON",()=>nrHeadDamage(true)),action("NR-301 HEAD DAMAGE VIS OFF",()=>nrHeadDamage(false)),
    action("NR-302 STUN READY ON",()=>nrStunReady(true)),action("NR-302 STUN READY OFF",()=>nrStunReady(false)),
    action("NR-303 BLOCK COWER COVER ON",()=>nrBlockCowerCover(true)),action("NR-303 BLOCK COWER COVER OFF",()=>nrBlockCowerCover(false)),
    action("NR-304 BLOCK PEEK COVER ON",()=>nrBlockPeekCover(true)),action("NR-304 BLOCK PEEK COVER OFF",()=>nrBlockPeekCover(false)),
    action("NR-305 NO EVASIVE DIVES ON",()=>nrNoEvasiveDive(true)),action("NR-305 NO EVASIVE DIVES OFF",()=>nrNoEvasiveDive(false)),
    action("NR-306 MOVE ANIMS BLOCK ON",()=>nrMovementAnimsBlocked(true)),action("NR-306 MOVE ANIMS BLOCK OFF",()=>nrMovementAnimsBlocked(false)),
    action("NR-307 LADDERS ON",()=>nrLadders(true)),action("NR-307 LADDERS OFF",()=>nrLadders(false)),
    action("NR-308 CLIMBOVERS ON",()=>nrClimbovers(true)),action("NR-308 CLIMBOVERS OFF",()=>nrClimbovers(false)),
    action("NR-309 DROP HEIGHT ON [EXP]",()=>nrDropFromHeight(true)),action("NR-309 DROP HEIGHT OFF",()=>nrDropFromHeight(false)),
    action("NR-310 CAR HEAVY ON [EXP]",()=>nrHeavyCar(true)),action("NR-310 CAR HEAVY OFF",()=>nrHeavyCar(false)),backItem()]};}

function sandboxMenu() {
    return {
        title: "SANDBOX",
        items: [
            submenu("MONEY DROPS", moneyMenu),
            submenu("OBJECT PLACER", objectPlacerMenu),
            action("ISLAND ACCESS MODE: " + (accessMode ? "ON" : "OFF"), toggleAccessMode),
            action("REPAIR CURRENT VEHICLE", fixCurrentCar),
            backItem()
        ]
    };
}


function playerLabMenu() {
    return {
        title: "PLAYER LAB",
        items: [
            action("HEAL NIKO", healPlayer),
            action("ARMOUR +100", armourPlayer),
            action("INVINCIBLE: " + (playerInvincible ? "ON" : "OFF"), togglePlayerInvincible),
            action("INVISIBLE: " + (playerInvisible ? "ON" : "OFF"), togglePlayerInvisible),
            action("FREEZE POSITION: " + (playerFrozen ? "ON" : "OFF"), togglePlayerFrozen),
            action("GRAVITY: NORMAL", () => setPlayerGravity(1.0, "NORMAL")),
            action("GRAVITY: MOON 0.35", () => setPlayerGravity(0.35, "MOON")),
            action("GRAVITY: FLOATY 0.10", () => setPlayerGravity(0.10, "FLOATY")),
            action("WINDSCREEN EJECTION: " + (playerWindscreenEject ? "ON" : "OFF"), toggleWindscreenEjection),
            action("EVERYONE IGNORE ME: " + (everyoneIgnorePlayer ? "ON" : "OFF"), toggleEveryoneIgnorePlayer),
            backItem()
        ]
    };
}

function travelLabMenu() {
    return {
        title: "TRAVEL / POSITION LAB",
        items: [
            submenu("MAP TELEPORTS", landmarkTeleportMenu),
            action("SAVE CURRENT POSITION", saveCurrentPosition),
            action("RETURN TO SAVED POSITION", returnToSavedPosition),
            action("NUDGE NORTH +10", () => nudgePlayer(0.0, 10.0, 0.0, "NORTH")),
            action("NUDGE SOUTH -10", () => nudgePlayer(0.0, -10.0, 0.0, "SOUTH")),
            action("NUDGE EAST +10", () => nudgePlayer(10.0, 0.0, 0.0, "EAST")),
            action("NUDGE WEST -10", () => nudgePlayer(-10.0, 0.0, 0.0, "WEST")),
            action("NUDGE UP +5", () => nudgePlayer(0.0, 0.0, 5.0, "UP")),
            backItem()
        ]
    };
}

function vehicleLabMenu() {
    return {
        title: "VEHICLE LAB",
        items: [
            action("CAR DAMAGE: " + (carDamageEnabled ? "ON" : "OFF"), toggleCarDamage),
            action("VISIBLE DAMAGE: " + (carVisibleDamageEnabled ? "ON" : "OFF"), toggleCarVisibleDamage),
            action("FULL PROOFS: " + (carProofsEnabled ? "ON" : "OFF"), toggleCarProofs),
            action("STRONG COLLISIONS: " + (carStrongEnabled ? "ON" : "OFF"), toggleCarStrong),
            action("WATERTIGHT: " + (carWatertightEnabled ? "ON" : "OFF"), toggleCarWatertight),
            action("INVISIBLE VEHICLE: " + (carInvisible ? "ON" : "OFF"), toggleCarInvisible),
            action("FREEZE VEHICLE: " + (carFrozen ? "ON" : "OFF"), toggleCarFrozen),
            action("UPRIGHT / FLIP RECOVERY", uprightCurrentCar),
            action("LOCK DOORS", () => setDoorLock(2, "LOCKED")),
            action("UNLOCK DOORS", () => setDoorLock(1, "UNLOCKED")),
            action("BOOST SPEED 10", () => setForwardSpeed(10.0, "10")),
            action("BOOST SPEED 25", () => setForwardSpeed(25.0, "25")),
            action("BOOST SPEED 40", () => setForwardSpeed(40.0, "40")),
            action("TRACTION: DRIFT 0.45", () => setCarTraction(0.45, "DRIFT")),
            action("TRACTION: NORMAL 1.0", () => setCarTraction(1.0, "NORMAL")),
            action("TRACTION: GRIP 1.8", () => setCarTraction(1.8, "GRIP")),
            action("LIGHTS: DIM 0.25X", () => setCarLightMultiplier(0.25, "0.25X")),
            action("LIGHTS: NORMAL 1X", () => setCarLightMultiplier(1.0, "1X")),
            action("LIGHTS: BRIGHT 3X", () => setCarLightMultiplier(3.0, "3X")),
            backItem()
        ]
    };
}

function worldLabMenu() {
    return {
        title: "WORLD LAB",
        items: [
            submenu("WEATHER", weatherMenu),
            action("MAD DRIVERS: " + (madDriversEnabled ? "ON" : "OFF"), toggleMadDrivers),
            action("WIND: CALM 0.0", () => setWind(0.0, "CALM")),
            action("WIND: NORMAL 1.0", () => setWind(1.0, "NORMAL")),
            action("WIND: STORM 5.0", () => setWind(5.0, "STORM")),
            action("WIND: EXTREME 10.0", () => setWind(10.0, "EXTREME 10")),
            action("WIND: INSANE 25.0 [EXP]", () => setWind(25.0, "INSANE 25")),
            action("WIND: ABSURD 50.0 [EXP]", () => setWind(50.0, "ABSURD 50")),
            backItem()
        ]
    };
}

function cityWarMenu() {
    return {
        title: "CITY WAR 2.0 LAB",
        items: [
            action("END WAR -> EMS AFTERMATH", requestStreetWarEnd),
            action("FORCE CLEANUP / OFF", () => clearStreetWar()),
            action("SKIRMISH / 3v3", () => setStreetWar(6, "SKIRMISH")),
            action("STREET WAR / 5v5", () => setStreetWar(10, "STREET WAR")),
            action("FULL WAR / 8v8", () => setStreetWar(16, "FULL WAR")),
            action("WAR ZONE PRESET / 8v8", warZonePreset),
            action("TEAM STATUS [" + streetWarState + "]: A " + streetWarTeamA.length + " / B " + streetWarTeamB.length, warTeamStatus),
            action("ACTOR ENGINE STATUS: " + managedActors.length + "/" + MAX_MANAGED_ACTORS, managedActorStatus),
            action("WAR SPAWN BUDGET: " + cityWarTotalSpawns + "/" + CITY_WAR_TOTAL_SPAWN_BUDGET, warTeamStatus),
            action("REINFORCEMENTS: " + (streetWarReinforcements ? "ON" : "OFF"), toggleWarReinforcements),
            action("PLAYER NEUTRAL: " + (streetWarPlayerNeutral ? "ON" : "OFF"), toggleWarPlayerNeutral),
            action("POLICE RESPONSE: " + (warPoliceResponseEnabled ? "ON" : "OFF"), toggleWarPoliceResponse),
            action("POLICE RESPONSE STATUS: " + warPoliceUnits.length + "/" + WAR_POLICE_MAX_UNITS, warPoliceStatus),
            action("EMS AFTERMATH: " + (warEmsEnabled ? "ON" : "OFF"), toggleWarEmsResponse),
            action("EMS STATUS", warEmsStatus),
            action("WEAPONS: SMG ONLY", () => setWarWeaponMode(0, "SMG")),
            action("WEAPONS: AR ONLY", () => setWarWeaponMode(1, "AR")),
            action("WEAPONS: TEAM MIX (A SMG / B AR)", () => setWarWeaponMode(2, "TEAM MIX")),
            action("FIGHTER HEALTH 250", () => setWarHealth(250)),
            action("FIGHTER HEALTH 400", () => setWarHealth(400)),
            action("FIGHTER HEALTH 600", () => setWarHealth(600)),
            action("ACCURACY 25", () => setWarAccuracy(25)),
            action("ACCURACY 50", () => setWarAccuracy(50)),
            action("ACCURACY 75", () => setWarAccuracy(75)),
            action("FIRE RATE 70", () => setWarShootRate(70)),
            action("FIRE RATE 110", () => setWarShootRate(110)),
            action("FIRE RATE 160", () => setWarShootRate(160)),
            action("USE COVER: " + (streetWarUseCover ? "ON" : "OFF"), toggleWarCover),
            action("CARJACKERS: " + (streetWarCarjackers ? "ON" : "OFF"), toggleWarCarjackers),
            action("DRIVE SPEED 15", () => setWarDriveSpeed(15.0)),
            action("DRIVE SPEED 25", () => setWarDriveSpeed(25.0)),
            action("DRIVE SPEED 35", () => setWarDriveSpeed(35.0)),
            action("WAR FIRE AMMO: " + (streetWarFireAmmo ? "ON" : "OFF"), toggleWarFireAmmo),
            action("PLAYER INCENDIARY HITS: " + (incendiaryHits ? "ON" : "OFF"), toggleIncendiaryHits),
            action("INCENDIARY CHANCE 25%", () => setIncendiaryChance(25)),
            action("INCENDIARY CHANCE 50%", () => setIncendiaryChance(50)),
            action("INCENDIARY CHANCE 100%", () => setIncendiaryChance(100)),
            action("MAD DRIVERS: " + (madDriversEnabled ? "ON" : "OFF"), toggleMadDrivers),
            action("FORCE RETARGET BOTH TEAMS", forceWarRetarget),
            backItem()
        ]
    };
}

function rootMenu() {
    return {
        title: "DYNAMIC LIBERTY v6.1.4",
        items: [
            submenu("TACTICAL SQUAD", recruitMenu),
            submenu("PED SPAWNER", pedSpawnerMenu),
            submenu("PED CONTROL", pedControlMenu),
            submenu("PED COMBAT LAB", pedCombatLabMenu),
            submenu("CITY WAR 2.0 LAB", cityWarMenu),
            submenu("DIRECTOR MODE", directorModeMenu),
            submenu("PLAYER LAB", playerLabMenu),
            submenu("TRAVEL / POSITION LAB", travelLabMenu),
            submenu("VEHICLE SPAWNER", vehicleCategoryMenu),
            submenu("VEHICLE TOOLS", vehicleToolsMenu),
            submenu("VEHICLE LAB", vehicleLabMenu),
            submenu("VEHICLE PHYSICS", vehiclePhysicsMenu),
            submenu("VEHICLE DOORS & LIGHTS", vehicleDoorsLightsMenu),
            submenu("TRAFFIC CONTROL", trafficControlMenu),
            submenu("TIME / CLOCK", timeMenu),
            submenu("WEATHER", weatherMenu),
            submenu("SANDBOX", sandboxMenu),
            submenu("WORLD CONTROL", worldMenu),
            submenu("WORLD LAB", worldLabMenu),
            submenu("POLICE", policeMenu),
            submenu("NATIVE RESCUE LAB", nativeRescueMenu),
            submenu("NATIVE RESEARCH", nativeResearchMenu),
            submenu("SCENARIOS", wowMenu),
            action("CLOSE TRAINER", () => { menuOpen = false; imguiClearPending = true; unlockPlayerControl(); })
        ]
    };
}

function squadRecruitMenu() {
    return {
        title: "SQUAD / RECRUIT",
        items: [
            action("RECRUIT 1   [" + bodyguardCount() + "/10]", () => recruitBodyguards(1)),
            action("RECRUIT 5", () => recruitBodyguards(5)),
            action("FILL SQUAD TO 10", () => recruitBodyguards(10)),
            action("REAPPLY SQUAD PROFILE", reapplySquadProfile),
            action("HEAL + ARMOUR ALL", healBodyguards),
            action("DISMISS ALL", dismissBodyguards),
            backItem()
        ]
    };
}

function squadOrdersMenu() {
    return {
        title: "TACTICAL ORDERS",
        items: [
            action("REGROUP / FOLLOW ME", squadRegroup),
            action("COMBAT NEARBY ENEMIES", squadCombatNearby),
            action("ATTACK ACTIVE PED", squadAttackActivePed),
            action("STAND DOWN", squadStandDown),
            action("HOLD POSITION", squadHoldPosition),
            action("GUARD CURRENT POSITION", squadGuardPosition),
            action("BOARD MY VEHICLE", squadEnterCar),
            backItem()
        ]
    };
}

function squadFormationMenu() {
    return {
        title: "FORMATION",
        items: [
            action("FORMATION ID 0", () => setSquadFormation(0)),
            action("FORMATION ID 1", () => setSquadFormation(1)),
            action("FORMATION ID 2", () => setSquadFormation(2)),
            action("FORMATION ID 3", () => setSquadFormation(3)),
            action("SPACING 1.0m", () => setSquadSpacing(1.0)),
            action("SPACING 2.0m", () => setSquadSpacing(2.0)),
            action("SPACING 4.0m", () => setSquadSpacing(4.0)),
            action("SPACING 8.0m", () => setSquadSpacing(8.0)),
            backItem()
        ]
    };
}

function squadLoadoutMenu() {
    const items = SQUAD_WEAPONS.map(w =>
        action((squadWeapon === w.id ? "* " : "") + w.label, () => setSquadWeapon(w.id, w.label))
    );
    items.push(backItem());
    return { title: "SQUAD LOADOUT", items };
}

function squadAimMenu() {
    return {
        title: "AIM / FIRE CONTROL",
        items: [
            action("ACCURACY 25", () => setSquadAccuracy(25)),
            action("ACCURACY 50", () => setSquadAccuracy(50)),
            action("ACCURACY 75", () => setSquadAccuracy(75)),
            action("ACCURACY 100", () => setSquadAccuracy(100)),
            action("SHOOT RATE 40", () => setSquadShootRate(40)),
            action("SHOOT RATE 75", () => setSquadShootRate(75)),
            action("SHOOT RATE 100", () => setSquadShootRate(100)),
            action("SHOOT RATE 150", () => setSquadShootRate(150)),
            action("USE COVER: " + (squadUseCover ? "ON" : "OFF"), toggleUseCover),
            action("COVERING FIRE: " + (squadCoveringFire ? "ON" : "OFF"), toggleCoveringFire),
            action("CLEAR LOS ONLY: " + (squadClearLosOnly ? "ON" : "OFF"), toggleClearLos),
            action("SIGNAL AFTER KILL: " + (squadSignalAfterKill ? "ON" : "OFF"), toggleSignalKill),
            backItem()
        ]
    };
}

function squadSurvivalMenu() {
    return {
        title: "SURVIVABILITY",
        items: [
            action("HEALTH 300", () => setSquadHealth(300)),
            action("HEALTH 500", () => setSquadHealth(500)),
            action("HEALTH 1000", () => setSquadHealth(1000)),
            action("HEALTH 2000", () => setSquadHealth(2000)),
            action("INVINCIBLE: " + (squadInvincible ? "ON" : "OFF"), toggleInvincible),
            action("HEADSHOT IMMUNE: " + (squadHeadshotImmune ? "ON" : "OFF"), toggleHeadshotImmune),
            action("MOVE WHEN INJURED: " + (squadMoveWhenInjured ? "ON" : "OFF"), toggleMoveInjured),
            action("DROP WEAPONS ON DEATH: " + (squadDropsWeapons ? "ON" : "OFF"), toggleDropsWeapons),
            action("CAN BE KNOCKED OFF BIKE: " + (squadCanBeKnockedOffBike ? "ON" : "OFF"), toggleBikeKnockoff),
            backItem()
        ]
    };
}

function squadVehicleAiMenu() {
    return {
        title: "VEHICLE CREW AI",
        items: [
            action("BOARD MY VEHICLE", squadEnterCar),
            action("DRIVE-BYS: " + (squadDrivebys ? "ON" : "OFF"), toggleDrivebys),
            action("USE CARS IN COMBAT: " + (squadUseCarsCombat ? "ON" : "OFF"), toggleCarsCombat),
            action("LEAVE CAR IN COMBAT: " + (squadLeaveCarCombat ? "ON" : "OFF"), toggleLeaveCarCombat),
            action("STAY IN CAR WHEN JACKED: " + (squadStayInCarWhenJacked ? "ON" : "OFF"), toggleStayCarJacked),
            action("CANT BE DRAGGED OUT: " + (squadCantBeDraggedOut ? "ON" : "OFF"), toggleCantDragged),
            backItem()
        ]
    };
}

function squadRoleMenu() {
    return {
        title: "ROLE PRESETS",
        items: [
            action("RIFLE TEAM", presetRifleTeam),
            action("TANK TEAM", presetTankTeam),
            action("DRIVE-BY CREW", presetDrivebyCrew),
            action("CHAOS CREW / MINIGUN", presetChaosCrew),
            backItem()
        ]
    };
}

function squadAiLabMenu() {
    return {
        title: "AI LAB / WEIRD GTA IV",
        items: [
            action("DRUNK AI: " + (squadDrunk ? "ON" : "OFF"), toggleDrunk),
            action("BLIND RAGE: " + (squadBlindRage ? "ON" : "OFF"), toggleBlindRage),
            action("SIGNAL AFTER KILL: " + (squadSignalAfterKill ? "ON" : "OFF"), toggleSignalKill),
            action("CLEAR LOS ONLY: " + (squadClearLosOnly ? "ON" : "OFF"), toggleClearLos),
            action("COVERING FIRE: " + (squadCoveringFire ? "ON" : "OFF"), toggleCoveringFire),
            backItem()
        ]
    };
}

function recruitMenu() {
    return {
        title: "TACTICAL SQUAD [" + bodyguardCount() + "/10]",
        items: [
            submenu("SQUAD / RECRUIT", squadRecruitMenu),
            submenu("TACTICAL ORDERS", squadOrdersMenu),
            submenu("FORMATION", squadFormationMenu),
            submenu("LOADOUT", squadLoadoutMenu),
            submenu("AIM / FIRE CONTROL", squadAimMenu),
            submenu("SURVIVABILITY", squadSurvivalMenu),
            submenu("VEHICLE CREW AI", squadVehicleAiMenu),
            submenu("ROLE PRESETS", squadRoleMenu),
            submenu("AI LAB / WEIRD GTA IV", squadAiLabMenu),
            backItem()
        ]
    };
}

function vehicleCategoryMenu() {
    const items = [];
    Object.keys(VEHICLES).forEach(cat => {
        items.push(submenu(cat.toUpperCase(), () => vehicleListMenu(cat)));
    });
    items.push(backItem());
    return { title: "VEHICLE SPAWNER", items };
}

function vehicleListMenu(category) {
    const items = VEHICLES[category].map(name =>
        action(name, () => spawnVehicle(name))
    );
    items.push(backItem());
    return { title: category.toUpperCase(), items };
}

function vehicleToolsMenu() {
    return {
        title: "VEHICLE TOOLS",
        items: [
            action("REPAIR CURRENT VEHICLE", fixCurrentCar),
            action("CLEAN CURRENT VEHICLE", cleanCurrentCar),
            backItem()
        ]
    };
}

function timeMenu() {
    const items = [];
    for (let h = 0; h < 24; h++) {
        const hh = h;
        items.push(action(String(h).padStart(2, "0") + ":00", () => setHour(hh)));
    }
    items.push(action(
        "FREEZE CLOCK: " + (frozenTime ? "ON" : "OFF"),
        toggleFreezeTime
    ));
    items.push(backItem());
    return { title: "TIME / CLOCK", items };
}

function worldMenu() {
    return {
        title: "WORLD CONTROL",
        items: [
            action("IMGUI STATUS TEST", imguiDiagnostic),
            action("AGGRESSIVE PEDS: OFF", () => setAggressiveMode(0)),
            action("AGGRESSIVE PEDS: FISTS", () => setAggressiveMode(1)),
            action("AGGRESSIVE PEDS: ARMED", () => setAggressiveMode(2)),
            action("TRAFFIC: EMPTY", () => trafficDensity(0.0, "EMPTY")),
            action("TRAFFIC: NORMAL", () => trafficDensity(1.0, "NORMAL")),
            action("TRAFFIC: RUSH HOUR", () => trafficDensity(1.8, "RUSH HOUR")),
            action("PEDS: EMPTY", () => pedDensity(0.0, "EMPTY")),
            action("PEDS: NORMAL", () => pedDensity(1.0, "NORMAL")),
            action("PEDS: CROWDED", () => pedDensity(1.8, "CROWDED")),
            action("WORLD SPEED: 25%", () => setGameSpeed(0.25, "25%")),
            action("WORLD SPEED: 50%", () => setGameSpeed(0.50, "50%")),
            action("WORLD SPEED: NORMAL", () => setGameSpeed(1.0, "NORMAL")),
            action("WORLD SPEED: 150%", () => setGameSpeed(1.5, "150%")),
            action("TOGGLE TRAINS", toggleTrains),
            backItem()
        ]
    };
}

function policeMenu() {
    const items = [
        action("POLICE SYSTEM: " + (policeDisabled ? "DISABLED" : "ENABLED"), togglePoliceDisabled),
        action("CLEAR NEARBY COPS", () => clearNearbyCops(140.0, true))
    ];

    for (let i = 0; i <= 6; i++) {
        const level = i;
        items.push(action("WANTED LEVEL " + i, () => setWanted(level)));
    }

    items.push(action(
        "POLICE HELIS: " + (policeHelisEnabled ? "ON" : "OFF"),
        togglePoliceHelis
    ));
    items.push(backItem());
    return { title: "POLICE", items };
}


function blackoutRun() {
    frozenHour = 3;
    n("SET_TIME_OF_DAY", 3, 0);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.55);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.55);
    n("SET_PED_DENSITY_MULTIPLIER", 0.75);
    n("SWITCH_POLICE_HELIS", false);
    policeHelisEnabled = false;
    notify("BLACKOUT RUN");
}

function swarmCity() {
    n("SET_PED_DENSITY_MULTIPLIER", 2.4);
    n("SET_CAR_DENSITY_MULTIPLIER", 0.65);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 0.65);
    setAggressiveMode(1);
    notify("SWARM CITY");
}

function convoyEscape() {
    const p = playerId();
    n("ALTER_WANTED_LEVEL", p, 5);
    n("APPLY_WANTED_LEVEL_CHANGE_NOW", p);
    n("SET_CAR_DENSITY_MULTIPLIER", 1.35);
    n("SET_RANDOM_CAR_DENSITY_MULTIPLIER", 1.35);
    n("SWITCH_POLICE_HELIS", true);
    policeHelisEnabled = true;
    notify("CONVOY ESCAPE");
}

function cashChaos() {
    moneyScatter(20, 5000);
    setAggressiveMode(1);
    n("SET_PED_DENSITY_MULTIPLIER", 1.8);
    notify("CASH CHAOS");
}

function wowMenu() {
    return {
        title: "SCENARIOS",
        items: [
            action("PURGE HOUR", purgeHour),
            action("NIGHT SIEGE", nightSiege),
            action("SLOW-MO MAYHEM", slowMoMayhem),
            action("CITY WAR 5v5 + POLICE + EMS", () => {
                warPoliceResponseEnabled = true;
                warEmsEnabled = true;
                setStreetWar(10, "SCENARIO 5v5");
                streetWarReinforcements = false;
            }),
            action("RESET WORLD", normalWorld),
            backItem()
        ]
    };
}

// ------------------------------------------------------------
// MENU ENGINE
// ------------------------------------------------------------
function currentMenu() {
    if (menuStack.length === 0) return rootMenu();
    return menuStack[menuStack.length - 1]();
}

function openSubmenu(getter) {
    menuStack.push(getter);
    selected = 0;
    markMenuDirty();
}

function goBack() {
    if (menuStack.length > 0) {
        menuStack.pop();
        selected = 0;
        markMenuDirty();
    } else {
        menuOpen = false;
        imguiClearPending = true;
        unlockPlayerControl();
    }
}

function runSelected() {
    const menu = currentMenu();
    if (!menu.items.length) return;

    const item = menu.items[selected];
    if (!item) return;

    if (item.submenu) {
        openSubmenu(item.submenu);
        return;
    }

    if (item.action) {
        item.action();
        if (menuOpen) markMenuDirty();
    }
}

function moveSelection(delta) {
    const menu = currentMenu();
    if (!menu.items.length) return;

    selected += delta;
    if (selected < 0) selected = menu.items.length - 1;
    if (selected >= menu.items.length) selected = 0;
    markMenuDirty();
}

// ------------------------------------------------------------
// IMGUIREDUX MENU RENDERER — V5.3 SINGLE-CLEAR FRAME
// Open: render every game frame.
// Close: submit one empty clear frame, then leave ImGui alone until reopened.
// This avoids continuously exercising Dear ImGui's implicit fallback window.
// ------------------------------------------------------------
const IMGUI_VISIBLE = 11;

function drawImGuiMenuContents() {
    const menu = currentMenu();
    if (!menu || !menu.items || !menu.items.length) return;

    if (selected >= menu.items.length) selected = menu.items.length - 1;

    let start = Math.max(0, selected - 4);
    let end = Math.min(menu.items.length, start + IMGUI_VISIBLE);
    if (end - start < IMGUI_VISIBLE) {
        start = Math.max(0, end - IMGUI_VISIBLE);
    }

    // ImGuiCond_Always = 1. Force position/size every frame so controller
    // mouse emulation cannot drag the trainer away.
    ImGui.SetNextWindowPos(34.0, 70.0, 1);
    ImGui.SetNextWindowSize(430.0, 520.0, 1);

    // ImGuiRedux Begin args after title/open:
    // noTitleBar, noResize, noMove, autoResize.
    // No titlebar + noMove + noResize = controller-only fixed trainer panel.
    const visible = ImGui.Begin(
        "Dynamic Liberty v6.1.4",
        true,
        true,
        true,
        true,
        false
    );

    // Do NOT call SetWindowPos/SetWindowSize after Begin().
    // SetNextWindowPos/SetNextWindowSize above already hard-lock the panel
    // every frame, while avoiding the tearing/side-effects of mid-window
    // repositioning.

    if (visible) {
        ImGui.TextColored("DYNAMIC LIBERTY v6.1.4", 0.25, 0.75, 1.0, 1.0);
        ImGui.Text(menu.title);
        ImGui.Separator();
        ImGui.Spacing();

        for (let i = start; i < end; i++) {
            const item = menu.items[i];
            const active = i === selected;
            const suffix = item.submenu ? "  >" : "";
            const label = (active ? "> " : "  ") + item.label + suffix;

            if (active) {
                ImGui.TextColored(label, 1.0, 0.82, 0.20, 1.0);
            } else {
                ImGui.Text(label);
            }
            ImGui.Spacing();
        }

        ImGui.Separator();
        ImGui.Text(
            (selected + 1) + "/" + menu.items.length +
            "   DPAD MOVE   A SELECT   B BACK"
        );

        if (menu.title.indexOf("TACTICAL") >= 0 || menu.title.indexOf("SQUAD") >= 0 ||
            menu.title.indexOf("FORMATION") >= 0 || menu.title.indexOf("AI LAB") >= 0) {
            ImGui.Text(
                "Squad " + bodyguardCount() + "/10 | Weapon " + squadWeapon +
                " | Acc " + squadAccuracy + " | Spacing " + squadSpacing.toFixed(1)
            );
        }
    }

    ImGui.End();
}

function renderImGuiFrame() {
    // While open, submit the trainer every frame.
    if (menuOpen) {
        ImGui.BeginFrame("DYNAMIC_LIBERTY_V600_FRAME");
        drawImGuiMenuContents();
        ImGui.EndFrame();
        return;
    }

    // On close, submit exactly ONE empty frame to clear the last menu image.
    // Then stop creating empty ImGui frames until the trainer opens again.
    if (imguiClearPending) {
        ImGui.BeginFrame("DYNAMIC_LIBERTY_V600_CLEAR");
        ImGui.EndFrame();
        imguiClearPending = false;
    }
}

function markMenuDirty() {}
function renderMenu(force = false) {}

// ------------------------------------------------------------
// BACKGROUND WORLD UPDATES
// ------------------------------------------------------------
function worldUpdate() {
    const now = Date.now();

    if (frozenTime && now - lastFreezeUpdate >= 300) {
        lastFreezeUpdate = now;
        n("SET_TIME_OF_DAY", frozenHour, 0);
    }

    // Access Mode and Police Disabled both suppress the restricted-island
    // wanted response. Police Disabled additionally suppresses random cops
    // and police helicopters.
    if ((accessMode || policeDisabled) && now - lastAccessEnforce >= 150) {
        lastAccessEnforce = now;
        const p = playerId();
        if (p !== null) {
            n("SET_MAX_WANTED_LEVEL", 0);
            n("CLEAR_WANTED_LEVEL", p);
            n("DONT_DISPATCH_COPS_FOR_PLAYER", p, true);
            n("SET_POLICE_IGNORE_PLAYER", p, true);

            if (policeDisabled) {
                n("SET_CREATE_RANDOM_COPS", false);
                n("SWITCH_POLICE_HELIS", false);
            }
        }
    }

    if (policeDisabled && now - lastPoliceAreaClear >= 1500) {
        lastPoliceAreaClear = now;
        clearNearbyCops(140.0, false);
    }

    if (aggressiveMode && !effectivePlayerIgnore() && now - lastAggroPulse >= 1100) {
        lastAggroPulse = now;
        pulseAggressivePeds();
    }

    if (streetWarLimit > 0 && now - lastStreetWarPulse >= 450) {
        lastStreetWarPulse = now;
        pulseStreetWar();
    }

    if (streetWarCarjackers && streetWarLimit > 0 && now - lastCarjackPulse >= 1200) {
        lastCarjackPulse = now;
        pulseWarCarjackers();
    }

    if ((warPoliceUnits.length || (warPoliceResponseEnabled &&
        (streetWarState === "ACTIVE" || streetWarState === "ENDING"))) &&
        now - lastWarPolicePulse >= WAR_POLICE_PULSE_MS) {
        lastWarPolicePulse = now;
        pulseWarPoliceResponse();
    }

    if ((warEmsUnit || streetWarState === "AFTERMATH") &&
        now - lastWarEmsPulse >= WAR_EMS_PULSE_MS) {
        lastWarEmsPulse = now;
        pulseWarEmsResponse();
    }

    if (managedActors.length && now - lastManagedActorPulse >= MANAGED_ACTOR_PULSE_MS) {
        lastManagedActorPulse = now;
        pulseManagedActors();
    }

    if ((incendiaryHits || streetWarFireAmmo) && now - lastIncendiaryPulse >= 120) {
        lastIncendiaryPulse = now;
        pulseIncendiaryHits();
    }

    updateBodyguardSquad();
}

// ------------------------------------------------------------
// FAST CONTROLLER MENU INPUT
// Tap = instant. Hold D-pad = quick repeat. A/B use release latches so
// B performs exactly one back action per press.
// ------------------------------------------------------------
let navHeld = 0;
let navRepeatAt = 0;
let aLatch = false;
let bLatch = false;

function updateMenuInput() {
    const now = Date.now();

    const up = btnPressed(BTN.UP);
    const down = btnPressed(BTN.DOWN);

    let dir = 0;
    if (up && !down) dir = -1;
    if (down && !up) dir = 1;

    if (dir !== 0) {
        if (navHeld !== dir) {
            navHeld = dir;
            moveSelection(dir);
            navRepeatAt = now + 230;
        } else if (now >= navRepeatAt) {
            moveSelection(dir);
            navRepeatAt = now + 85;
        }
    } else {
        navHeld = 0;
    }

    const a = btnPressed(BTN.A);
    if (a && !aLatch) {
        aLatch = true;
        runSelected();
    } else if (!a) {
        aLatch = false;
    }

    const b = btnPressed(BTN.B);
    if (b && !bLatch) {
        bLatch = true;
        goBack();
    } else if (!b) {
        bLatch = false;
    }
}

// ------------------------------------------------------------
// MAIN LOOP
// ------------------------------------------------------------
while (true) {
    const lb = btnPressed(BTN.LB);
    const rb = btnPressed(BTN.RB);

    if (lb && rb) {
        if (!comboLatch) {
            comboLatch = true;
            menuOpen = !menuOpen;

            if (menuOpen) {
                menuStack = [];
                selected = 0;
                aLatch = true;
                bLatch = true;
                navHeld = 0;
                lockPlayerControl();
                markMenuDirty();
            } else {
                imguiClearPending = true;
                unlockPlayerControl();
                // One empty ImGui frame will clear the previous menu draw data.
            }
        }
    } else {
        comboLatch = false;
    }

    if (menuOpen) {
        updateMenuInput();

    } else {
        // Release latches outside the trainer.
        if (!btnPressed(BTN.A)) aLatch = false;
        if (!btnPressed(BTN.B)) bLatch = false;
    }

    renderImGuiFrame();
    worldUpdate();
    wait(0);
}
