# MINECONF, the Minecraft Conf and Bedrock Daemon

This project is intended to help you manage your configuration files easily and to be able to use the power of
GitHub Actions to have the configuration files modified directly on GitHub and deployed automatically to your server!

## Setup
First things first, make sure your fork is private. You do not want your configurations to be public. To do so you need
to duplicate the repository, you can use the guide here: https://help.github.com/en/github/creating-cloning-and-archiving-repositories/duplicating-a-repository 

Change the folder name `.github_` to `.github`, I changed the name on the main repo because I don't the github action to trigger on the public repository.

Have your Bedrock dedicated server at `/opt/minecraft` and have this repo at `~/mineconf` of the deployment user

Move your configuration files to your `mineconf` fork:

```sh
cd /opt/minecraft
mv whitelist.json permissions.json server.properties HOME/mineconf
```

Make Symlinks to the configuration files and the service, e.g:
```sh
ln -s $HOME/mineconf/whitelist.json /opt/minecraft/whitelist.json
ln -s $HOME/mineconf/permissions.json /opt/minecraft/permissions.json
ln -s $HOME/mineconf/server.properties /opt/minecraft/server.properties

ln -s $HOME/mineconf/minecraft.service /etc/systemd/system/minecraft.conf
```

You need to add your public key to Github so the github actions can interact with your repository without prompting for a password, to
do so you can follow this guide: https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account

You can create a new key to add to GitHub actions so GitHub has access to your machine for automated deployment.

Add the command `sudo systemctl restart minecraft` to the `sudoers` file with rule `NOPASSWD`, again for GitHub actions to not prompt for
a password on deployment.


## minecraft.service: The Minecraft Service Daemon for Linux.

The configuration files include a `minecraft.service` file that is intended to be used to run your Minecraft Bedrock dedicated server.
The goal of this service is to have your linux server run your minecraft server in the background without needing to have a user connected and
typing manually the command on the machine.

For example my setup uses a Ubuntu Server LTS (18.04) with `minecraft.service` running as a background. I setup my machine so if there is an outage,
the computer will turn right back on when electricity comes back. There's is many other advantages to working like this, and this comes from setups
I put in place for many of my clients, where their apps still run in production and never complained. Happy client, happy week end.

### minecraft.service configuration
All you have to do is replace user and group by a user and group of your choice on your machine. You can also just create a user minecraft and put him
in the group minecraft, whatever you want.

After you're done with that, you can now start and enable the service, to have it run on machine boot.

```sh
sudo systemctl start minecraft
sudo systemctl enable minecraft
```

## GitHub Deploy Action
The deployment workflow is fairly simple and I wish to keep it simple for this repo as for many this is an introduction to CI/CD. So I just use an SSH action,
the command that runs on it is: go to the mineconf repo location, stash the changes made on the machine, pull the newest changes and restart the minecraft
service.

All the deployment is contained inside the `01-config-deployment.yml`

There is some secrets that needs to be added which are:
- HOST
- PRIVATE_KEY
- USERNAME

## Collaborations
Any collaboration is welcome, just make a pull request to this repository.
