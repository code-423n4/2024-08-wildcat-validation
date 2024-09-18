## [N-01] `HooksInstanceNotFound()` error thrown instead of `HooksTemplateNotFound()`

https://github.com/code-423n4/2024-08-wildcat/blob/main/src/HooksFactory.sol#L503-L505

The deployMarket() function checks `hooksTemplate` variable against `address(0)` i.e checks if hooks template exists or not. If the condition evaluates to `true` instead of throwing `HooksTemplateNotFound()` error , it is throwing `HooksInstanceNotFound()` error.

## [N-02] `getHooksTemplatesCount()` function not used 

https://github.com/code-423n4/2024-08-wildcat/blob/main/src/HooksFactory.sol#L228

In the `getHooksTemplates` function instead of using `getHooksTemplatesCount()` function code has been rewritten for calculating count of hooks template leading to code repitition.  