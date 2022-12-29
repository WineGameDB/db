WineGameDB documentation:

"games" table:
  Required fields:
    "id":                WineGameDB-internal game id
    "internal_name":     The internal name of the game (Epic: AppName, GOG: Game ID)
    "title":             The actual title of the game as it's shown on the storefront
    "store":             The storefront the game was obtained at. Used for differentiating between different versions. May be "Epic" or "GOG"
    "status":            Playability rating of the game:
                          - 0: Does not work
                          - 1: Works OOTB. Note that this assumes
                              (1) the game is running in a clean prefix and
                              (2) you're using the latest version of Wine-GE to run the game
                          - 2: Works with workarounds
                          - 3: Main game works, some content (multiplayer) is inaccessible
                          - 4: Unknown
  Optional fields:
    "status_conditions": The game might have a different playability rating in certain scenarios. Conditions are structured as follows:
                         ```
                         condition1
                         playability rating if condition1 is true
                         condition2
                         playability rating if condition2 is true
                         ```

"workarounds" table:
  Required fields:
    "id":                WineGameDB-internal workaround id
    "title":             The workaround's title (in your launcher this could be used to display what's currently being done)
  Optional fields:
    "arg_titles":        If the workaround takes arguments, these are the titles. Separated by newline characters (\n)

"games_workarounds" table:
Joins together games and workarounds. Iterate over this with your game_id, find all workarounds that are listed for it, and run them
  Required fields:
    "game_id":           The Game ID the workaround is for
    "workaround_id":     The workaround that should be applied
    "is_optional":       If the main game works without this, but extra features/functionality is added, this will be true
    "is_conditional":    If the workaround is only required in certain scenarios, this will be true
  Optional fields:
    "args":              The arguments supplied to the workaround (again separated with \n)
    "optional_feature":  If the workaround is optional, this will describe what's added by running it
    "conditions":        The condition(s) required for the workaround (also separated with \n)


Documentation on specific workarounds:
It is up to your launcher to actually do the work of running a workaround, this is only a written guide for you to implement:
 - "override_exe"
   Launch a different executable to the one defined in the game's metadata. The path will be relative to the game's installation folder
 - "start_params"
   Launch the executable and add these start parameters
 - "env_var"
   Set an environment variable
 - "copy_file"
   Copy a file. Args: "src", "dst", "symlink=false"; should be self-explanatory
   If dst is a folder, copy src into it. If it is a file, copy src and rename it if necessary. Overwrite files if necessary
   Example:
   ```
   copy_file
   $GAMEDIR/EasyAntiCheat/easyanticheat_x64.so
   $GAMEDIR/FallGuys_client_game_Data/Plugins/x86_64/
   true
   ```
 - "edit_ini"
   Edit an INI file. Args are (1) path to the file, (2) section, (3) key name and (4) value to set
   Note: Even though it isn't valid INI, some games use INI files without sections. Thus, the 2nd argument (section) might be empty
   Example:
   ```
   edit_ini
   $GAMEDIR/FallGuys_client.ini

   TargetApplicationPath
   FallGuys_client_game.exe
   ```
 - "virtualdesktop"
   Wine's Virtual Desktop setting.
   Add a new string value in HKCU\Software\Wine\Explorer\Desktops. Name it "Default" and set it to the monitor's resolution ($SCREEN_RES). In addition to that, add another string value in HKCU\Software\Wine\Explorer. Name it "Desktop" and set its value to "Default"
 - "nocrashdialog"
   Add a DWORD registry value named "ShowCrashDialog" in HKCU\Software\Wine\WineDbg. Set it to 0
 - "d3dcompiler"
   MS Direct3D compiler (d3dcompiler_XX.dll), first argument is used to specify the version
   Obtain the respective DLL files (a mirror is available in this repo), copy them to sys32/syswow64 and set them to native
 - "vcruntime"
   MS Visual C++ Runtime
   Obtain the installers (https://docs.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#visual-studio-2015-2017-2019-and-2022) and run them (optionally with the /silent argument)
   Note: Install both the x64 and x86 installers
 - "dxvk"
   doitsujin's DXVK (https://github.com/doitsujin/dxvk)
   Arg: `version` (version of DXVK to install, may be empty to mean "latest")
   Download the release and either run the setup or perform what it does yourself
 - "vkd3d"
   HansKristian-Work's VKD3D-Proton (https://github.com/HansKristian-Work/vkd3d-proton)
   Arg: `version` (version of VKD3D to install, may be empty to mean "latest")
   Download the release and either run the setup or perform what it does yourself
 - "custom_wine"
   Use a custom Wine version (instead of latest Wine-Staging)
   Arguments:
    - name:    `wine-staging`, `wine-ge` or `wine_lutris`
    - version: Specific version to use (tagname of GH release). May be empty to mean "latest"


Variables:
Some workarounds will have special variables that change based on the system configuration. They are indicated by a $ (e.g."$SCREEN_WIDTH")
 - "SCREEN_WIDTH"
   The horizontal resolution of the user's main monitor (in pixels, e.g. 1920)
 - "SCREEN_HEIGHT"
   The vertical resoltion of the user's main monitor (in pixels, e.g. 1080)
 - "SCREEN_RES"
   Shorthand for `$SCREEN_WIDTHx$SCREEN_HEIGHT`
 - "GPU_VENDOR"
   The user's main GPU (valid values are "nvidia", "amd" and "intel")
 - "GAMEDIR"
   The install directory of the game, with no trailing /


Conditions:
Conditions are rather basic for now. They are usually used with variables
  - `value1=value2`  Check equality, e.g. `$GPU_VENDOR=nvidia`
  - `value1!=value2` Check inequality, e.g. `$GPU_VENDOR!=intel`


Acknowledgements:
  - This project hevily borrows from Lutris' idea of game install scripts & Proton's protonfixes
  - While this DB is primarily intended for the Heroic Games Launcher, other projects are free to use it as well
