### [L-1] Some functions with declared returns value in its signature do not have return statemnts anywahere within the functions.

**Description** Several functions within the protocol have returns value declared in their signature but the return values are completely ignored within said functions.


[`HookFactory::_deployMarket #L393-489`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L393-L489)

[`HookFactory::deployMarket #L491-516`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L491-L516)

[`HookFactory::deployMarketAndHooks #L518-545`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L518-L545)

[`AccessControlHooks::_onCreateMarket #L159-194`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/AccessControlHooks.sol#L159-L194)

[`AccessControlHooks::_readAddress #L504-508`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/AccessControlHooks.sol#L504-L508)

[`FixedTermLoanHooks::_onCreateMarket #L170-223`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/FixedTermLoanHooks.sol#L170-L223)

[`FixedTermLoanHooks::getPreviousLenderStatus #L347-351`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/FixedTermLoanHooks.sol#L347-L351)

[`FixedTermLoanHooks::_readAddress #L541-545`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/FixedTermLoanHooks.sol#L541-L545)

[`WildcatMarketBase::_isSanctioned #L254-273`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L254-L273)

[`WildcatMarketBase::_calculateCurrentStatePointers #L358-360`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L358-L360)

[`WildcatMarketBase::_getUpdatedState #L406-465`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L406-L465)

[`WildcatMarketBase::_calculateCurrentState #L476-534`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L476-L534)

[`WildcatMarketBase::_applyWithdrawalBatchPayment #L665-695`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L665-L695)

[`WildcatMarketBase::_runctimeConstant #L736-743`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L736-L743)

[`WildcatMarketBase::_runctimeConstant #L745-752`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L745-L752)

[`WildcatMarketBase::_isFlaggedByChainalysis #L754-767`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L754-L767)

[`WildcatMarketBase::_createEscrowForUnderlyingAsset #L769-793`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketBase.sol#L769-L793)

[`WildcatMarketWithdrawals::getAccountWithdrawalStatus #L40-49`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L40-L49)

[`WildcatMarketWithdrawals::_queueWithdrawal #L80-130`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L80-L130)

[`WildcatMarketWithdrawal::_processUnpaidWithdrawalBatch #L319-346`](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L319-L346)


**Impact:** Ignoring return statement after it has been declatred in the function signature may lead to compilation erros, function incompleteness and deployemnt errors.

**Tools Used:** Manual Review.


**Recommendations:** Add a return value check to avoid unexpected crash of the contract.


### [G-1] `public` function not used within the contract could be marked external.


**Description** Functions marked `external` are more gas efficient than `public` functions.

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