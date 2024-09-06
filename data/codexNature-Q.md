### Casting from `uint256` to `uint16` could cause `originalReserveRatioBips


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
