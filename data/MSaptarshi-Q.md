# [L-1]  Protocol might be vulnerable to chain reorg 
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L319
## Impact
Wildcat uses `create` to deploy new instances of hooks.
A user Imagine that Alice deploys a hooks instance. Bob sees that the network block reorg happens and calls `deployHooksInstance`. Thus, it creates a hook with an address to which bob controls.
Ethereum chain is suspected to undergo reorg previously, Optimistic rollups (Optimism/Arbitrum) are also suspect to reorgs since if someone finds a fraud the blocks will be reverted, even though the user receives a confirmation and already created a quest.
## Recommendation
Use `create2` to deploy them