# OUTDATE - DO NOT USE

# Cardano-Node
We are going to build a Cardano Node! 

## What is Cardano?
Cardano (**$ADA**) is a cryptocurrency (similar to Bitcoin and Ethereum, but different in many ways as it is considered to be a 3rd-generation cryptocurrency) that aims to be the most decentralized cryptocurrency that will benefit the world. Cryptocurrencies are all racing to bring the revolutionary technology of blockchain to the real world. A blockchain is simply a ledger (or list of transactions) that is powered by a bunch of connected computers. The blockchain is decentralized (not owned by a single entity/person) and powered by many (connected nodes/computers). The Bitcoin/Ethereum network is powered by a bunch of computers connected together.

The Bitcoin and Ethereum blockchain operates as Proof-of-Work while the Cardano blockchain operates as Proof-of-Stake. 

*What is Proof-of-Work?* **Proof-of-Work** simply means that miners (aka computers with ~~sold out~~ graphic cards or GPUs) WORK to solve complex problems and these complex problems validate the network and it's transactions. The miner that successfully solves the problem gets rewards with Bitcoin/Ethereum. Hence, Proof-of-**WORK**. This is a very energy-intensive process and will only keep increasing as demand continues to grow for cryptocurrency.

*What is Proof-of-Stake?* **Proof-of-Stake** means that validators/stakers (aka people with an X amount/stake of $ADA) can use their current X amount of $ADA to validate the network. Stakers have a choice of running their own node which connects directly to the Cardano blockchain or to "delegate" their stake/amount of **$ADA** to a node. Stakers then earn a reward for validating the network. Staking requires a lot less energy than Proof-of-Work cryptocurrencies. 

So today, we will be building up a Cardano Node on AWS.

## Useful Links
1. Cardano homepage to learn more: https://cardano.org/
2. Cardano-Node instructions: https://cardano-community.github.io/guild-operators/#/README
3. AWS Platform to host our node: https://aws.com
4. Free Udemy Course on Plutus (smart contract programming): https://www.udemy.com/course/plutus-reliable-smart-contracts/
5. Free Udemy Course on Marlowe (smart contract non-programming): https://www.udemy.com/course/marlowe-programming-language/

## Additional Resources
1. Harden your Ubuntu Servers: https://www.lifewire.com/harden-ubuntu-server-security-4178243
2. Harden your Ubuntu Servers 2: https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3
3. EDEN Pool's Useful Youtube Videos: https://www.youtube.com/channel/UCZvLM73pSD3aSJv6_Egf8Fg

---

## Table of Content
1. [Part 1 - Secure the Node](https://github.com/dangitsdavid/cardano-node/blob/main/README.md#part-1-secure-the-node)
2. [Part 2 - Set up Pre-reqs](https://github.com/dangitsdavid/cardano-node/blob/main/README.md#part-2---set-up-the-pre-reqs)
3. [Part 3 - Prep the Cardano Node](https://github.com/dangitsdavid/cardano-node/blob/main/README.md#part-3---start-the-relay-node)
4. [Part 4 - Create an AMI copy and deploy to AWS for Produce Nodes](https://github.com/dangitsdavid/cardano-node/blob/main/README.md#part-3---start-the-relay-node)

## Note: You will need the at least 2-3 servers set up
- 1 Producer Node
- 1-2 Relay Nodes

---
## Part 1: Secure the Node

### Create an EC2 Instance on AWS
- Create an EC2 instance with Ubuntu (I used Ubuntu Server 20.04 LMS)
- Select t2.medium and 24-30 GB for EBS Storage
- We can leave default VPC and Security Group for now (will be a security risk if not addressed later since it is open to the world!)
- Tag it! (i.e., Name, CardanoNode) 
- Save your Private Key (i.e., mykeypair.pem) so that you can connect to it later

### Connect into your Cardano Node
- You can choose whatever SSH method you like, I am running on Windows and my command line has SSH configured. Let's SSH into our Ubuntu Server using the server's public IP address and our generate private key (mykeypair.pem / mykeypair.cer) as ubuntu (default AWS account user for Ubuntu)  
```
ssh ubuntu@12.345.678.90 -i mykeypair.pem  
<yes>
```
- Once connected, lets update the server!  
```
sudo apt-get update
sudo apt-get upgrade
```

### Secure your Node: Create a non-root user with sudo access and allow it SSH permissions
- Open up a separate terminal, we will need to generate a public key with the private key.
```
cd /PathOfSSHFile
ssh-keygen -y -f mykeypair.pem
<copy the ssh-rsa AAAAAABBBBBBCCCC...>
```
- Create the user
```
sudo su  
sudo adduser <username>  
<password>  
<password>  
<ENTER to leave defaults for user info>  
<Y to continue>
```
- Give that user the power of sudo
```
sudo adduser <username> sudo
```
- Log on as that new user and we will need to add SSH to your non-root account
```
sudo su - <username>
mkdir ~/.ssh
cd ~/.ssh
```
- We can add SSH access to our non-root user by adding the public key in a file called authorized_keys
```
nano authorized_keys
<paste your generated public key from earlier>
```
- Now let's try connecting to your Cardano Node with the non-root account
```
ssh <username>@12.345.678.90 -i mykeypair.pem  
<yes>
```

### Secure your Cardano Node: Root User & Firewall
- We are going to change our SSH port (OPTIONAL)
```
sudo nano /etc/ssh/sshd_config
# Uncomment the Port 22 and change the number to a random one (i.e., Port 1234)
Port 1234
```
- In the same file, we will disable root login
```
# It should look something like:
# PermitRootLogin prohibit-password
# We will need to change it to:
PermitRootLogin no
```
- We can now delete the ubuntu user since we won't be needing it anymore 
- (MAKE SURE YOUR NEW USER HAS SUDO ACCESS AND LOGIN WORKS BEFORE DELETING!)
```
sudo userdel -r ubuntu
```
- Check firewall (ufw) status  
```
sudo ufw status  
```
- Set your firewall configuration and allow SSH on port 22 (or to whatever previously set as your SSH port)
```
sudo ufw allow proto tcp from any to any port 22  
```
- Allow node port  
```
sudo ufw allow proto tcp from any to any port 6001  
```
- Activate the firewall and restart ssh
```
sudo ufw enable  
<y>
sudo systemctl restart ssh
```
- Check that your SSH is running on the right port (when using a different SSH port than port 22)
```
sudo systemctl status ssh 

```
- Check firewall (ufw) status  
```
sudo ufw status  
```

### Add your Security Group rules for AWS
- Go to your EC2 Instance and edit it's security group under the EC2 instance's Security tab OR by navigating to the left options bar EC2 > Network & Security > Security Groups
- We will need to just add inbound rules for:
```
# Type                # Protocol   # Port range                         # Source
SSH (or Custom TCP)     TCP         22 (or different SSH port #)        0.0.0.0/0 (or your own IP address)
Custom TCP              TCP         6001                                0.0.0.0/0
```
- Keep in mind, we only need to create inbound rules because security groups are ttateful, which means that creating inbound rules will automatically create outbound rules

---

## Part 2 - Set up the Pre-reqs

### Pre-requisites (prereqs.sh) - Creating our file structure
- Create a tmp directory, change to that tmp directory, install CURL, download the prereq.sh script, and then change it's permissions.
```
mkdir "$HOME/tmp"
cd "$HOME/tmp"
curl -sS -o prereqs.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/prereqs.sh
chmod 755 prereqs.sh
```
- You can view and familiarize yourself with the syntax of the prereq script before proceeding:
```
./prereqs.sh -h
```

```
Usage: prereqs.sh [-f] [-s] [-i] [-l] [-c] [-w] [-p] [-b <branch>] [-n <mainnet|testnet|launchpad|guild|staging>] [-t <name>] [-m <seconds>]
Install pre-requisites for building cardano node and using CNTools

-f    Force overwrite of all files including normally saved user config sections in env, cnode.sh and gLiveView.sh
      topology.json, config.json and genesis files normally saved will also be overwritten
-s    Skip installing OS level dependencies (Default: will check and install any missing OS level prerequisites)
-n    Connect to specified network instead of mainnet network (Default: connect to cardano mainnet network)
      eg: -n testnet
-t    Alternate name for top level folder, non alpha-numeric chars will be replaced with underscore (Default: cnode)
-m    Maximum time in seconds that you allow the file download operation to take before aborting (Default: 60s)
-l    Use IOG fork of libsodium - Recommended as per IOG instructions (Default: system build)
-c    Install/Upgrade and build CNCLI with RUST
-w    Install/Upgrade Vacuumlabs cardano-hw-cli for hardware wallet support
-p    Install/Upgrade PostgREST binary to query postgres DB as a service
-b    Use alternate branch of scripts to download - only recommended for testing/development (Default: master)
-i    Interactive mode (Default: silent mode)
```
- Run the prereqs.sh script and bashrc!
```
./prereqs.sh
. "${HOME}/.bashrc"
```
- Now we wait for the prereqs script to run (may take some time - so feel free to go grab a drink)

---

## Part 3 - Prep the Cardano Node Image

### Setting up your Cardano Node
- Let's clone the Cardano Node repository and go to the folder
```
cd ~/git
git clone https://github.com/input-output-hk/cardano-node
cd cardano-node
```
- Let's make sure we are up to date with the repository
```
git fetch --tags --all
git pull
git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)
```
- We are in a deteached HEAD state, let's run a git status to confirm
```
git status
# It should return the following:
# HEAD detached at <version number>
# nothing to commit, working tree clean
```
- Time to build the Cardano Node using the cabal-build-all.sh script
```
$CNODE_HOME/scripts/cabal-build-all.sh -o
```
- This will probably take some time to run (30+ minutes). Feel free to take a short break while it runs.
- Once completed, we can check the cardano-cli (command line interface) and the cardano-node versions
```
cardano-cli version
cardano-node version
```
- Before continuing, please make sure to edit the CNODE_PORT to match what we've set earlier (i.e., port 6001)
```
nano $CNODE_HOME/scripts/env
CNODE_PORT=6001
```
```
# You will only need to uncomment and provide a name for pool on your block producer node.
POOL_NAME="MyFirstPool"

<CTRL+X to exit editor>
<Y to save changes>
<ENTER to save changes>
```
- Now we will create an AMI (backup image) of the Cardano Node to use for future Relay/Producer Nodes

---

## Part 4 - Create an AMI copy and deploy to AWS for Produce Nodes
- In the EC2 Dashboard, we will right-click on our EC2 instance > Image and templates > Create image
- Name your image as CardanoNode
- Description can be anything
- Once we have the AMI, we can use it to launch the Relay and Producer Nodes.
- More Info on the the differences between Relay and Producer nodes here: https://iohk.zendesk.com/hc/en-us/articles/900001951746-What-are-Block-producing-nodes-and-relay-nodes

---

## Part 5 - Start the Relay Node

- Go to the cnode/scripts directory and run the cnode.sh or deploy-as-systemd.sh script to sync the node to the blockchain! We will be using tmux to split the windows so we have more screen estate!
```
cd $CNODE_HOME/scripts
```
##$ cnode.sh route
```
tmux
./cnode.sh
# Listening on http://127.0.0.1:<port>
```
- You can read up more about tmux here: 
- https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/
##$ deploy-as-systemd.sh route
```
./deploy-as-systemd.sh
<select Y for Relay Node and N for Core/Block Producer Nodes>
```

- Since we have no view, let's open a new window and run the gLiveView script
```
<Press CTRL+B>
<Press %>
# That will split the panes in half
<Press CTRL+B>
<Press the right key arrow key to move to the right pane (you can move in between panes with CTRL+B and Left/Right Arrow Key)>
./gLiveView.sh
```
- Now we have to our node sync with the entire blockchain (May take a long time depending how large the blockchain is already - a few hours to a day or longer)

---

## Postponing documentation until I get my stake pool up and running since some info seems to be outdated.
