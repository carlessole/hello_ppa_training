# Take Home Test from Canonical - Carles Solé Grau

## TO REMOVE

```bash
sudo apt install devscripts
sudo apt install build-essential devscripts debhelper
```

Reference:
[Minimum Complete Example of Debian Packaging and Launchpad PPA (hellodeb)](https://metebalci.com/blog/a-minimum-complete-example-of-debian-packaging-and-launchpad-ppa-hellodeb/)

## TO REMOVE

Pick one existing Debian package from the Ubuntu archive. The selected package should be simple and easy to work with.

## 1. Download the source code of that Debian package

Go to [Ubuntu Packages](https://packages.ubuntu.com/) and select the desired Ubuntu version.
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
For this exercise, I will use [pull-pkg](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/#pull-pkg), as described in the guide provided in the test instructions. It is one of the simplest and fastest ways to achieve it.

```bash
sudo apt update && sudo apt install ubuntu-dev-tools
```

[pull-pkg](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/how-to/get-package-source/#pull-pkg) offers several ways to specify the version or state of the package you want to retrieve.  
For this exercise, we will use the simplest approach:

```bash
pull-lp-source hello
```

### Initialize Git repository

Initialize a Git repository in the package source directory to track all changes from this point onward.
Example:

```bash
cd hello-2.10
git init
```

From now on, try to commit each step individually for easier tracking and review.
