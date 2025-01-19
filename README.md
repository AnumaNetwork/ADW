# ADW

ADW is a dedicated desktop process manager for [Anuma node](https://github.com/anumanetwork/anumad).


ADW offers a miniature console using which user can re-build the Anuma stack, upgrading Anuma to the latest version directly from GitHub. The build process is automated via a series of scripts that, if
needed, fetch required tools (git, go, gcc) and build Anuma on the host computer (the build includes various Anuma utilities including `txgen`, `wallet`, `anumactl` and others and can be executed against any specific Git branch).  ADW console can also be used to migrate Kasparov database if building a version with an updated database schema.

ADW process configuration (available via a simple JSON editor) allows user to specify command-line arguments for executables, as such it is possible to configure ADW to run multiple instances of Anuma or potentially run multiple networks simultaneously (provided Anuma nodes do not pro-actively auto-discover each-other)

Like many desktop applications, ADW can run in the tray bar, out of the way.

ADW is built using [NWJS](https://nwjs.io) and is compatible Windows, Linux and Mac OS X.


## Building ADW

### Pre-requisites

- [Node.js 14.0.0+](https://nodejs.org/)
- Emanator - `npm install emanator@latest`

**NOTE:** ADW build process builds and includes latest Anuma binaries from Git master branches. 
To build from specific branches, you can use `--branch...` flags (see below).

#### Generating ADW installers
```
npm install emanator@latest
git clone git@github.com:anumanetwork/adw
cd adw
# run emanate with one or multiple flags below
#  --portable   create a portable zipped application
#  --innosetup  generate Windows setup executable
#  --dmg        generate a DMG image for Mac OS X
#  --all        generate all OS compatible packages
# following flags can be used to reset the environment
#  --clean		clean build folders: purges cloned `GOPATH` folder
#  --reset		`--clean` + deletes downloaded/cached NWJS and NODE binaries
emanate [--portable | --innosetup | --dmg | --all]
```


DMG - Building DMG images on Mac OS requires `sudo` access in order to use system tools such as `diskutil` to generate images: 
```
sudo emanate --dmg
```

To build the windows portable deployment, run the following command:
```
emanate --portable
```

To build the Windows installer, you need to install [Innosetup](https://jrsoftware.org/isdl.php) and run:
```
emanate --innosetup
```


Emanator stores build files in the `~/emanator` folder

#### Running ADW from development environment


In addition to Node.js, please download and install [Latest NWJS SDK https://nwjs.io](https://nwjs.io/) - make sure that `nw` executable is available in the system PATH and that you can run `nw` from command line.

On Linux / Darwin, as good way to install node and nwjs is as follows:

```
cd ~
mkdir bin
cd bin

#node - (must be 14.0+)
wget https://nodejs.org/dist/v14.4.0/node-v14.4.0-linux-x64.tar.xz
tar xvf node-v14.4.0-linux-x64.tar.xz
ln -s node-v14.4.0-linux-x64 node

#nwjs
wget https://dl.nwjs.io/v0.46.2/nwjs-sdk-v0.46.2-linux-x64.tar.gz
tar xvf nwjs-sdk-v0.46.2-linux-x64.tar.gz
ln -s nwjs-sdk-v0.46.2-linux-x64 nwjs

```
Once done add the following to `~/.bashrc`
```
export PATH = /home/<user>/bin/node/bin:/home/<user>/bin/nwjs:$PATH
```
The above method allows you to deploy latest binaries and manage versions by re-targeting symlinks pointing to target folders.

Once you have node and nwjs working, you can continue with ADW.

ADW installation:
```
npm install emanator@latest
git clone git@github.com:anumanetwork/adw
cd adw
npm install
emanate --local-binaries
nw .
```

#### Building installers from specific Anuma Git branches

`--branch` argument specifies common branch name for anuma and kasparov, for example:
```
emanate --branch=v0.4.0-dev 
```
The branch for each repository can be overriden using `--branch-<repo-name>=<branch-name>` arguments as follows:
```
emanate --branch=v0.4.0-dev --branch-anumad=v0.3.0-dev
emanate --branch-miningsimulator=v0.1.2-dev
```

**NOTE:** ADW `build` command in ADW console operates in the same manner and accepts `--branch...` arguments.


## ADW Process Manager

### Configuration

ADW runtime configuration is declared using a JSON object.  

Each instance of the process is declared using it's **type** (for example: `anumad`) and a unique **identifier** (`aw0`).  Most process configuration objects support `args` property that allows
passing arguments or configuration options directly to the process executable.  Depending on the process type, the configuration is passed via command line arguments (kasparov*) or configuration file (anumad).

Supported process types:
- `anumad` - Anuma full node
- `anumaminer` - Anuma sha256 miner

**NOTE:** For Anuma, to specify multiple connection endpoints, you must use an array of addresses as follows: ` "args" : { "connect" : [ "peer-addr-port-a", "peer-addr-port-b", ...] }`

#### Default Configuration File
```js
{
	"anumad:aw0": {
		"args": {
			"rpclisten": "0.0.0.0:12412",
			"listen": "0.0.0.0:12413",
			"profile": 7000,
			"rpcuser": "user",
			"rpcpass": "pass"
		}
	},

```

### Data Storage

ADW stores it's configuration file as `~/.adw/config.json`.  Each configured process data is stored in `<datadir>/<process-type>-<process-identifier>` where `datadir` is a user-configurable location.  The default `datadir` location is `~/.adw/data/`.  For example, `anumad` process with identifier `aw0` will be stored in `~/.adw/data/anumad-aw0/` and it's logs in `~/.adw/data/anumad-aw0/logs/anumad.log`

### Anuma Binaries

ADW can run Anuma from 2 locations - an integrated `bin` folder that is included with ADW redistributables and `~/.adw/bin` folder that is created during the Anuma build process. 

## ADW Console

ADW Console provides following functionality:
- Upgrading kasparov using `migrate` command
- `start` and `stop` controls stack runtime
- Anumad RPC command execution
- Use of test wallet app (ADW auto-configures kasparov address)
- Rebuilding Anuma software stack from within the console

### Using Anumad RPC

Anumad RPC can be accessed via ADW Console using the process identifier. For example:
```
$ aw0 help
$ aw0 getinfo
```
Note that RPC methods are case insensitive.

To supply RPC call arguments, you must supply and array of JSON-compliant values (numbers, double-quote-enclosed strings and 'true'/'false').  For example:
```
$ aw0 getblock "000000b22ce2fcea335cbaf5bc5e4911b0d4d43c1421415846509fc77ec643a7"
{
  "hash": "000000b22ce2fcea335cbaf5bc5e4911b0d4d43c1421415846509fc77ec643a7",
  "confirmations": 83,
  "size": 673,
  "blueScore": 46241,
  ...
}
```

