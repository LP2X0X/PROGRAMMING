- Anonymous methods are interesting, in that they can access the local variables of the method that defines them. Formally speaking, such variables are termed outer variables of the anonymous method.
- The following important points about the interaction between an anonymous method scope and the scope of the defining method should be mentioned:
	1. An anonymous method cannot access ref or out parameters of the defining method.
	2. An anonymous method cannot have a local variable with the same name as a local variable in the outer method.
	3. An anonymous method can access instance variables (or static variables, as appropriate) in the outer class scope.
	4. An anonymous method can declare local variables with the same name as outer class member variables (the local variables have a distinct scope and hide the outer class member variables).