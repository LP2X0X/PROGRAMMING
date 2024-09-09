- `std::exit()` is a function that causes the program to [[Normal termination|terminate normally]].
- `std::exit()` performs a number of cleanup functions. First, objects with static storage duration are destroyed. Then some other miscellaneous file cleanup is done if any files were used. Finally, control is returned back to the OS, with the argument passed to `std::exit()` used as the `status code`.

```ad-note
`std::exit` is called implicitly when `main()` returns.
```