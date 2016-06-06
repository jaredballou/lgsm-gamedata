# Game Data Files
## General Info

These files are the proof of concept of my new method of supporting all the games LGSM covers. It's basically a hierarchial way to define three things:

 * **Script Parameters:** These are things like server_executable, game name, local directories, and all the rest.
 * **Server Parameters:** These are the command-line switches that we give to the actual game server daemon. There is a little bit of smarts around the Source and GoldSource parsers, we feed it "minus parameters" and "plus parameters", and it spits them out in a somewhat sane order.
 * **Server Settings:** These are the items that go into \_default.yaml for each game. They include the values for the two types of parameters, and are overridden hierarchially by sourcing \_default.yaml, then \_common.yaml, then $instance.yaml from the cfg/servers directory.

The new method uses a hierarchy of files like so:
 * **engine:** Engine-specific files, like Source or Unreal4.
 * **game:** Each directory correlates to a game, the name of the directory is what the installer will name the script it will install for the root instance.
  * **cfg:** Templates for game-specific configs
  * **name.yaml:** The core gamedata file for this game
  * **mods:** Optional path where mod config files are stored.
 * **platform:** Content delivery platforms. Currently only Steam is supported.

The gamedata files themselves use YAML with Python dictionary replacement, so they are merged from the bottom up properly and then interpolated. This lets us keep variable names in the gamedata and script config files, and have it all process at runtime.
 * **include:** An ordered list of files that this gamedata file should include. Omit the .yaml extension, the path is relative to the directory this README is in.
 * **settings:** Configuration settings. These can be processed as server parameters, replacement values for the templates to generate game configs, and anything else that needs to be set. Each item will be referenced by name, and has the following fields:
  * **parm:** If this setting needs to be fed to the server server_executable, put the prefix needed here. Set as an empty string if you just want to feed the value of the setting without any interpolation.
  * **default:** Default value. This will populate the lgsm\/%(game)\/cfg\/\_default.yaml file.
  * **desc:** Description of the setting
  * **value:** Override the default setting. This should only be used for things like mods, where we basically want a different default that will be forced. For the most part, this should be left alone.
 * **dependencies:** Files that need to be updated to work, Glibc and such. Each item is the name of the file, and has the following fields:
  * **checksum:** The checksum of the file to use. This is an MD5 checksum, and will pull down "%(name).%(checksum)" from the dependencies repo.
 * **scriptactions:** These are the command-line actions that the LGSM script invokes to actually do the work. Each item will show up in the list of actions on the help screen, and has the following fields:
  * **module:** Either a string with a single module, or an array, that will need to be satisfied before this action can be invoked.
  * **function:** Name of the function to invoke to run the action.
  * **parms:** Parameters to pass to the function. These are sent as a dict.
  * **desc:** Description
  * **aliases:** Additional aliases for invoking this action
 * **config:** This is the special key-value dictionary that drives the script config. After loading all the gamedata files, the settings section is parsed into LGSM and then this is laid on top as the final layer. For the most part, the only things that go in here are script-global items like the lgsm data directory, and pre-flight variables like date and paths.
 * **modules:** This is not yet implemented, but will include a list of required Python modules that need to be pulled down/updated for this game. This will support Pip or files from GitHub.

## TODO

 * [ ] Look into better handling of parms, especially with the "-" and "+" ordering in Source.
  * Perhaps put a "before" and "after" field in the parms, so we can do a little more complex ordering?
 * [ ] Clean up gamedata files for all engines/games.
 * [ ] When \_default.yaml updates, read all other configs. Add in commented key/value/comment lines so that other configs have the keys and default values available.
 * [ ] Add dependency installation for games, simple array of packages needed for debian,ubuntu,redhat for each game.
 * [ ] Allow values to append or replace individual items, i.e. for dependencies layer on the needed packages from \_default \_engine and game data files.
 * [ ] Parser should read the value and identify variable names, and make sure that this key is declared after those variables that the value references.
