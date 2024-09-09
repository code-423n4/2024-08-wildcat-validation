## Impact
Function removeControllerFactory is not removing factory address from "_addAllowedSenderOnChain" after removing from "_controllerFactories".

## Proof of Concept
```
  function removeControllerFactory(address factory) external onlyOwner {
    if (!_controllerFactories.remove(factory)) {
      revert ControllerFactoryDoesNotExist();
    }
    emit ControllerFactoryRemoved(factory);
  }
```

## Tools Used
Manual Review
## Recommended Mitigation
It should remove factory address  from "AllowedSenderOnChain"