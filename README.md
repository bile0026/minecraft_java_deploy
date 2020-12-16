# minecraft_java_deploy
deploy/update minecraft java server

Sets up a Minecraft Java server based on the `java_url` variable. Just grab the latest one from https://www.minecraft.net/en-us/download/server to deploy the latest version. I'm assuming you have some knowledge of how to run Ansible playbooks, and won't go into much detail on that. Minecraft server will run as a service and automatically start on boot. Should work on all RedHat and Debian-based operating systems. If the specified minecraft service already exists the minecraft_server_update roll will be called and will upgrade the server without any changes to settings. Update role will put a backup of the server into the `backup_folder` prior to pushing new files in.

* Make sure to deploy from a tag, not from master branch. Tags should be stable, master branch should be considered development and may not always be in a stable state.
```
git clone <repo_url>
git checkout tag/<tag_name>
```

* Once deployment finishes, be sure to give the server a few minutes to generate the world depending on the speed of your CPU.
* You can also use the update task to update the server.properties file.

# Current caveats
* Currently just disables firewalld on RedHat-based servers. Working on modifying instead.
* Disables SELINUX on systems running it. Working on modificiations to allow instead.
* Have to specify version manually in the version variable. Check here for latest version: https://www.minecraft.net/en-us/download/server I hope to have this automatically grab the latest version in the future.

Run the playbook with this command, substituting your credentials. -k is used to prompt for user password, -K is for the sudo/become password. If you want to use a private key switch out -k (lowercase) with --key-file <path>. This playbook requires become as it will install packages and create services.

```
ansible-playbook -i hosts deploy_minecraft.yml -u <username> -k -K
```

Set the vars as required if any are different from defaults. Change these in `group_vars/all.yml', or setup specific group/host vars if you are looking for a scaled deployment.

# Configurable variables
* gamertags are case-sensitive and might contain spaces. Double check these careully if you want to use the whitelist.
* as a preventative measure, whitelist is not automatically enabled if whitelist is created. Set the `whitelist` variable to enable it when you are sure it's correct.
* pay attention to variables that use quote around true/false values. Make sure to preserve the "" or the templates may not work correctly. These are strings, not literal true/false values.

```
# the jar file to download and run from https://www.minecraft.net/en-us/download/server
java_url: https://...

# will be used to name not only the server but folders and services.
minecraft_server_name: my_server

# this setting will depend on resource levels of your host machine. More players = more resources.
max_players: 10

gamemode: survival

difficulty: easy

# users must log in to their microsoft account to connect
online_mode: "true"

# enables/enforces the whitelist on the server
whitelist: "false"

# server port needs to be unique for each server running on the same host
server_port: 19134

level_name: my_level

message_otd: Welcome to my server

level_seed:

# server permission variables set uuids. Add more as needed. Bypass is whether to bypass user limit.
op_permissions:
   - level: 4
     uuid: xxxxxxxx
     gamertag: gamertag1
     bypass: "false"
  - level: 3
    uuid: xxxxxxy
    gamertag: gamertag2
    bypass: "false"
  - level: 2
    uuid: xxxxxxz
    gamertag: gamertag3
    bypass: "false"
  - level: 1
    uuid: xxxxxxz
    gamertag: gamertag4
    bypass: "false"

# whitelist variables set gamertags. Use "" on true/false. Add more as needed
whitelist_accounts:
  - playerlimit: "true"
    gamertag: gamertag1
  - playerlimit: "false"
    gamertag: gamertag2
```