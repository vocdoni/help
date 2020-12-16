# Guide - Deploy Vochain Explorer

### Dependences

Golang 1.14+

### Running

####  Docker

Navigate to `vocexplorer/docker/vocexplorer`

```text
docker-compose build
docker-compose up
```

That's it!

####  Manual

 **1. Frontend**

On the command line navigate into the directory `./frontend` and run

```text
env GOARCH=wasm GOOS=js go build -ldflags "-s -w" -trimpath -o ../static/main.wasm
```

to compile the frontend code into wasm.

 **2. Static files**

Make a copy of the `wasm_exec.js` file from `$GOROOT/misc/wasm/` directory and put it in the `./static` directory. This must be from the same golang version that you used to build `main.wasm`.

Get back to the root path and run `yarn` to install and compile the required style assets.

If you want to renew the styles, run `yarn gulp`, or in case you want to watch for changes, `yarn gulp sass:watch`.

There's also a gulp task for watching `.go` files changes in `./frontend` files and regenerating the `main.wasm` file: `yarn gulp go:watch`.

You can watch both `.go` and `.scss` file changes by just using

```text
yarn gulp watch
```

 **4. Backend**

After steps 1, and 2, navigate back into `vocexplorer` and run

```text
go run main.go
```

Then in your favourite web browser navigate to localhost at the specified port.

Options for `main.go`:

*  `--chain` `(string)` vochain network to connect to \(eg. main, dev \(default "main"\)
*  `--dataDir` `(string)` directory where data is stored \(default "/Users/natewilliams/.vocexplorer"\)
*  `--disableGzip` use to disable gzip compression on web server
*  `--hostURL` `(string)` url to host block explorer \(default "[http://localhost:8081](http://localhost:8081)"\)
*  `--logLevel` `(string)` log level &lt;debug, info, warn, error&gt; \(default "error"\)
*  `--refreshTime` `(int)` Number of seconds between each content refresh \(default 10\)
*  `--vochainCreateGenesis` create own/testing genesis file on vochain
*  `--vochainGenesis` `(string)` use alternative genesis file for the voting chain
*  `--vochainLogLevel` `(string)` voting chain node log level \(default "error"\)
*  `--vochainNodeKey` `(string)` user alternative vochain private key \(hexstring\[64\]\)
*  `--vochainP2PListen` `(string)` p2p host and port to listent for the voting chain \(default "0.0.0.0:26656"\)
*  `--vochainPeers` `(stringArray)` comma separated list of p2p peers
*  `--vochainSeeds` `(stringArray)` comma separated list of p2p seed nodes

 **Note: when updating to a new version of this program, you may need to refresh your data store:**

```text
rm -rf DBPATH
```

Where DBPATH by default is `~/.vocexplorer/CHAIN_ID` and CHAIN\_ID is the ID of the blockchain you're exploring, eg. `vocdoni-release-06`
