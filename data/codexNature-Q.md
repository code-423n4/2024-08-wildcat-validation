### [L-1] Some functions with declared returns value in its signature do not have return statemnts anywahere within the functions.

**Description:** Several functions within the protocol have returns value declared in their signature but the return values are completely ignored within said functions.


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



**Recommendation:** Add a return value check to avoid unexpected crash of the contract.



### [L-2] Casting from `uint256` to `uint16` could cause `originalReserveRatioBips


**Description:** In `MarketConstraintHooks::_calculateTemporaryReserveRatioBips` function the unsafe casting of `originalReserveRatioBips` from `uint256` to `uint16`.
Also as seen below in `TemporaryReserveRatio` struct `originalAnnualInterestBips` and `originalReserveRatioBips` are casted as `uint16` as against `uint256` in `_calculateTemporaryReserveRatioBips` function.

`MarketConstraintHooks.sol#L7-L11`

```solidity
struct TemporaryReserveRatio {
@>  uint16 originalAnnualInterestBips;
@>  uint16 originalReserveRatioBips;
    uint32 expiry;
}
```

`MarketConstraintHooks.sol#L148-169`

```javascript
      function _calculateTemporaryReserveRatioBips(
        uint256 annualInterestBips,
        uint256 originalAnnualInterestBips,
        uint256 originalReserveRatioBips
    ) internal pure returns (uint16 temporaryReserveRatioBips) {
        // Calculate the relative reduction in the interest rate in bips,
        // bound to a maximum of 100%
        uint256 relativeDiff =
            MathUtils.mulDiv(10000, originalAnnualInterestBips - annualInterestBips, originalAnnualInterestBips);

        // If the reduction is 25% (2500 bips) or less, return the original reserve ratio
        if (relativeDiff <= 2500) {
@>          temporaryReserveRatioBips = uint16(originalReserveRatioBips);
        } else {
            // Calculate double the relative reduction in the interest rate in bips,
            // bound to a maximum of 100%
            uint256 boundRelativeDiff = MathUtils.min(10000, 2 * relativeDiff);

            // If the bound relative diff is lower than the existing reserve ratio, return the latter.
@>          temporaryReserveRatioBips = uint16(MathUtils.max(boundRelativeDiff, originalReserveRatioBips));
        }
    }

```

**Impact:** Impact is quite low as the Max Reserve Ratio has been set to a max of 10,000, hence the likelihood of an overflow/underflow is doubtful.

**Tools Used:** Manual Review.


**Recommended Mitigation:** It is recommended that casting be uniform.
