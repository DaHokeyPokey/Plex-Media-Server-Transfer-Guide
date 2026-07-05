# **Plex Media Server Transfer Guide**

## How to transfer your Plex Media Server from Windows to Linux/Docker

This guide was created to help anyone that wants to transfer their Plex Media Server from Windows to Linux/Docker and keep as much of their existing server as possible, including invited users, watch history and metadata.

> **Important:** Moving Plex from Windows to Linux/Docker is not always perfect. This guide is meant to help preserve everything, but you should still make a full backup before doing anything.

This guide will not help you set up Docker, Docker Compose, or how to set up a Linux OS.

---

## **The Process**

To move our server over, we have to prepare our current Windows server to be shut down and moved.

Go to your server, then go to:

```text
Settings > Library
```

Make sure all scanning is turned off and automatic trash removal is turned off.

![How your settings should look like](/images/picture1.png)

Make sure to save your changes.

Now sign out of Plex and shut down the server.

---

## **Back Up Your Plex Media Server**

Next we should make a backup of our Plex Media Server just in case things go south.

### **1. Find your Plex Media Server folder**

Open any folder and type in the following path:

```text
%LOCALAPPDATA%\Plex Media Server
```

You can also go to the folder manually by going to:

```text
C:\Users\YOUR-USERNAME\AppData\Local\Plex Media Server
```

Replace `YOUR-USERNAME` with your Windows login username.

![Path](/images/picture2.png)

You should see the following folders:

![Folders](/images/picture3.png)

---

### **2. Show hidden folders if needed**

Now look for the `AppData` folder.

If you can't find it, `AppData` is a hidden folder. To see hidden folders go to:

```text
View > Show > Hidden Items
```

After that you should now see `AppData`.

![Hidden](/images/picture4.png)

![Appdata](/images/picture5.png)

---

### **3. Go to the Plex folder**

Now that we can see `AppData`, go to:

```text
AppData > Local
```

You should now see your `Plex Media Server` folder.

![Plex Media Server Folder](/images/picture6.png)

---

## **Archive the Plex Media Server Folder**

For this next part I'll be using 7-Zip. You can download it here, but any archiving program should work.

To archive this data, move back one folder so we can see the parent folder:

```text
Plex Media Server
```

Right click the folder, go to:

```text
7-Zip > Add to archive
```

You can choose whatever format you want, but since we are moving from Windows to Linux/Docker I would recommend:

```text
tar
```

Take a look at your settings and make sure they look right. If they do, hit start.

> **Optional:** You can exclude the `Cache` folder from the backup if you want to save space and make the transfer faster.

This is another one of those things that depends on the size of the file and how powerful your PC is. Using my file and PC as a reference, it took my Ryzen 3600 about 25/30 mins for a `~50GB` file.

While this is taking place, lets start to prepare our new server.

---

## **Prepare the New Linux/Docker Server**

For this example I'll be using:

* Ubuntu 22.04 Desktop
* Docker Compose

The idea here is to start the server up once to populate all of the files, replace the files from the backup we just created, and log into our server with all of our settings.

---

## **Docker Compose File**

[Plex docker compose file example](/examples/plex/compose.yaml)

```yaml
services:
  Plex:
    image: lscr.io/linuxserver/plex:latest
    network_mode: host
    runtime: nvidia
    container_name: Plex
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - VERSION=docker
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - ${PLEXAPP}:/config
      - ${MEDIA}:/plexmedia
    restart: unless-stopped
```

---

## **Example `.env` File**

The compose file above uses variables like `${PUID}`, `${PGID}`, `${TZ}`, `${PLEXAPP}`, and `${MEDIA}`.

You can define those in a `.env` file in the same folder as your `compose.yaml`.

Example:

```env
PUID=1000
PGID=1000
TZ=America/Chicago
PLEXAPP=/path/to/plex/config
MEDIA=/path/to/media
```

To find your user ID and group ID, run:

```bash
id
```

You should see something like this:

```text
uid=1000(username) gid=1000(username)
```

Use those numbers for `PUID` and `PGID`.

---

## **Review the Compose File**

Before going any further lets review the compose file and make sure we have all the prerequisites needed before running our server.

### `network_mode: host`

This sets the container network to host mode.

Host mode means that the container will be using the host PC's IP address.

It's recommended to run the Plex container in host mode over bridge mode. I'll be using host mode for this example.

If you are set on using bridge mode, please see the documents linked above.

---

### `runtime: nvidia`

This is used when you want to use a NVIDIA GPU for hardware transcoding.

For Intel Quick Sync, see the document linked above.

> **Note:** Hardware transcoding requires Plex Pass. This option can be commented out if you are not using a NVIDIA GPU.

Example if you are not using NVIDIA:

```yaml
# runtime: nvidia
```

You can also comment out:

```yaml
# - NVIDIA_VISIBLE_DEVICES=all
```

---

### `environment`

This section is where we enter different variables to adjust certain settings in the container.

#### `PUID`

The user ID that will be assigned for permissions.

To keep it simple, you want the user ID to match the user ID that will be launching/starting Docker Compose.

This basically makes the user inside the container have the same permissions as the user outside the container.

#### `PGID`

The group ID that will be assigned for permissions.

To keep it simple, you want the group ID for the user that will be launching/starting Docker Compose.

#### `TZ`

The time zone that you want the container to use.

Example:

```text
America/Chicago
```

#### `VERSION`

This is the version of Plex Media Server that you will be running.

For most people I would recommend:

```yaml
- VERSION=docker
```

This lets the Docker image handle the Plex version.

Other options you might see:

| Value            | What it does                                                                              |
| ---------------- | ----------------------------------------------------------------------------------------- |
| `docker`         | Uses the Plex version included with the Docker image. Good default option.                |
| `latest`         | Uses the latest version your Plex account is allowed to download.                         |
| `public`         | For Plex Pass users that want the latest public release instead of Plex Pass/beta builds. |
| Specific version | Lets you pin Plex to a specific version.                                                  |

#### `NVIDIA_VISIBLE_DEVICES`

This is used when you want to use a NVIDIA GPU for hardware transcoding.

```yaml
- NVIDIA_VISIBLE_DEVICES=all
```

> **Note:** This requires Plex Pass. This option can be commented out if you are not using NVIDIA hardware transcoding.

---

## **Volumes**

This is the section where we assign the host folders to the container.

### Config Path

```yaml
- /path/config:/config
```

On the left side of the colon, we want to enter the path where we want to store the container's information on the host machine.

The right side is the folder path inside the container.

### Media Path

```yaml
- /path/media:/plexmedia
```

On the left side of the colon, this is the path where your media is located on the host machine.

The right side is the folder in the container where the media will be mounted.

So if your compose file has this:

```yaml
- /mnt/storage/media:/plexmedia
```

Then inside Plex you would use:

```text
/plexmedia
```

Not:

```text
/mnt/storage/media
```

This is important because Plex sees the container path, not the host path.

### Optional Prerolls Path

```yaml
- /path/prerolls:/prerolls
```

Same as above. This mounts my prerolls to `/prerolls` inside the container.

This is optional and can be commented out.

### Restart Policy

```yaml
restart: unless-stopped
```

This tells the container what to do if the host machine is restarted.

`unless-stopped` basically means that the container will start on reboot or anytime the host machine goes down, unless the container was stopped by a user.

---

## **Run the Container Once**

Now that we have our Docker Compose file all mapped out, we are almost ready to run it.

Technically speaking you can place this file anywhere and run it, but lets be organized and place it in the same folder we used for our volumes.

Example:

If the config file path is:

```text
/path/config
```

Then we want to put the `compose.yaml` file in:

```text
/path
```

Once the file is in the correct path, open up a terminal and enter the following:

```bash
cd /path/to/compose/file
```

Make sure the file is there:

```bash
ls
```

You should see your `compose.yaml` file.

Now we can start up the container:

```bash
docker compose up -d
```

To make sure our container is running:

```bash
docker ps
```

You should see the Plex container up and running.

---

## **Check the Config Folder**

Next lets make sure the folders populated in the correct location/as intended.

One thing we want to make sure populated in the config folder is:

```text
Preferences.xml
```

This is very important as this is where we have to enter all the information we need to make sure we save our invited users and server identity.

If everything is where you want, lets go ahead and stop the container.

Double check and make sure your terminal is in the correct folder.

```bash
docker compose down
```

Make sure it's not running:

```bash
docker ps
```

Hopefully by this time your archive is finished.

---

## **Move the Backup to Linux**

Move your zipped file from your Windows machine to your Linux machine.

Next we want to unzip/extract this archive.

Now we are going to copy and paste all of the files and folders from the archive file to the config folder.

You'll be asked if you want to merge, replace, or rename the files.

Choose:

```text
Replace
```

This will replace the files created by the container with the archived files we just copied and pasted.

---

## **Fix Permissions**

After copying the files over, we want to make sure the permissions are correct.

If your `PUID` and `PGID` are both `1000`, you can run:

```bash
sudo chown -R 1000:1000 /path/to/plex/config
```

Or you can use your current Linux user and group:

```bash
sudo chown -R "$(id -u):$(id -g)" /path/to/plex/config
```

Make sure you replace this path:

```text
/path/to/plex/config
```

With the actual path you used for `${PLEXAPP}`.

---

## **Back Up `Preferences.xml`**

Now we want to back up:

```text
Preferences.xml
```

Quick and dirty way is just to copy the file and label it:

```text
Preferences(1).xml
```

Another way is to open the file, copy it to an empty file, and save it as:

```text
Preferences.xml.bak
```

This is a better method to me because the names are the same, so you know which backup is for which file.

Now that we have a backup, open `Preferences.xml` in whatever text editor you are comfortable with.

---

## **Get the Windows Registry Information**

The information that we need to enter in the XML file is located in the Windows Registry.

To get the information we need, head over to the Windows PC.

Press the Windows key, enter:

```text
Run
```

And select `Run`.

In Run, enter:

```text
regedit
```

Click run. This will open up a new window.

Select:

```text
HKEY_CURRENT_USER > Software > Plex, Inc. > Plex Media Server
```

Or as a full path:

```text
HKEY_CURRENT_USER\Software\Plex, Inc.\Plex Media Server
```

You should see your Plex registry values.

---

## **Values We Need to Copy**

We need to enter the following fields in the XML file:

* `AnonymousMachineIdentifier`
* `MachineIdentifier`
* `ProcessedMachineIdentifier`
* `PlexOnlineToken`
* `PlexOnlineUsername`
* `PlexOnlineMail`
* `FriendlyName`
* `PublishServerOnPlexOnlineKey`

For these values:

* `AnonymousMachineIdentifier`
* `MachineIdentifier`
* `ProcessedMachineIdentifier`

They are pre-populated in the XML file. We will be replacing these values with the values from the Windows PC.

The rest of the items we will have to enter into the XML file if they are missing.

> **Security warning:** Do not share screenshots or examples that show your `PlexOnlineToken`, machine identifiers, email address, or registry values. Treat the Plex token like a password.

After entering all your information, your XML file should now have the needed values.

Save it and close out the text editor.

---

## **Start the Container Again**

We are now ready to start up the container again.

Go back to your terminal and make sure you are in the correct folder.

```bash
docker compose up -d
```

Now we can log into the server.

If you are on the same machine as the Plex server, you can use:

```text
http://127.0.0.1:32400/web
```

If you are on another computer on your network, use:

```text
http://HOST-IP:32400/web
```

Example:

```text
http://192.168.1.50:32400/web
```

If done correctly, everything should be as it was on your Windows PC.

---

## **Important: Turn Off Scanning Again**

> **FIRST THING YOU NEED TO DO:**
> Go to the settings and make sure all scanning and automatic trash removal are turned off.

Migrations are not perfect. If the settings didn't transfer over, the server might start to scan your folders before they are mapped correctly.

If Plex scans before your new paths are correct, it might think your media is missing and remove items from your libraries. This can mess with your watch history, metadata, and library items.

---

## **Add the New Library Paths**

Now lets add our new paths to our libraries.

Remember, you want to add the container path, not the host path.

Example:

If your compose file has:

```yaml
- /mnt/storage/media:/plexmedia
```

Then in Plex, add the library path as:

```text
/plexmedia
```

Scan your library and double check to make sure the file path for your media is correct.

Once that is completed, you can remove the old Windows library paths and set up how you like to handle your scans and automatic trash removal.

---

## **Final Verification Checklist**

Before turning automatic scanning or automatic trash removal back on, verify the following:

* Plex opens successfully at `http://HOST-IP:32400/web`
* Your libraries are visible
* Your media is available from the new Linux/Docker paths
* Watch history is still present
* Invited users still show up
* Remote access is working, if you use it
* Hardware transcoding works, if configured
* Old Windows library paths have not been removed until the new paths are confirmed working

---

## **Finished**

Congrats, you have now just transferred your Plex Media Server from your Windows PC to your Linux/Docker PC.
