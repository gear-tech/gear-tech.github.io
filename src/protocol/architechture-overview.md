# Architechture overview

```mermaid
flowchart TB
    gear-cli-->vara-runtime
    vara-runtime-->pallets
    subgraph pallets
        pallet-gear
        pallet-gear-bank
        pallet-gear-gas
        pallet-gear-messenger
        pallet-gear-program
        pallet-gear-payment
        pallet-gear-scheduler
        pallet-gear-staking-rewards
        pallet-gear-voucher
    end
```

🚧 *To be added soon.*
