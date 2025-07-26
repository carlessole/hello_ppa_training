# Take Home Test from Canonical - Carles Solé Grau


An initial general reading of this tutorial helps to understand all the necessary steps to generate and publish a package to Launchpad PPA:
- [Minimum Complete Example of Debian Packaging and Launchpad PPA (hellodeb)](https://metebalci.com/blog/a-minimum-complete-example-of-debian-packaging-and-launchpad-ppa-hellodeb/)

---

Pick an existing Debian package from the Ubuntu archive. The selected package should be simple and easy to work with.

## 1. Download the source code of that Debian package

1. Go to [Ubuntu Packages](https://packages.ubuntu.com/) and select your desired Ubuntu version.  
    I am working on Ubuntu 25.4: [Plucky](https://packages.ubuntu.com/plucky/).

2. Choose a simple package to avoid overcomplicating the exercise.  
    [hello](https://packages.ubuntu.com/plucky/hello) is a good choice because it is an example package based on GNU Hello.
    It is also the one used in [the reference link](https://pastebin.ubuntu.com/p/hZ4sH647Jt/).

3. In the **Download** section, select your target architecture to get instructions for downloading the package:
    - [https://packages.ubuntu.com/plucky/amd64/hello/download](https://packages.ubuntu.com/plucky/amd64/hello/download)

4. Get the source code:

    There are multiple ways to get the source code of a package:
    - [Get the source of a package](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/): Link provided in the test assignment instructions.
    - [apt-src](https://manpages.ubuntu.com/manpages/plucky/man1/apt-src.1p.html)
    - [How to get source code of package using the apt command on Debian or Ubuntu](https://www.cyberciti.biz/faq/how-to-get-source-code-of-package-using-the-apt-command-on-debian-or-ubuntu/)

    Feel free to use the method you are most comfortable with.  
    For this exercise, I will use [pull-pkg](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/#pull-pkg), as described in the official guide.  

    It is one of the simplest and fastest methods.
    ```bash
    sudo apt update && sudo apt install ubuntu-dev-tools
    ```

    Then fetch the source:
    ```bash
    pull-lp-source hello
    ```

### Initialize Git repository

Initialize a Git repository in the package source directory to track all changes:

```bash
cd hello-2.10
git init
```

From this point onward, try to commit each step individually for easier tracking and review.

## 2. Add an executable bash script called "testing.sh"

This script must output the following message to **standard error (STDERR)** when executed:

```
this is a test from Carles Solé Grau
```

1. Create a `testing.sh` file and place it in the `debian` directory.

    This is because the `debian/` directory is the standard location recognized by the Debian packaging system for defining what gets installed, where, and how. See: [Debian Policy Manual – Source Packages](https://www.debian.org/doc/debian-policy/ch-source.html)  
    Content of `debian/testing.sh`:

    ```bash
    #!/bin/bash

    # Use '>&2' to redirect output to STDERR
    echo "this is a test from Carles Solé Grau" >&2
    ```

2. Make the script executable:

    ```bash
    chmod +x debian/testing.sh
    ```

3. Verify the message goes to STDERR:

    ```bash
    debian/testing.sh > stdout.txt 2> stderr.txt
    ```

    Then check `stderr.txt` to confirm the output.  
    Don't forget to delete the temporary files after testing.

4. To install this script on your system when installing the `.deb` package, add/edit the file `debian/install` with this line:

    ```
    debian/testing.sh /usr/bin
    ```
    `/usr/bin` has been used as the installation path because the [reference example link provided](https://pastebin.ubuntu.com/p/hZ4sH647Jt/) also points to this location.

## 3. Echo a string during the Debian package installation via STDOUT

This message must be printed to **standard output (STDOUT)**, so `debian/testing.sh` can not be reused.

1. Create a [maintainer script](https://www.debian.org/doc/debian-policy/ch-binary.html#prompting-in-maintainer-scripts) that runs during installation.  
We will use the `postinst` script, which is executed after the package is installed.  
Create or edit `debian/postinst`:

    ```bash
    #!/bin/bash
    set -e

    # Output to STDOUT
    echo "this is a test from Carles Solé Grau"

    # Hook for debhelper (even if not required) to avoid a warning
    #DEBHELPER#

    exit 0
    ```

2. Make it executable:

    ```bash
    chmod +x debian/postinst
    ```

### Validate the prompt during installation

Before proceeding and pushing all this code to the Launchpad PPA, it is a good idea to validate everything locally first.

Build the `.deb` package following [Ubuntu - Build packages](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/build-packages/)

1. Install the prerequisites:
    ```bash
    sudo apt update
    sudo apt install sbuild debhelper ubuntu-dev-tools piuparts
    ```
2. Since we just want to test the .deb package locally, we can use:
    ```
    sbuild -c <RELEASE>-<ARCH>[-shm]
    ```
    However, I had some issues with this tool, so I ended up using `debuild` instead:
    - [debuild - build a Debian package](https://manpages.ubuntu.com/manpages/plucky/en/man1/debuild.1.html)
    - [debuild - build a Debian package - Examples](https://manpages.ubuntu.com/manpages/plucky/en/man1/debuild.1.html#examples)

    ```bash
    debuild -i -us -uc -b
    ```
    `-us`and `-uc` to avoid signing sources and changes.

    Before running it, make sure `help2man` is installed on your system:
    ```bash
    sudo apt update
    sudo apt install help2man
    ```

If everything goes well, a `.deb` file should be generated one directory up.

3. Test the installation:

    ```bash
    sudo dpkg -i ../hello_2.10-*.deb
    ```

    You should see the following printed in the terminal:
    `/usr/bin` has been used as in the installation path as the [reference example link provided](https://pastebin.ubuntu.com/p/hZ4sH647Jt/) points also there.
    ```bash
    this is a test from Carles Solé Grau
    ```

4. After installation run:
    ```bash
    dpkg -S testing.sh; testing.sh
    ```

    Outuput should look like:
    ```bash
    hello: /usr/bin/testing.sh
    this is a test from Carles Solé Grau
    ```

## 4. Push all your changes to a public Git repository (e.g. Launchpad Git, GitHub, or GitLab)

Choose your preferred Git provider and push all your changes there.

## 5. Upload (dput) the source change to your PPA on the Launchpad

The provided links:
- https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/upload-packages-to-ppa/
- https://github.com/canonical/ubuntu-packaging-guide/blob/2.0-preview/docs/how-to/upload-packages-to-ppa.rst

It appears that the official documentation page for uploading a package to a Launchpad PPA is still under construction.
Therefore, we have followed the necessary steps using various resources found online.

### Update changelog

- [The Launchpad PPA - launchpad-dependencies](https://documentation.ubuntu.com/launchpad/en/latest/developer/explanation/launchpad-ppa/#launchpad-dependencies): Step 4.
- [debchange - Tool for maintenance of the debian/changelog file in a source package](https://manpages.ubuntu.com/manpages/trusty/man1/debchange.1.html)

The first step is to add a new entry to the `debian/changelog`. You can do this manually or use the [dch](https://manpages.ubuntu.com/manpages/trusty/man1/debchange.1.html) tool:
```bash
dch -i
```

Add an entry like this:
```
hello (2.10-5selracnoble1) noble; urgency=medium

    * Install 'testing.sh' in '/usr/bin' and print a prompt message to STDOUT during installation

 -- Carles Sole Grau <carlessg11@gmail.com>  Sat, 26 Jul 2025 21:59:05 +0200
```

The third column specifies which Ubuntu version the package is intended to be built for.

### Build source package
To upload the source changes to the Launchpad PPA, you need to generate and sign them:

- [WiKi Ubuntu - UbuntuDevelopment - Uploading](https://wiki.ubuntu.com/UbuntuDevelopment/Uploading): "In order for your upload to be accepted by the archive maintenance software it must be signed by a GPG key that is associated with a launchpad account that has upload rights for the package."
- [Ubuntu Mantainers Handbook - PackageBuilding ](https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/PackageBuilding.md): "In order for a source package to be accepted by Launchpad, it must be signed"

You can follow the steps described in [Using GPG to manage OpenPGP keys](https://documentation.ubuntu.com/launchpad/en/latest/user/how-to/import-openpgp-key/#using-gpg-to-manage-openpgp-keys) to generate GPG keys and link them to your personal Launchpad account.

Now you can build a signed source package:
- [Building with debuild](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/build-packages/#building-with-debuild)
- [debuild - build a Debian package](https://manpages.ubuntu.com/manpages/plucky/en/man1/debuild.1.html)

```bash
debuild -S -sa -k"<YOUR-GPG-KEY-ID>"
```
Replace `<YOUR-GPG-KEY-ID>` with your GPG key identifier (you can find it with `gpg --list-keys`).  
This command generates the source package and signs it with your key, leaving the files ready to be uploaded to the Launchpad PPA.


### Upload the changes
Follow this guide to upload the source changes: [Upload a package to a PPA](https://documentation.ubuntu.com/launchpad/en/latest/user/how-to/packaging/ppa-package-upload/).

In my case, I chose the [Upload a package to a PPA - FTP](https://documentation.ubuntu.com/launchpad/en/latest/user/how-to/packaging/ppa-package-upload/#ftp) method:

Example `~/.dput.cf` configuration:
```
[selrac-packaging-training]
fqdn = ppa.launchpad.net
method = ftp
incoming = ~selrac/packaging-training
login = anonymous
allow_unsigned_uploads = 0
```
Then simply run:
```bash
dput selrac-packaging-training ../hello_2.10-5ubuntuplucky2_source.changes
```

After a short while, you should see a new Published Package appear in your Launchpad PPA.

## 6. Install your package from your PPA
In your Launchpad PPA, under the `Adding this PPA to your system` section, you will find the commands needed to add this PPA to your system.
You can also read more about it in this guide:
- [Add a Personal Package Archive (PPA)](https://help.ubuntu.com/stable/ubuntu-help/addremove-ppa.html.en)

```bash
sudo add-apt-repository ppa:selrac/packaging-training
sudo apt update
```

After this, a new entry/file will be generated in `/etc/apt/sources.list.d/`

```bash
sudo apt install hello
```

## 7. Paste the message printed during the installation via standard out (STDOUT)

After each `install`, run:
```bash
sudo apt remove hello
```
This allows you to perform a complete installation each time.

### Full output: sudo apt install hello

```bash
selrac@carles-evert hello_ppa/hello-2.10 (feat/take_home_test_canonical_carles_sole_grau) » sudo apt install hello
Installing:                     
  hello

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 0
  Download size: 49,4 kB
  Space needed: 276 kB / 226 GB available

Get:1 https://ppa.launchpadcontent.net/selrac/packaging-training/ubuntu plucky/main amd64 hello amd64 2.10-5ubuntuplucky2 [49,4 kB]
Fetched 49,4 kB in 0s (199 kB/s) 
Selecting previously unselected package hello.
(Reading database ... 342128 files and directories currently installed.)
Preparing to unpack .../hello_2.10-5ubuntuplucky2_amd64.deb ...
Unpacking hello (2.10-5ubuntuplucky2) ...
Setting up hello (2.10-5ubuntuplucky2) ...
this is a test from Carles Solé Grau
Processing triggers for man-db (2.13.0-1) ...
Processing triggers for install-info (7.1.1-1) ...
```

### STDOUT: sudo apt install hello 2> /dev/null
```bash
selrac@carles-evert hello_ppa/hello-2.10 (feat/take_home_test_canonical_carles_sole_grau) » sudo apt install hello 2> /dev/null 
Installing:                     
  hello

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 0
  Download size: 49,4 kB
  Space needed: 276 kB / 226 GB available

Get:1 https://ppa.launchpadcontent.net/selrac/packaging-training/ubuntu plucky/main amd64 hello amd64 2.10-5ubuntuplucky2 [49,4 kB]
Fetched 49,4 kB in 0s (202 kB/s) 
Selecting previously unselected package hello.
(Reading database ... 342128 files and directories currently installed.)
Preparing to unpack .../hello_2.10-5ubuntuplucky2_amd64.deb ...
Unpacking hello (2.10-5ubuntuplucky2) ...
Setting up hello (2.10-5ubuntuplucky2) ...
this is a test from Carles Solé Grau
Processing triggers for man-db (2.13.0-1) ...
Processing triggers for install-info (7.1.1-1) ...
```

### STDERR: sudo apt install hello > /dev/null
```bash
selrac@carles-evert hello_ppa/hello-2.10 (feat/take_home_test_canonical_carles_sole_grau) » sudo apt install hello > /dev/null

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
```




