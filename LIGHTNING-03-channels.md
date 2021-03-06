# Lightning Payment Channels

### Setup
We continue to use the `identity_pubkey`s of Exchange A and Exchange B which were set up [here](LIGHTNING-01-peers.md)

```shell
$ export X_A_ID_PUBKEY=0279a2f2b499a5345f17d35eff76d5ab45d252560f63347b96e42f8b80045ca435
$ export X_B_ID_PUBKEY=026832da661d53ee23e88909b70ed1768825deb22f26b0d5519ad0c78a1e528676
```

### Exchange A opens 0.16 BTC payment channel to Exchange B
Check open payment channels for Exchange A
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
    "channels": []
}
```

Exchange A tries to open `0.3 BTC` channel to Exchange B, getting limit error imposed by the [spec](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md#requirements)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons openchannel --node_key=$X_B_ID_PUBKEY --local_amt=30000000 --ticker=BTC
[lncli] rpc error: code = Unknown desc = funding amount is too large, the max channel size is: 0.16777216 BTC
```

Exchange A opens `0.16 BTC` channel to Exchange B with the following [BTC funding transaction](https://testnet.blockchain.info/tx/091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06)
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons openchannel --node_key=$X_B_ID_PUBKEY --local_amt=16000000 --ticker=BTC
{
    "funding_txid": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06"
}
```

Funding transaction must be confirmed before a channel is opened.
The default number of confirmations is 1.

Once the channel is opened, Exchange A lists the `Bitcoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
    "channels": [
	{
	    "active": true,
	    "remote_pubkey": "026832da661d53ee23e88909b70ed1768825deb22f26b0d5519ad0c78a1e528676",
	    "channel_point": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06:0",
	    "chan_id": "1418272143300296704",
	    "capacity": "16000000",
	    "local_balance": "15991312",
	    "remote_balance": "0",
	    "commit_fee": "8688",
	    "commit_weight": "600",
	    "fee_per_kw": "12000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	}
    ]
}
```

Exchange B lists the `Bitcoin` payment channel as follows
```
$ lncli --rpcserver=localhost:10002 --no-macaroons listchannels
{
    "channels": [
	{
	    "active": true,
	    "remote_pubkey": "0279a2f2b499a5345f17d35eff76d5ab45d252560f63347b96e42f8b80045ca435",
	    "channel_point": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06:0",
	    "chan_id": "1418272143300296704",
	    "capacity": "16000000",
	    "local_balance": "0",
	    "remote_balance": "15991312",
	    "commit_fee": "8688",
	    "commit_weight": "552",
	    "fee_per_kw": "12000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	}
    ]
}
```

Exchange A's `Bitcoin` balance decreased due to the committed amount in payment channel
```
$ lncli --rpcserver=localhost:10001 --no-macaroons walletbalance --ticker=BTC
{
    "balance": "48991564"
}
$ lncli --rpcserver=localhost:10001 --no-macaroons walletbalance --ticker=LTC
{
    "balance": "1000000000"
}
```


### Checking Swap Routes
Exchange A has no swap route to Exchange B yet
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
```

Exchange B has no swap route to Exchange A yet
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
[lncli] rpc error: code = Unknown desc = unable to find a path to destination
```



### Exchange B opens 0.1 LTC payment channel to Exchange A
Check open payment channels for Exchange B (only BTC channel to Exchange A exists)
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons listchannels
{
    "channels": [
	{
	    "active": true,
	    "remote_pubkey": "0279a2f2b499a5345f17d35eff76d5ab45d252560f63347b96e42f8b80045ca435",
	    "channel_point": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06:0",
	    "chan_id": "1418272143300296704",
	    "capacity": "16000000",
	    "local_balance": "0",
	    "remote_balance": "15991312",
	    "commit_fee": "8688",
	    "commit_weight": "552",
	    "fee_per_kw": "12000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	}
    ]
}
```

Exchange B opens `0.1 LTC` channel to Exchange A with the following [LTC funding transaction](https://chain.so/tx/LTCTEST/17285434991d24039ed8eac6761873e89fa9c90c4c70b0a94255e6c67673536f)
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons openchannel --node_key=$X_A_ID_PUBKEY --local_amt=10000000 --ticker=LTC
{
    "funding_txid": "17285434991d24039ed8eac6761873e89fa9c90c4c70b0a94255e6c67673536f"
}
```

Exchange B lists the `Litecoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons listchannels
{
    "channels": [
	{
	    "active": true,
	    "remote_pubkey": "0279a2f2b499a5345f17d35eff76d5ab45d252560f63347b96e42f8b80045ca435",
	    "channel_point": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06:0",
	    "chan_id": "1418272143300296704",
	    "capacity": "16000000",
	    "local_balance": "0",
	    "remote_balance": "15991312",
	    "commit_fee": "8688",
	    "commit_weight": "552",
	    "fee_per_kw": "12000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	},
	{
	    "active": true,
	    "remote_pubkey": "0279a2f2b499a5345f17d35eff76d5ab45d252560f63347b96e42f8b80045ca435",
	    "channel_point": "17285434991d24039ed8eac6761873e89fa9c90c4c70b0a94255e6c67673536f:0",
	    "chan_id": "527244412821045248",
	    "capacity": "10000000",
	    "local_balance": "9963800",
	    "remote_balance": "0",
	    "commit_fee": "36200",
	    "commit_weight": "600",
	    "fee_per_kw": "50000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	}
    ]
}
```

Exchange A lists the `Litecoin` payment channel as follows
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons listchannels
{
    "channels": [
	{
	    "active": true,
	    "remote_pubkey": "026832da661d53ee23e88909b70ed1768825deb22f26b0d5519ad0c78a1e528676",
	    "channel_point": "091420448e9ae1e2f207ffc77abad6fb4d18bbd253053ba74d82c721f0d43d06:0",
	    "chan_id": "1418272143300296704",
	    "capacity": "16000000",
	    "local_balance": "15991312",
	    "remote_balance": "0",
	    "commit_fee": "8688",
	    "commit_weight": "600",
	    "fee_per_kw": "12000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	},
	{
	    "active": true,
	    "remote_pubkey": "026832da661d53ee23e88909b70ed1768825deb22f26b0d5519ad0c78a1e528676",
	    "channel_point": "17285434991d24039ed8eac6761873e89fa9c90c4c70b0a94255e6c67673536f:0",
	    "chan_id": "527244412821045248",
	    "capacity": "10000000",
	    "local_balance": "0",
	    "remote_balance": "9963800",
	    "commit_fee": "36200",
	    "commit_weight": "552",
	    "fee_per_kw": "50000",
	    "unsettled_balance": "0",
	    "total_satoshis_sent": "0",
	    "total_satoshis_received": "0",
	    "num_updates": "0",
	    "pending_htlcs": []
	}
    ]
}
```

NOTE: Exchange B's `Litecoin` balance is locked until funding transaction is confirmed
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons walletbalance --ticker=BTC
{
    "balance": "32500000"
}
$ lncli --rpcserver=localhost:10002 --no-macaroons walletbalance --ticker=LTC
{
    "balance": "500000000"
}
```

Exchange B's `Litecoin` balance after funding transaction is confirmed
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons walletbalance --ticker=BTC
{
    "balance": "32500000"
}
$ lncli --rpcserver=localhost:10002 --no-macaroons walletbalance --ticker=LTC
{
    "balance": "489964850"
}
```


### Checking Swap Routes
Exchange A has the following swap routes to Exchange B
```shell
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
{
    "routes": [
	{
	    "total_time_lock": 480138,
	    "total_fees": "0",
	    "total_amt": "100000",
	    "hops": [
		{
		    "chan_id": "527244412821045248",
		    "chan_capacity": "10000000",
		    "amt_to_forward": "100000",
		    "fee": "0",
		    "expiry": 479562
		},
		{
		    "chan_id": "1418272143300296704",
		    "chan_capacity": "16000000",
		    "amt_to_forward": "1000",
		    "fee": "0",
		    "expiry": 1289922
		}
	    ]
	}
    ]
}
$ lncli --rpcserver=localhost:10001 --no-macaroons queryswaproutes --dest=$X_B_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
{
    "routes": [
	{
	    "total_time_lock": 1290059,
	    "total_fees": "0",
	    "total_amt": "10",
	    "hops": [
		{
		    "chan_id": "1418272143300296704",
		    "chan_capacity": "16000000",
		    "amt_to_forward": "10",
		    "fee": "0",
		    "expiry": 1289915
		},
		{
		    "chan_id": "527244412821045248",
		    "chan_capacity": "10000000",
		    "amt_to_forward": "1000",
		    "fee": "0",
		    "expiry": 479535
		}
	    ]
	}
    ]
}
```

Exchange B has the following swap routes to Exchange A
```shell
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=BTC --out_ticker=LTC
{
    "routes": [
	{
	    "total_time_lock": 480139,
	    "total_fees": "0",
	    "total_amt": "100000",
	    "hops": [
		{
		    "chan_id": "527244412821045248",
		    "chan_capacity": "10000000",
		    "amt_to_forward": "100000",
		    "fee": "0",
		    "expiry": 479563
		},
		{
		    "chan_id": "1418272143300296704",
		    "chan_capacity": "16000000",
		    "amt_to_forward": "1000",
		    "fee": "0",
		    "expiry": 1289922
		}
	    ]
	}
    ]
}
$ lncli --rpcserver=localhost:10002 --no-macaroons queryswaproutes --dest=$X_A_ID_PUBKEY --in_amt=1000 --in_ticker=LTC --out_ticker=BTC
{
    "routes": [
	{
	    "total_time_lock": 1290059,
	    "total_fees": "0",
	    "total_amt": "10",
	    "hops": [
		{
		    "chan_id": "1418272143300296704",
		    "chan_capacity": "16000000",
		    "amt_to_forward": "10",
		    "fee": "0",
		    "expiry": 1289915
		},
		{
		    "chan_id": "527244412821045248",
		    "chan_capacity": "10000000",
		    "amt_to_forward": "1000",
		    "fee": "0",
		    "expiry": 479536
		}
	    ]
	}
    ]
}
```

Note that until the funding transaction for both channels have 6 confirmations, `queryswaproutes` will return `[lncli] rpc error: code = Unknown desc = unable to find a path to destination` even though the channels may appear active in `listchannels`.
