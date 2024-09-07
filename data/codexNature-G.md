### [G-1] `public` function not used within the contract could be marked external.


**Description:** Functions marked `external` are more gas efficient than `public` functions.

Mark the below function as `external` since there were not used internally.

[`WildcatSanctionsEscrow::escrowedAsset #L30-32`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L30-L32)

[`WildcatSanctionsEscrow::releaseEscrow #L34-45`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L34-L45)

[`WildcatSanctionsSentinel::overrideSanction #L96-99`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L96-L99)

[`WildcatSanctionSentinel::removeSanctionOverride #L104-107`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L104-L107)

[`WildcatSanctionSentinel::createEscrow #L121-142`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsSentinel.sol#L121-L142)

[`WildcatMarketToken::balanceOf #L17-22`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketToken.sol#L17-L22)

[`WildcatMarketWithdrawals::executeWithdrawal #L194-210`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L194C12-L210)

[`WildcatMarketWithdrawals::repayAndProcessUnpaidWithdrawalBatches #L283-317`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L283-L317)

**Tools Used:** Aderyn


**Recoomendations:** Change the `public` statment to `external` to save gas.