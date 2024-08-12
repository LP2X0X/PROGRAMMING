- There are two common situations:
	- You are looking to optimize select parts of your application by directly manipulating memory outside the management of the .NET 5 Runtime.
	- You are calling methods of a C-based .dll or COM server that demand pointer types as parameters. Even in this case, you can often bypass pointer types in favor of the System.IntPtr type and members of the System.Runtime.InteropServices.Marshal type.
---

- Allow unsafe code:
![[Pasted image 20240618111252.png|center|700]]

---

![[Pasted image 20240618110536.png|center|700]]
