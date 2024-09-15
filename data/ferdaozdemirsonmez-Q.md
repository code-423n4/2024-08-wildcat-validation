##Title:
Hardcoded Protocol Fee Cap in setProtocolFeeBips Function

## Summary:
The setProtocolFeeBips function in the WildcatMarketConfig contract contains a hardcoded protocol fee cap of 1000. Hardcoding such values limits flexibility and can create maintenance challenges if the fee cap needs to be adjusted in the future.

## Vulnerability Details:
Within the setProtocolFeeBips function, the protocol fee cap is set as a hardcoded value of 1000. This use of a "magic number" makes the code less flexible and harder to maintain. A constant or dynamically configurable parameter would allow easier adjustments and clearer understanding of the fee limit.

## Impact:
Having this hardcoded value reduces the contract's adaptability. If the protocol evolves and the fee cap needs to be changed, the contract would need to be redeployed or manually updated in multiple places.
Proof of Concept:
solidity
if (_protocolFeeBips > 1_000) revert_ProtocolFeeTooHigh();

## Recommended Mitigation Steps:
Define the fee cap as a constant or make it a configurable parameter, allowing easier updates and reducing the reliance on hardcoded values. Example:
solidity
uint16 constant MAX_PROTOCOL_FEE_BIPS = 1000;

