Verum-NodeJS-Mining-Pool
======================

High performance Node.js (with native C addons) mining pool for Verum. Comes with lightweight example front-end script which uses the pool's AJAX API.


#### Table of Contents
* [Usage](#usage)
  * [Requirements](#requirements)
  * [1. Downloading & Installing](#1-downloading--installing)
  * [2. Create wallet](#2-create-wallet)
  * [3. Configuration](#3-configuration)
  * [4. Starting the Pool](#4-start-the-pool)
  * [5. Host the front-end](#5-host-the-front-end)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Features](#features)
* [Credits](#credits)
* [License](#license)

Usage
===

#### Requirements
* Latest [Verum Daemon](https://github.com/VerumProject/VerumCore)
* [Node.js](http://nodejs.org/) v14.21.3 (Recommend to install using [NVM](https://github.com/creationix/nvm))
* [Redis](http://redis.io/) key-value store v2.6+
  * Ubuntu: 
```
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
 ```
* Ubuntu Packages:
```sudo apt-get install libssl-dev libboost-all-dev libsodium-dev```

##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.

#### 1) Downloading & Installing

```bash
git clone https://github.com/VerumProject/Verum-NodeJS-Mining-Pool
cd Verum-NodeJS-Mining-Pool
npm i
```
_*Please note that if you installed NVM that you also have to install node trough NVM with `nvm install 14.21.3`_

#### 2) Create wallet
1. Go to the location where you extracted or compiled your Verum Daemon executables.
2. To create the wallet, use this command: `./Verum-Service --generate-container --container-file pool.wallet --container-password YourOwnPassword` (You can change YourOwnPassword or pool.wallet with your own information)
3. When you executed you will see a wallet address output in the terminal. Copy it and already enter it in the `config.json` file in [Step 3](#3-configuration)
4. I suggest at this point to make a backup of the `pool.wallet` file to another location on your server and also onto your pc. I HIGHLY recommend this in case of issues in the future.
5. In order to start the Payment Processor (Wallet), use this command `./Verum-Service --generate-container --container-file pool.wallet --container-password YourOwnPassword --rpc-legacy-security`

**Note:** Do NOT open the `pool.wallet` file with `./Verum-Wallet`. This will break the wallet file from opening with `./Verum-Serivce`. If you did however do that, please restore your back in step 4.

_*We're working hard to migrate the pool from using `./Verum-Service` to `./Verum-Wallet-API`._


#### 3) Configuration

Edit the `config.json` file and change the following keys:
- **poolHost**: This is your pool hostname, this will be shown on the website where the miners connect to.
- **poolAddress**: This is the wallet address of the payment processor in order to make payments
- **sslCert**: The certificate file for your SSL connection
- **sslKey**: The certificate key file for your SSL connection
- **sslCA**: The certificate authority file for your SSL connection
- **password**: This is your password for the frontend admin panel where you can see stats
- **daemon -> host**: If you have your daemon running somewhere else, you can change it to a different ip (Make sure to run the daemon with `--rpc-bind-ip 0.0.0.0` in order to make a successful connection)
- **wallet -> host**: If you have your payment processor (Verum-Service) running somewhere else, you can change it to a different ip (Make sure to run the payment processor with `--bind-ip 0.0.0.0` in order to make a successful connection)

#### 4) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis
* `chartsDataCollector` - Processes miners and workers hashrate stats and charts
* `telegramBot`	- Processes telegram bot commands


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

#### 5) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.

Edit the `website_example/config.json` file and change the following variables:
- **api**: This is the url of your domain or ip address of the API (Make sure to check what port is used for http or https)

### Monitoring Your Pool

* To inspect and make changes to redis I suggest using redis-commander (`npm i -g redis-commander`)
* To monitor server load for CPU, Network, IO, etc - I suggest using [Netdata](https://github.com/firehol/netdata)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever), [PM2](https://github.com/Unitech/pm2) or [tmux](https://github.com/tmux/tmux)


Features
===

#### Optimized pool server
* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* Support for Cryptonight (Original, Monero v7, Stellite v7), Cryptonight Light (Original, Aeon v7, IPBC) and Cryptonight Heavy (Sumokoin) algorithms.
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Ability to configure multiple ports - each with their own difficulty
* Miner login (wallet address) validation
* Workers identification (specify worker name as the password)
* Variable difficulty / share limiter
* Set fixed difficulty on miner client by passing "address" param with "+[difficulty]" postfix
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* SSL support for both pool and API servers
* RBPPS (PROP) payment system

#### Live statistics API
* Currency network/block difficulty
* Current block height
* Network hashrate
* Pool hashrate
* Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, payout estimate, etc)
* Blocks found (pending, confirmed, and orphaned)
* Historic charts of pool's hashrate, miners count and coin difficulty
* Historic charts of users's hashrate and payments

#### Mined blocks explorer
* Mined blocks table with block status (pending, confirmed, and orphaned)
* Blocks luck (shares/difficulty) statistics
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)

#### Smart payment processing
* Splintered transactions to deal with max transaction size
* Minimum payment threshold before balance will be paid out
* Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option
* Option to enable dynamic transfer fee based on number of payees per transaction and option to have miner pay transfer fee instead of pool owner (applied to dynamic fee only)
* Control transactions priority with config.payments.priority (default: 0).
* Set payment ID on miner client when using "[address].[paymentID]" login
* Integrated payment ID addresses support for Exchanges

#### Admin panel
* Aggregated pool statistics
* Coin daemon & wallet RPC services stability monitoring
* Log files data access
* Users list with detailed statistics

#### Pool stability monitoring
* Detailed logging in process console & log files
* Coin daemon & wallet RPC services stability monitoring
* See logs data from admin panel

#### Extra features
* An easily extendable, responsive, light-weight front-end using API to display data
* Onishin's [keepalive function](https://github.com/perl5577/cpuminer-multi/commit/0c8aedb)
* Support for merged mining
* Support for slush mining system (disabled by default)
* E-Mail Notifications on worker connected, disconnected (timeout) or banned (support MailGun, SMTP and Sendmail)
* Telegram channel notifications when a block is unlocked
* Top 10 miners report
* Multilingual user interface


Credits
---------

* [fancoder](//github.com/fancoder) - Developper on cryptonote-universal-pool project from which current project is forked.
* [dvandal](//github.com/dvandal) - Developer of cryptonote-nodejs-pool software
* [StarlightJS](//github.com/Verum256) - Developer of Verum-NodeJS-Mining-Pool software

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
