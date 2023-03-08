# **Plex Media Server Transfer Guide**
## How to transfer you Plex Media Server from Windows to Linux/Docker


  This guide was created to help anyone that wants to transfer their Plex Media Server from Windows to Linux/Docker and keep all of their invited             users,watch history and metadata. This guide will not help you setup docker and docker compose or how to setup up a Linux OS.

### **The Process:**

To move our server over, we have to prepare our current Windows server to be shut down and moved.

Go to your server, then to Settings > Library and make sure all scanning is removed and automatic trash removal is removed

![How your settings should look like](/images/picture1.png)

Make sure to save you changes.

Now sign out of plex and shut down the server.

Next we should make a backup of our Plex Media Server just in case things go south.

1. Open any folder and type in the the following path. Replace DaHokeyPokey with your windows login username.

![Path](/images/picture2.png)

2. You should see the following folders

![Folders](/images/picture3.png)

3. Now look for "Appdata" folder. If you can't find it, this is "Appdata" is a hidden folder. To see hidden folders go to View -> Show -> Hidden Items. After that you should now see Appdata.

![Hidden](/images/picture4.png)

![Appdata](/images/picture5.png)

4. Now that we can see Appdata go to Appdata -> Local. You should now see you Plex Media Server folder.

![Plex Media Server Folder](/images/picture6.png)

5. 

Next thing we want to do is archive this file and make a copy of it and store it somewhere else just in case something breaks, gets corrupted, etc. If you have good internet I would highly suggest uploading it to whatever cloud provided you are using.

For this next part I'll be using 7Zip, you can download it here, but any archiving program should work.

To archive this data lets move back one folder so we can see the parent folder Plex Media Server.
Right click, go to 7zip, select add to archive
You can choose are format you want, since we are moving from Windows to Linux/Docker I would recommend tar.
Take a look at my settings and make sure they match, if they do, hit start.

This is another one of things that depending on the size of the file and how powerful your pc is, this can take some time. Using my file and pc as a reference
it took my Ryzen 3600 about 25/30 mins for ~50GB file.

While this is taking place, lets start to prepare are new server.

For this example I'll be using Ubuntu 22.04 Desktop with docker compose.
The idea here is to start the server up once to populate all of the files, replace the files from the back-up we just created and log into our server with all of our settings.

[Plex docker compose file example](/examples/plex/compose.yaml)
```
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
      - VERSION=public
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - ${PLEXAPP}:/config
      - ${MEDIA}:/plexmedia
    restart: unless-stopped
```

Before going any further lets review the compose file and make sure we have all the prerequiests needed before running our server.

Its recommended to run the container in host mode over bridge mode. I'll be using host mode for this example, if you are set in using bridge, please see the document linked about.

newtork_mode: host - This sets the containers network to host mode. Host mode means that the container will be using the host pc's ip address.
runtime: nvidia - This is used when you want to use a nvidia gpu for hardware transcoding. For intel quicksync see document linked above. (This requires plex pass. This option can be commented out.)
environment - This section is where we enter different variable to adjust certain settings in the container.
    
    PUID - The user id that will be assigned for permissions. To keep it simple you want the user id to match your user id that will be launching/starting docker compose. This basically makes the user inside the container have the same permission as the user outside the container.
    
    PGID - The group id that will be assigned for permissions. To keep it simple you want the group id for the user that will be launching/starting docker compose.

    TZ - The time zone that you want the container to use. See this list for your timezone.

    VERSION - This is the version of the Plex Media Server that you will be running. If you have plex pass you want to use public, if not use docker.

    NVIDIA_VISIBLE_DEVICES - This is used when you want to use a nvidia gpu for hardware transcoding. (This requires plex pass. This option can be commented out.)

    Volumes - This is the section where we assigned the host folders to the container.

    /Path/config:/config - On the left side of the colon, we want to enter the path where we want to store the containers information on the host machine. The right side is the folder path to the folder that will be populated into the host machines folder provide on the left side.

    /Path/media:/media - On the left side of the colon, this is the path where your media is located on the host machine. The right side is the folder in the container where the media would be mounted in the container.

    /Path/prerolls:/prerolls - Same as above. Mounted my prerolls to /prerolls inside the container. (Optional field, can be commented out.)

    restart - This tells the container what to do if the host machine is restarted.
    unless-stopped - This basically means that the container will start on reboot or anytime the host machine goes down, unless the container was stopped by a user.

Now that we have our docker compose file all mapped out we are almost ready to run it.

Technically speaking you can place this file anywhere and run it, but lets be organized and place it in the same folder we used for our volumes.

Example: for the config file we gave the path /Path/config, we want to put the compose.yaml file in /Path.

Once the file is in the correct path, open up a terminal and enter the following

cd /path to compose.yaml file
make sure the file is there
ls
you should see the following

now we can start up the container.

docker compose up -d

to make sure our container is running

docker ps

you should see the following if the container is up and running.

next lets make sure the folders populated in the correct location/as intended. One thing we want to make sure populated in the config file is the Preferences.xml. This is very important as this is where we have to enter all the information we need to make sure we save our invited users.
if everything is where you want, lets go ahead and stop the container.
double check and make sure your terminal is in the correct folder.

docker compose down

make sure its not running

docker ps

Hopefully by this time your archive is finished.

Move your zipped file from your windows machine to your linux machine.
Next we want to unzipped this archive
Now we are going to copy and paste of the files and folders from the archive file to the config file
You'll be asked if you want to merge,replace, or rename the files, choose to replace the files. This will replace the files created by the container and replaced them with the archived files we just copied and pasted.

Now we want to back up Preferences.xml.
Quick and dirty way is just to copy the file and label it Preferences(1).xml
Another way is to open the file copy it to an empty file and save it as Preferences.xml.bak. This is a better method to me, because the name are the same so you know which bak is for which file.
Now that we have a backup open, Preferences.xml in whatever text editor you are comfortable with.

The information that we need to enter in the xml file is located in the Windows Registry.
To get the information we need we need to head over to the Windows PC, press the windows key, enter run and select run.

In Run, enter regedit and click run, this will open up a new window
Select HKEY_Current_USER -> Software -> Plex Inc -> Plex Media Server. You should see the following below.

We need to enter the following fields in the xml file.

- AnonymousMachineIdentifier
- MachineIdentifier
- ProcessedMachineIdentifier
- PlexOnlineToken
- PlexOnlineUsername
- PlexOnlineMail
- FriendlyName
- PublishServerOnPlexOnlineKey

For AnonymousMachineIdentifier, MachineIdentifier, ProcessedMachineIdentifier, these are pre populated in the xml file, we will be replacing this values with the values from the windows pc. The rest of the items we will have to enter into the xml file.

After entering all your information, your xml file should now look like this.

Save it and close out the text editor.

We are now ready to start up the container again.

go back to your terminal, make sure you are in the correct folder

docker compose up -d

now we can log into the server at 127.0.0.1:32400/web or HOST-IP:32400/web

If done correctly you everything should be as it was on your windows pc.

FIRST THING YOU NEED TO DO IS GO TO THE SETTINGS AND MAKE SURE ALL SCANNING AND AUTO TRASH IS TURN OFF.
Migration are not perfect and if the settings didn't transfer over, the server might start to scan your folders and they are not mapped at the moment and this will erase all of you watch history and library.

Now lets add our new paths to our librares scan your library, double check to make sure the file path for your media is correct.

Once that is completed, you can remove our old library paths and setup how you like to handle your scans and auto trash.

Congrats you have now just transfered your Plex Media Server from your Windows PC to your Linux/Docker PC.
