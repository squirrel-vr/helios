# Table of contents

1. [Prerequisites](#prerequisites)
2. [Fresh Install](#fresh-install)
3. [Update Pack](#update-pack)

# Prerequisites

## Install and configure Nebula

Nebula lives in its own folder, separate from the repo; its .env points it at the repo.

```
git clone https://github.com/dscalzi/Nebula.git
cd Nebula
npm i
```

Create a file named `.env` in the Nebula folder (example):

```
JAVA_EXECUTABLE=C:\Program Files\Eclipse Adoptium\jdk-25.0.4.101-hotspot\bin\java.exe
ROOT=C:\Users\bogda\Projects\minecraft\helios
BASE_URL=https://helios.vr-ukraine.com
HELIOS_DATA_FOLDER=C:\Users\bogda\AppData\Roaming\Helios Launcher
```

JAVA_EXECUTABLE must be the path to java.exe on your own PC. "Eclipse Adoptium" is the vendor of the free Temurin JDK,
not the Eclipse IDE.

__Install Temurin JDK 17 from adoptium.net__

then run where java in a terminal (or look under `C:\Program Files\Eclipse Adoptium\`) and paste the path it shows.

- `ROOT` is the cloned GitHub repo of helois.
- `BASE_URL` must end with a slash. Every mod URL in the distribution is built from it, so a typo here breaks every
  download.
- `HELIOS_DATA_FOLDER` is only used for local testing. Rename the last part once you rebrand the launcher.

Then create the folder structure inside the repo:

```
npm run start -- init root
```

This creates `servers/`, `meta/distrometa.json` and `schemas/` in your repo folder.

[go up](#table-of-contents)
# Fresh Install

## 1. Import the pack into Nebula

Nebula has a dedicated command that turns a CurseForge export into a Helios server, including mods, configs and the
loader.

1. Create the folder modpacks/curseforge inside your repo and copy the export zip there.
2. Run the import, choosing a short server ID with no spaces:
   ```
   npm run start -- generate server-curseforge mypack MyPack-1.0.0.zip
   ```
   ### Fabric packs: skip this command. The CurseForge import is built around Forge packs. With Fabric it can miss the loader and drop most mods, so use the manual route below instead.
3. Nebula creates `servers/mypack-<mcversion>/` in your repo. Check its contents:
    - `forgemods/` or `fabricmods/` with required, `optionalon` and `optionaloff` subfolders. Imported mods land in
      required.
    - `files/` with everything from the export's overrides folder (`config`, `options.txt`, and so on), which is copied
      into
      each player's game folder.
    - `servermeta.json` with the loader version filled in.
4. Compare the mod count in required with your CurseForge profile. If any mods are missing, usually because their
   authors block third-party downloads, download those jars manually from CurseForge and drop them into required.
5. Move mods you want players to be able to switch off (minimaps, shaders, performance tweaks) into `optionalon` or
   `optionaloff`. Helios shows these as toggles.

__If the import command fails, build the server manually instead:__

   ```
   npm run start -- generate server mypack 1.20.1 --forge 47.3.0
   ```

Use `--fabric <loader version>` for Fabric packs. Then copy the jars from your CurseForge instance's (in the
CurseForge app: profile ⋮ menu → Open Folder) mods folder into `forgemods/required` (or `fabricmods/required`) and the
folders config and other settings into `files/`.
`files/` mirrors the root of the game folder. Copy the config folder itself, so you end up with `files/config/...`, and
put loose files like options.txt directly in files/.
Protect player settings. Files in files/ are re-checked every launch, so a tracked options.txt would reset everyone's
keybinds and video settings each time they play. Mark personal files as untracked in servermeta.json; Helios then
skips validating them, so players' own changes are not reverted:

   ```json lines
   "untrackedFiles": [
  {
    "appliesTo": [
      "files"
    ],
    "patterns": [
      "options.txt",
      "servers.dat",
      "config/**/*-client.toml"
    ]
  }
]
   ```

Keep server-relevant configs tracked so everyone stays in sync with the server.

## 2.Configure server metadata

Two JSON files control what players see in the launcher: one per server and one for the whole distribution.
Edit `servers/mypack-<mcversion>/servermeta.json`:

   ```json lines
   {
  "meta": {
    "version": "1.0.0",
    "name": "MyPack (Minecraft 1.20.1)",
    "description": "Our modded survival server",
    "icon": "",
    "address": "play.example.com:25565",
    "mainServer": true,
    "autoconnect": true
  },
  "forge": {
    "version": "47.3.0"
  },
  "untrackedFiles": []
}
   ```

- address is your server's IP or domain with port. With autoconnect on, Play joins the server directly.
- mainServer: true makes it the default selection. Only one server should have it.
- version is your pack version. Bump it on every update so you can tell versions apart.
- For the icon, drop a square .png into the server folder and leave icon empty.
- Remove the discord block unless you set up Discord Rich Presence.

Edit `meta/distrometa.json`: set rss to a news feed if you want the launcher's news panel, otherwise remove it along
with the discord block.

Adding more servers later (for example a creative world or a second pack) works the same way: run generate server or
generate server-curseforge with a new ID, fill in its folder and servermeta.json, and regenerate. Players see it in
the launcher's server selector.

## 3. Generate distribution.json and publish it

Nebula hashes every file and writes distribution.json to the repo root; pushing the repo publishes it.

1. Generate a test copy first and install it into your local Helios data folder:
   ```
   npm run start -- generate distro distribution_dev --installLocal
   ```
2. Generate the real file:
   ```
   npm run start -- generate distro
   ```
3. Open `distribution.json` and spot-check that mod URLs start with your BASE_URL.
4. Commit and push everything from the repo folder:
   ```
      git add .
      git commit -m "Pack 1.0.0"
      git push
   ```

5. Wait for the Pages deployment (watch the Actions tab), then
   open https://your-user.github.io/mypack-distro/distribution.json in a browser. If you see the JSON, it is live.
6. Also open one mod URL from the JSON to confirm the jar downloads.
   GitHub Pages caches files for about 10 minutes, so a fresh push can take a little while to reach players.

[go up](#table-of-contents)

## Update pack

1. Add updated and/or new mods to `servers/mypack-<mcversion>/forgemods/required` (or `fabricmods/required`, or
   `optionalon` and `optionaloff` for optional mods)
2. Add or update files that should be in the root of the modpack in `servers/mypack-<mcversion>/files`
3. __Important!__ Bump version in the `server/mypack-<mcversion>/servermeta.json` file:
   ```json lines
   ...
   "meta": {
       "version": "1.0.1", // update this to 1.0.2 and so on
       "name": "VRUkraineSMP",
   ```
4. Return to the Nebula directory and run this command to update distro:
   ```
   npm run start -- generate distro
   ```
5. Return to the `helios` directory, commit, and push the changes:
     ```
      git add .
      git commit -m "(Some message, for example: Update pack to version 1.0.2)"
      git push
      ```
[go up](#table-of-contents)
