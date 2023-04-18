# code-remote

Launch a dev-container from a local folder or git URL directly in visual studio code from the command line.

## Installation 

The `code-mirror` command is a simple scripts in `bash`, so you can easily fetch it for a specific version from github:

    PROGRAM="code-remote"
    REPO="harcokuppens/code-remote" 
    LATEST_TAG="$(curl -s https://api.github.com/repos/${REPO}/releases/latest | jq -r '.tag_name')"
    INSTALL_DIR=/usr/local/bin # make sure INSTALL_DIR is in your PATH environment variable
    DOWNLOAD_URL="https://raw.githubusercontent.com/${REPO}/${LATEST_TAG}/bin/"
    curl -Lo "${INSTALL_DIR}/${PROGRAM}"  "$DOWNLOAD_URL/${PROGRAM}"
    chmod a+x "${INSTALL_DIR}/${PROGRAM}"
      
Requirements:  

* `bash` shell
* `mirror` tool from https://github.com/harcokuppens/mirror/
* `docker` tool 
* `code` commandline tool of Visual Studio Code

## Description

Help info from `code-remote --help`:


    NAME
      code-remote - Open a dev container in visual studio code from the command line directly.

    USAGE
      code-remote [-v|-f|-n]  FOLDER   [WORKSPACEFOLDER]
      code-remote [-f|-n]     GIT-URL  [WORKSPACEFOLDER]
      code-remote --try|-t    LANGUAGE

        LANGUAGE: the following languages are available in a try dev-container:
                  python go php node java rust dotnet cpp sqlserver

      More documentation
        * code-remote --help
        * https://github.com/harcokuppens/code-remote

    OPTIONS    
      -v    instead of bind mount the host folder is mirrored in a named volume and mounted     
      -n    no mount, by default workspace folder set to /. Can be combined with -f.
      -f    still use workspace folder set in devcontainer.json in cases when it would be ignored 
      -t, --try 
            try a sample development containers 
      
    DESCRIPTION
      When opening dev container
        * from a FOLDER by default this host folder is mounted (bind mount).

          By default the FOLDER is mounted at /workspaces/<FOLDERBASENAME>, however we can
          change the mount location with the 'workspaceMount' field in devcontainer.json.

          There is a small exception: for a dev container using docker compose, there is
          no automatic bind mount done. You can still configure that by using the
          'mounts' field in the devcontainer.json file.

          This mimics the 'Dev Containers: Open folder in Container...'
          command in the visual studio code application.

        * from a  GIT-URL a named volume is created with the clone of the git repo,
          which is then mounted in the dev container.

          When using a GIT-URL the named volume is always mounted at /workspaces/<URLBASENAME>
          and cannot be changed.

          This mimics the 'Dev Containers: Clone Repository in Container Volume...'
          command in the visual studio code application.

      The workspace folder in visual studio code is set by default to the mount location.
      However an end user can specify a  different 'workspaceFolder' in devcontainer.json.
      There is a small exception: for a GIT-URL the 'workspaceFolder' in devcontainer.json
      is by default ignored, however by supplying the -f option code-remote will still use it.
      The workspace folder can still always be overruled by the end user by using the
      WORKSPACEFOLDER argument on the command line. Finally we must emphasize that changing
      the workspace folder doesn't change the mount location. And if still we got the
      wrong workspace folder in visual studio code you can always open it in a different
      working folder using the menu 'File -> Open Folder...'.

      Tweaks:
       * If for a FOLDER you want to have a volume mount instead of a bind mount, then
         supply the -v option which causes the FOLDER to be used like a GIT-URL.
         Everything is done the same as for a GIT-URL except that instead of the
         repository the folder is mirrored into a a named volume which is then mounted
         in the dev container! Useful if you want to be fully isolated from the host.
         The mount is then always in /workspaces/<FOLDERBASENAME>.
         The 'workspaceFolder' in devcontainer.json is then also by default ignored,
         however by supplying the -f option code-remote will still use it.
       * If for a GIT-URL you want to have a host folder mounted(bind mount) instead,
         then git clone the url into a FOLDER, and open that FOLDER with code-remote.

      IMPORTANT: if you have opened a dev container an then restart it with changed
         options, like -v, then first delete the old container first to let visual
         studio code rebuild it, otherwise  visual studio code will reuse this old
         container with the old mount settings. If you forgot to do this, you can
         also rebuild the container from visual studio code with the command:
            'Dev Containers: Rebuild Container...'

      Notes about the named volume:
        * on first usage the named volume is automatically created from the GIT-URL/FOLDER
          with naming convention:

             code-volume-<BASENAME_OF_URL_OR_FOLDER>
             code-volume-<ESCAPED_URL_OR_FOLDER>

        * when using a named volume its name is notified in the terminal
        * the named volume is reused when you later open the dev container again
        * only when you delete the named volume with 'docker image rm VOLNAME'
          then a new fresh named volume is created from the GIT-URL/FOLDER.
        * The reason for this behavior is that you easily can continue developing from
          an existing volume. When you close visual studio code, and later open it
          again you get the same volume opened again from which you can continue.
        * Sometimes a container does use a special user instead of root. Then you
          must use 'sudo chown -R USER /workspaces/*' to make this user the owner
          of these files. By default the files in the volume are owned by root.
