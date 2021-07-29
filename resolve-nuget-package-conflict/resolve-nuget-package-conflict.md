# Resolving Nuget Package Conflicts

## What is Happening
Nuget package conflicts arise when two packages have a common dependency with contradicting version requirements. For example, your project depends on packages A and B. Both A and B require C. Package A requires C, from version 2.1.0, while package B requires C not greater than version 2.0.0.

```
                    +-----------+      >= v2.1.0
         /--------->| Package A |-----------------\
        |           +-----------+                \|/
 +-----------+                               +-----------+ 
 | MyProject |                               | Package C |
 +-----------+                               +-----------+  
        |             +-----------+                 /|\
         \----------->| Package B |------------------|
                      +-----------+        <= v2.0.0
```
When possible you can upgrade package A and B to their latest version and hoping that the conflict resolves itself. In some occasions updates will not help the situation. 

## Binding Redirection
One way to resolve this conflict is to tell 


```c
System.IO.FileLoadException : Could not load file or assembly 'Moq, Version=4.1.1308.2120, Culture=neutral, PublicKeyToken=69f491c39445e920' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference. (Exception from HRESULT: 0x80131040)
```