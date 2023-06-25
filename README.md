# DbScriptExtensions

DbScriptExtensions is a Lua framework for the [Eluna Lua Engine ©](https://github.com/ElunaLuaEngine/Eluna) that allows you to create, load, modify and delete database entities via your own Lua code.

## Table of Contents

  - [Installation](#installation)
  - [Examples](#examples)
    - [Creating a new database object](#creating-a-new-database-object)
    - [Get unused primary keys](#get-unused-primary-keys)
    - [Loading a database object](#loading-a-database-object)
    - [Updating an existing database object](#updating-an-existing-database-object)
    - [Delete an existing database object](#delete-an-existing-database-object)
  - [FAQ](#faq)

## Installation
* Copy the content of this repository to the folder `lua_scripts/extensions/DbScriptExtensions`.
  
  To check if you copied it correctly, make sure you find the README file under `lua_scripts/extensions/DbScriptExtensions/README.md`
* Run the world server of the emulator of your choice that supports [Eluna Lua Engine ©](https://github.com/ElunaLuaEngine/Eluna)
* After the server finished loading all Lua scripts, it will show you the message `"[DbScriptExtensions] Table classes generated"`
* Close the world server
* Open the file `lua_scripts/extensions/DbScriptExtensions/DbScriptExtensions.ext` and edit the line `DbScriptExtensions_GenerateTableFiles = true` to `DbScriptExtensions_GenerateTableFiles = false`
* Run the world server again and use DbScriptExtensions in your Lua scripts

## Examples
### Creating a new database object

```lua
local newCreature = DbCreature()
--[[
    Make sure that the primary key of a table (in this example "guid" of table "creature") isn't already existing, otherwise the object will not properly track changes to existing values in the database.

    All column values are automatically set to the default database values when you're creating a new object.
]]
newCreature.guid = 123123123 
newCreature:Save()
```

### Get unused primary keys
```lua
-- Use this while creating new entries without a fixed primary key
local race, class = DbPlayercreateinfo():GetNewPrimaryKeys() -- This will return race = 12, class = 12 because the highest race and class in the table playercreateinfo are ID 11. It will always return all primary key columns.
```

### Loading a database object
```lua
-- Way 1
local existingCreature = DbCreature() -- Create new object of database table creature
existingCreature:Load(123) -- Load data of entry with guid = 123 (guid is the primary key of creature)

-- Way 2
local existingCreature = DbCreature():Load(123)

-- You can now access the columns like a normal Lua table. Attribute names are always the same as the column names in the table (case sensitive!).
print(existingCreature.map) -- Prints the map id of the loaded creature

-- Load entry with combined primary keys
local existingPlayerCreateInfo = DbPlayercreateinfo():Load(1, 4) -- Table playercreateinfo has a combined primary key based on column race and class. The order will always be the same as in the database which means we're loading the entry with race = 1 AND class = 4
```

### Updating an existing database object
```lua
local existingCreature = DbCreature():Load(123) 
existingCreature.map = 1337 -- Set map column value to 1337
existingCreature:Save() -- Save changes to database, will insert a new entry with guid 123 or update existing entry with guid 123
```

### Delete an existing database object
```lua
local existingCreature = DbCreature():Load(123) 
existingCreature:Delete() -- Delete row with guid = 123
```

## FAQ
### Which emulators does it support?
I only tested it on AzerothCore and TrinityCore, but it should work on all emulators supported by Eluna.

### What can i do when i see errors in my world server console?
Set `DbScriptExtensions_PrintQueries = true` in your file `lua_scripts/extensions/DbScriptExtensions/DbScriptExtensions.ext`. Now restart the world server. It should show all DbScriptExtensions SQL queries that it tries to run. Now you can open up an issue on the GitHub repository with the query it tried to execute and a snippet of your code that leads to this error. Make sure to mention which emulator you're using.

### What can i do when i updated my emulator and database and now i see errors in my world server console?
There were probably some changes in your database because of the update. Try to regenerate the table files by deleting all files under `lua_scripts/extensions/DbScriptExtensions/Mapping` (Do not delete the Mapping folder! Regenerating will fail if you do this!) and setting `DbScriptExtensions_GenerateTableFiles = true` in your file `lua_scripts/extensions/DbScriptExtensions/DbScriptExtensions.ext`. Now you can reload Eluna by typing `reload eluna` into your console or by typing `.reload eluna` ingame. Afterwards set the option back to false and restart your world server. If the error still persists, report the problem as explained in the first question.

### Are all column names case sensitive?
Yes.

### Does DbScriptExtensions support my custom tables?
Yes. After adding a new table to your world database, just regenerate the table files as explained in second question. Make sure to only use MySQL data types that are supported by your emulator. If i missed a type and you encounter an error, make sure to report it as a GitHub issue.

### How do i figure out the class name of a specific table?
The mapping is done in the file `lua_scripts\extensions\DbScriptExtensions\Mapping\DbScriptExtensions_Mappings.ext`. You can find all class names in there.

### Are there any attributes that could help me to create objects in a generic way?
There are several internal helper attributes that you can use. Also check the Queryable class for functions that you could call.
Example of internal helper attributes:
```lua
    local creature = DbCreature():Load(123)
    local guid = creature.__columns["guid"] -- Access column directly by name, same value as creature.guid
    local guidColumnId = creature.__columnIds["guid"] -- Will return 0 because guid is the first column in the table creature (it's 0 based because the ElunaQuery functions are 0 based. Lua is usually 1 based)
    local guidColumnName = creature.__columnById[guidColumnId] -- Will return "guid" because column 1 is guid as seen in the line above
    local amountOfColumns = creature.__columnIdCounter -- Will return 24 because the table creature has 24 columns
    local elunaReadColumnFunction = creature.__columnFunctions["guid"] -- Will return function ElunaQuery:GetUInt32 because this is the function to read the value out of an query object. Use like this: elunaReadColumnFunction(query, columnId)
    local queryObject = WorldDBQuery("SELECT * FROM creature WHERE guid = 123")
    if (queryObject) then
        local guid = elunaReadColumnFunction(queryObject, guidColumnId) -- Will return 123
    end
```

### Where can i get support?
You can find me on the official [Eluna Discord server](https://discord.gg/Ed5HK3Dc). Ask your question in the support channel and tag me (kittykaev).

### Can i support your work?
Sure. Contact me via Discord (kittykaev).
