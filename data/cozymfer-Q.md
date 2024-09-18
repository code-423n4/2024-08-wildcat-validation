###  Modifier SphereXConfig::spherexOnlyOperatorOrAdmin() is different from logic inside the funciton

**Description**
The modifier SphereXconfig::spherexOnlyOperatorOrAdmin() is named with the logic of an Operator "OR" an Admin role, but the logic inside function uses && which is a both operator AND admin roles.

**Impact**
User needs to be have the Operator and Admin roles when this modifier is called. Without intention of having both roles they could potentially lock functionality or access for the WildcatArchController::updateSphereXEngineOnRegisteredContracts function. 

**Recommended mitigation**

Make change to the logic of the function to match the or in the name to create the intended function, using "||" instead of "&&". Instead can change name to match logic inside function from spherexOnlyOperatorOrAdmin() to spherexOnlyOperatorAndAdmin(). Or just create both functions just depends on intention of use.
