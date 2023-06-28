# RHAPI

- [Introduction](#introduction)
- [User Interface Helpers](#user-interface-helpers)
- [Data Fields](#data-fields)
- [Database Access](#database-access)
- [Data input/output](#data-input-output)
- [Heat Generation](#heat-generation)
- [Class Ranking Methods](#class-ranking-methods)
- [Race Points Methods](#race-points-methods)
- [LED](#led-)
- [Video Receivers](#video-receivers)
- [Active Race](#active-race)
- [Event Results](#event-results)
- [Language and Translation](#language-and-translation)
- [Hardware Interface](#hardware-interface)
- [Sensors Interface](#sensors-interface)


## Introduction

Most plugin interfaces provide access to `RHAPI`, an object providing a wide range of properties and methods across RotorHazard's internal systems. Using RHAPI, one can manipulate nearly every facet of a race event and RotorHazard's behavior.

### API Version

The API version can be read from the `API_VERSION_MAJOR` and `API_VERSION_MINOR` properties.

### Language translation shortcut

The language translation function can be accessed directly via `RHAPI.__()` in addition to its location within `RHAPI.language`.



## User Interface Helpers

Interact with RotorHazard's frontend user interface.
These methods are accessed via `RHAPI.ui` 

### UI Panels

Add custom UI panels to RotorHazard's frontend pages. Panels may contain Options and Quickbuttons.

Panels are represented with the `UIPanel` class, which has the following properties:
- `name` (string): Internal identifier for this panel
- `label` (string): Text used as visible panel header
- `page` (string): Page to add panel to
- `order` (int): Not yet implemented


#### ui.panels
_Read only_
The list of registered panels. Returns `List[UIPanel]`.

#### ui.register_panel(name, label, page, order=0)
Register a UI panel and assign it to a page. Returns all panels as `list[UIPanel]`.

- `name` (string): Internal identifier for this panel
- `label` (string): Text used as visible panel header
- `page` (string): Page to add panel to; one of "format", "settings"
- `order` _optional_ (int): Not yet implemented 

### Quickbuttons

Provides a simple interface to add a UI button and bind it to a function. Quickbuttons appear on assigned UI panels.

Quickbuttons are represented with the `QuickButton` class, which has the following properties:
- `panel` (string): `name` of panel where button will appear
- `name` (string): Internal identifier for this quickbutton
- `label` (string): Text used for visible button label
- `function` (callable): Function to run when button is pressed
- `args` (any): Argument passed to `function` when called

Pass a dict to `args` and parse it in your `function` if multiple arguments are needed.

#### ui.register_quickbutton(panel, name, label, function)
Register a Quickbutton and assign it to a UI panel. Returns all buttons as `list[QuickButton]`.

- `panel` (string): `name` of panel previously registered with `ui.register_panel`
- `name` (string): Internal identifier for this quickbutton
- `label` (string): Text used for visible button label
- `function` (callable): Function to run when button is pressed
- `args` (any): Argument passed to `function` when called


### Pages

Add custom pages to RotorHazard's frontend with [Flask Blueprints](https://flask.palletsprojects.com/blueprints/). 

#### ui.blueprint_add(blueprint)
Adds a Flask Blueprint which can be used to provide access to custom pages/URLs.

- `blueprint` (blueprint): Flask Blueprint object

### UI Messages

Send messages to RotorHazard's frontend.

#### ui.message_speak(message)
Sends a message which is parsed by the text-to-speech synthesizer.

- `message` (string): Text of message to be spoken

#### ui.message_notify(message)
Sends a message which appears in the message center and notification bar.

- `message` (string): Text of message to display

#### ui.message_alert(message)
Sends a message which appears as a pop-up alert.

- `message` (string): Text of message to display



## Data Fields

Create and access new data structures.
These methods are accessed via `RHAPI.fields` 


### Options
Options are simple storage variables which persist to the database and can be presented  to users through frontend UI. Each option takes a single value.

#### fields.options
_Read only_
Provides a list of options registered by plugins. Does not include built-in options.

#### fields.register_option(field, panel=None, order=0)
Register an option and optioanlly assign it to be desiplayed on a UI panel.

- `field` (UIField): See [UI Fields](Plugins.md#ui-fields)
- `panel` _optional_ (string): `name` of panel previously registered with `ui.register_panel`
- `order` _optional_ (int): Not yet implemented


### Pilot Attributes

Pilot Attributes are simple storage variables which persist to the database and can be presented to users through frontend UI. Pilot Attribute values are unique to/stored individually for each pilot.

Pilot Attributes are represented with the `PilotAttribute` class, which has the following properties:
- `id` (int): ID of pilot to which this attribute is assigned
- `name` (string): Name of attribute
- `value` (string): Value of attribute

#### fields.pilot_attributes
_Read only_
Provides a list of registered `PilotAttribute`s.

#### fields.register_pilot_attribute(field)
Register an attribute to be displayed in the UI or otherwise made accessible to plugins.

- `field` (UIField): See [UI Fields](Plugins.md#ui-fields)



## Database Access

Read and modify database values.
These methods are accessed via `RHAPI.db` 

### Global
Properties and methods spanning the entire stored event.

#### db.event_results
_Read only_
Returns cumulative totals for all saved races as `dict`.

#### db.clear_all()
Resets database to default state, clearing all pilots, heats, race classes, races, race formats, frequency sets, and options.

### Pilots
A pilot is an individual participant. In order to participate in races, pilots can be assigned to multiple heats.

Pilots are represented with the `Pilot` class, which has the following properties:
- `id` (int): Internal identifier 
- `callsign` (string): Callsign
- `team` (string): Team designation
- `phonetic` (string): Phonetically-spelled callsign, used for text-to-speech
- `name` (string): Real name
- `color` (string): Hex-encoded color
- `used_frequencies` (string): Serialized list of frequencies this pilot has been assigned when starting a race, ordered by recency
- `active` (boolean): Not yet implemented

The sentinel value `RHUtils.PILOT_ID_NONE` should be used when no pilot is defined.


#### db.pilots
_Read only_
All pilot records. Returns `list[Pilot]`.

#### db.pilot_by_id(pilot_id)
A single pilot record. Does not include custom attributes. Returns `Pilot`.
- `pilot_id` (int): ID of pilot record to retrieve

#### db.pilot_attributes(pilot_or_id)
All custom attributes assigned to pilot. Returns `list[PilotAttribute]`.
- `pilot_or_id` (pilot|int): Either the pilot object or the ID of pilot

#### db.pilot_attribute_value(pilot_or_id, name, default_value=None)
The value of a single custom attribute assigned to pilot. Returns `string`.
- `pilot_or_id` (pilot|int): Either the pilot object or the ID of pilot
- `name` (string): attribute to retrieve
- `default_value` _(optional)_: value to return if attribute does not exist

#### db.pilot_add(name=None, callsign=None, phonetic=None, team=None, color=None)
Add a new pilot to the database. Returns the new `Pilot`.
- `name` _(optional)_ (string): Name for new pilot
- `callsign` _(optional)_ (string): Callsign for new pilot
- `phonetic` _(optional)_ (string): Phonetic spelling for new pilot callsign
- `team` _(optional)_ (string): Team for new pilot
- `color` _(optional)_ (string): Color for new pilot

#### db.pilot_alter(pilot_id, name=None, callsign=None, phonetic=None, team=None, 
Alter pilot data. Returns the altered `Pilot`
- `pilot_id` (int): ID of pilot to alter
- `name` _(optional)_ (string): New name for pilot
- `callsign` _(optional)_ (string): New callsign for pilot
- `phonetic` _(optional)_ (string): New phonetic spelling of callsign for pilot
- `team` _(optional)_ (string): New team for pilot
- `color` _(optional)_ (string): New color for pilot
- `attributes` _(optional)_ (list[dict]): Attributes to alter, attribute values assigned to respective keys

#### db.pilot_delete(pilot_or_id)
Delete pilot record. Fails if pilot is associated with saved race. Returns `boolean` success status.
- `pilot_or_id` (int): ID of pilot to delete 

#### db.pilots_clear()
Delete all pilot records. No return value.


### Heats
Heats are collections of pilots upon which races are run. A heat may first be represented by a heat plan which defines methods for assigning pilots. The plan must be seeded into pilot assignments in order for a race to be run.

Heats are represented with the `Heat` class, which has the following properties:
- `id` (int): Internal identifier
- `name` (string): User-facing name
- `class_id` (int): ID of associated race class
- `results` (dict|None): Calculated cumulative heat summary results
- `_cache_status`: Internal use only
- `order` (int): Not yet implemented
- `status` (HeatStatus): Current status of heat as `PLANNED` or `CONFIRMED`
- `auto_frequency` (boolean): True to assign pilot seats automatically, False for direct assignment
- `active` (boolean): Not yet implemented

The sentinel value `RHUtils.HEAT_ID_NONE` should be used when no heat is defined.

#### db.heats
_Read only_
All heat records. Returns `list[Heat]`

#### db.heat_by_id(heat_id)
A single heat record. Returns `Heat`.
- `heat_id` (int): ID of heat record to retrieve

#### db.heats_by_class(raceclass_id)
All heat records associated with a specific class. Returns `list[Heat]`
- `raceclass_id` (int): ID of raceclass used to retrieve heats

#### db.heat_results(heat_or_id)
The calculated summary result set for all races associated with this heat. Returns `dict`.
- `heat_or_id` (int|Heat): Either the heat object or the ID of heat

#### db.heat_max_round(heat_id)
The highest-numbered race round recorded for selected heat. Returns `int`.
- `heat_id` (int): ID of heat

#### db.heat_add(name=None, raceclass=None, auto_frequency=None)
Add a new heat to the database. Returns the new `Heat`.
- `name` _(optional)_ (string): Name for new heat
- `raceclass` _(optional)_ (int): Raceclass ID for new heat
- `auto_frequency` _(optional)_ (boolean): Whether to enable auto-frequency

#### db.heat_duplicate(source_heat_or_id, dest_class=None)
Duplicate a heat record. Returns the new `Heat`.
- `source_heat_or_id` (int|heat): Either the heat object or the ID of heat to copy from
- `dest_class` _(optional)_ (int): Raceclass ID to copy heat into

#### db.heat_alter(heat_id, name=None, raceclass=None, auto_frequency=None, status=None)
Alter heat data. Returns tuple of this `Heat` and affected races as `list[SavedRace]`.
- `heat_id` (int): ID of heat to alter
- `name` _(optional)_ (string): New name for heat
- `raceclass` _(optional)_ (int): New raceclass ID for heat
- `auto_frequency` _(optional)_ (boolean): New auto-frequency setting for heat
- `status` _(optional)_ (HeatStatus): New status for heat

#### db.heat_delete(heat_or_id)
Delete heat. Fails if heat has saved races associated or if there is only one heat left in the database. Returns `boolean` success status.
- `heat_or_id` (int|heat): ID of heat to delete

#### db.heats_clear()
Delete all heat records. No return value.


### Heat &rarr; Slots
Slots are data structures containing a pilot assignment or assignment method. Heats contain one or more Slots corresponding to pilots who may participate in the Heat. When a heat is calculated, the `method` is used to reserve a slot for a given pilot. Afterward, `pilot` contains the ID for which the space is reserved. A Slot assignment is only a reservation, it does not mean the pilot has raced regardless of heat `status`.

Slots are represented with the `HeatNode` class, which has the following properties:
- `id` (int): Internal identifier
- `heat_id` (int): ID of heat to which this slot is assigned
- `node_index` (int): slot number
- `pilot_id` (int|None): ID of pilot assigned to this slot
- `color` (string): hexadecimal color assigned to this slot
- `method` (ProgramMethod): Method used to implement heat plan
- `seed_rank` (int): Rank value used when implementing heat plan
- `seed_id` (int): ID of heat or class used when implementing heat plan

`Database.ProgramMethod` defines the method used when a heat plan is converted to assignments:
- `ProgramMethod.NONE`: No assignment made
- `ProgramMethod.ASSIGN`: Use pilot already defined in `pilot_id`
- `ProgramMethod.HEAT_RESULT`: Assign using `seed_id` as a heat designation
- `ProgramMethod.CLASS_RESULT`: Assign using `seed_id` as a race class designation

Import `ProgramMethod` with:
`from Database import ProgramMethod`

#### db.slots
_Read only_
All slot records. Returns `list[HeatNode]`.

#### db.slots_by_heat(heat_id)
Slot records associated with a specific heat. Returns `list[HeatNode]`.
- `heat_id` (int): ID of heat used to retrieve slots

#### db.slot_alter(slot_id, pilot=None, method=None, seed_heat_id=None, seed_raceclass_id=None, seed_rank=None)
Alter slot data. Returns tuple of associated `Heat` and affected races as `list[SavedRace]`.
- `slot_id` (int): ID of slot to alter
- `pilot` _(optional)_ (int): New ID of pilot assigned to slot
- `method` _(optional)_ (ProgramMethod): New seeding method for slot
- `seed_heat_id` _(optional)_ (): New heat ID to use for seeding
- `seed_raceclass_id` _(optional)_ (int): New raceclass ID to use for seeding
- `seed_rank` _(optional)_ (int): New rank value to use for seeding

#### db.slot_alter_fast(slot_id, pilot=None, method=None, seed_heat_id=None, seed_raceclass_id=None, seed_rank=None
Like `slot_alter`, but intended to be used when many alterations need done at once. Does not check for some types of invalid input. Does not trigger events, clear results, or update cached data. These operations must be done manually if required. No return value.
- `slot_id` (int): ID of slot to alter
- `pilot` _(optional)_ (int): New ID of pilot assigned to slot
- `method` _(optional)_ (ProgramMethod): New seeding method for slot
- `seed_heat_id` _(optional)_ (): New heat ID to use for seeding
- `seed_raceclass_id` _(optional)_ (int): New raceclass ID to use for seeding
- `seed_rank` _(optional)_ (int): New rank value to use for seeding

With `method` set to `ProgramMethod.NONE`, most other fields are ignored. Only use `seed_heat_id` with `ProgramMethod.HEAT_RESULT`, and `seed_raceclass_id` with `ProgramMethod.CLASS_RESULT`, otherwise the assignment is ignored.


### Race Classes
Race Classes are groups of related heats. Classes may be used by the race organizer in many different ways, such as splitting sport and pro pilots, practice/qualifying/mains, primary/consolation bracket, etc.

Race Classes are represented with the `RaceClass` class, which has the following properties:
- `id` (int): Internal identifier
- `name` (string): User-facing name.
- `description` (string): User-facing long description, accepts markdown
- `format_id` (int): ID for class-wide required race format definition
- `win_condition` (string): ranking algorithm
- `results` (dict|None): Calculated cumulative race class results
- `_cache_status`: Internal use only.
- `ranking` (dict|None): Calculated race class ranking
- `rank_settings` (string): JSON-serialized arguments for ranking algorithm
- `_rank_status`: Internal use only.
- `rounds` (int): Number of expected/planned rounds each heat will be run
- `heat_advance_type` (atAdvanceType'
- `order` (int): Not yet implemented
- `active` (boolean): Not yet implemented

The sentinel value `RHUtils.CLASS_ID_NONE` should be used when no race class is defined.

`Database.HeatAdvanceType` defines how the UI will automatically advance heats after a race is finished.
- `HeatAdvanceType.NONE`: Do nothing
- `HeatAdvanceType.NEXT_HEAT`: Advance heat; if all `rounds` run advance race class
- `HeatAdvanceType.NEXT_ROUND`: Advance heat if `rounds` has been reached; advance race class after last heat in class

#### db.raceclasses
_Read only_
All race class records. Returns `list[RaceClass]`.

#### db.raceclass_by_id(raceclass_id)
A single race class record. Returns `RaceClass`.
- `raceclass_id` (int): ID of race class record to retrieve

#### db.raceclass_add(name=None, description=None, raceformat=None, win_condition=None, rounds=None, heat_advance_type=None)
Add a new race class to the database. Returns the new `RaceClass`.
- `name` _(optional)_ (string): Name for new race class
- `description` _(optional)_ (string): Description for new race class
- `raceformat` _(optional)_ (int): ID of format to assign
- `win_condition` _(optional)_ (string): Class ranking identifier to assign
- `rounds` _(optional)_ (int): Number of rounds to assign to race class
- `heat_advance_type` _(optional)_ (HeatAdvanceType): Advancement method to assign to race class

#### db.raceclass_duplicate(source_class_or_id)
Duplicate a race class. Returns the new `RaceClass`.
- `source_class_or_id` (int|RaceClass): Either the race class object or the ID of race class

#### db.raceclass_alter(raceclass_id, name=None, description=None, raceformat=None, win_condition=None, rounds=None, heat_advance_type=None, rank_settings=None)
Alter race class data. Returns tuple of this `RaceClass` and affected races as `list[SavedRace]`.
- `raceclass_id` (int): ID of race class to alter
- `name` _(optional)_ (string): Name for new race class
- `description` _(optional)_ (string): Description for new race class
- `raceformat` _(optional)_ (int): ID of format to assign
- `win_condition` _(optional)_ (string): Class ranking identifier to assign
- `rounds` _(optional)_ (int): Number of rounds to assign to race class
- `heat_advance_type` _(optional)_ (HeatAdvanceType): Advancement method to assign to race class
- `rank_settings` _(optional)_ (string):

#### db.raceclass_results(raceclass_or_id)
The calculated summary result set for all races associated with this race class. Returns `dict`.
- `raceclass_or_id` (int|RaceClass): Either the race class object or the ID of race class

#### db.raceclass_ranking(raceclass_or_id)
The calculated ranking associated with this race class. Returns `dict`.
- `raceclass_or_id` (int|RaceClass): Either the race class object or the ID of race class

#### db.raceclass_delete(raceclass_or_id)
Delete race class. Fails if race class has saved races associated. Returns `boolean` success status.
- `raceclass_or_id` (int|RaceClass): Either the race class object or the ID of race class

#### db.raceclasses_clear()
Delete all race classes. No return value.

### Race Formats

The sentinel value `RHUtils.FORMAT_ID_NONE` should be used when no race format is defined.

#### db.raceformats
_Read only_
#### db.raceformat_by_id(format_id)
format_id
#### db.raceformat_add(name=None, unlimited_time=None, race_time_sec=None, lap_grace_
name=None, unlimited_time=None, race_time_sec=None, lap_gracesec=None, staging_fixed_tones=None, staging_delay_tones=None, start_delay_min_ms=None, start_delay_max_ms=None, start_behavior=None, win_condition=None, number_laps_win=None, team_racing_mode=None, points_method=None)
#### db.raceformat_duplicate(source_format_or_id)
source_format_or_id
#### db.raceformat_alter(raceformat_id, name=None, unlimited_time=None, race_time_sec=None, 
raceformat_id, name=None, unlimited_time=None, race_time_sec=None,lap_grace_sec=None, staging_fixed_tones=None, staging_delay_tones=None, start_delay_min_ms=None, start_delay_max_ms=None, start_behavior=None, win_condition=None, number_laps_win=None, team_racing_mode=None, points_method=None, points_settings=None)
#### db.raceformat_delete(raceformat_id)
raceformat_id
#### db.raceformats_clear()

### Frequency Sets

The sentinel value `RHUtils.FREQUENCY_ID_NONE` should be used when no frequency is defined.

#### db.frequencysets
_Read only_
#### db.frequencyset_by_id(set_id)
set_id
#### db.frequencyset_add(name=None, description=None, frequencies=None, enter_ats=None, exit
name=None, description=None, frequencies=None, enter_ats=None, exi_ats=None)
#### db.frequencyset_duplicate(source_set_or_id)
source_set_or_id
#### db.frequencyset_alter(set_id, name=None, description=None, frequencies=None, enter_
set_id, name=None, description=None, frequencies=None, enterats=None, exit_ats=None)
#### db.frequencyset_delete(set_or_id)
set_or_id
#### db.frequencysets_clear()

### Saved Races

#### db.races
_Read only_
#### db.race_by_id(race_id)
race_id
#### db.race_by_heat_round(heat_id, round_number)
heat_id, round_number
#### db.races_by_heat(heat_id)
heat_id
#### db.races_by_raceclass(raceclass_id)
raceclass_id
#### db.race_results(race_or_id)
race_or_id
#### db.races_clear()

### Saved Race &rarr; Pilot Runs

#### db.pilotruns
_Read only_
#### db.pilotrun_by_id(run_id)
run_id
#### db.pilotrun_by_race(race_id)
race_id

### Saved Race &rarr; Pilot Run &rarr; Laps

#### db.laps
_Read only_
#### db.laps_by_pilotrun(run_id)
run_id

### Options

#### db.options
_Read only_
#### db.option(name, default=False, as_int=False)
name, default=False, as_int=False
#### db.option_set(name, value)
name, value
#### db.options_clear()



## Data input/output

View and import/export data from the database via registered `DataImporters` and `DataExporters`.
These methods are accessed via `RHAPI.io` 

#### io.exporters
_Read only_
#### io.run_export()
#### io.importers
_Read only_
#### io.run_import(importer_id, data, import_args=None)



## Heat Generation

View and Generate heats via registered `HeatGenerators`.
These methods are accessed via `RHAPI.heatgen` 

#### heatgen.generators
_Read only_
#### heatgen.run_export(generator_id, generate_args)
generator_id, generate_args



## Class Ranking Methods

View registered `RaceClassRankMethods`.
These methods are accessed via `RHAPI.classrank` 

#### classrank.methods
_Read only_



## Race Points Methods

View registered `RacePointsMethods`.
These methods are accessed via `RHAPI.points` 

#### points.methods
_Read only_



## LED

Activate and manage connected LED displays via registered `LEDEffects`.
These methods are accessed via `RHAPI.led` 

#### led.enabled
_Read only_
#### led.effects
_Read only_
#### led.effect_by_event(event)
event
#### led.effect_set(event, name)
event, name
#### led.clear
#### led.display_color(seat_index, from_result=False)
seat_index, from_result=False
#### led.activate_effect(args)
args



## Video Receivers

View and manage connected Video Receiver devices.
These methods are accessed via `RHAPI.vrxcontrol` 

#### vrxcontrol.enabled
_Read only_
#### vrxcontrol.status
_Read only_
#### vrxcontrol.devices
_Read only_
#### vrxcontrol.kill
#### vrxcontrol.devices_by_pilot(seat, pilot_id)
seat, pilot_id



## Active Race

View and manage the currently active race.
These methods are accessed via `RHAPI.race` 

#### race.pilots
_Read only_
#### race.teams
_Read only_
#### race.slots
_Read only_
#### race.seat_colors
_Read only_
#### race.heat
_Read only_
#### race.frequencyset
_Read only_
#### race.raceformat
_Read only_
#### race.status
_Read only_
#### race.stage_time_internal
_Read only_
#### race.start_time
_Read only_
#### race.start_time_internal
_Read only_
#### race.end_time_internal
_Read only_
#### race.seats_finished
_Read only_
#### race.laps
_Read only_
#### race.any_laps_recorded
_Read only_
#### race.laps_raw
_Read only_
#### race.laps_active_raw(filter_late_laps=False)
filter_late_laps=False
#### race.results
_Read only_
#### race.team_results
_Read only_
#### race.scheduled
_Read only_

#### race.schedule(sec_or_none, minutes=0)
sec_or_none, minutes=0
#### race.stage
#### race.stop(doSave=False)
doSave=False



## Event Results

View or clear result data for all races, heats, classes, and event totals.
These methods are accessed via `RHAPI.eventresults` 

#### eventresults.results
_Read only_
#### eventresults.results_clear



## Language and Translation

View and retrieve loaded translation strings.
These methods are accessed via `RHAPI.language` 

#### language.languages
_Read only_
#### language.dictionary
_Read only_
#### language.\_\_(text, domain='')
text, domain=''



## Hardware Interface

View information provided by the harware interface layer.
These methods are accessed via `RHAPI.interface` 

#### interface.seats
_Read only_



## Sensors Interface

View data collected by environmental sensors such as temperature, voltage, and current.
These methods are accessed via `RHAPI.sensors` 

#### sensors.sensors_dict
_Read only_
#### sensors.sensor_names
_Read only_
#### sensors.sensor_objs
_Read only_
#### sensors.sensor_obj(name)
name