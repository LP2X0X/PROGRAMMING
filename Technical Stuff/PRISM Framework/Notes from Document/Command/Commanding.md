- Actions or operations that the user can perform through the UI are typically defined as commands.
- Commands provide a convenient way to represent actions or operations that can be easily bound to controls in the UI. **They encapsulate the actual code that implements the action or operation and help to keep it decoupled from its actual visual representation in the view.**
```ad-important
Interaction between the UI controls in the view and the command can be two-way. In this case, the command can be invoked as the user interacts with the UI, and the UI can be automatically enabled or disabled as the underlying command becomes enabled or disabled.
```