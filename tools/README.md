# openpilot tools

## System Requirements

openpilot is developed and tested on **Ubuntu 20.04**, which is the primary development target aside from the [supported embedded hardware](https://github.com/commaai/openpilot#running-on-a-dedicated-device-in-a-car).

Running natively on any other system is not recommended and will require modifications. On Windows you can use WSL, and on macOS or incompatible Linux systems, it is recommended to use the dev containers.

## Native setup on Ubuntu 20.04

**1. Clone openpilot**

NOTE: This repository uses Git LFS for large files. Ensure you have [Git LFS](https://git-lfs.com/) installed and set up before cloning or working with it.

Either do a partial clone for faster download:
``` bash
git clone --filter=blob:none --recurse-submodules --also-filter-submodules https://github.com/commaai/openpilot.git
```

or do a full clone:
``` bash
git clone --recurse-submodules https://github.com/commaai/openpilot.git
```

**2. Run the setup script**

``` bash
cd openpilot
git lfs pull
tools/ubuntu_setup.sh
```

Activate a shell with the Python dependencies installed:
``` bash
poetry shell
```

**3. Build openpilot**

``` bash
scons -u -j$(nproc)
```

## Dev Container on any Linux or macOS

openpilot supports [Dev Containers](https://containers.dev/). Dev containers provide customizable and consistent development environment wrapped inside a container. This means you can develop in a designated environment matching our primary development target, regardless of your local setup.

Dev containers are supported in [multiple editors and IDEs](https://containers.dev/supporting), including Visual Studio Code. Use the following [guide](https://code.visualstudio.com/docs/devcontainers/containers) to start using them with VSCode.

#### X11 forwarding on macOS

GUI apps like `ui` or `cabana` can also run inside the container by leveraging X11 forwarding. To make use of it on macOS, additional configuration steps must be taken. Follow [these](https://gist.github.com/sorny/969fe55d85c9b0035b0109a31cbcb088) steps to setup X11 forwarding on macOS.

## WSL on Windows

[Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about) should provide a similar experience to native Ubuntu. [WSL 2](https://docs.microsoft.com/en-us/windows/wsl/compare-versions) specifically has been reported by several users to be a seamless experience.

Follow [these instructions](https://docs.microsoft.com/en-us/windows/wsl/install) to setup the WSL and install the `Ubuntu-20.04` distribution. Once your Ubuntu WSL environment is setup, follow the Linux setup instructions to finish setting up your environment. See [these instructions](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps) for running GUI apps.

**NOTE**: If you are running WSL and any GUIs are failing (segfaulting or other strange issues) even after following the steps above, you may need to enable software rendering with `LIBGL_ALWAYS_SOFTWARE=1`, e.g. `LIBGL_ALWAYS_SOFTWARE=1 selfdrive/ui/ui`.
## Resolving Dependency and Environment Issues with Pyenv and Poetry in WSL

When working with Python development tools like `pyenv` and `poetry` within the Windows Subsystem for Linux (WSL), users may encounter conflicts and issues due to path and environment settings. This guide aims to address these issues by outlining the steps necessary to properly configure your environment to avoid conflicts between Windows and Linux tools.

### Problem Description

Users might experience some dependency issues thatr manifest as windows paths being used in linux.  These will likely manifest in ways that break your virtual environmentsl, poetry, or you may notice things like paths beginning with /mnt/c/users/username   instead of your /home/username, or see a mention of CMD not being able to start from a UNC path..  There is a number of bugs which may arise from certain deployment tools in windows, or even visual studio being installed..  These errors occur because WSL is incorrectly referring to the Windows version of `pyenv` instead of a version installed within the WSL environment (Ubuntu). This misconfiguration can also interfere with `poetry` and other virtual environment tools, leading to a non-functional Python environment.

### Solution Overview

The solution involves ensuring that each tool is installed within its respective environment and that their paths are configured correctly to prevent overlap. Here are the steps to set up `pyenv` correctly within WSL:

#### 1. Install `pyenv` in WSL

First, install `pyenv` directly in your WSL environment, not Windows. This ensures that the Linux version of the tool is used. You can install `pyenv` using the following command:

```bash
curl https://pyenv.run | bash
```

#### 2. Configure Environment Variables

After installation, add the following lines to your `~/.bashrc` file, replacing `USER` with your actual username:

```bash
export PATH="/home/USER/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Reload your bash configuration:

```bash
source ~/.bashrc
```

#### 3. Disable Windows Path Appending in WSL

To prevent WSL from using Windows paths (which can cause conflicts), modify the WSL configuration:

```bash
sudo vi /etc/wsl.conf
```

Add the following configuration to disable Windows path appending:

```ini
[interop]
appendWindowsPath = false
```

After saving your changes and exiting the editor, you'll need to restart WSL to apply the new configurations. Now, while you could manually close each WSL session and patiently wait for about 8 seconds, let’s be real—ain’t nobody got time for that! Instead, do what the pros do. Once you’ve confirmed that everything is saved, simply run the following command to instantly shut down all WSL sessions:

```
wsl --shutdown
```
*This command ensures that all changes are applied promptly and you can start fresh with your newly configured settings. *

Then, reopen WSL and check that no Windows paths are included:

```bash
echo $PATH
```

#### 4. Optional: Configuring Specific Paths

If you need specific Windows paths (e.g., for VS Code), you can manually add them to your path in WSL:

```bash
export PATH="$PATH:/mnt/c/Users/{username}/AppData/Local/Programs/Microsoft VS Code/bin"
```
Alternatively, you can configure settings globally or locally for your WSL environments:

- Use the \`.wslconfig\` file, typically located in your Windows user directory (C:\\Users\\YourUsername\\.wslconfig), to apply global settings across all installed distributions on WSL 2.
- Use the \`wsl.conf\` file, found inside the \`/etc\` directory of each Linux distribution (e.g., /etc/wsl.conf), to apply settings locally, specific to each Linux distribution installed on either WSL 1 or WSL 2.



```ini
[boot]
command = export PATH="$PATH:/mnt/c/Users/YOUR_USERNAME/AppData/Local/Programs/Microsoft VS Code/bin"
```

## CTF
Learn about the openpilot ecosystem and tools by playing our [CTF](/tools/CTF.md).

## Directory Structure

```
├── ubuntu_setup.sh     # Setup script for Ubuntu
├── mac_setup.sh        # Setup script for macOS
├── cabana/             # View and plot CAN messages from drives or in realtime
├── camerastream/       # Cameras stream over the network
├── joystick/           # Control your car with a joystick
├── lib/                # Libraries to support the tools and reading openpilot logs
├── plotjuggler/        # A tool to plot openpilot logs
├── replay/             # Replay drives and mock openpilot services
├── scripts/            # Miscellaneous scripts
├── serial/             # Tools for using the comma serial
├── sim/                # Run openpilot in a simulator
├── ssh/                # SSH into a comma device
└── webcam/             # Run openpilot on a PC with webcams
```
