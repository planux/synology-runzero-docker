## RunZero Network Explorer on Synology NAS

This will show you how to deploy RunZero Network Explorer in a non-persistent container, on a Synology NAS using Synology Container Manager, which is Synology's repackaging of Docker. This Dockerfile also uses `ubuntu:latest` instead of `debian:stable-slim` like is listed in the official docs. You can learn more about RunZero here https://www.runzero.com 

Shout out to Pearce Barry @ RunZero Engineering for helping with this; adaptation to use Synology Container Manager by Paul Lanzi.
 
PREREQUISITES
- Synology NAS using DSM 7
- Synology Container Manager
- Knowledge on how to set firewall settings and ports on your Synology.

TESTED ON
- DS220+
- DS3018xs

## Step 1: Provision a Network Explorer

1. With your web broser, go to https://console.runzero.com (log in as necessary) and Click Deploy-> Deploy an Explorer
2. On the left side, select "Linux". Ensure that 64-bit (x86_64) is selected.
3. Scroll down to the URL for the Explorer. It will be in the format of:
`https://console.runzero.com/download/explorer/<Explorer Identifier>/<Explorer Version>/runzero-explorer-linux-amd64.bin`
Copy this URL. You will need it in the subsequent steps. Note the fields for "Explorer Identifier" and "Explorer Version" in the URL above.
- Note: You do NOT need to download the Explorer binary.

## Step 2: Create the dockerfile

1. Log into your Synology
2. If they are not already installed, install the 1) Text Edit package and the 2) Container Manager from the Package Center
3. Using the File Explorer, create a new folder for the runZero Network Explorer Docker project. Generally these would be in `<some volume>/docker/<name of project>` but you can choose any file system and folder you like. For instance, my project is located at `/volume1/docker/runzero-network-explorer`
4. Create a subfolder within the project folder for the dockerfile. For instance, mine is located at `/volume1/docker/runzero-network-explorer/docker/`
5. Using the Text Editor, create a file Dockerfile in this directory. For instance, the full path for mine is: `/volume1/docker/runzero-network-explorer/docker/Dockerfile`. Note that the file name is case sensitive.
6. The file contents should be:
   
```
FROM ubuntu:latest

WORKDIR /opt/rumble

ENV AGENT_URL=<the URL that you captured in step 1.3 above>

ENV RUMBLE_AGENT_HOST_ID=<the Explorer Identifier from step 1.3 above>

ENV RUMBLE_AGENT_LOG_DEBUG=false

ADD ${AGENT_URL} runzero-explorer.bin

RUN chmod +x runzero-explorer.bin

RUN apt update && apt install -y chromium-browser

USER root

ENTRYPOINT [ "/opt/rumble/runzero-explorer.bin", "manual"]
```
- Explanation:
  - For non-persistent containers an Explorer Identifier needs to be persisted through an environment variable.
  - The argument `manual` tells runZero not to look for SystemD or upstart.
  - The line `RUN apt update && apt install -y chromium-browser` installs Chromium so that the Explorer can capture screenshots. It can be removed if you do not want your Explorer to capture screenshots.

## Step 3: Create a Project, Image and Container for the runZero Network Explorer

1. On your Synology, open the Container Manager
2. Select Project, then "Create". Project Name = `runzero-network-explorer`, Path = `<path you created in step 2.3>`, Source = choose "Create docker-compose.yml"
3. In the "Create docker-compose.yml box", enter:
```
version: '3'
services:
  myservice:
    build:
      context: <path that you created in step 2.3>/dockerfile
      dockerfile: Dockerfile
    image: runzero-network-explorer/latest
```
4. Click Next
5. You do NOT need to setup a Web Portal, so leave that box unchecked. Click Next.
6. The Summary screen should look like this (including "Start the project once it is created"):
7. The build will begin. This will take several minutes as Container Manager downloads the Ubuntu base image, Chromium and runZero Network Explorer (and dependencies). When it completes it should display a message like this:

## Step 4: Verifying all is working correctly

1. Looking at the Container Manager -> Project screen, you should see (the green icon indicates the container is running successfully):
2. Looking at the Container Manager -> Container screen, click on Log. You should see messages like this indicating that the new Explorer is communicating with the runZero service
3. Navigate to the runZero console, select Deploy->View Explorers. The newly deployed Explorer should appear like this:
4. On the runZero console, and configure at least one Task to use the new Explorer
5. After at least one Task has successfully completed, view an individual Asset and scroll down to the Services section (at the very bottom). Look for service attributes such as `tcp.win` and `tcp.winScale`. The presence of these attributes indicates that runZero's raw connection based probes are working correctly through the Docker networking.


