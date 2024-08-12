The conditional operator (?:), also known as the ternary conditional operator, is a shorthand method of  writing a simple if/else statement. The syntax works like this:  
```ad-note
condition ? first_expression : second_expression;
```
First, both types of first_expression and second_expression must have implicit conversions to from one to another, or, new in C# 9.0, each must have an implicit conversion to a target type. Second, the conditional operator can be used only in assignment statements.