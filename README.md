# rust-node-dev-container
VS Code Rust + Node Dev Container

## Using this definition with an existing folder

Just follow these steps:

1. If this is your first time using a development container, please follow the [getting started steps](https://aka.ms/vscode-remote/containers/getting-started) to set up your machine.

2. To use VS Code's copy of this definition:
   1. Start VS Code and open your project folder.
   2. Press <kbd>F1</kbd> select and **Dev Containers: Add Development Container Configuration Files...** from the command palette.
   3. Select the Alpine definition.

3. To use latest copy of this definition from the repository:
   1. Clone this repository.
   2. Copy the contents of this folder in the cloned repository to the root of your project folder.
   3. Start VS Code and open your project folder.

4. After following step 2 or 3, the contents of the `.devcontainer` folder in your project can be adapted to meet your needs.

5. Finally, press <kbd>F1</kbd> and run **Dev Containers: Reopen Folder in Container** to start using the definition.


# [Sharing Git credentials with your container](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials)

Sharing Git credentials with your container
The Dev Containers extension provides out of the box support for using local Git credentials from inside a container. In this section, we'll walk through the two supported options.

If you do not have your user name or email address set up locally, you may be prompted to do so. You can do this on your local machine by running the following commands:

```sh
git config --global user.name "Your Name"
git config --global user.email "your.email@address"
```

The extension will automatically copy your local .gitconfig file into the container on startup so you should not need to do this in the container itself.

## Using a credential helper
If you use HTTPS to clone your repositories and have a [credential helper configured](https://docs.github.com/get-started/getting-started-with-git/caching-your-github-credentials-in-git) in your local OS, no further setup is required. Credentials you've entered locally will be reused in the container and vice versa.

## Using SSH keys

There are some cases when you may be cloning your repository using SSH keys instead of a credential helper. To enable this scenario, the extension will automatically forward your local SSH agent if one is running.

First, you should ensure that SSH agent forwarding is enabled on the host. Search for the AllowAgentForwarding option in sshd_config (usually located at /etc/ssh/sshd_config or %programdata%\ssh\sshd_config). If not already, you should set it to yes and reload or restart the sshd service.

You can add your local SSH keys to the agent if it is running by using the ssh-add command. For example, run this from a terminal or PowerShell:

```sh
ssh-add
```

It will add the default files (~/.ssh/id_rsa, .ssh/id_dsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ed25519, and ~/.ssh/identity). If you'd like to specify a key, simply append the path to it after the command.

On Windows and Linux, you may get an error because the agent is not running (macOS typically has it running by default). Follow these steps to resolve the problem:

### Windows:

Start a local Administrator PowerShell and run the following commands:

```sh
# Make sure you're running as an Administrator
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
Get-Service ssh-agent
```

### Linux:

First, start the SSH Agent in the background by running the following in a terminal:

```sh
eval "$(ssh-agent -s)"
```

Then add these lines to your ~/.bash_profile or ~/.zprofile (for Zsh) so it starts on login:

```sh
if [ -z "$SSH_AUTH_SOCK" ]; then
   # Check for a currently running instance of the agent
   RUNNING_AGENT="`ps -ax | grep 'ssh-agent -s' | grep -v grep | wc -l | tr -d '[:space:]'`"
   if [ "$RUNNING_AGENT" = "0" ]; then
        # Launch a new instance of the agent
        ssh-agent -s &> $HOME/.ssh/ssh-agent
   fi
   eval `cat $HOME/.ssh/ssh-agent` > /dev/null
   ssh-add 2> /dev/null
fi
```

On the last line, replace <your ssh key> with your specific ssh key.

For example 

```sh
ssh-add $HOME/.ssh/id_ed25519 2> /dev/null
```

If you encounter any issues, you may want to check the Dev Containers extension's known limitations.


# Creating a non-root user
While any images or Dockerfiles that come from the Dev Containers extension will include a non-root user with a UID/GID of 1000 (typically either called vscode or node), many base images and Dockerfiles do not. Fortunately, you can update or create a Dockerfile that adds a non-root user into your container.

Running your application as a non-root user is recommended even in production (since it is more secure), so this is a good idea even if you're reusing an existing Dockerfile. For example, this snippet for a Debian/Ubuntu container will create a user called user-name-goes-here, give it the ability to use sudo, and set it as the default:

```Dockerfile
ARG USERNAME=user-name-goes-here
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# ********************************************************
# * Anything else you want to do like clean up goes here *
# ********************************************************

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
```


Tip: If you hit an error when building about the GID or UID already existing, the image you selected likely already has a non-root user you can take advantage of directly.


In either case, if you've already built the container and connected to it, run Dev Containers: Rebuild Container from the Command Palette (F1) to pick up the change. Otherwise run Dev Containers: Open Folder in Container... to connect to the container.


