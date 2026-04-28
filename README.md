# Pawn.CMD
🚀 Plugin-powered (component) command processor for open.mp (Open Multiplayer, OMP) server
## Natives
```pawn
native PC_RegAlias(const cmd[], const alias[], ...);
native PC_SetFlags(const cmd[], flags);
native PC_GetFlags(const cmd[]);
native PC_RenameCommand(const cmd[], const newname[]);
native PC_CommandExists(const cmd[]);
native PC_DeleteCommand(const cmd[]);
native PC_SetDescription(const cmd[], const description[]);
native PC_GetDescription(const cmd[], dest[], maxlength = sizeof dest);

native CmdArray:PC_GetCommandArray();
native CmdArray:PC_GetAliasArray(const cmd[]);
native PC_GetArraySize(CmdArray:arr);
native PC_GetCommandName(CmdArray:arr, index, dest[], size = sizeof dest);
native PC_FreeArray(&CmdArray:arr);

native PC_EmulateCommand(playerid, const cmdtext[]);
```
## Callbacks
```pawn
forward PC_OnInit();
forward OnPlayerCommandReceived(playerid, cmd[], params[], flags);
forward OnPlayerCommandPerformed(playerid, cmd[], params[], result, flags);
```
## How to install
Extract the archive in your server folder. Place .dll/.so inside **components** folder. Update your server.cfg/config.json

## Configuration (pawncmd.cfg in components folder)
**The values in parentheses are default values**
* CaseInsensitivity (**true**)
* LegacyOpctSupport (**true**)
* LocaleName (**"C"**)
* UseCaching (**true**)
## Example command
```pawn
#include <Pawn.CMD>

cmd:help(playerid, params[]) // you can also use 'CMD' or 'COMMAND' instead of 'cmd'
{
  // code here
  return 1;
}
```
## Registering aliases
You can register aliases for a command
```pawn
#include <Pawn.CMD>

cmd:help(playerid, params[])
{
  // code here
  return 1;
}
alias:help("commands", "cmds")
```

## Using Descriptions
You can add descriptions to your commands like this.
```
desc:test("This is a test command.")
CMD:test(playerid) {
  // code here
  return 1;
}

description:test("This is a test command.")
CMD:test(playerid) {
  // code here
  return 1;
}

CMDD:test(playerid)("This is a test command.") {
  // code here
  return 1;
}
```
You can also change the descriptions in operation.
```
PC_SetDescription("test", "new description");
```
You can also get the description for a list of admin commands or something similar.
```
new description[32];
PC_SetDescription("test", description);
```
> [!NOTE]
> **Command Descriptions & Initialization**
>
> Command descriptions are processed asynchronously when the script loads. If you attempt to fetch descriptions inside `OnGameModeInit`, they may return as empty strings because the plugin is still indexing the data.
>
> To resolve this, you have two options:
> 1. **Recommended:** Use the `PC_OnInit` callback. It is triggered only after the plugin has fully finished its internal setup.
> 2. **Alternative:** If you must use `OnGameModeInit`, implement a **1,000ms delay** before calling `PC_GetDescription`.

## Using flags
You can set flags for a command
```pawn
#include <Pawn.CMD>

enum (<<= 1)
{
  CMD_ADMIN = 1, // 0b00000000000000000000000000000001
  CMD_MODER,     // 0b00000000000000000000000000000010
  CMD_JR_MODER   // 0b00000000000000000000000000000100
};

new pPermissions[MAX_PLAYERS];

flags:ban(CMD_ADMIN)
cmd:ban(playerid, params[])
{
  // code here
  return 1;
}
alias:ban("block")

flags:kick(CMD_ADMIN | CMD_MODER)
cmd:kick(playerid, params[])
{
  // code here
  return 1;
}

flags:jail(CMD_ADMIN | CMD_MODER | CMD_JR_MODER)
cmd:jail(playerid, params[])
{
  // code here
  return 1;
}

public OnPlayerCommandReceived(playerid, cmd[], params[], flags)
{
  if (!(flags & pPermissions[playerid]))
  {
    printf("player %d doesn’t have access to command '%s'", playerid, cmd);

    return 0;
  }

  return 1;
}

public OnPlayerCommandPerformed(playerid, cmd[], params[], result, flags)
{
  if (result == -1)
  {
    SendClientMessage(playerid, 0xFFFFFFFF, "SERVER: Unknown command.");

    return 0;
  }

  return 1;
}

public PC_OnInit()
{
  const testAdminPlayerId = 1, testModerPlayerId = 2, testJrModerPlayerId = 3, testSimplePlayerId = 4;

  pPermissions[testAdminPlayerId] = CMD_ADMIN | CMD_MODER | CMD_JR_MODER;
  pPermissions[testModerPlayerId] = CMD_MODER | CMD_JR_MODER;
  pPermissions[testJrModerPlayerId] = CMD_JR_MODER;
  pPermissions[testSimplePlayerId] = 0;

  PC_EmulateCommand(testAdminPlayerId, "/ban 4 some reason"); // ok
  PC_EmulateCommand(testModerPlayerId, "/ban 8 some reason"); // not ok, moder doesn’t have access to 'ban'
  PC_EmulateCommand(testModerPlayerId, "/kick 15 some reason"); // ok
  PC_EmulateCommand(testModerPlayerId, "/jail 16 some reason"); // ok, moder can use commands of junior moderator
  PC_EmulateCommand(testJrModerPlayerId, "/jail 23 some reason"); // ok
  PC_EmulateCommand(testSimplePlayerId, "/ban 42 some reason"); // not ok
}
```
##### If you want to use Pawn.CMD in a filterscript, put this define before including:
```pawn
#define FILTERSCRIPT
```
