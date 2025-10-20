# GetUsedAssemblyReferences-repro
Small repro to show that GetUsedAssemblyReferences returns not used assembly references

Current structure:

`ProjectC` has no dependencies. It has `internal class ClassC` that is visible to `ProjectB`
`ProjectB` depends on `ProjectC` and has two classes: `internal class ClassB` that derives from `ClassC`, and `class ClassB_UsedByA`
`ProjectA` depends on `ProjectB` and has `class ClassA` that derives from `ClassB_UsedByA`

With this setup - `ProjectA` has nothing to do with `ProjectC`, only `ProjectB` does.

`ReferenceTrimmer` package has been added, in order to make it easier to see what assembly references are considered `used`.

# Repro

```
dotnet build /p:EnableReferenceTrimmer=true /p:EnableReferenceTrimmerDiagnostics=true
```

if we now open `.\ProjectA\obj\Debug\net9.0\_ReferenceTrimmer_UsedReferences.log` (update path based on .net version) - we can see the following:

```
...\GetUsedAssemblyReferences-repro\ProjectB\obj\Debug\net9.0\ref\ProjectB.dll
...\dotnet\packs\Microsoft.NETCore.App.Ref\9.0.10\ref\net9.0\System.Runtime.dll
```

Meaning the solution builds file (without `ProjectA` referenceing `ProjectC`) and only used reference of `ProjectA` is `ProjectB`, which is expected

Now to build with `ProjectA` having a reference to `ProjectC` - we add `/p:Add_A_To_C_Reference=true` to the command line

```
dotnet build /p:Add_A_To_C_Reference=true /p:EnableReferenceTrimmer=true /p:EnableReferenceTrimmerDiagnostics=true
```

Now if we open the same file `.\ProjectA\obj\Debug\net9.0\_ReferenceTrimmer_UsedReferences.log` - it now contains `ProjectC`

```
...\GetUsedAssemblyReferences-repro\ProjectB\obj\Debug\net9.0\ref\ProjectB.dll
...\GetUsedAssemblyReferences-repro\ProjectC\obj\Debug\net9.0\ref\ProjectC.dll
...\dotnet\packs\Microsoft.NETCore.App.Ref\9.0.10\ref\net9.0\System.Runtime.dll
```

My confusion is why is the `ProjectC` considered as used assembly reference to `ProjectA`? We didn't change any code - we just built with adding this reference