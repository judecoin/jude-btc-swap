# Swap CLI

The CLI defaults to **mainnet** (from version 0.6.0 onwards).
For testing and to familiarise yourself with the tool, we recommend you to try it on testnet first.
To do that, pass the `--testnet` flag with the actual command:

```shell
swap --testnet <SUBCOMMAND>
```

Take note that applying `--testnet` results in transacting on Bitcoin testnet and on judecoin stagenet, not on judecoin testnet.

The two main commands of the CLI are:

- `buy-jude`: for swapping BTC to JUDE with a particular seller
- `list-sellers`: for discovering available sellers through a rendezvous point

Running `swap --help` gives us roughly the following output:

```
swap 0.8.0
The COMIT guys <hello@comit.network>
CLI for swapping BTC for JUDE

USAGE:
    swap [FLAGS] [OPTIONS] <SUBCOMMAND>

FLAGS:
        --debug      Activate debug logging
    -h, --help       Prints help information
    -j, --json       Outputs all logs in JSON format instead of plain text
        --testnet    Swap on testnet and assume testnet defaults for data-dir and the blockchain related parameters
    -V, --version    Prints version information

OPTIONS:
        --data-base-dir <data>    The base data directory to be used for mainnet / testnet specific data like database, wallets etc

SUBCOMMANDS:
    buy-jude         Start a BTC for JUDE swap
    list-sellers    Discover and list sellers (i.e. ASB providers)

    cancel          Try to cancel an ongoing swap (expert users only)
    help            Prints this message or the help of the given subcommand(s)
    history         Show a list of past, ongoing and completed swaps
    refund          Try to cancel a swap and refund the BTC (expert users only)
    resume          Resume a swap
```

## Swapping BTC for JUDE

Running `swap buy-jude --help` gives us roughly the following output:

```
swap-buy-jude 0.8.0
Start a BTC for JUDE swap

USAGE:
    swap buy-jude [FLAGS] [OPTIONS] --change-address <bitcoin-change-address> --receive-address <judecoin-receive-address> --seller <seller>

FLAGS:
    -h, --help       Prints help information
        --testnet    Swap on testnet and assume testnet defaults for data-dir and the blockchain related parameters
    -V, --version    Prints version information

OPTIONS:
        --change-address <bitcoin-change-address>           The bitcoin address where any form of change or excess funds should be sent to
        --receive-address <judecoin-receive-address>          The judecoin address where you would like to receive judecoin
        --seller <seller>                                   The seller's address. Must include a peer ID part, i.e. `/p2p/`
        
        --electrum-rpc <bitcoin-electrum-rpc-url>           Provide the Bitcoin Electrum RPC URL
        --bitcoin-target-block <bitcoin-target-block>       Estimate Bitcoin fees such that transactions are confirmed within the specified number of blocks
        --judecoin-daemon-address <judecoin-daemon-address>     Specify to connect to a judecoin daemon of your choice: <host>:<port>
        --tor-socks5-port <tor-socks5-port>                 Your local Tor socks5 proxy port [default: 9050]
```

This command has three core options:

- `--change-address`: A Bitcoin address you control. Will be used for refunds of any kind.
- `--receive-address`: A judecoin address you control. This is where you will receive the judecoin after the swap.
- `--seller`: The multiaddress of the seller you want to swap with.

## Discovering sellers

Running `swap list-sellers --help` gives us roughly the following output:

```
swap-list-sellers 0.8.0
Discover and list sellers (i.e. ASB providers)

USAGE:
    swap list-sellers [FLAGS] [OPTIONS]

FLAGS:
    -h, --help       Prints help information
        --testnet    Swap on testnet and assume testnet defaults for data-dir and the blockchain related parameters
    -V, --version    Prints version information

OPTIONS:
        --rendezvous-point <rendezvous-point>       Address of the rendezvous point you want to use to discover ASBs
        --tor-socks5-port <tor-socks5-port>         Your local Tor socks5 proxy port [default: 9050]
```

Running `swap --testnet list-sellers --rendezvous-point /dnsaddr/rendezvous.coblox.tech/p2p/12D3KooWQUt9DkNZxEn2R5ymJzWj15MpG6mTW84kyd8vDaRZi46o` will give you something like:

```
Connected to rendezvous point, discovering nodes in 'jude-btc-swap-testnet' namespace ...
Discovered peer 12D3KooWPZ69DRp4wbGB3wJsxxsg1XW1EVZ2evtVwcARCF3a1nrx at /dns4/ac4hgzmsmekwekjbdl77brufqqbylddugzze4tel6qsnlympgmr46iid.onion/tcp/8765
+----------------+----------------+----------------+--------+----------------------------------------------------------------------------------------------------------------------------------------+
| PRICE          | MIN_QUANTITY   | MAX_QUANTITY   | STATUS | ADDRESS                                                                                                                                |
+====================================================================================================================================================================================================+
| 0.00665754 BTC | 0.00010000 BTC | 0.00100000 BTC | Online | /dns4/ac4hgzmsmekwekjbdl77brufqqbylddugzze4tel6qsnlympgmr46iid.onion/tcp/8765/p2p/12D3KooWPZ69DRp4wbGB3wJsxxsg1XW1EVZ2evtVwcARCF3a1nrx |
+----------------+----------------+----------------+--------+----------------------------------------------------------------------------------------------------------------------------------------+
```

or this if a node is not reachable:

```
Connected to rendezvous point, discovering nodes in 'jude-btc-swap-testnet' namespace ...
Discovered peer 12D3KooWPZ69DRp4wbGB3wJsxxsg1XW1EVZ2evtVwcARCF3a1nrx at /dns4/ac4hgzmsmekwekjbdl77brufqqbylddugzze4tel6qsnlympgmr46iid.onion/tcp/8765
+-------+--------------+--------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------+
| PRICE | MIN_QUANTITY | MAX_QUANTITY | STATUS      | ADDRESS                                                                                                                                |
+============================================================================================================================================================================================+
| ???   | ???          | ???          | Unreachable | /dns4/ac4hgzmsmekwekjbdl77brufqqbylddugzze4tel6qsnlympgmr46iid.onion/tcp/8765/p2p/12D3KooWPZ69DRp4wbGB3wJsxxsg1XW1EVZ2evtVwcARCF3a1nrx |
+-------+--------------+--------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------+
```

## Automating discover and swapping

The `buy-jude` and `list-sellers` command have been designed to be composed.
[This script](./discover_and_take.sh) is example of what can be done.
Deciding on the seller to use is non-trivial to automate which is why it is not implemented as part of the tool.

## Tor

By default, the CLI will look for Tor at the default socks port `9050` and automatically route all traffic with a seller through Tor.
This allows swapping with sellers that are only reachable with an onion address.

Disclaimer:
Communication with public blockchain explorers (Electrum, public JUDE nodes) currently goes through clearnet.
For complete anonymity it is recommended to run your own blockchain nodes.
Use `swap buy-jude --help` to see configuration options.
