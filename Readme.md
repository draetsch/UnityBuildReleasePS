# Unity Powershell Scripts for Batch Build & Release

This is a set of Powershell scripts I wrote for our Unity game which allows us
to build multiple configurations easily, and to release to Steam and Itch
repeatably.

## Build tool

The **build** tool performs a number of tasks:

1. Increments the version number
2. Tags the git repo
3. Builds the combination of variants in one go:
   * Multiple platform targets
   * Development mode
   * Different #define modes (currently just used to enable/disable Steam specific code)
4. Places output in sensible version-and-variant labelled folders
5. Protects you from silly mistakes e.g. forgetting to build from a clean copy

```
Old Doorways Unity Build Tool
Usage:
  build.ps1 [-src:sourcefolder] [-major|-minor|-patch|-hotfix] [-keepversion] [-force] [-devonly] [-prodonly] [-skipsteam] [-test] [-dryrun]

  -src         : Source folder (current folder if omitted), must contain buildconfig.json
  -major       : Increment major version i.e. [x++].0.0.0
  -minor       : Increment minor version i.e. x.[x++].0.0
  -patch       : Increment patch version i.e. x.x.[x++].0
  -hotfix      : Increment hotfix version i.e. x.x.x.[x++]
               : (-patch is assumed if none are supplied)
  -keepversion : Keep current version number
  -force       : Move version tag
  -devonly     : Build development version only (builds both otherwise)
  -prodonly    : Build production version only (builds both otherwise)
  -skipsteam   : Skip the Steam build
  -test        : Testing mode, don't fail on dirty working copy etc
  -dryrun      : Don't perform any actual actions, just report what would happen
  -help        : Print this help
```

## Release tool

The **release** tool automates uploading builds to Steam and Itch.io:

```
Old Doorways Release Tool
Usage:
  release.ps1 [-src:sourcefolder] -version:ver -service:svc [-windows:bool] [-mac:bool] [-dryrun]

  -src         : Source folder (current folder if omitted), must contain buildconfig.json
  -version:ver : Version to release
  -service:svc : 'steam' or 'itch'
  -windows:b   : Whether to release for Windows (default true)
  -mac:b       : Whether to release for Mac (default true)
  -dryrun      : Don't perform any actual actions, just report what would happen
  -help        : Print this help
```

## Requirements

1. Unity project using 2017.3 or newer
2. [Multibuild](https://github.com/sinbad/UnityMultiBuild) installed - this tool
   uses its batch mode for simplicity
3. Powershell v5.1+ installed
4. Steamworks SDK installed if you're releasing to Steam
5. Itch.io's `butler` tool installed if you're releasing to Itch

## How to use

1. Place the contents of this folder anywhere you like
2. Make sure you have [Multibuild](https://github.com/sinbad/UnityMultiBuild) in
   your project assets
3. Place a file called `buildconfig.json` in your Unity project root, see `buildconfig_template.json`, see below for more discussion
4. Run `build.ps1` or `release.ps1` as above, use `-help` to get assistance
  * You can either run them from your Unity project root (run `\path\to\build.ps1`), or from their own directory and specify the Unity project path with `-src:\path\to\unity\project`


## Build Config

The build config file is just a JSON file telling the tools how to behave. There
is a template in `buildconfig_template.json`, and the meaning of the properties
is as follows:

* **BuildDir**: Path to the location in which build output will be produced.
  It can be relative, in which case it is relative to the Unity project folder.
  * Subdirectories will be created while building, of the form {BuildDir}/{version}/{general|steam}/{target}/
* **ReleaseDir**: Path to the folder where zipped release versions will be created
* **Targets**: Array of strings of targets to build. These must match the enum names of [`MultiBuild.Target`](https://github.com/sinbad/UnityMultiBuild/blob/866b2bb2d2d816e6244b7df5f33335df425f1802/Assets/MultiBuild/Editor/Settings.cs#L9), e.g. "Mac64"
* **AssemblyInfo**: Path to the AssemblyInfo.cs file containing the version number.
  This will be updated when bumping the version number
* **ItchAppId**: The id of your application at [Itch.io](https://itch.io) if you're releasing there
* **ItchChannelByTarget**: Dictionary mapping the target names specified previously to Itch.io channel names
* **SteamAppId**: The ID of your app on Steam if you're releasing there
* **SteamDepotsByTarget**: A dictionary mapping target names to Depot IDs in Steam.
  Currently only 1 depot per target is supported (1 per platform)
* **SteamLogin**: The login name you'll use to upload to Steam

