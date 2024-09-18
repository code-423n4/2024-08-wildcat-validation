## Replace abi.encodeWithSignature and abi.encodeWithSelector with abi.encodeCall
Using abi.encodeCall provides compile-time safety checks for function signatures and parameter types. It prevents potential errors due to typos or incorrect parameter types.
```
// Example usage
abi.encodeWithSignature("transfer(address,uint256)", to, amount);
abi.encodeWithSelector(IERC20.transfer.selector, to, amount);
```
Recommended practice
```
abi.encodeCall(IERC20.transfer, (to, amount));
```

## Use require Statements for Input Validation
Some functions in WildcatMarketConfig.sol do not perform adequate input validation, potentially allowing unexpected values to be passed, leading to unintended behavior.
```
function setMaxTotalSupply(uint256 _maxTotalSupply) external {
    maxTotalSupply = _maxTotalSupply;
}
```
Recommended Practice
```
function setMaxTotalSupply(uint256 _maxTotalSupply) external {
    require(_maxTotalSupply > 0, "Invalid max total supply");
    maxTotalSupply = _maxTotalSupply;
}
```

