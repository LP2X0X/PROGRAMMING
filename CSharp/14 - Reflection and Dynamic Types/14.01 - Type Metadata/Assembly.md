---
tags:
 - csharp
 - reflection
 - metadata
---

## The Assembly Metadata Token

When you examine an assembly's IL using `ildasm.exe`, the **Assembly token** describes the assembly itself — its identity. This is the metadata the runtime reads first to know *what* it's loading.

The Assembly token records:

- **Name** — the simple name of the assembly (e.g., `CarLibrary`)
- **Version** — a four-part version number: `major.minor.build.revision`
- **Culture** — the locale for satellite assemblies (neutral if not localized)
- **Public key token** — an 8-byte hash of the public key, used for [[The Role of .NET Assemblies|strong-named assemblies]]
- **Hash algorithm** — the algorithm used to hash files in the assembly (typically SHA1)

### Example ildasm Output

```il
.assembly CarLibrary
{
  .ver 1:0:0:0
  .hash algorithm 0x00008004  // SHA1
  .publickeytoken = (null)     // not strong-named
}
```

### Why This Matters

The Assembly token is the assembly's "passport." When another assembly depends on this one, the compiler copies these identity values into an [[AssemblyRef]] token in the *consuming* assembly. At load time, the runtime compares the AssemblyRef against the actual Assembly token to verify it found the correct version and, for strong-named assemblies, that the signature is valid.

```ad-note
If the Assembly token has a null public key token, the assembly is not strong-named. Strong naming adds cryptographic verification — the runtime will refuse to load an assembly if the public key token in the AssemblyRef doesn't match the loaded assembly's actual key.
```

## See Also

- [[AssemblyRef]]
- [[TypeDef and TypeRef]]
- [[The Role of .NET Assemblies]]
