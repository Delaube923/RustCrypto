# RustCrypto

### Spot

```shell
➜  ~ cd xcrypto
➜  ~ ls
➜  ~ Cargo.lock  Cargo.toml  binance  logger  private_key.pem  pyalgo  spot.json  src  usdt.json
➜  ~ cd binance/spot
```
build debug

```shell
➜  ~ cargo build
```

or release

```shell
➜  ~ cargo build -r
```

You will find the binary file in target/debug or targer/release

### USDT future

```shell
➜  ~ cd xcrypto
➜  ~ ls
➜  ~ Cargo.lock  Cargo.toml  binance  logger  private_key.pem  pyalgo  spot.json  src  usdt.json
➜  ~ cd binance/usdt
```

build debug

```shell
➜  ~ cargo build
```

or release

```shell
➜  ~ cargo build -r
```

You will find the binary file in target/debug or targer/release


## Run

Before you run the program, two files is required, one is the configuration file, and the other is your private key.

For these two files, you can refer to `usdt.json` and `spot.json` for the configuration file, and `private_key.pem` for the private key.

After compilation is completed, jump to the target/debug or target/release. **Before you run the program, you need to do the following things**.
- **Set position mode to Single-Side Position**
- **Move configuration file, private key and binary file to the same directory**

You can run spot trading by this command

```shell
./spot -c=spot.json -l=info
```

The argument `-c=spot.json` is the abbreviation of `--config=spot.json`, representing the path to the configuration file. The `-l=info` is the abbreviation of `--level=info`, indicating the log level, this parameter is optional, its default value is `info`. The configuration file looks like this.

```json
{
    "apikey":"your api key",
    "pem": "private_key.pem",
    "margin": false,
    "local": "ws://localhost:8111"
}
```

- **apikey** can be generated through the Binance.
- **pem** is the path where your private key is located, which in this example is `private_key.pem` located in the current directory.
- **margin** determine to use spot account or cross-margin account
- **local** is the websocket address bind to, and the strategy will communicate with the trading system by connecting to this address


For usdt future, it is similar to spot trading.

```shell
./usdt -c=usdt.json -l=info
```

The configuration file looks like this.

```json
{
    "apikey": "your api key",
    "pem": "private_key.pem",
    "local": "ws://localhost:8111"
}
```

The meaning of each field is the same as the `spot.json`

## Position

After you run the binary, pos.db will appear in the current directory where you executed it. This file is a SQLite3 database that is used to store the position holdings for different sessions.

If you need to modify the position recorded by the system, you can make changes to pos.db using SQL. After completing the modifications, simply restart the system.

## Market Stream
you can subscribe market stream via `ssession.subscribe(symbol: str,stream: str)`
- bbo: best bid or ask's price or quantity in real-time for a specified symbol

```python
sub = ssession.subscribe("btcusdt","bbo")
```

- depth: `depth` and `depth:100ms` are supported

```python
sub = ssession.subscribe("btcusdt","depth")
# sub = ssession.subscribe("btcusdt","depth:100ms")
```

- kline: `1s`, `1m`, `3m` `5m`, `15m`, `30m`, `1h`, `8h`, `12h`, `1d`, `3d`, `1w`, `1M` is avaliable

```python
sub = ssession.subscribe("btcusdt","kline:1m")
```

## Python strategy package


```shell
➜  ~ cd xcrypto
➜  ~ ls
➜  ~ Cargo.lock  Cargo.toml  binance  logger  private_key.pem  pyalgo  spot.json  src  usdt.json
➜  ~ cd pyalgo
```

install debug

```shell
➜  ~ maturin develop
```

install release

```shell
➜  ~ maturin develop -r
```

You can also package as wheels

```shell
➜  ~ maturin build 
```

or 

```shell
➜  ~ maturin build -r
```
The `.whl` file is located in the target/wheels


## Implement strategy

In the `pyalgo/example` directory, there are some demos. To implement any strategy, the following classes is all you need.

- **Engine**, used to create sessions. DepthSubscription and BarSubscription are subscribed through the created session. Here is an example. 

```python
from pyalgo import *

if __name__ == "__main__":
    # The only argument is interval
    # Because the underlying WebSocket is non-blocking 
    # In order to avoid wasted CPU cycles, an appropriate sleep interval is required. 
    # You can also set it to 0, and it will become a busy loop.
    eng = Engine(0.0001)
    # create session
    session = eng.make_session(
        addr="ws://localhost:8111", session_id=1, name="test", trading=True
    )   
    # subscribe depth, you can also subscribe kline by session.subscribe("dogeusdt","kline:1m)
    sub = session.subscribe("dogeusdt", "depth")
    # set callback
    # when you recv depth, it will print data
    sub.on_data = lambda x:print(x)
    eng.run()
```
