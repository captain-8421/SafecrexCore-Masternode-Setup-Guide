SafeCrex-Masternode-Setup-Guide
 
<img src="https://avatars1.githubusercontent.com/u/62416254?s=400&u=ad4ef48524c7a39acd5da813fbc6e624940ccc03&v=4">

SafeCrex Masternode Setup Guide
==========================

## Introduction

This guide is for a single masternode, on a Ubuntu 18.04 64-bit server (VPS) running headless and will be controlled from the wallet on your local computer (Control wallet). The wallet on the the VPS will be referred to as the Remote wallet.

You will need your masternode server details for progressing through this guide including the public IP address.

### VPS Server Recommendations ###

[Digital Ocean](https://www.digitalocean.com/products/droplets/)

[Vultr](https://www.vultr.com/products/cloud-compute/#compute)

**For either of the above VPS services, the economical $5 plan will be sufficient for your masternode.**

---

First the basic requirements:

1. 1,000 SFCX
1. A main computer (your everyday computer) – This will run the Safecrex Core control wallet, hold your collateral 1,000 SFCX and can be opened and closed without affecting the masternode.
1. Masternode Server (VPS Suggested – The wallet daemon that will be on 24/7).
1. A unique IP address for your VPS / Remote wallet.

**For security reasons, you’re are going to need a different IP for each masternode you plan to host.**

## Configuration

**Step 1:** Using the control wallet, enter the debug console `Tools > Debug console` and type the following command:

```
masternode genkey
```

*(This will be the masternode’s privkey variable. We will need this later…)*

**Step 2:** Using the control wallet, enter the following command:

```
getaccountaddress "chooseAnyNameForYourMasternode"
```

**Step 3:** Still in the control wallet, send 1,000 SFCX to the wallet address you generated in Step 2. 

Be 100% sure that you entered the address correctly. You can verify this when you paste the address into the **"Pay To:"** field, the label will auto populate with the name you chose. 

Also make sure this is exactly **1,000 SFCX**; No less, no more.

**Be absolutely sure the send to address is copied correctly and then check it again. We cannot help you if you send 10,000 BTCT to an incorrect address.**

Please allow at least 1 block confirmation to complete before moving on.

**Step 4:** Still in the control wallet, enter the command into the console:

```
masternode outputs
```

*This gets the proof of transaction of sending 1,000 SFCX*

**Step 5:** Still on the main computer, go into the BTCT data directory:

OS | Path to BTCT
------------ | -------------
Windows | `%Appdata%/Roaming/SafecrexCore/`
macOS | `~/Library/Application\ Support/SafecrexCore/`
Linux | `~/.safecrexcore/`

Find the masternode.conf file, edit it in your favorite text editor and add the following line to it:

```
<Name of Masternode(Use the name you entered earlier for simplicity)> <Unique VPS Public IP address>:9792 <The result of Step 1> <Result of Step 4> <The number after the long line in Step 4>
```

Example:

```
mn1 127.0.0.2:9792 93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg 2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c 0
```

Substitute with your own values and without the "<>"s.

Lastly, close the control wallet and open again to load the new configuration file.

## VPS Remote Wallet Install

Install the latest version of the Safecrex Core wallet onto your masternode. The latest version can be found here: [Safecrex core release](https://github.com/safecrex/safecrexcore/releases).

**Step 1:** Log in to your VPS via SSH:

```
cd ~
```

**Step 2:** From your home directory, download the latest version from the BTCT GitHub repository:

```
wget https://github.com/safecrex/safecrexcore/blob/master/wallet/Safecrex-linux64.tar.gz
```

Always check the releases page for the latest version and update the URL to reflect the most current version.

**Step 3:** Unzip & Extract:

```
tar -zxvf Safecrex-linux64.tar.gz
```

**Step 4:** Copy the files to the local bin. **Requires sudo**

```
cp -r sfcx/safecrexd /usr/local/bin
```
```
cp -r sfcx/safecrex-cli /usr/local/bin
```

**Step 5:** Note: If this is the first time running the wallet in the VPS, you’ll need to attempt to start the wallet:

```
safecrexd --daemon
```

**Step 6:** Stop the daemon after the blockchain downloads:

You can verify you have the entire blockchain by comparing the results of `safecrex-cli getinfo` with the [Safecrex Block Explorer](https://blockinfo.safecrex.trade/)

Once verified:

```
safecrex-cli stop
```

**Step 7:** Navigate to the btct data directory:

```
cd ~/.safecrexcore
```

**Step 8:** Open the configuration file by typing:

```
nano safecrex.conf
```

**Step 9:** Make the config look like this with your values:

```
rpcuser=someuser
rpcpassword=somepass
server=1
daemon=1
rpcport=9792
rpcconnect=127.0.0.1
masternode=1
masternodeprivkey={Result of Step 1}
externalip={Your VPS unique public ip address}:9792

```

*Make sure to replace rpcuser and rpcpassword with your own and remove {}.*

**Step 10:** Save and exit the file:

```
Ctr+x to exit and press Y to save changes and press enter to close
```

**Please be sure to have port 9792 open on your server firewall if applicable for your control wallet to be able start the masternode remotely.**

## Start the Masternode

**Step 1:** Navigate back to the Safecrex VPS server:


**Step 2:** Start the wallet daemon:

```
safecrexd --daemon
```

**Step 3:** From the Control wallet debug console(Local wallet):

```
masternode start-alias mn1
```

Where "mn1" is the name of your masternode alias we shown in example,you can choose yours.

**The following should appear.**

```
{
  "alias": "mn1",
  "result": "successful"
}


```


**Step 4:** Back in the VPS (remote wallet), Use the following command to check status:

```
safecrex-cli masternode status
```

You should see something like:

```
{
  "outpoint": "2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c",
  "service": "127.0.0.2:9792",
  "payee": "SRKc3HceTAcNEGQWcNmDNF1oJzMySddk2C",
  "status": "Masternode successfully started"
}
```

If you see status Not capable masternode: Hot node, waiting for remote activation, you need to wait a bit longer for the blockchain to reach consensus. It's possible it may take 60 to 120 minutes before the activation can be done. You can also try restarting the VPS wallet `safecrex-cli stop` and then `safecrexd --daemon` command again.

Safecrex core Masternode Setup is Complete!


## Tearing down a Masternode

1. `safecrex-cli stop` from the masternode to stop the wallet.
1. Then from your control wallet, edit your masternode.conf, delete the mn1 masternode line entry.
1. Restart the control wallet.
1. Your 1,000 SFCX will now be unlocked.
