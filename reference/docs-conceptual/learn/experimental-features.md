---
ms.date: 12/14/2020
title: Using Experimental Features in PowerShell
description: Lists the currently available experimental features and how to use them.
---
# Using Experimental Features in PowerShell

The Experimental Features support in PowerShell provides a mechanism for experimental features to
coexist with existing stable features in PowerShell or PowerShell modules.

An experimental feature is one where the design is not finalized. The feature is available for users
to test and provide feedback. Once an experimental feature is finalized, the design changes become
breaking changes.

> [!CAUTION]
> Experimental features aren't intended to be used in production since the changes are allowed to be
> breaking. Experimental features are not officially supported. However, we appreciate any feedback
> and bug reports. You can file issues in the
> [GitHub source repository](https://github.com/PowerShell/PowerShell/issues/new/choose).

For more information about enabling or disabling these features, see
[about_Experimental_Features](/powershell/module/microsoft.powershell.core/about/about_experimental_features).

## Available features

This article describes the experimental features that are available and how to use the feature.

|                            Name                            |   6.2   |   7.0   |   7.1   |   7.2   |
| ---------------------------------------------------------- | :-----: | :-----: | :-----: | :-----: |
| PSTempDrive (mainstream in PS 7.0+)                        | &check; |         |         |         |
| PSUseAbbreviationExpansion (mainstream in PS 7.0+)         | &check; |         |         |         |
| PSNullConditionalOperators (mainstream in PS 7.1+)         |         | &check; |         |         |
| PSUnixFileStat (non-Windows only - mainstream in PS 7.1+)  |         | &check; |         |         |
| PSCommandNotFoundSuggestion                                | &check; | &check; | &check; | &check; |
| PSImplicitRemotingBatching                                 | &check; | &check; | &check; | &check; |
| Microsoft.PowerShell.Utility.PSManageBreakpointsInRunspace |         | &check; | &check; | &check; |
| PSDesiredStateConfiguration.InvokeDscResource              |         | &check; | &check; | &check; |
| PSNativePSPathResolution                                   |         |         | &check; | &check; |
| PSCultureInvariantReplaceOperator                          |         |         | &check; | &check; |
| PSNotApplyErrorActionToStderr                              |         |         | &check; | &check; |
| PSSubsystemPluginModel                                     |         |         | &check; | &check; |
| PSAnsiProgress                                             |         |         |         | &check; |
| PSAnsiRendering                                            |         |         |         | &check; |

## Microsoft.PowerShell.Utility.PSManageBreakpointsInRunspace

In PowerShell 7.0, the experiment enables the **BreakAll** parameter on the `Debug-Runspace` and
`Debug-Job` cmdlets to allow users to decide if they want PowerShell to break immediately in the
current location when they attach a debugger.

In PowerShell 7.1, this experiment also adds the **Runspace** parameter to the `*-PSBreakpoint`
cmdlets.

- `Disable-PSBreakpoint`
- `Enable-PSBreakpoint`
- `Get-PSBreakpoint`
- `Remove-PSBreakpoint`
- `Set-PSBreakpoint`

The **Runspace** parameter specifies a **Runspace** object to interact with breakpoints in the
specified runspace.

```powershell
Start-Job -ScriptBlock {
    Set-PSBreakpoint -Command Start-Sleep
    Start-Sleep -Seconds 10
}

$runspace = Get-Runspace -Id 1

$breakpoint = Get-PSBreakPoint -Runspace $runspace
```

In this example, a job is started and a breakpoint is set to break when the `Set-PSBreakPoint` is
run. The runspace is stored in a variable and passed to the `Get-PSBreakPoint` command with the
**Runspace** parameter. You can then inspect the breakpoint in the `$breakpoint` variable.

## PSAnsiRendering

This experiment was added in PowerShell 7.2. The feature enables changes how the PowerShell engine
outputs text and add the `$PSStyle` automatic variable to control ANSI rendering of string output.

```powershell
PS> $PSStyle | Get-Member

   TypeName: System.Management.Automation.PSStyle

Name             MemberType Definition
----             ---------- ----------
Equals           Method     bool Equals(System.Object obj)
FormatHyperlink  Method     string FormatHyperlink(string text, uri link)
GetHashCode      Method     int GetHashCode()
GetType          Method     type GetType()
ToString         Method     string ToString()
Background       Property   System.Management.Automation.PSStyle+BackgroundColor Background {get;}
Blink            Property   string Blink {get;}
BlinkOff         Property   string BlinkOff {get;}
Bold             Property   string Bold {get;}
BoldOff          Property   string BoldOff {get;}
Foreground       Property   System.Management.Automation.PSStyle+ForegroundColor Foreground {get;}
Formatting       Property   System.Management.Automation.PSStyle+FormattingData Formatting {get;}
Hidden           Property   string Hidden {get;}
HiddenOff        Property   string HiddenOff {get;}
Italic           Property   string Italic {get;}
ItalicOff        Property   string ItalicOff {get;}
OutputRendering  Property   System.Management.Automation.OutputRendering OutputRendering {get;set;}
Progress         Property   System.Management.Automation.PSStyle+ProgressConfiguration Progress {get;}
Reset            Property   string Reset {get;}
Reverse          Property   string Reverse {get;}
ReverseOff       Property   string ReverseOff {get;}
Strikethrough    Property   string Strikethrough {get;}
StrikethroughOff Property   string StrikethroughOff {get;}
Underline        Property   string Underline {get;}
UnderlineOff     Property   string UnderlineOff {get;}
```

The base members return strings of ANSI escape sequences mapped to their names. The values are
settable to allow customization.

For more information, see [about_Automatic_Variables](/powershell/module/Microsoft.PowerShell.Core/About/about_Automatic_Variables.md)

> [!NOTE]
> For C# developers, you can access `PSStyle` as a singleton. Usage will look like this:
>
> ```csharp
> string output = $"{PSStyle.Instance.Foreground.Red}{PSStyle.Instance.Bold}Hello{PSStyle.Instance.Reset}";
> ```
>
> `PSStyle` exists in the System.Management.Automation namespace.

Along with access to `$PSStyle`, this introduces changes to the PowerShell engine. The PowerShell
formatting system is updated to respect `$PSStyle.OutputRendering`.

- `StringDecorated` type is added to handle ANSI escaped strings.
- `string IsDecorated` boolean property is added to return if the string contains ANSI escape
  sequences based on if the string contains ESC or C1 CSI.
- The `Length` property returns _only_ the length for the text without the ANSI escape sequences.
- `StringDecorated Substring(int contentLength)` method returns a substring starting at index 0 up
  to the content length that is not a part of ANSI escape sequences. This is needed for table
  formatting to truncate strings and preserve ANSI escape sequences that don't take up printable
  character space.
- `string ToString()` method stays the same and returns the plaintext version of the string.
- `string ToString(bool Ansi)` method returns the raw ANSI embedded string if the `Ansi` parameter
  is true. Otherwise, a plaintext version with ANSI escape sequences removed is returned.

The `FormatHyperlink(string text, uri link)` returns a string containing ANSI escape sequence used
to decorate hyperlinks. Some terminal hosts, like the
[Windows Terminal](https://www.microsoft.com/p/windows-terminal/9n0dx20hk701), support this markup,
which makes the rendered text clickable in the terminal.

## PSAnsiProgress

This experiment was added in PowerShell 7.2. The feature adds the `$PSStyle.Progress` member and
allows you to control progress view bar rendering.

- `$PSStyle.Progress.Style` - An ANSI string setting the rendering style.
- `$PSStyle.Progress.MaxWidth` - Sets the max width of the view. Set to `0` for console width.
  Defaults to `120`
- `$PSStyle.Progress.View` - An enum with values, `Minimal` and `Classic`. `Classic` is the existing
  rendering with no changes. `Minimal` is a single line minimal rendering. `Minimal` is the default.

The following example updates the rendering style to a minimal progress bar.

```powershell
$PSStyle.Progress.View.Minimal
```

> [!NOTE]
> You must have the **PSAnsiRendering** experimental feature enabled to use this feature.

## PSCommandNotFoundSuggestion

Recommends potential commands based on fuzzy matching search after a **CommandNotFoundException**.

```powershell
PS> get
```

```Output
get: The term 'get' is not recognized as the name of a cmdlet, function, script file, or operable
program. Check the spelling of the name, or if a path was included, verify that the path is correct
and try again.

Suggestion [4,General]: The most similar commands are: set, del, ft, gal, gbp, gc, gci, gcm, gdr,
gcs.
```

## PSCultureInvariantReplaceOperator

When the left-hand operand in a `-replace` operator statement is not a string, that operand is
converted to a string.

When this feature is disabled, the `-replace` operator does a culture-sensitive string conversion.
For example, if your culture is set to French (fr), the value `1.2` is converted to the string
`1,2`.

```powershell
PS> [cultureinfo]::CurrentCulture = 'fr'
PS> 1.2 -replace ','
12
PS> [cultureinfo]::CurrentCulture = 'en'
PS> 1.2 -replace ','
1.2
```

With the feature enabled:

```powershell
PS> [cultureinfo]::CurrentCulture = 'fr'
PS> 1.2 -replace ','
1.2
```

## PSDesiredStateConfiguration.InvokeDscResource

Enables compilation to MOF on non-Windows systems and enables the use of `Invoke-DSCResource`
without an LCM.

## PSImplicitRemotingBatching

This feature examines the command typed in the shell, and if all the commands are implicit remoting
proxy commands that form a simple pipeline, then the commands are batched together and invoked as a
single remote pipeline.

Example:

```powershell
# Create remote session and import TestIMod module
$s = nsn -host remoteMachine
icm $s { ipmo 'C:\Users\user\Documents\WindowsPowerShell\Modules\TestIMod\TestIMod.psd1' }
Import-PSSession $s -mod testimod

$maxProcs = 1000
$filter = 'pwsh','powershell*','wmi*'

# Without batching, this pipeline takes approximately 12 seconds to run
Measure-Command { Get-AllProcesses -MaxCount $maxProcs | Select-Custom $filter | Group-Stuff $filter }
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 12
Milliseconds      : 463

# But with the batching experimental feature enabled, it takes approximately 0.20 seconds
Measure-Command { Get-AllProcesses -MaxCount $maxProcs | Select-Custom $filter | Group-Stuff $filter }
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 209
```

As seen above, with the batching feature enabled, all three implicit remoting proxy commands,
`Get-AllProcesses`, `Select-Custom`, `Group-Stuff`, run in the remote session and the result from
the pipeline is the only data returned to the client. This decreases the amount of data sent back
and forth between client and remote session, and also reduces the amount of object serialization and
de-serialization.

## PSNativePSPathResolution

If a PSDrive path that uses the FileSystem provider is passed to a native command, the resolved file
path is passed to the native command. This means a command like `code temp:/test.txt` now works as
expected.

Also, on Windows, if the path starts with `~`, that is resolved to the full path and passed to the
native command. In both cases, the path is normalized to the directory separators for the relevant
operating system.

- If the path is not a PSDrive or `~` (on Windows), then path normalization doesn't occur
- If the path is in single quotes, then it's not resolved and treated as literal

## PSNotApplyErrorActionToStderr

When this experimental feature is enabled, error records redirected from native commands, like when
using redirection operators (`2>&1`), are not written to the `$Error` variable and the preference
variable `$ErrorActionPreference` does not affect the redirected output.

Many native commands write to `stderr` as an alternative stream for additional information. This
behavior can cause confusion when looking through errors or the additional output information can
be lost to the user if `$ErrorActionPreference` is set to a state that mutes the output.

When a native command has a non-zero exit code, `$?` is set to `$false`. If the exit code is zero,
`$?` is set to `$true`.

## PSNullConditionalOperators

Introduces new operators for Null conditional member access operators - `?.` and `?[]`. Null member
access operators can be used on scalar types and array types. Return the value of the accessed member
if the variable is not null. If the value of the variable is null, then return null.

```powershell
$x = $null
${x}?.propname
${x?}?.propname

${x}?[0]
${x?}?[0]

${x}?.MyMethod()
```

The property `propname` is accessed and it's value is returned only if `$x` is not null. Similarly,
the indexer is used only if `$x` is not null. If `$x` is null, then null is returned.

The `?.` and `?[]` operators are member access operators and do not allow a space in between the
variable name and the operator.

Since PowerShell allows `?` as part of the variable name, disambiguation is required when the
operators are used without a space between the variable name and the operator. To disambiguate, the
variables must use `{}` around the variable name like: `${x?}?.propertyName` or `${y}?[0]`.

> [!NOTE]
> This feature has moved out of the experimental phase and is a mainstream feature in PowerShell 7.1
> and higher.

## PSTempDrive

Creates the `TEMP:` PSDrive mapped to user's temporary directory path.

> [!NOTE]
> This feature has moved out of the experimental phase and is a mainstream feature in PowerShell 7
> and higher.

## PSUnixFileStat

This feature provides new behavior to include data from the Unix **stat** API in the output of the
file system provider to facilitate a more Unix-like file listing. It adds a new note property in the
filesystem provider named **UnixStat** that includes a common expression of the `stat(2)` API from
the underlying Unix type system.

The output from `Get-ChildItem` should look something like this:

```powershell
dir | select -first 4 -skip 5


    Directory: /Users/jimtru/src/github/forks/JamesWTruher/PowerShell-1

UnixMode   User      Group           LastWriteTime        Size Name
--------   ----      -----           -------------        ---- ----
drwxr-xr-x jimtru    staff        10/23/2019 13:16         416 test
drwxr-xr-x jimtru    staff         11/8/2019 10:37         896 tools
-rw-r--r-- jimtru    staff         11/8/2019 10:37      112858 build.psm1
-rw-r--r-- jimtru    staff         11/8/2019 10:37      201297 CHANGELOG.md
```

> [!NOTE]
> This feature has moved out of the experimental phase and is a mainstream feature in PowerShell 7.1
> and higher.

## PSUseAbbreviationExpansion

This feature enables tab-completion of abbreviated cmdlets and functions:

For example:

- `i-psdf<tab>` returns `Import-PowerShellDataFile`.
- `u-akvmssdr<tab>` returns `Undo-AzKeyVaultManagedStorageSasDefinitionRemoval`

This only works for tab completion (interactive use), so `i-psdf` is not a valid cmdlet name in a
script.

> [!NOTE]
> This feature has moved out of the experimental phase and is a mainstream feature in PowerShell 7
> and higher.

## PSSubsystemPluginModel

This feature enables the subsystem plugin model in PowerShell. The feature makes it possible to
separate components of `System.Management.Automation.dll` into individual subsystems that reside in
their own assembly. This separation reduces the disk footprint of the core PowerShell engine and
allows these components to become optional features for a minimal PowerShell installation.

Currently, only the **CommandPredictor** subsystem is supported. This subsystem is used along with
the PSReadLine module to provide custom prediction plugins. In future, **Job**,
**CommandCompleter**, **Remoting** and other components could be separated into subsystem
assemblies outside of `System.Management.Automation.dll`.

The experimental feature includes a new cmdlet,
[Get-PSSubsystem](xref:Microsoft.PowerShell.Core.Get-PSSubsystem). This cmdlet is only available
when the feature is enabled. This cmdlet returns information about the subsystems that are available
on the system.
