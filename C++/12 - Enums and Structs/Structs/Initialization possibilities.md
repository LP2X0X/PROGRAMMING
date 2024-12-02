### If an aggregate is defined with an initialization list:
	- If an explicit initialization value exists, that explicit value is used.
	- If an initializer is missing and a default member initializer exists, the default is used.
	- If an initializer is missing and no default member initializer exists, value initialization occurs.

### If an aggregate is defined with no initialization list:
	- If a default member initializer exists, the default is used.
	- If no default member initializer exists, the member remains uninitialized.

```ad-tip
Always provide default values for your members. This ensures that your members will be initialized even if the variable definition doesn’t include an initializer list.
```