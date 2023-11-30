# Calibration network Deal Storage Provider (SP)

## 32GiB SP service
### t017840
Every deal accepted by this SP will be aggeregated into 32GiB sectors, which is the minimum size for calibration network. 
This miner has a preset sealing capacity of 2x32GiB sectors per day, defined as sectors in waitdeals will be flushed every 12h.   

Maximum sealing time will happen when deals come in as the first to start a publish message, and first to start a new sector:  
1. Publish storage deal messages are aggregated and published every 1h, deals can wait up to 1h
2. Sector waiting for more deals to come in, deals can wait up to 12h 
3. Sealing takes around 6h to complete 

  **Users should expect time to get deals on chain to be up to 19h, so deals should be issued with a minimum 24h to deal start epoch (Current epoch + 2880).**

  **Looking Glass** - http://38.70.220.87:8123/ - This service shows the SP's Boost logs, so you can check on deal proposal status without having to wait 12h+. Filter by client ID.
  
### On chain published address and peer ID
`/ip4/38.70.220.87/tcp/10241`

`12D3KooWQtcfpAhnu89D3YkyA1ZVXmDBmCLwA62LEfHinfLcHzoL`

### Deal acceptance parameters
- min. deal size: 256B (verified deals needs to be minimum 1MiB)
- max. deal size: 512MiB
- unverified price per GiB: 0.0000000005 FIL
- verified price per GiB: 0 FIL
- accept online storage deals: yes
- accept offline storage deals: no
- accept online retrieval deals: yes
- accept offline retrieval deals: no

### Configurations parameters
#### libp2p-info
```
Provider: t017840
Agent: boost-1.7.4+git.52cb7cd
Peer ID: 12D3KooWQtcfpAhnu89D3YkyA1ZVXmDBmCLwA62LEfHinfLcHzoL
Peer Addresses:
  /ip4/38.70.220.87/tcp/10241
Protocols:
  /fil/datatransfer/1.2.0
  /fil/retrieval/qry/0.0.1
  /fil/retrieval/qry/1.0.0
  /fil/retrieval/transports/1.0.0
  /fil/storage/ask/1.0.1
  /fil/storage/ask/1.1.0
  /fil/storage/mk/1.0.1
  /fil/storage/mk/1.1.0
  /fil/storage/mk/1.1.1
  /fil/storage/mk/1.2.0
  /fil/storage/mk/1.2.1
  /fil/storage/status/1.0.1
  /fil/storage/status/1.1.0
  /fil/storage/status/1.2.0
  /floodsub/1.0.0
  /ipfs/graphsync/1.0.0
  /ipfs/graphsync/2.0.0
  /ipfs/id/1.0.0
  /ipfs/id/push/1.0.0
  /ipfs/ping/1.0.0
  /legs/head/indexer/ingest/mainnet/0.0.1
  /libp2p/autonat/1.0.0
  /libp2p/balancer/forwarding/0.0.1
  /meshsub/1.0.0
  /meshsub/1.1.0
```
#### Storage Ask
- Price / epoch / Gib	500,000,000 atto
- Verified Price / epoch / Gib	0 atto
- Min Piece Size	256 B
- Max Piece Size	512 MiB
#### Deal publish
- Deal publish period	1h
#### Boost config.toml configuration
```
[Libp2p]
  AnnounceAddresses = ["/ip4/38.70.220.87/tcp/10241"]
[Dealmaking]
  ConsiderOnlineStorageDeals = true
  ConsiderOfflineStorageDeals = false
  ConsiderOnlineRetrievalDeals = true
  ConsiderOfflineRetrievalDeals = false
  ConsiderVerifiedStorageDeals = true
  ConsiderUnverifiedStorageDeals = true
  ExpectedSealDuration = "6h0m0s"
  MaxDealStartDelay = "336h0m0s"
  StartEpochSealingBuffer = 480
  DealProposalLogDuration = "24h0m0s"
  RetrievalLogDuration = "24h0m0s"
  HttpTransferMaxConcurrentDownloads = 20
  HttpTransferStallCheckPeriod = "30s"
  HttpTransferStallTimeout = "5m0s"
[Wallets]
  Miner = "f017840"
[ContractDeals]
  Enabled = true
  AllowlistContracts = []
[LotusDealmaking]
  ConsiderOnlineStorageDeals = true
  ConsiderOfflineStorageDeals = false
  ConsiderOnlineRetrievalDeals = true
  ConsiderOfflineRetrievalDeals = false
  ConsiderVerifiedStorageDeals = true
  ConsiderUnverifiedStorageDeals = true
[IndexProvider]
  Enable = true
```
#### Lotus-miner config.toml configuration
```
[Dealmaking]
  PublishMsgPeriod = "1h0m0s"
  MaxDealsPerPublishMsg = 8
[Sealing]
  MaxWaitDealsSectors = 2
  MaxSealingSectors = 8
  MaxSealingSectorsForDeals = 8
  PreferNewSectorsForDeals = true
  CommittedCapacitySectorLifetime = "12960h0m0s"
  WaitDealsDelay = "12h0m0s"
  AlwaysKeepUnsealedCopy = true
  FinalizeEarly = true
  MakeNewSectorForDeals = true
  BatchPreCommits = false
  AggregateCommits = false
```


## DEAL EXAMPLES

### Client side (Using boost):
FAKE DEALS (BOOST)
#### 1. Make a random name and file. Here it's a 32GiB file
```
UUID=`uuidgen | awk -F"-" '{print $1}'`
dd if=/dev/urandom of=$UUID.deal bs=1K count=32317564
```
#### 2. Make a car of the file
```
boostx generate-car ./$UUID.deal ./$UUID.car
Payload CID:  bafyreibdo6167w27ok4r4nxapiuqy4kt4xz2v6ashhlpmtkqzayj4b5w4
```
#### 3. Then you need to calculate the commp and piece size for the generated car file:
```
boostx commp ./$UUID.car
  CommP CID:  baga6ea4seaqlulkipc|sbgopuqe7afpe7|gdkooaudctsmdcuz6ovgc6xvrkong
  Piece size:  34359738368
  Car file size: 33317564668 
```
#### 4. Place the generated car file on a public HTTP server, so that a storage provider can later fetch it.
#### 5. Finally, trigger an online storage deal with a given storage provider:
```
boost deal --verified=false \
           --provider=t017840 \
           --commp=baga6ea4seaqlulkipc|sbgopuqe7afpe7|gdkooaudctsmdcuz6ovgc6xvrkong \
           --http-url=https://some_http_server/Your_UUID_file.car \
           --car-size=33317564668 \
           --duration=600001 \
           --storage-price=500000000 \
           --piece-size=34359738368 \
           --payload-cid=bafyreibdo6167w27ok4r4nxapiuqy4kt4xz2v6ashhlpmtkqzayj4b5w4
```
#### 6. Deal accepted in Boost
![image](https://github.com/benjaminh83/fvm-calib-deal-miners/assets/14029124/38ddb01f-2d0f-4220-af27-93822124eabb)
