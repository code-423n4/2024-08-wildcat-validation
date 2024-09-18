# Functions not called internally within a contract should be marked as `external` not `public`

## Summary
Functions that are marked `public` cost more gas than functions marked `external`. To save gas, any functions that are not intended to be called within a contract should be marked as `external` while only functions that are intended to be called both internally and externally should be marked as `public`.

In the `WildcatSanctionsSentinel` contract, the following functions are marked as `public` but never called within the contract
1. `overrideSanction`, see https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L96-L99
2. `removeSanctionOverride`, see https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L104-L107
3. `createEscrow`, see https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L121-L142

The visibility of the above functions should be changed from `public` to `external` to save gas since they are not called internally in the `WildcatSanctionsSentinel` contract.