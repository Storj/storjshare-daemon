Storj Share Daemon
==================

[![Build Status](https://img.shields.io/travis/Storj/storjshare-daemon.svg?style=flat-square)](https://travis-ci.org/Storj/storjshare-daemon)
[![Coverage Status](https://img.shields.io/coveralls/Storj/storjshare-daemon.svg?style=flat-square)](https://coveralls.io/r/Storj/storjshare-daemon)
[![NPM](https://img.shields.io/npm/v/storjshare-daemon.svg?style=flat-square)](https://www.npmjs.com/package/storjshare-daemon)
[![License](https://img.shields.io/badge/license-AGPL3.0-blue.svg?style=flat-square)](https://raw.githubusercontent.com/Storj/storjshare-daemon/master/LICENSE)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg?style=flat-square)](https://hub.docker.com/r/computeronix/storjshare-daemon)

Daemon + CLI for farming data on the Storj network, suitable for standalone
use or inclusion in other packages.

## Installation of Pre-built Binaries

storjshare daemon is available as a snap package in the store, installable on [supported](https://snapcraft.io/docs/core/install) systems. These packages are securely confined and automatically updated.

### Latest stable release

```
snap install storjshare
```

### Latest build from github master

```
snap install storjshare --edge
```

## Manual Installation

Make sure you have the following prerequisites installed:

* Git
* Node.js LTS (6.9.x)
* NPM
* Python 2.7
* GCC/G++/Make

### Node.js + NPM

#### GNU+Linux & Mac OSX

```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

Close your shell and open an new one. Now that you can call the `nvm` program, 
install Node.js (which comes with NPM):

```
nvm install --lts
```

#### Windows

Download [Node.js LTS](https://nodejs.org/en/download/) for Windows, launch the
installer and follow the setup instructions. Restart your PC, then test it from
the command prompt:

```
node --version
npm --version
```

### Build Dependencies

#### GNU+Linux

Debian based (like Ubuntu)
```
apt install git python build-essential
```

Red Hat / Centos
```
yum groupinstall 'Development Tools'
```
You might also find yourself lacking a C++11 compiler - [see this](http://hiltmon.com/blog/2015/08/09/c-plus-plus-11-on-centos-6-dot-6/)

#### Mac OSX

```
xcode-select --install
```

#### Windows

```
npm install --global windows-build-tools
```

---

Once build dependencies have been installed for your platform, install the 
package globally using Node Package Manager:

```
npm install --global storjshare-daemon
```

## Usage (CLI)

Once installed, you will have access to the `storjshare` program, so start by 
asking it for some help.

```
storjshare --help

  Usage: storjshare [options] [command]


  Commands:

    start       start a farming node
    stop        stop a farming node
    restart     restart a farming node
    status      check status of node(s)
    logs        tail the logs for a node
    create      create a new configuration
    destroy     kills the farming node
    killall     kills all shares and stops the daemon
    daemon      starts the daemon
    help [cmd]  display help for [cmd]

  Options:

    -h, --help     output usage information
    -V, --version  output the version number
```

You can also get more detailed help for a specific command.

```
storjshare help create

  Usage: storjshare-create [options]

  generates a new share configuration

  Options:

    -h, --help                 output usage information
    --sjcx <addr>              specify the sjcx address (required)
    --key <privkey>            specify the private key
    --storage <path>           specify the storage path
    --size <maxsize>           specify share size (ex: 10GB, 1TB)
    --rpcport <port>           specify the rpc port number
    --rpcaddress <addr>        specify the rpc address
    --maxtunnels <tunnels>     specify the max tunnels
    --tunnelportmin <port>     specify min gateway port
    --tunnelportmax <port>     specify max gateway port
    --manualforwarding         do not use nat traversal strategies
    --logfile <path>           specify the logfile path
    --noedit                   do not open generated config in editor
    -o, --outfile <writepath>  write config to path
```

## Usage (Programmatic)

The Storj Share daemon uses a local [dnode](https://github.com/substack/dnode) 
server to handle RPC message from the CLI and other applications. Assuming the 
daemon is running, your program can communicate with it using this interface. 
The example that follows is using Node.js, but dnode is implemented in many
[other languages](https://github.com/substack/dnode#dnode-in-other-languages).

```js
const dnode = require('dnode');
const daemon = dnode.connect(45015);

daemon.on('remote', (rpc) => {
  // rpc.start(configPath, callback);
  // rpc.stop(nodeId, callback);
  // rpc.restart(nodeId, callback);
  // rpc.status(callback);
  // rpc.destroy(nodeId, callback);
  // rpc.save(snapshotPath, callback);
  // rpc.load(snapshotPath, callback);
  // rpc.killall(callback);
});
```

You can also easily start the daemon from your program by creating a dnode 
server and passing it an instance of the `RPC` class exposed from this package.

```js
const storjshare = require('storjshare-daemon');
const dnode = require('dnode');
const api = new storjshare.RPC();

dnode(api.methods).listen(45015, '127.0.0.1');
```

## Configuring the Daemon

The Storj Share daemon loads configuration from anywhere the 
[rc](https://www.npmjs.com/package/rc) package can read it. The first time you 
run the daemon, it will create a directory in `$HOME/.config/storjshare`, so 
the simplest way to change the daemon's behavior is to create a file at 
`$HOME/.config/storjshare/config` containing the following:

```json
{
  "daemonRpcPort": 45015,
  "daemonRpcAddress": "127.0.0.1",
  "daemonLogFilePath": "",
  "daemonLogVerbosity": 3
}
```

Modify these parameters to your liking, see `example/daemon.config.json` for 
detailed explanation of these properties.

## Debugging the Daemon

The daemon logs activity to the configured log file, which by default is 
`$HOME/.config/storjshare/logs/daemon.log`. However if you find yourself 
needing to frequently restart the daemon and check the logs during 
development, you can run the daemon as a foreground process for a tighter 
feedback loop.

```
storjshare killall
storjshare daemon --foreground
```

## Migrating from [`storjshare-gui`](https://github.com/storj/storjshare-gui) or [`storjshare-cli`](https://github.com/storj/storjshare-cli)
#### storjshare-gui
If you are using the `storjshare-gui` package you can go on with the latest
GUI release. You don't need to migrate but if you like you can do it. If you
choose to migrate from the old storjshare-gui to the CLI version of
storjshare-daemon, please follow the instructions below.

#### storjshare-cli
Storj Share provides a simple method for creating new shares, but if you were 
previously using the `storjshare-cli` package superceded by this one, you'll 
want to migrate your configuration to the new format. To do this, first you'll 
need to dump your private key **before** installing this package.

> If you accidentally overwrote your old `storjshare-cli` installation with 
> this package, don't worry - just reinstall the old package to dump the key, 
> then reinstall this package.

### Step 0: Dump Your Private Key

#### storjshare-gui
Open `%AppData%\Storj Share\settings.json` in any texteditor.
For each GUI drive you will find the private key and the dataDir. Use these
information and go on with Step 1 and 2.
```
{
  "tabs": [
    {
      "key": "4154e85e87b323611cba45ab1cd51203f2508b1da8455cdff8b641cce827f3d6",
      "address": "1K1rPg...",
      "storage": {
        "dataDir": "D:\\Storj\\storjshare-5f4722"
      }
    },
    {
      "key": "0b0341a9913bb84b51485152a1b0a8a6ed68fa4f9a4fedb26c61ff778ce61ec8",
      "address": "1K1rPg...",
      "storage": {
        "dataDir": "D:\\Storj\\storjshare-48a1c4"
      }
  ],
  "appSettings": {...}
}
```

#### storjshare-cli
You can print your cleartext private key from storjshare-cli, using the 
`dump-key` command:

```
storjshare dump-key
 [...]  > Unlock your private key to start storj  >  ********

 [info]   Cleartext Private Key:
 [info]   ======================
 [info]   4154e85e87b323611cba45ab1cd51203f2508b1da8455cdff8b641cce827f3d6
 [info]   
 [info]   (This key is suitable for importing into Storj Share GUI)
```

If you are using a custom data directory, be sure to add the `--datadir <path>`
option to be sure you get the correct key. Also be sure to note your defined 
payout address and data directory.

### Step 1: Install Storj Share and Create Config

Now that you have your private key, you can generate a new configuration file. 
To do this, first install the `storjshare-daemon` package globally and use the 
`create` command. You'll need to remove the `storjshare-cli` package first, so 
make sure you perform the previous step for all shared drives before 
proceeding forward.

```
npm remove -g storjshare-cli
npm install -g storjshare-daemon
```

Now that you have Storj Share installed, use the `create` command to generate 
your configuration.

```
storjshare create --key 4154e8... --sjcx 1K1rPg... --storage <datadir> -o <writepath>
```

This will generate your configuration file given the parameters you passed in, 
write the file to the path following the `-o` option, and open it in your text 
editor. Here, you can make other changes to the configuration following the 
detailed comments in the generated file.

### Step 2: Use The New Configuration

Now that you have successfully migrated your configuration file, you can use 
it to start the share.

```
storjshare start --config path/to/config.json

  * daemon is not running, starting...

  * starting share with config at path/to/config.json
```

## License

Storj Share - Daemon + CLI for farming data on the Storj network.  
Copyright (C) 2017 Storj Labs, Inc

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see http://www.gnu.org/licenses/.

