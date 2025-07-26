# Take Home Test from Canonical - Carles Solé Grau

## TO REMOVE

```bash
sudo apt install devscripts
sudo apt install build-essential devscripts debhelper
```

Reference:
[Minimum Complete Example of Debian Packaging and Launchpad PPA (hellodeb)](https://metebalci.com/blog/a-minimum-complete-example-of-debian-packaging-and-launchpad-ppa-hellodeb/)

## TO REMOVE

Pick an existing Debian package from the Ubuntu archive. The selected package should be simple and easy to work with.

## 1. Download the source code of that Debian package

Go to [Ubuntu Packages](https://packages.ubuntu.com/) and select your desired Ubuntu version.
In my case: [Plucky](https://packages.ubuntu.com/plucky/).

Choose a simple package to avoid overcomplicating the exercise.

I selected [hello](https://packages.ubuntu.com/plucky/hello) because it is an example package based on GNU Hello.
It is also the one used in [the reference link](https://pastebin.ubuntu.com/p/hZ4sH647Jt/).

In the **Download** section, select your target architecture to get instructions for downloading the package:
[https://packages.ubuntu.com/plucky/amd64/hello/download](https://packages.ubuntu.com/plucky/amd64/hello/download)

There are multiple ways to get the source code of a package:
- [Get the source of a package](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/): Link provided in the test assignment instructions.
- [apt-src](https://manpages.ubuntu.com/manpages/plucky/man1/apt-src.1p.html)
- [How to get source code of package using the apt command on Debian or Ubuntu](https://www.cyberciti.biz/faq/how-to-get-source-code-of-package-using-the-apt-command-on-debian-or-ubuntu/)

Feel free to use the method you are most comfortable with.
For this exercise, I will use [pull-pkg](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/#pull-pkg), as described in the official guide. It is one of the simplest and fastest methods.

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

Create a `testing.sh` file and place it in the `debian` directory.  
This is because the `debian/` directory is the standard location recognized by the Debian packaging system for defining what gets installed, where, and how. See: [Debian Policy Manual – Source Packages](https://www.debian.org/doc/debian-policy/ch-source.html)

Content of `debian/testing.sh`:

```bash
#!/bin/bash

# Use '>&2' to redirect output to STDERR
echo "this is a test from Carles Solé Grau" >&2
```

Make the script executable:

```bash
chmod +x debian/testing.sh
```

To verify the message goes to STDERR:

```bash
debian/testing.sh > stdout.txt 2> stderr.txt
```

Then check `stderr.txt` to confirm the output.  
Don't forget to delete the temporary files after testing.

To install this script on your system when installing the `.deb` package, add/edit the file `debian/install` with this line:

```
debian/testing.sh /usr/bin
```
`/usr/bin` has been used as the installation path because the [reference example link provided](https://pastebin.ubuntu.com/p/hZ4sH647Jt/) also points to this location.

## 3. Echo a string during the Debian package installation via STDOUT

This message must be printed to **standard output (STDOUT)**, so `debian/testing.sh`. can not be reused.

To do this, create a [maintainer script](https://www.debian.org/doc/debian-policy/ch-binary.html#prompting-in-maintainer-scripts) that runs during installation.  
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

Make it executable:

```bash
chmod +x debian/postinst
```

### Validate the prompt during installation

Before proceeding and pushing all this code to the Launchpad PPA, it is a good idea to validate everything locally first.

Build the `.deb` package following [Ubuntu - Build packages](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/build-packages/)

Install the prerequisites:
```bash
sudo apt update
sudo apt install sbuild debhelper ubuntu-dev-tools piuparts
```
Since we just want to test the .deb package locally, we can use:
```
sbuild -c <RELEASE>-<ARCH>[-shm]
```
However, I had some issues with this tool, so I ended up using `debuild` instead:
```
debuild -us -uc 
```

Before running it, make sure `help2man` is installed on your system:
```
sudo apt update
sudo apt install help2man
```

If everything goes well, a `.deb` file should be generated one directory up. To test the installation:

```bash
sudo dpkg -i ../hello_2.10-*.deb
```

You should see the following printed in the terminal:
`/usr/bin` has been used as in the installation path as the [reference example link provided](https://pastebin.ubuntu.com/p/hZ4sH647Jt/) points also there.
```
this is a test from Carles Solé Grau
```

After installation run:
```
dpkg -S testing.sh; testing.sh
```

Outuput should look like:
```
hello: /usr/bin/testing.sh
this is a test from Carles Solé Grau
```

## 4. Push all your changes to a public Git repository (e.g. Launchpad Git, GitHub, or GitLab)

Choose your preferred Git provider and push all your changes there.
