# zephyr-cryptonote-nodejs-pool

Forked from [cryptonote-nodejs-pool](https://github.com/dvandal/cryptonote-nodejs-pool)

High performance Node.js (with native C addons) mining pool for CryptoNote based coins. Comes with lightweight example front-end script which uses the pool's AJAX API. Support for Cryptonight (Original, Monero v7, Stellite v7), Cryptonight Light (Original, Aeon v7, IPBC) Cryptonight Fast (Electronero/Crystaleum), and Cryptonight Heavy (Sumokoin) algorithms.

## Zephyr Specifics

Zephyr requires a modified version of `cryptoforknote-util` called `zephyr-cryptonote-util`

[zephyr-cryptoforknote-util](https://github.com/ZephyrProtocol/zephyr-cryptoforknote-util.git)

A PR will be submitted to the original repo to add the required changes to the original repo.
If you want to support Zephyr with your pool, you will need to use this modified version but please be aware of the changes (they may conflict if you aren't using the repo from MoneroOcean!)

package.json

```json
  "dependencies": {
    ...
    "cryptoforknote-util": "git+https://github.com/ZephyrProtocol/zephyr-cryptoforknote-util.git",
    ...
  }
```

config.json

```json
    ...
  "cnVariant": 0,
  "cnBlobType": 13, // 13 for Zephyr - this points to the modified zephyr-cryptoforknote-util
  "includeHeight": true,
  "isRandomX": true,
    ...
```

Tested/running on ubuntu 20.04 / nodejs 14 / redis npm package 3.1.2 / npm package async 1.5.2

#### Table of Contents

- [zephyr-cryptonote-nodejs-pool](#zephyr-cryptonote-nodejs-pool)
  - [Zephyr Specifics](#zephyr-specifics)
      - [Table of Contents](#table-of-contents)
- [Features](#features)
      - [Optimized pool server](#optimized-pool-server)
      - [Live statistics API](#live-statistics-api)
      - [Mined blocks explorer](#mined-blocks-explorer)
      - [Smart payment processing](#smart-payment-processing)
      - [Admin panel](#admin-panel)
      - [Pool stability monitoring](#pool-stability-monitoring)
      - [Extra features](#extra-features)
- [Usage](#usage)
      - [Requirements](#requirements)
        - [Seriously](#seriously)
      - [1) Downloading \& Installing](#1-downloading--installing)
      - [2) Configuration](#2-configuration)
      - [3) Start the pool](#3-start-the-pool)
      - [4) Host the front-end](#4-host-the-front-end)
      - [5) Customize your website](#5-customize-your-website)
      - [SSL](#ssl)
      - [Upgrading](#upgrading)
    - [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
    - [Monitoring Your Pool](#monitoring-your-pool)
- [Community / Support](#community--support)
      - [Pools Using This Software](#pools-using-this-software)
  - [Referral Links](#referral-links)
  - [Donations](#donations)
  - [Credits](#credits)
  - [License](#license)

# Features

#### Optimized pool server

- TCP (stratum-like) protocol for server-push based jobs
  - Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
- Support for Cryptonight (Original, Monero v7, Stellite v7), Cryptonight Light (Original, Aeon v7, IPBC) and Cryptonight Heavy (Sumokoin) algorithms.
- IP banning to prevent low-diff share attacks
- Socket flooding detection
- Share trust algorithm to reduce share validation hashing CPU load
- Clustering for vertical scaling
- Ability to configure multiple ports - each with their own difficulty
- Miner login (wallet address) validation
- Workers identification (specify worker name as the password)
- Variable difficulty / share limiter
- Set fixed difficulty on miner client by passing "address" param with "+[difficulty]" postfix
- Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
- SSL support for both pool and API servers
- RBPPS (PROP) payment system

#### Live statistics API

- Currency network/block difficulty
- Current block height
- Network hashrate
- Pool hashrate
- Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, payout estimate, etc)
- Blocks found (pending, confirmed, and orphaned)
- Historic charts of pool's hashrate, miners count and coin difficulty
- Historic charts of users's hashrate and payments

#### Mined blocks explorer

- Mined blocks table with block status (pending, confirmed, and orphaned)
- Blocks luck (shares/difficulty) statistics
- Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)

#### Smart payment processing

- Splintered transactions to deal with max transaction size
- Minimum payment threshold before balance will be paid out
- Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
- Prevent "transaction is too big" error with "payments.maxTransactionAmount" option
- Option to enable dynamic transfer fee based on number of payees per transaction and option to have miner pay transfer fee instead of pool owner (applied to dynamic fee only)
- Control transactions priority with config.payments.priority (default: 0).
- Set payment ID on miner client when using "[address].[paymentID]" login
- Integrated payment ID addresses support for Exchanges

#### Admin panel

- Aggregated pool statistics
- Coin daemon & wallet RPC services stability monitoring
- Log files data access
- Users list with detailed statistics

#### Pool stability monitoring

- Detailed logging in process console & log files
- Coin daemon & wallet RPC services stability monitoring
- See logs data from admin panel

#### Extra features

- An easily extendable, responsive, light-weight front-end using API to display data
- Onishin's [keepalive function](https://github.com/perl5577/cpuminer-multi/commit/0c8aedb)
- Support for merged mining
- Support for slush mining system (disabled by default)
- E-Mail Notifications on worker connected, disconnected (timeout) or banned (support MailGun, SMTP and Sendmail)
- Telegram channel notifications when a block is unlocked
- Top 10 miners report
- Multilingual user interface

# Usage

#### Requirements

- Coin daemon(s) (find the coin's repo and build latest version from source)
- [Node.js](http://nodejs.org/) v11.0+
  - For Ubuntu:

```
 curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash
 sudo apt-get install -y nodejs
```

- Or use NVM(https://github.com/creationix/nvm) for debian/ubuntu.

- [Redis](http://redis.io/) key-value store v2.6+
  - For Ubuntu:

```
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
```

Dont forget to tune redis-server:

```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1024 > /proc/sys/net/core/somaxconn
```

Add this lines to your /etc/rc.local and make it executable

```
chmod +x /etc/rc.local
```

- libssl required for the node-multi-hashing module

  - For Ubuntu: `sudo apt-get install libssl-dev`

- Boost is required for the cryptoforknote-util module
  - For Ubuntu: `sudo apt-get install libboost-all-dev`
- libsodium
  - For Ubuntu: `sudo apt-get install libsodium-dev`

##### Seriously

Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.

[**Redis warning**](http://redis.io/topics/security): It'sa good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).

**Do not run the pool as root** : create a new user without ssh access to avoid security issues :

```bash
sudo adduser --disabled-password --disabled-login your-user
```

To login with this user :

```
sudo su - your-user
```

#### 1) Downloading & Installing

Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/dvandal/cryptonote-nodejs-pool.git pool
cd pool

npm update
```

#### 2) Configuration

Copy the `config_examples/COIN.json` file of your choice to `config.json` then overview each options and change any to match your preferred setup.

Explanation for each field:

```javascript
/* Pool host displayed in notifications and front-end */
"poolHost": "your.pool.host",

/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "monero", // Must match the parentCoin variable in config.js

/* Used for front-end display */
"symbol": "XMR",

/* Minimum units in a single coin, see COIN constant in DAEMON_CODE/src/cryptonote_config.h */
"coinUnits": 1000000000000,

/* Number of coin decimals places for notifications and front-end */
"coinDecimalPlaces": 4,

/* Coin network time to mine one block, see DIFFICULTY_TARGET constant in DAEMON_CODE/src/cryptonote_config.h */
"coinDifficultyTarget": 120,

"blockchainExplorer": "http://blockexplorer.arqma.com/block/{id}",  //used on blocks page to generate hyperlinks.
"transactionExplorer": "http://blockexplorer.arqma.com/tx/{id}",    //used on the payments page to generate hyperlinks

/* Set daemon type. Supported values: default, forknote (Fix block height + 1), bytecoin (ByteCoin Wallet RPC API) */
"daemonType": "default",

/* Set Cryptonight algorithm settings.
   Supported algorithms: cryptonight (default). cryptonight_light and cryptonight_heavy
   Supported variants for "cryptonight": 0 (Original), 1 (Monero v7), 3 (Stellite / XTL)
   Supported variants for "cryptonight_light": 0 (Original), 1 (Aeon v7), 2 (IPBC)
   Supported blob types: 0 (Cryptonote), 1 (Forknote v1), 2 (Forknote v2), 3 (Cryptonote v2 / Masari) */
"cnAlgorithm": "cryptonight",
"cnVariant": 1,
"cnBlobType": 0,
"includeHeight":false, /*true to include block.height in job to miner*/
"includeAlgo":"cn/wow", /*wownero specific change to include algo in job to miner*/
"isRandomX": true,
/* Logging */
"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},
"childPools":[ {"poolAddress":"your wallet",
                    "intAddressPrefix": null,
                    "coin": "MCN",  	//must match COIN name in the child pools config.json
                    "childDaemon": {
                        "host": "127.0.0.1",
                        "port": 26081
                    },
                    "pattern": "^Vdu",  //regex to identify which childcoin the miner specified in password. eg) Vdu is first 3 chars of a MCN wallet address.
                    "blockchainExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_block",
                    "transactionExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_transaction",
                    "api": "https://multi-miner.smartcoinpool.net/apiMerged1",
                    "enabled": true
                    }
]
/* Modular Pool Server */
"poolServer": {
    "enabled": true,
    "mergedMining":false,
    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "your wallet",

    /* This is the Public address prefix used for miner login validation. */
    "pubAddressPrefix": 343,

    /* This is the Integrated address prefix used for miner login validation. */
    "intAddressPrefix": 340,

    /* This is the Subaddress prefix used for miner login validation. */
    "subAddressPrefix": 439,

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem" // The SSL certificate authority chain

    "ports": [
        {
            "port": 3333, // Port for mining apps to connect to
            "difficulty": 2000, // Initial difficulty miners are set to
            "desc": "Low end hardware" // Description of port
        },
        {
            "port": 4444,
            "difficulty": 15000,
            "desc": "Mid range hardware"
        },
        {
            "port": 5555,
            "difficulty": 25000,
            "desc": "High end hardware"
        },
        {
            "port": 7777,
            "difficulty": 500000,
            "desc": "Cloud-mining / NiceHash"
        },
        {
            "port": 8888,
            "difficulty": 25000,
            "desc": "Hidden port",
            "hidden": true // Hide this port in the front-end
        },
        {
            "port": 9999,
            "difficulty": 20000,
            "desc": "SSL connection",
            "ssl": true // Enable SSL
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 100, // Minimum difficulty
        "maxDiff": 100000000,
        "targetTime": 60, // Try to get 1 share per this many seconds
        "retargetTime": 30, // Check to see if we should retarget every this many seconds
        "variancePercent": 30, // Allow time to vary this % from target without retargeting
        "maxJump": 100 // Limit diff percent increase/decrease in a single retargeting
    },

    /* Set difficulty on miner client side by passing <address> param with +<difficulty> postfix */
    "fixedDiff": {
        "enabled": true,
        "separator": "+", // Character separator between <address> and <difficulty>
    },

    /* Set payment ID on miner client side by passing <address>.<paymentID> */
    "paymentId": {
        "addressSeparator": ".", // Character separator between <address> and <paymentID>
        "validation": true // Refuse login if non alphanumeric characters in <paymentID>
        "validations": ["1,16", "64"], //regex quantity. range 1-16 characters OR exactly 64 character
        "ban": true  // ban the miner for invalid paymentid
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
        "enabled": true,
        "min": 10, // Minimum percent probability for share hashing
        "stepDown": 3, // Increase trust probability % this much with each valid share
        "threshold": 10, // Amount of valid shares required before trusting begins
        "penalty": 30 // Upon breaking trust require this many valid share before trusting
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, // How many seconds to ban worker for
        "invalidPercent": 25, // What percent of invalid shares triggers ban
        "checkThreshold": 30 // Perform check when this many shares have been submitted
    },

    /* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards 'loyal' miners by valuing younger shares higher than older shares. Remember adjusting the weight!
    More about it here: https://mining.bitcoin.cz/help/#!/manual/rewards */
    "slushMining": {
        "enabled": false, // Enables slush mining. Recommended for pools catering to professional miners
        "weight": 300 // Defines how fast the score assigned to a share declines in time. The value should roughly be equivalent to the average round duration in seconds divided by 8. When deviating by too much numbers may get too high for JS.
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 300, // How often to run in seconds
    "maxAddresses": 50, // Split up payments if sending to more than this many addresses
    "mixin": 5, // Number of transactions yours is indistinguishable from
    "priority": 0, // The transaction priority
    "transferFee": 4000000000, // Fee to pay for each transaction
    "dynamicTransferFee": true, // Enable dynamic transfer fee (fee is multiplied by number of miners)
    "minerPayFee" : true, // Miner pays the transfer fee instead of pool owner when using dynamic transfer fee
    "minPayment": 100000000000, // Miner balance required before sending payment
    "maxPayment": null, // Maximum miner balance allowed in miner settings
    "maxTransactionAmount": 0, // Split transactions by this amount (to prevent "too big transaction" error)
    "denomination": 10000000000 // Truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, // How often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 60,
    "poolFee": 0.8, // 0.8% pool fee (1% total fee total including donations)
    "soloFee": 0, // solo fee
    "finderReward": 0.2, // 0.2 finder reward
    "devDonation": 0.2, // 0.2% donation to send to pool dev
    "networkFee": 0.0, // Network/Governance fee (used by some coins like Loki)

    /* Some forknote coins have an issue with block height in RPC request, to fix you can enable this option.
       See: https://github.com/forknote/forknote-pool/issues/48 */
    "fixBlockHeightRPC": false
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, // How many second worth of shares used to estimate hash rate
    "updateInterval": 3, // Gather stats and broadcast every this many seconds
    "bindIp": "0.0.0.0", // Bind API to a specific IP (set to 0.0.0.0 for all)
    "port": 8117, // The API port
    "blocks": 30, // Amount of blocks to send at a time
    "payments": 30, // Amount of payments to send at a time
    "password": "your_password", // Password required for admin stats
    "ssl": false, // Enable SSL API
    "sslPort": 8119, // The SSL port
    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem", // The SSL certificate authority chain
    "trustProxyIP": false // Proxy X-Forwarded-For support
},

/* Coin daemon connection details (default port is 18981) */
"daemon": {
    "host": "127.0.0.1",
    "port": 18981
},

/* Wallet daemon connection details (default port is 18980) */
"wallet": {
    "host": "127.0.0.1",
    "port": 18982,
    "password": "--rpc-password"
},

/* Redis connection info (default port is 6379) */
"redis": {
    "host": "127.0.0.1",
    "port": 6379,
    "auth": null, // If set, client will run redis auth command on connect. Use for remote db
    "db": 0, // Set the REDIS database to use (default to 0)
    "cleanupInterval": 15 // Set the REDIS database cleanup interval (in days)
}

/* Pool Notifications */
"notifications": {
    "emailTemplate": "email_templates/default.txt",
    "emailSubject": {
        "emailAdded": "Your email was registered",
        "workerConnected": "Worker %WORKER_NAME% connected",
        "workerTimeout": "Worker %WORKER_NAME% stopped hashing",
        "workerBanned": "Worker %WORKER_NAME% banned",
        "blockFound": "Block %HEIGHT% found !",
        "blockUnlocked": "Block %HEIGHT% unlocked !",
        "blockOrphaned": "Block %HEIGHT% orphaned !",
        "payment": "We sent you a payment !"
    },
    "emailMessage": {
        "emailAdded": "Your email has been registered to receive pool notifications.",
        "workerConnected": "Your worker %WORKER_NAME% for address %MINER% is now connected from ip %IP%.",
        "workerTimeout": "Your worker %WORKER_NAME% for address %MINER% has stopped submitting hashes on %LAST_HASH%.",
        "workerBanned": "Your worker %WORKER_NAME% for address %MINER% has been banned.",
        "blockFound": "Block found at height %HEIGHT% by miner %MINER% on %TIME%. Waiting maturity.",
        "blockUnlocked": "Block mined at height %HEIGHT% with %REWARD% and %EFFORT% effort on %TIME%.",
        "blockOrphaned": "Block orphaned at height %HEIGHT% :(",
        "payment": "A payment of %AMOUNT% has been sent to %ADDRESS% wallet."
    },
    "telegramMessage": {
        "workerConnected": "Your worker _%WORKER_NAME%_ for address _%MINER%_ is now connected from ip _%IP%_.",
        "workerTimeout": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has stopped submitting hashes on _%LAST_HASH%_.",
        "workerBanned": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has been banned.",
        "blockFound": "*Block found at height* _%HEIGHT%_ *by miner* _%MINER%_*! Waiting maturity.*",
        "blockUnlocked": "*Block mined at height* _%HEIGHT%_ *with* _%REWARD%_ *and* _%EFFORT%_ *effort on* _%TIME%_*.*",
        "blockOrphaned": "*Block orphaned at height* _%HEIGHT%_ *:(*",
        "payment": "A payment of _%AMOUNT%_ has been sent."
    }
},

/* Email Notifications */
"email": {
    "enabled": false,
    "fromAddress": "your@email.com", // Your sender email
    "transport": "sendmail", // The transport mode (sendmail, smtp or mailgun)

    // Configuration for sendmail transport
    // Documentation: http://nodemailer.com/transports/sendmail/
    "sendmail": {
        "path": "/usr/sbin/sendmail" // The path to sendmail command
    },

    // Configuration for SMTP transport
    // Documentation: http://nodemailer.com/smtp/
    "smtp": {
        "host": "smtp.example.com", // SMTP server
        "port": 587, // SMTP port (25, 587 or 465)
        "secure": false, // TLS (if false will upgrade with STARTTLS)
        "auth": {
            "user": "username", // SMTP username
            "pass": "password" // SMTP password
        },
        "tls": {
            "rejectUnauthorized": false // Reject unauthorized TLS/SSL certificate
        }
    },

    // Configuration for MailGun transport
    "mailgun": {
        "key": "your-private-key", // Your MailGun Private API key
        "domain": "mg.yourdomain" // Your MailGun domain
    }
},

/* Telegram channel notifications.
   See Telegram documentation to setup your bot: https://core.telegram.org/bots#3-how-do-i-create-a-bot */
"telegram": {
    "enabled": false,
    "botName": "", // The bot user name.
    "token": "", // The bot unique authorization token
    "channel": "", // The telegram channel id (ex: BlockHashMining)
    "channelStats": {
        "enabled": false, // Enable periodical updater of pool statistics in telegram channel
        "interval": 5 // Periodical update interval (in minutes)
    },
    "botCommands": { // Set the telegram bot commands
        "stats": "/stats", // Pool statistics
         "enable": "/enable", // Enable telegram notifications
        "disable": "/disable" // Disable telegram notifications
    }
},

/* Monitoring RPC services. Statistics will be displayed in Admin panel */
"monitoring": {
    "daemon": {
        "checkInterval": 60, // Interval of sending rpcMethod request
        "rpcMethod": "getblockcount" // RPC method name
    },
    "wallet": {
        "checkInterval": 60,
        "rpcMethod": "getbalance"
    }
},

/* Prices settings for market and price charts */
"prices": {
    "source": "cryptonator", // Exchange (supported values: cryptonator, altex, crex24, cryptopia, stocks.exchange, tradeogre, maplechange)
    "currency": "USD" // Default currency
},

/* Collect pool statistics to display in frontend charts  */
"charts": {
    "pool": {
        "hashrate": {
            "enabled": true, // Enable data collection and chart displaying in frontend
            "updateInterval": 60, // How often to get current value
            "stepInterval": 1800, // Chart step interval calculated as average of all updated values
            "maximumPeriod": 86400 // Chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
        },
        "miners": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "workers": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "difficulty": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "price": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "profit": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        }

    },
    "user": { // Chart data displayed in user stats block
        "hashrate": {
            "enabled": true,
            "updateInterval": 180,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "worker_hashrate": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 60,
            "maximumPeriod": 86400
        },
        "payments": { // Payment chart uses all user payments data stored in DB
            "enabled": true
        }
    },
    "blocks": {
        "enabled": true,
        "days": 30 // Number of days displayed in chart (if value is 1, display last 24 hours)
    }
}
```

#### 3) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:

- `pool` - Which opens ports for miners to connect and processes shares
- `api` - Used by the website to display network, pool and miners' data
- `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
- `payments` - Sends out payments to miners according to their balances stored in redis
- `chartsDataCollector` - Processes miners and workers hashrate stats and charts
- `telegramBot` - Processes telegram bot commands

By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.

To keep your pool up, on operating system with systemd, you can create add your pool software as a service.  
Use this [example](https://github.com/dvandal/cryptonote-nodejs-pool/blob/master/deployment/cryptonote-nodejs-pool.service) to create the systemd service `/lib/systemd/system/cryptonote-nodejs-pool.service`
Then enable and start the service with the following commands :

```
sudo systemctl enable cryptonote-nodejs-pool.service
sudo systemctl start cryptonote-nodejs-pool.service
```

#### 4) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.

Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to (override daemon setting if set) */
var poolHost = "poolhost.com";

/* Number of coin decimals places (override daemon setting if set) */
"coinDecimalPlaces": 4,

/* Contact email address. */
var email = "support@poolhost.com";

/* Pool Telegram URL. */
var telegram = "https://t.me/YourPool";

/* Pool Discord URL */
var discord = "https://discordapp.com/invite/YourPool";

/*Pool Facebook URL */
var facebook = "https://www.facebook.com/<YourPoolFacebook";

/* Market stat display params from https://www.cryptonator.com/widget */
var marketCurrencies = ["{symbol}-BTC", "{symbol}-USD", "{symbol}-EUR", "{symbol}-CAD"];

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/light.css";

/* Default language */
var defaultLang = 'en';

```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:

- `custom.css` for creating your own pool style
- `custom.js` for changing the functionality of your pool website

Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.

#### SSL

You can configure the API to be accessible via SSL using various methods. Find an example for nginx below:

- Using SSL api in `config.json`:

By using this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "https://poolhost:8119";`

- Inside your SSL Listener, add the following:

```javascript
location ~ ^/api/(.*) {
    proxy_pass http://127.0.0.1:8117/$1$is_args$args;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js` to include the /api. For example:  
`var api = "http://poolhost/api";`

You no longer need to include the port in the variable because of the proxy connection.

- Using his own subdomain, for example `api.poolhost.com`:

```bash
server {
    server_name api.poolhost.com
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /your/ssl/certificate;
    ssl_certificate_key /your/ssl/certificate_key;

    location / {
        more_set_headers 'Access-Control-Allow-Origin: *';
        proxy_pass http://127.0.01:8117;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "//api.poolhost.com";`

You no longer need to include the port in the variable because of the proxy connection.

#### Upgrading

When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.

- Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
- Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
- Run `npm update` to force updating/reinstalling of the dependencies.
- Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here:

- Daemon https://wiki.bytecoin.org/wiki/JSON_RPC_API
- Wallet https://wiki.bytecoin.org/wiki/Wallet_JSON_RPC_API

Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```

### Monitoring Your Pool

- To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
- To monitor server load for CPU, Network, IO, etc - I suggest using [Netdata](https://github.com/firehol/netdata)
- To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever) or [PM2](https://github.com/Unitech/pm2)

# Community / Support

- [GitHub Issues](https://github.com/dvandal/cryptonote-nodejs-pool/issues)
- [Telegram Group](http://t.me/CryptonotePool)

#### Pools Using This Software

- https://pool.leviar.io/
- https://pool.croat.community/

## Referral Links

- NiceHash Miner - Test your mining pool: [https://www.nicehash.com/?refby=938d7799-8f8e-4935-975e-897a1567b1ed](https://www.nicehash.com/?refby=938d7799-8f8e-4935-975e-897a1567b1ed)
- Binance Exchange - Buy and Sell cryptos: [https://www.binance.com/en/register?ref=92696209](https://www.binance.com/en/register?ref=92696209)
- Coinbase Wallet - Buy 100$ USD and get 10$ USD free: [https://www.coinbase.com/join/vandal_y](https://www.coinbase.com/join/vandal_y)
- Shakepay Wallet - Buy 100$ CAD and get 30$ CAD free: [https://shakepay.me/r/VDAIT0G](https://shakepay.me/r/VDAIT0G)

## Donations

Thanks for supporting my works on this project! If you want to make a donation to [Dvandal](https://github.com/dvandal/), the developper of this project, you can send any amount of your choice to one of theses addresses:

- Bitcoin (BTC): `392gS9zuYQBghmMpK3NipBTaQcooR9UoGy`
- Bitcoin Cash (BCH): `qp46fz7ht8xdhwepqzhk7ct3aa0ucypfgv5qvv57td`
- Monero (XMR): `49WyMy9Q351C59dT913ieEgqWjaN12dWM5aYqJxSTZCZZj1La5twZtC3DyfUsmVD3tj2Zud7m6kqTVDauRz53FqA9zphHaj`
- Dash (DASH): `XgFnxEu1ru7RTiM4uH1GWt2yseU1BVBqWL`
- Ethereum (ETH): `0x8c42D411545c9E1963ff56A91d06dEB8C4A9f444`
- Ethereum Classic (ETC): `0x4208D6775A2bbABe64C15d76e99FE5676F2768Fb`
- Litecoin (LTC): `LS9To9u2C95VPHKauRMEN5BLatC8C1k4F1`
- USD Coin (USDC): `0xb5c6BEc389252F24dd3899262AC0D2754B0fC1a3`
- Augur (REP): `0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064`
- Basic Attention Token (BAT): `0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064`
- Chainlink (LINK): `0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064`
- Dai (DAI): `0xF2a50BcCEE8BEb7807dA40609620e454465B40A1`
- Orchid (OXT): `0xf52488AAA1ab1b1EB659d6632415727108600BCb`
- Tezos (XTZ): `tz1T1idcT5hfyjfLHWeqbYvmrcYn5JgwrJKW`
- Zcash (ZCH): `t1YTGVoVbeCuTn3Pg9MPGrSqweFLPGTQ7on`
- 0x (ZRX): `0x4e52AAfC6dAb2b7812A0a7C24a6DF6FAab65Fc9a`

## Credits

- [fancoder](//github.com/fancoder) - Developper on cryptonote-universal-pool project from which current project is forked.
- [dvandal](//github.com/dvandal) - Developer of cryptonote-nodejs-pool software

## License

Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
zephyr-cryptonote-nodejs-池
从cryptonote-nodejs-pool分叉

基于 CryptoNote 的硬币的高性能 Node.js（带有本机 C 插件）挖掘池。附带使用池的 AJAX API 的轻量级示例前端脚本。支持 Cryptonight（原始、Monero v7、Stellite v7）、Cryptonight Light（原始、Aeon v7、IPBC）Cryptonight Fast（Electronero/Crystaleum）和 Cryptonight Heavy（Sumokoin）算法。

和风细节
cryptoforknote-utilZephyr 需要一个名为的修改版本zephyr-cryptonote-util

zephyr-cryptoforknote-util

PR 将提交到原始存储库，以将所需的更改添加到原始存储库。如果您想在您的池中支持 Zephyr，您将需要使用此修改版本，但请注意这些更改（如果您不使用 MoneroOcean 的存储库，它们可能会发生冲突！）

包.json

  "dependencies": {
    ...
    "cryptoforknote-util": "git+https://github.com/ZephyrProtocol/zephyr-cryptoforknote-util.git",
    ...
  }
配置.json

    ...
  "cnVariant": 0,
  "cnBlobType": 13, // 13 for Zephyr - this points to the modified zephyr-cryptoforknote-util
  "includeHeight": true,
  "isRandomX": true,
    ...
在 ubuntu 20.04 / nodejs 14 / redis npm package 3.1.2 / npm package async 1.5.2 上测试/运行

目录
zephyr-cryptonote-nodejs-池
和风细节
目录
功能 -优化矿池服务器 -实时统计 API -开采区块浏览器 -智能支付处理 -管理面板 -矿池稳定性监控 -额外功能
使用 -要求 -认真 - 1）下载和安装 - 2）配置 - 3）启动池 - 4）托管前端 - 5）自定义您的网站 - SSL -升级
来自 CLI 的 JSON-RPC 命令
监控您的泳池
社区/支持 -使用此软件的矿池
推荐链接
捐款
制作人员
执照
特征
优化池服务器
用于基于服务器推送的作业的 TCP（类层）协议
与旧的 HTTP 协议相比，它具有更高的哈希率、更低的网络/CPU 服务器负载、更低的孤块百分比以及更少出错的可能性
支持 Cryptonight（原始、Monero v7、Stellite v7）、Cryptonight Light（原始、Aeon v7、IPBC）和 Cryptonight Heavy（Sumokoin）算法。
IP封禁，防止低差异共享攻击
套接字泛洪检测
共享信任算法以减少共享验证哈希CPU负载
垂直扩展的集群
能够配置多个端口 - 每个端口都有自己的难度
矿工登录（钱包地址）验证
工人身份识别（指定工人姓名作为密码）
可变难度/份额限制器
通过使用“+[difficulty]”后缀传递“address”参数，在矿工客户端上设置固定难度
用于水平扩展的模块化组件（池服务器、数据库、统计/API、支付处理、前端）
对池和 API 服务器的 SSL 支持
RBPPS (PROP) 支付系统
实时统计API
货币网络/区块难度
当前区块高度
网络算力
矿池算力
每个矿工的个人统计数据（算力、提交的股份、待处理余额、总支付额、预计支付额等）
找到的区块（待定、确认和孤立）
矿池算力、矿工数量和硬币难度​​的历史图表
用户算力和支付的历史图表
开采区块浏览器
包含区块状态（待处理、已确认和孤立）的已开采区块表
方块运气（份额/难度）统计
基于chainradar.com的通用区块和交易浏览器
智能支付处理
分割交易以处理最大交易大小
支付余额之前的最低付款门槛
截断支付金额精度以减少大宗交易的规模/复杂性的最小面额
使用“ payment.maxTransactionAmount ”选项防止“交易太大”错误
根据每笔交易的收款人数量启用动态转账费的选项，以及让矿工代替矿池所有者支付转账费的选项（仅适用于动态费用）
使用 config. payment.priority 控制交易优先级（默认值：0）。
使用“[地址].[付款ID]”登录时在矿机客户端设置付款ID
集成支付 ID 解决了对交易所的支持
管理面板
聚合池统计数据
硬币守护进程和钱包RPC服务稳定性监控
日志文件数据访问
详细统计的用户列表
水池稳定性监测
流程控制台和日志文件中的详细记录
硬币守护进程和钱包RPC服务稳定性监控
从管理面板查看日志数据
额外功能
一个易于扩展、响应灵敏、轻量级的前端，使用 API 来显示数据
Onishin的keepalive功能
支持合并挖矿
支持slush挖矿系统（默认禁用）
关于工作人员连接、断开连接（超时）或禁止的电子邮件通知（支持 MailGun、SMTP 和 Sendmail）
区块解锁时的 Telegram 频道通知
十大矿工报告
多语言用户界面
用法
要求
硬币守护进程（找到硬币的存储库并从源代码构建最新版本）
Node.js v11.0+
对于Ubuntu：
 curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash
 sudo apt-get install -y nodejs
或者对 debian/ubuntu 使用 NVM（https://github.com/creationix/nvm）。

Redis键值存储 v2.6+

对于Ubuntu：
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
不要忘记调整 redis-server：

echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1024 > /proc/sys/net/core/somaxconn
将此行添加到 /etc/rc.local 并使其可执行

chmod +x /etc/rc.local
节点多哈希模块所需的 libssl

对于Ubuntu：sudo apt-get install libssl-dev
cryptoforknote-util 模块需要 Boost

对于Ubuntu：sudo apt-get install libboost-all-dev
钠

对于Ubuntu：sudo apt-get install libsodium-dev
严重地
这些都是合法的要求。如果您使用系统包管理器附带的旧版本 Node.js 或 Redis，那么您将会遇到问题。按照链接的说明获取最新的稳定版本。

Redis 警告：了解并理解您正在使用的软件是个好主意 - 使用 Redis 的一个好地方是数据持久性。

不要以 root 身份运行池：创建一个没有 ssh 访问权限的新用户以避免安全问题：

sudo adduser --disabled-password --disabled-login your-user
要使用此用户登录：

sudo su - your-user
1) 下载并安装
克隆存储库并运行npm update以安装所有依赖项：

git clone https://github.com/dvandal/cryptonote-nodejs-pool.git pool
cd pool
安装
npm install
2）配置
复制config_examples/COIN.json您选择的文件，config.json然后概述每个选项并更改任何选项以匹配您的首选设置。

每个字段的解释：

/* Pool host displayed in notifications and front-end */
"poolHost": "your.pool.host",

/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "monero", // Must match the parentCoin variable in config.js

/* Used for front-end display */
"symbol": "XMR",

/* Minimum units in a single coin, see COIN constant in DAEMON_CODE/src/cryptonote_config.h */
"coinUnits": 1000000000000,

/* Number of coin decimals places for notifications and front-end */
"coinDecimalPlaces": 4,

/* Coin network time to mine one block, see DIFFICULTY_TARGET constant in DAEMON_CODE/src/cryptonote_config.h */
"coinDifficultyTarget": 120,

"blockchainExplorer": "http://blockexplorer.arqma.com/block/{id}",  //used on blocks page to generate hyperlinks.
"transactionExplorer": "http://blockexplorer.arqma.com/tx/{id}",    //used on the payments page to generate hyperlinks

/* Set daemon type. Supported values: default, forknote (Fix block height + 1), bytecoin (ByteCoin Wallet RPC API) */
"daemonType": "default",

/* Set Cryptonight algorithm settings.
   Supported algorithms: cryptonight (default). cryptonight_light and cryptonight_heavy
   Supported variants for "cryptonight": 0 (Original), 1 (Monero v7), 3 (Stellite / XTL)
   Supported variants for "cryptonight_light": 0 (Original), 1 (Aeon v7), 2 (IPBC)
   Supported blob types: 0 (Cryptonote), 1 (Forknote v1), 2 (Forknote v2), 3 (Cryptonote v2 / Masari) */
"cnAlgorithm": "cryptonight",
"cnVariant": 1,
"cnBlobType": 0,
"includeHeight":false, /*true to include block.height in job to miner*/
"includeAlgo":"cn/wow", /*wownero specific change to include algo in job to miner*/
"isRandomX": true,
/* Logging */
"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},
"childPools":[ {"poolAddress":"your wallet",
                    "intAddressPrefix": null,
                    "coin": "MCN",  	//must match COIN name in the child pools config.json
                    "childDaemon": {
                        "host": "127.0.0.1",
                        "port": 26081
                    },
                    "pattern": "^Vdu",  //regex to identify which childcoin the miner specified in password. eg) Vdu is first 3 chars of a MCN wallet address.
                    "blockchainExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_block",
                    "transactionExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_transaction",
                    "api": "https://multi-miner.smartcoinpool.net/apiMerged1",
                    "enabled": true
                    }
]
/* Modular Pool Server */
"poolServer": {
    "enabled": true,
    "mergedMining":false,
    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "your wallet",

    /* This is the Public address prefix used for miner login validation. */
    "pubAddressPrefix": 343,

    /* This is the Integrated address prefix used for miner login validation. */
    "intAddressPrefix": 340,

    /* This is the Subaddress prefix used for miner login validation. */
    "subAddressPrefix": 439,

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem" // The SSL certificate authority chain

    "ports": [
        {
            "port": 3333, // Port for mining apps to connect to
            "difficulty": 2000, // Initial difficulty miners are set to
            "desc": "Low end hardware" // Description of port
        },
        {
            "port": 4444,
            "difficulty": 15000,
            "desc": "Mid range hardware"
        },
        {
            "port": 5555,
            "difficulty": 25000,
            "desc": "High end hardware"
        },
        {
            "port": 7777,
            "difficulty": 500000,
            "desc": "Cloud-mining / NiceHash"
        },
        {
            "port": 8888,
            "difficulty": 25000,
            "desc": "Hidden port",
            "hidden": true // Hide this port in the front-end
        },
        {
            "port": 9999,
            "difficulty": 20000,
            "desc": "SSL connection",
            "ssl": true // Enable SSL
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 100, // Minimum difficulty
        "maxDiff": 100000000,
        "targetTime": 60, // Try to get 1 share per this many seconds
        "retargetTime": 30, // Check to see if we should retarget every this many seconds
        "variancePercent": 30, // Allow time to vary this % from target without retargeting
        "maxJump": 100 // Limit diff percent increase/decrease in a single retargeting
    },

    /* Set difficulty on miner client side by passing <address> param with +<difficulty> postfix */
    "fixedDiff": {
        "enabled": true,
        "separator": "+", // Character separator between <address> and <difficulty>
    },

    /* Set payment ID on miner client side by passing <address>.<paymentID> */
    "paymentId": {
        "addressSeparator": ".", // Character separator between <address> and <paymentID>
        "validation": true // Refuse login if non alphanumeric characters in <paymentID>
        "validations": ["1,16", "64"], //regex quantity. range 1-16 characters OR exactly 64 character
        "ban": true  // ban the miner for invalid paymentid
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
        "enabled": true,
        "min": 10, // Minimum percent probability for share hashing
        "stepDown": 3, // Increase trust probability % this much with each valid share
        "threshold": 10, // Amount of valid shares required before trusting begins
        "penalty": 30 // Upon breaking trust require this many valid share before trusting
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, // How many seconds to ban worker for
        "invalidPercent": 25, // What percent of invalid shares triggers ban
        "checkThreshold": 30 // Perform check when this many shares have been submitted
    },

    /* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards 'loyal' miners by valuing younger shares higher than older shares. Remember adjusting the weight!
    More about it here: https://mining.bitcoin.cz/help/#!/manual/rewards */
    "slushMining": {
        "enabled": false, // Enables slush mining. Recommended for pools catering to professional miners
        "weight": 300 // Defines how fast the score assigned to a share declines in time. The value should roughly be equivalent to the average round duration in seconds divided by 8. When deviating by too much numbers may get too high for JS.
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 300, // How often to run in seconds
    "maxAddresses": 50, // Split up payments if sending to more than this many addresses
    "mixin": 5, // Number of transactions yours is indistinguishable from
    "priority": 0, // The transaction priority
    "transferFee": 4000000000, // Fee to pay for each transaction
    "dynamicTransferFee": true, // Enable dynamic transfer fee (fee is multiplied by number of miners)
    "minerPayFee" : true, // Miner pays the transfer fee instead of pool owner when using dynamic transfer fee
    "minPayment": 100000000000, // Miner balance required before sending payment
    "maxPayment": null, // Maximum miner balance allowed in miner settings
    "maxTransactionAmount": 0, // Split transactions by this amount (to prevent "too big transaction" error)
    "denomination": 10000000000 // Truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, // How often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 60,
    "poolFee": 0.8, // 0.8% pool fee (1% total fee total including donations)
    "soloFee": 0, // solo fee
    "finderReward": 0.2, // 0.2 finder reward
    "devDonation": 0.2, // 0.2% donation to send to pool dev
    "networkFee": 0.0, // Network/Governance fee (used by some coins like Loki)

    /* Some forknote coins have an issue with block height in RPC request, to fix you can enable this option.
       See: https://github.com/forknote/forknote-pool/issues/48 */
    "fixBlockHeightRPC": false
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, // How many second worth of shares used to estimate hash rate
    "updateInterval": 3, // Gather stats and broadcast every this many seconds
    "bindIp": "0.0.0.0", // Bind API to a specific IP (set to 0.0.0.0 for all)
    "port": 8117, // The API port
    "blocks": 30, // Amount of blocks to send at a time
    "payments": 30, // Amount of payments to send at a time
    "password": "your_password", // Password required for admin stats
    "ssl": false, // Enable SSL API
    "sslPort": 8119, // The SSL port
    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem", // The SSL certificate authority chain
    "trustProxyIP": false // Proxy X-Forwarded-For support
},

/* Coin daemon connection details (default port is 18981) */
"daemon": {
    "host": "127.0.0.1",
    "port": 18981
},

/* Wallet daemon connection details (default port is 18980) */
"wallet": {
    "host": "127.0.0.1",
    "port": 18982,
    "password": "--rpc-password"
},

/* Redis connection info (default port is 6379) */
"redis": {
    "host": "127.0.0.1",
    "port": 6379,
    "auth": null, // If set, client will run redis auth command on connect. Use for remote db
    "db": 0, // Set the REDIS database to use (default to 0)
    "cleanupInterval": 15 // Set the REDIS database cleanup interval (in days)
}

/* Pool Notifications */
"notifications": {
    "emailTemplate": "email_templates/default.txt",
    "emailSubject": {
        "emailAdded": "Your email was registered",
        "workerConnected": "Worker %WORKER_NAME% connected",
        "workerTimeout": "Worker %WORKER_NAME% stopped hashing",
        "workerBanned": "Worker %WORKER_NAME% banned",
        "blockFound": "Block %HEIGHT% found !",
        "blockUnlocked": "Block %HEIGHT% unlocked !",
        "blockOrphaned": "Block %HEIGHT% orphaned !",
        "payment": "We sent you a payment !"
    },
    "emailMessage": {
        "emailAdded": "Your email has been registered to receive pool notifications.",
        "workerConnected": "Your worker %WORKER_NAME% for address %MINER% is now connected from ip %IP%.",
        "workerTimeout": "Your worker %WORKER_NAME% for address %MINER% has stopped submitting hashes on %LAST_HASH%.",
        "workerBanned": "Your worker %WORKER_NAME% for address %MINER% has been banned.",
        "blockFound": "Block found at height %HEIGHT% by miner %MINER% on %TIME%. Waiting maturity.",
        "blockUnlocked": "Block mined at height %HEIGHT% with %REWARD% and %EFFORT% effort on %TIME%.",
        "blockOrphaned": "Block orphaned at height %HEIGHT% :(",
        "payment": "A payment of %AMOUNT% has been sent to %ADDRESS% wallet."
    },
    "telegramMessage": {
        "workerConnected": "Your worker _%WORKER_NAME%_ for address _%MINER%_ is now connected from ip _%IP%_.",
        "workerTimeout": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has stopped submitting hashes on _%LAST_HASH%_.",
        "workerBanned": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has been banned.",
        "blockFound": "*Block found at height* _%HEIGHT%_ *by miner* _%MINER%_*! Waiting maturity.*",
        "blockUnlocked": "*Block mined at height* _%HEIGHT%_ *with* _%REWARD%_ *and* _%EFFORT%_ *effort on* _%TIME%_*.*",
        "blockOrphaned": "*Block orphaned at height* _%HEIGHT%_ *:(*",
        "payment": "A payment of _%AMOUNT%_ has been sent."
    }
},

/* Email Notifications */
"email": {
    "enabled": false,
    "fromAddress": "your@email.com", // Your sender email
    "transport": "sendmail", // The transport mode (sendmail, smtp or mailgun)

    // Configuration for sendmail transport
    // Documentation: http://nodemailer.com/transports/sendmail/
    "sendmail": {
        "path": "/usr/sbin/sendmail" // The path to sendmail command
    },

    // Configuration for SMTP transport
    // Documentation: http://nodemailer.com/smtp/
    "smtp": {
        "host": "smtp.example.com", // SMTP server
        "port": 587, // SMTP port (25, 587 or 465)
        "secure": false, // TLS (if false will upgrade with STARTTLS)
        "auth": {
            "user": "username", // SMTP username
            "pass": "password" // SMTP password
        },
        "tls": {
            "rejectUnauthorized": false // Reject unauthorized TLS/SSL certificate
        }
    },

    // Configuration for MailGun transport
    "mailgun": {
        "key": "your-private-key", // Your MailGun Private API key
        "domain": "mg.yourdomain" // Your MailGun domain
    }
},

/* Telegram channel notifications.
   See Telegram documentation to setup your bot: https://core.telegram.org/bots#3-how-do-i-create-a-bot */
"telegram": {
    "enabled": false,
    "botName": "", // The bot user name.
    "token": "", // The bot unique authorization token
    "channel": "", // The telegram channel id (ex: BlockHashMining)
    "channelStats": {
        "enabled": false, // Enable periodical updater of pool statistics in telegram channel
        "interval": 5 // Periodical update interval (in minutes)
    },
    "botCommands": { // Set the telegram bot commands
        "stats": "/stats", // Pool statistics
         "enable": "/enable", // Enable telegram notifications
        "disable": "/disable" // Disable telegram notifications
    }
},

/* Monitoring RPC services. Statistics will be displayed in Admin panel */
"monitoring": {
    "daemon": {
        "checkInterval": 60, // Interval of sending rpcMethod request
        "rpcMethod": "getblockcount" // RPC method name
    },
    "wallet": {
        "checkInterval": 60,
        "rpcMethod": "getbalance"
    }
},

/* Prices settings for market and price charts */
"prices": {
    "source": "cryptonator", // Exchange (supported values: cryptonator, altex, crex24, cryptopia, stocks.exchange, tradeogre, maplechange)
    "currency": "USD" // Default currency
},

/* Collect pool statistics to display in frontend charts  */
"charts": {
    "pool": {
        "hashrate": {
            "enabled": true, // Enable data collection and chart displaying in frontend
            "updateInterval": 60, // How often to get current value
            "stepInterval": 1800, // Chart step interval calculated as average of all updated values
            "maximumPeriod": 86400 // Chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
        },
        "miners": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "workers": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "difficulty": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "price": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "profit": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        }

    },
    "user": { // Chart data displayed in user stats block
        "hashrate": {
            "enabled": true,
            "updateInterval": 180,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "worker_hashrate": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 60,
            "maximumPeriod": 86400
        },
        "payments": { // Payment chart uses all user payments data stored in DB
            "enabled": true
        }
    },
    "blocks": {
        "enabled": true,
        "days": 30 // Number of days displayed in chart (if value is 1, display last 24 hours)
    }
}
3）启动池
node init.js
默认情况下使用该文件config.json，但可以使用命令参数指定文件-config=file，例如：

node init.js -config=config_backup.json
该软件包含四个不同的模块：

pool- 为矿工打开连接和处理共享的端口
api- 网站用来显示网络、矿池和矿工数据
unlocker- 处理区块候选并在区块解锁时增加矿工的余额
payments- 根据存储在 redis 中的余额向矿工发送付款
chartsDataCollector- 处理矿工和工人的哈希率统计数据和图表
telegramBot- 处理电报机器人命令
默认情况下，运行该init.js脚本将启动所有四个模块。您可以选择使用命令参数让脚本启动仅启动特定模块-module=name，例如：

node init.js -module=api
使用 tmux 在单模块模式下运行池的示例屏幕截图。

为了保持池正常运行，在带有 systemd 的操作系统上，您可以创建将池软件添加为服务。
使用此示例创建 systemd 服务/lib/systemd/system/cryptonote-nodejs-pool.service ，然后使用以下命令启用并启动该服务：

sudo systemctl enable cryptonote-nodejs-pool.service
sudo systemctl start cryptonote-nodejs-pool.service
4）托管前端
只需将目录内容托管website_example在能够提供简单静态文件的文件服务器上即可。

编辑文件中的变量website_example/config.js以使用池的特定配置。变量解释：

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to (override daemon setting if set) */
var poolHost = "poolhost.com";

/* Number of coin decimals places (override daemon setting if set) */
"coinDecimalPlaces": 4,

/* Contact email address. */
var email = "support@poolhost.com";

/* Pool Telegram URL. */
var telegram = "https://t.me/YourPool";

/* Pool Discord URL */
var discord = "https://discordapp.com/invite/YourPool";

/*Pool Facebook URL */
var facebook = "https://www.facebook.com/<YourPoolFacebook";

/* Market stat display params from https://www.cryptonator.com/widget */
var marketCurrencies = ["{symbol}-BTC", "{symbol}-USD", "{symbol}-EUR", "{symbol}-CAD"];

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/light.css";

/* Default language */
var defaultLang = 'en';
5) 定制您的网站
包含以下文件，以便您可以自定义池网站，而无需对index.html其他前端文件进行重大更改，从而降低将更新与您自己的更改合并的难度：

custom.css创建您自己的泳池风格
custom.js用于更改您的矿池网站的功能
然后只需通过 nginx、Apache、Google Drive 或任何可以托管静态内容的方式提供文件即可。

SSL协议
您可以使用各种方法将 API 配置为可通过 SSL 访问。下面是 nginx 的示例：

在以下位置使用 SSL API config.json：
通过使用它，您将需要api更新website_example/config.js. 例如：
var api = "https://poolhost:8119";

在 SSL 侦听器中，添加以下内容：
location ~ ^/api/(.*) {
    proxy_pass http://127.0.0.1:8117/$1$is_args$args;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
通过添加此内容，您将需要更新中的api变量以website_example/config.js包含 /api。例如：
var api = "http://poolhost/api";

由于代理连接，您不再需要在变量中包含端口。

使用他自己的子域，例如api.poolhost.com：
server {
    server_name api.poolhost.com
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /your/ssl/certificate;
    ssl_certificate_key /your/ssl/certificate_key;

    location / {
        more_set_headers 'Access-Control-Allow-Origin: *';
        proxy_pass http://127.0.01:8117;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
api通过添加此内容，您将需要更新website_example/config.js. 例如：
var api = "//api.poolhost.com";

由于代理连接，您不再需要在变量中包含端口。

升级中
更新到最新代码时，不仅要更新git pull此存储库中的最新代码，还要更新 Node.js 模块以及任何可能已更改的配置文件。

在池目录（init.js 脚本所在的位置）中执行此操作git pull以获取最新代码。
通过删除node_modules带有rm -r node_modules.
运行npm update以强制更新/重新安装依赖项。
将您的config.json示例与此存储库中的最新示例或解释每个配置字段的设置说明中的示例进行比较。您可能需要修改或添加任何新的更改。
来自 CLI 的 JSON-RPC 命令
JSON-RPC 命令的文档可以在此处找到：

守护进程https://wiki.bytecoin.org/wiki/JSON_RPC_API
钱包https://wiki.bytecoin.org/wiki/Wallet_JSON_RPC_API
Curl 可用于从命令行使用 JSON-RPC 命令。以下是调用块 100 的示例getblockheaderbyheight：

curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
监控您的泳池
要检查并更改 redis，我建议使用redis-commander
要监控 CPU、网络、IO 等服务器负载 - 我建议使用Netdata
为了让你的池节点脚本在后台运行，记录到文件，并在崩溃时自动重新启动 - 我建议使用forever或PM2
社区/支持
GitHub 问题
电报群
使用此软件的池
https://pool.leviar.io/
https://pool.croat.community/
推荐链接
NiceHash Miner - 测试您的矿池：https://www.nicehash.com/? refby=938d7799-8f8e-4935-975e-897a1567b1ed
币安交易所 - 买卖加密货币：https://www.binance.com/en/register? ref=92696209
Coinbase 钱包 - 购买 100 美元送 10 美元免费：https://www.coinbase.com/join/vandal_y
Shakepay 钱包 - 购买 100 美元即可免费获得 30 美元：https ://shakepay.me/r/VDAIT0G
捐款
感谢您支持我在这个项目上的工作！如果您想向该项目的开发者Dvandal捐款，您可以将您选择的任意金额发送到以下地址之一：

比特币（BTC）：392gS9zuYQBghmMpK3NipBTaQcooR9UoGy
比特币现金（BCH）：qp46fz7ht8xdhwepqzhk7ct3aa0ucypfgv5qvv57td
门罗币（XMR）：49WyMy9Q351C59dT913ieEgqWjaN12dWM5aYqJxSTZCZZj1La5twZtC3DyfUsmVD3tj2Zud7m6kqTVDauRz53FqA9zphHaj
达世币（达世币）：XgFnxEu1ru7RTiM4uH1GWt2yseU1BVBqWL
以太坊（ETH）：0x8c42D411545c9E1963ff56A91d06dEB8C4A9f444
以太经典（ETC）：0x4208D6775A2bbABe64C15d76e99FE5676F2768Fb
莱特币（LTC）：LS9To9u2C95VPHKauRMEN5BLatC8C1k4F1
美元硬币（USDC）：0xb5c6BEc389252F24dd3899262AC0D2754B0fC1a3
预言家（REP）：0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064
基本注意力令牌（BAT）：0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064
链环（LINK）：0x5A66CE95ea2428BC5B2c7EeB7c96FC184258f064
戴（DAI）：0xF2a50BcCEE8BEb7807dA40609620e454465B40A1
兰花（OXT）：0xf52488AAA1ab1b1EB659d6632415727108600BCb
特所思（XTZ）：tz1T1idcT5hfyjfLHWeqbYvmrcYn5JgwrJKW
大零现金（ZCH）：t1YTGVoVbeCuTn3Pg9MPGrSqweFLPGTQ7on
0x（ZRX）：0x4e52AAfC6dAb2b7812A0a7C24a6DF6FAab65Fc9a
制作人员
fancoder - cryptonote-universal-pool 项目的开发人员，当前项目是从该项目分叉的。
dvandal - cryptonote-nodejs-pool 软件的开发者
执照
根据 GNU 通用公共许可证 v2 发布

http://www.gnu.org/licenses/gpl-2.0.html
