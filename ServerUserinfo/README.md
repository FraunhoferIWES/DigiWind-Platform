# FB-I-Server user information
Documentation of useful user information for FBI server.

# Contents
- [FB-I-Server user information](#fb-i-server-user-information)
- [Contents](#contents)
- [Basic Infos](#basic-infos)
- [Access to the server and file management](#access-to-the-server-and-file-management)
  - [Access with remote desktop](#access-with-remote-desktop)
  - [Access with SSH](#access-with-ssh)
    - [Advanced Setup with SSH Key Based Authentication](#advanced-setup-with-ssh-key-based-authentication)
- [Ubuntu Command Line Basics](#ubuntu-command-line-basics)
- [Optimization solvers](#optimization-solvers)
- [Working with MATLAB](#working-with-matlab)
  - [YALMIP and solvers](#yalmip-and-solvers)
  - [CoolProp](#coolprop)
- [Working with Python](#working-with-python)
  - [Installation of Miniconda](#installation-of-miniconda)
  - [Some basics of Miniconda](#some-basics-of-miniconda)
  - [Calling MATLAB from Python](#calling-matlab-from-python)
- [Working with Docker](#working-with-docker)
- [Working with terminal multiplexers](#working-with-terminal-multiplexers)

# Basic Infos
The physical machine is an Ubuntu 20.04 x64 system, with 64 physical cores and 258GiB memory.

The physical machine is split into virtual containers, named `iet3gold`, `iet3bismuth` and `iet3mercury`, each with a separate IP adress. More details on this can be found in the config/admin sections.

If you are only a daily user of the FBI-Server, your main interest is the container **`iet3bismuth`** with IP address **`128.131.134.48`**.

`iet3bismuth` is running the Linux distribution `Ubuntu 20.04`, as of 11/2021. For basic infos using the command prompt, see [Ubuntu command line basics](#ubuntu-command-line-basics).

# Access to the server and file management
As a user, you have 2 primary ways of accessing `iet3bismuth`, either via command line by use of the Secure Shell ([SSH](#access-with-ssh)) protocol or via [remote desktop](#access-with-remote-desktop).

A requirement for any type of access is that you are inside the TU Wien-Network (e.g. local access point at TU Wien or via VPN) and that your IP adress (e.g. your fixed TU Wien-VPN IP) can communicate through the firewall of the remote server (IP: `128.131.134.48`). Firewall port `3389` has to be open if you want to access via [remote desktop](#access-with-remote-desktop) and port `22` has to be open for you to be able to access via [SSH](#access-with-ssh). Administration is done by IET IT-admins, e.g. Franz Trummer. Please ask for access there. To simply check if you can access the server, type in your command prompt on your local computer:

```ps
C:\Users\username> ping 128.131.134.48
```

Furthermore, your IET network account has to be specified to network group `iet3-staff` by IET IT-admins, e.g. Franz Trummer. This should be default for employees in IET FB-I and your local user is thus created automatically at first login.

Your **login credentials** are, by default, the **same as those of your IET employee account**.

Note: For IET-external users, e.g. students, a user account has to be created manually, with the password beeing set at first login. In this case, please contact the `iet3bismuth`-Admins.

## Access with remote desktop
Just open the remote desktop client (Windows: the "old" client often performs better) and connect to the remote Computer `128.131.134.48`. Username with prefixed domain `ITE\` ! Thus, you get the basic Ubuntu Desktop experience.

## Access with SSH
SSH access is recommended for fast, basic operations.

The basic way to connect remotely via ssh is by typing in your command prompt (Terminal, [PowerShell](https://de.wikipedia.org/wiki/PowerShell), "Eingabeaufforderung", etc.):

```ps
C:\Users\username> ssh username@128.131.134.48
```
Here, the login credentials are, again, the same as those of your IET employee account.

### Advanced Setup with SSH Key Based Authentication
A more conveniant way of SSH acess is achieved by SSH Key-Based Authentication. You can then access the server, for example, by simply typing
```ps
C:\Users\username> ssh bismuth
```
and you are not even asked for a password.

**How to set this up?**
The main steps are given below, but you can find more detailed tutorials online, for example, [here](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement).

1. Do you already have a password protected SSH key pair? For example the one you use for TUgitLab authentication? If so, continue to step 2 . Otherwise, create a new key pair by running
    ```ps
    C:\Users\username> ssh-keygen -t ed25519
    ```
    in your command prompt.

2. Copy your public key via SSH-Copy-ID. Just run
    ```ps
    C:\Users\username> ssh-copy-id -i ~/.ssh/id_ed25519 username@128.131.134.48
    ```
    in your command prompt. If your preferred key does not have its default name `~/.ssh/ed25519`, you have to change this part accordingly. If your command prompt does not recognize the command `ssh-copy-id`, you can use Git Bash. If this is not working, you can also use `scp` in a [PowerShell](https://de.wikipedia.org/wiki/PowerShell) prompt:
    ```ps
    # Make sure that the .ssh directory exists in your server's home folder
    C:\Users\username> ssh username@128.131.134.48 mkdir .ssh
    
    # Use scp to copy the public key file generated previously to authorized_keys on your server
    C:\Users\username> scp "C:\Users\username\.ssh\id_ed25519.pub" username@128.131.134.48:.ssh\authorized_keys
    ```

3. Add the created key to your Windows ssh-agent, so that you dont have to type the password for every login. Just run the following lines in your command prompt. Note: This might only work in [PowerShell](https://de.wikipedia.org/wiki/PowerShell).
    ```ps
    # By default the ssh-agent service is disabled. We want to configure it to automatically start with Windows.
    C:\Users\username> Get-Service -Name ssh-agent | Set-Service -StartupType Automatic

    # Make sure you're running as an Administrator
    C:\Users\username> Start-Service ssh-agent
    
    # This should return a status of Running
    C:\Users\username> Get-Service ssh-agent
    
    # Now load your key files into ssh-agent
    C:\Users\username> ssh-add "C:\Users\username\.ssh\id_ed25519"
    ```
    **Note:** If you followed these steps correctly, but the command `ssh-add` returns `communication with agent failed`, this could be   [a known Windows issue](https://github.com/PowerShell/Win32-OpenSSH/issues/1234). As a workaround, you create/install a dummy sshd service like this:
    ```ps
    C:\Users\username> sc.exe create sshd binPath=C:\Windows\System32\OpenSSH\ssh.exe
    ```
    After this, try again to load your key files into ssh-agent via `ssh-add`.


4. Optional, but recommended: Set up a config file, to make your login even more simple. In your default `.ssh` path (in Windows, for example `C:\Users\username\.ssh\` , create a file `config` (without file ending) and type in the following lines:
    ```
    Host iet3bismuth bismuth
        HostName 128.131.134.48
        User username
        IdentityFile ~/.ssh/id_ed25519
    ```
    You can also download this [example config file](files/config) and change accordingly.

You should now be able to login to `iet3bismuth`, by simply typing:
```ps
C:\Users\username> ssh bismuth
```

**Note:** If you created more than 1 ssh key on your local machine (e.g. a different key for authenitication with [TUgitLab](https://gitlab.tuwien.ac.at/)), you might want to add the following lines to your `config` file:

    
    Host gitlab.tuwien.ac.at
        HostName gitlab.tuwien.ac.at
        IdentityFile ~/.ssh/id_rsa
    

where the file path should point to your ssh key used for gitLab authentication, of course.



# Ubuntu Command Line Basics
Use Google or look into Linux command line cheat sheets (e.g. [here](https://cheatography.com/davechild/cheat-sheets/linux-command-line/)  ) for basic commands, e.g. directory operations as `cd dir`, `cd ..` or `ls`.

To open MATLAB command prompt in the shell (without opening the GUI), simply type 


```ps
user@iet3bismuth:~$ matlab -nodesktop
```
You can skip the ` -nodesktop` option entirely if you are connected via SSH. To exit the MATLAB session, type `exit`.

To run a MATLAB script in a single MATLAB session, which is, at the end of the script, automatically terminated, u can use
```ps
user@iet3bismuth:~$ matlab -batch name
```
, where `name` is the name of the matlab script in the working folder without file extension.

# Optimization solvers
Currently, the optimization solvers [GUROBI](https://www.gurobi.com/) and [MOSEK](https://www.mosek.com/) are installed on the FBI-server. More details on the installation and licensing can be found in the config/admin sections of this repository.

To browse for the installed solvers, look into the folder `/opt` on `iet3bismuth`. E.g., just change with `cd /opt` to the system-wide installation folder and screen the folder with `ls`.

To work with these solvers in MATLAB, see the sections [Working with MATLAB](#working-with-matlab) and [YALMIP and solvers](#yalmip-and-solvers).


# Working with MATLAB
To open MATLAB command prompt in the shell (without opening the GUI), simply type 
```ps
user@iet3bismuth:~$ matlab -nodesktop
```
You can skip the ` -nodesktop` option entirely if you are connected via SSH. To exit the MATLAB session, type `exit`.

To run a MATLAB script in a single MATLAB session, which is, at the end of the script, automatically terminated, u can use
```ps
user@iet3bismuth:~$ matlab -batch name
```
, where `name` is the name of the matlab script in the working folder without file extension.

## YALMIP and solvers
To automatically add latest YALMIP, GUROBI and MOSEK version to your MATLAB path, create a folder `\MATLAB` in your user home directory (if not existing already) and copy the file [startup.m](files/startup.m) to this folder. 

To browse for currently installed YALMIP versions, change with `cd /opt` to the system-wide installation folder and screen the folder with `ls`.

To run a script with a specific version of YALMIP, manually add the path via `addpath`. For example, add the line
```
addpath(genpath('/opt/YALMIP-R20200116'));
```
at the beginning of your MATLAB script. This can be useful if you face certain compatibility issues, e.g. with GUROBI solver. Important: To make sure MALTAB is using the desired version, make sure to remove any other versions from the search path. For example:
```
rmpath(genpath('/opt/YALMIP-R20200116'));
```

To add different versions than the default "latest" version of GUROBI and MOSEK, just add the path to the respective folder located in `/opt` manually, similar to the way described above for YALMIP. You can also modify the [startup](files/startup.m) file, if you want to do this automatically for every MATLAB session.


## CoolProp
[CoolProp](http://www.coolprop.org/) is a very useful C++ library for thermophysical material and mixture properties. The only interface for MATLAB that is supported as of version 6.2 is via Python.

If you don't have a Python installation on `iet3bismuth`, check the short installation guide in [Working with Python](#working-with-python).

If you already have a Python installation (i.e. Miniconda), I recommend creating a new environment specifically for MATLAB. To do so, run (in a command prompt on `iet3bismuth`:
```ps
(base) user@iet3bismuth:~$ conda create --name matlabenv python=3.7
```
Then activate the environment by
```ps
(base) user@iet3bismuth:~$ conda env activate matlabenv
```
and install CoolProp with `pip`:
```ps
(matlabenv) user@iet3bismuth:~$ pip install CoolProp
```
(Note: As of 03/2021 CoolProp is not working with Python 3.9 and `pip`. If you want to run Python 3.9, you have to install it from the unofficial `conda-forge` channel, i.e. `(matlabenv) user@iet3bismuth:~$ conda install conda-forge::coolprop`)

To now use the installed CoolProp python package in MATLAB, u have to tell MATLAB the location to your python executable, via `pyenv`. Open a MATLAB session and run:
```python
>> pyenv('Version','/home/username/miniconda3/envs/matlabenv/bin/python.exe')
```
Change the path accordingly to where your python executable for the created environment `matlabenv` is located. The above path should be your default location.

To check if CoolProp is running in MATLAB, try, e.g.:
```python
>> py.CoolProp.CoolProp.PropsSI('T','P',101325,'Q',0,'Water')
```
to get the normal boiling point temperature of water.

Nothing else to do here, MATLAB remembers the pyversion for your next session.

# Working with Python
You are free to install and use your favorite Python distribution on `iet3bismuth`. In the following section, the setup of the Anaconda Python distribution is explained.

## Installation of Miniconda
[Miniconda](https://docs.conda.io/en/latest/miniconda.html) is a free minimal installer for conda. It is a small, bootstrap version of Anaconda that includes only conda, Python, the packages they depend on, and a small number of other useful packages.

1. In a shell session on `iet3bismuth`, move to your Downloads folder via `cd Downloads` and download the latest Miniconda shell script by running:
	```ps
	user@iet3bismuth:~$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
	```

2. After that, run the miniconda installation script:
	```ps
	user@iet3bismuth:~$ bash Miniconda3-latest-Linux-x86_64.sh
	```
	
3. Follow the prompts on the installer screens and accept the defaults. To make the changes take effect, close and then re-open your terminal window and test your installation via the command `conda list`.
	
## Some basics of Miniconda
The most important concept of conda is that it allows you to create separate environments containing files, packages, and their dependencies that will not interact with other environments.

When you begin using conda, you already have a default environment named `base`. You don't want to put programs into your base environment, though. Create separate environments to keep your programs isolated from each other.

Check out this short [Getting started with conda](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-environments) guide. (At least the first page)

## Calling MATLAB from Python
The [MATLAB Engine API](https://de.mathworks.com/help/matlab/matlab-engine-for-python.html) for Python provides a package for Python to call MATLAB as a computational engine. 

The API is already preinstalled on `iet3bismuth`. The installation for MATLAB version 2020b is located in `/opt/matlab20bPythonengine/lib`. Hence, to import the API as a python module, you have to add this folder to the path, as given below.
```python
import sys
sys.path.append("/opt/matlab20bPythonengine/lib")
import matlab.engine
```


# Working with Docker
[Docker](https://docs.docker.com/get-started/overview/) is an open source containerization platform which provides the ability to package and run an application in an isolated environment called a container.

By default, Docker is only accessible with root privileges (i.e. sudo or admin privileges). If you want to use Docker as a regular user, you need to have your user added to the `docker` group. Therefore, please ask your `iet3bismuth` admins.

Some helpful commands regarding docker:
- `docker ps -a` #shows all docker containers
- `docker ps --filter "status=exited"` #shows all stopped docker containers
- `docker container rm <Docker ID>` #removes docker with specific ID
- `docker restart <Docker ID>` #restarts docker with specific ID or name

# Working with terminal multiplexers
A terminal multiplexer is an application that can be used to multiplex several separate terminal sessions inside a single terminal window. It is also used to detach and reattach sessions from a terminal.

This is **very useful** if you execute **one or several long simulation/optimization runs** in a terminal. When working in a multiplexer session, your process will not be terminated when you log out (or close the window, shut down your local machine or lose network connection). Instead, you can easily reattach to said session.

Commonly used multiplexers include [GNU Screen](https://wiki.ubuntuusers.de/Screen/), [tmux](https://wiki.ubuntuusers.de/tmux/) and [Byobu](https://www.byobu.org/home). We recommend using **Byobu**, since it is very easy to get started with it. All of byobu's functionality is conveniently mapped to F1 to F12. It has a help menu to see keybindings and offers window tabs in an easy to interpret format.

Byobu is already preinstalled on `iet3bismuth`. Just run
```ps
user@iet3bismuth:~$ byobu
```
in a new terminal session to start it. Press F6 to exit byobu. Check the [homepage](https://www.byobu.org/home) or other help pages to learn more.
