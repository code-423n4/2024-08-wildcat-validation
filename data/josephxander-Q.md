## `HooksFactory` can be forgotten to be registered with `ArchController` via `HooksFactory::registerWithArchController`


The `HooksFactory::registerWithArchController` docs states that the `HooksFactory` contract needs to be executed once at deployment. But this is left as a method without a check to ensure it is called in timely manner.


### PoC
```javascript
//src/HooksFactory
//ln#68-77

 /**
   * @dev Registers the factory as a controller with the arch-controller, allowing
   *      it to register new markets.
   *      Needs to be executed once at deployment.
   *      Does not need checks for whether it has already been registered as the
   *      arch-controller will revert if it is already registered.
   */
  function registerWithArchController() external override {
    IWildcatArchController(_archController).registerController(address(this));
} //@audit As per 3rd line of natspec, call this directly in constructor.

```

### MITIGATION

Call `HooksFactory::registerWithArchController` in `HooksFactory::constructor`