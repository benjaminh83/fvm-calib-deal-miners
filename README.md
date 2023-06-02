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
  
### On chain published address and peer ID
`/ip4/194.182.34.236/tcp/10241`

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
  AnnounceAddresses = ["/ip4/194.182.34.236/tcp/10241"]
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
