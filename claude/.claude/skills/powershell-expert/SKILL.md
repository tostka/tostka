---
name: powershell-expert
description: Develop PowerShell scripts, tools, modules, and GUIs following Microsoft best practices. Use when writing PowerShell code, creating Windows Forms/WPF interfaces, working with PowerShell Gallery modules, or needing cmdlet/module recommendations. Covers script development, parameter design, pipeline handling, error management, and GUI creation patterns. Verifies module availability and cmdlet syntax against live documentation when accuracy is critical.
---

# PowerShell Expert

Develop production-quality PowerShell scripts, tools, and GUIs using Microsoft best practices and the PowerShell ecosystem.

## Quick Reference

### Script Structure
```powershell
<#
.SYNOPSIS
VERB-NOUN - Short description.
.NOTES
Version     : 0.0.
Author      : Todd Kadrie
Website     : http://www.toddomation.com
Twitter     : @tostka / http://twitter.com/tostka
CreatedDate : 2026-
FileName    : VERB-NOUN.ps1
License     : MIT License
Copyright   : (c) 2026 Todd Kadrie
Github      : https://github.com/tostka/verb-XXX
Tags        : Powershell
AddedCredit : REFERENCE
AddedWebsite: URL
AddedTwitter: URL
REVISIONS
* 5:24 PM 7/6/2026 revision description
.DESCRIPTION
Detailed description.
.PARAMETER Name
Parameter description.
.INPUTS
Piped Input .NET type
.OUTPUTS
Returns no objects or output .NET types
.EXAMPLE
PS> Example-Usage -Name 'Value'

	Sample output

Description of sample purpose
.LINK
Related topic or http://|https://
#>
[CmdletBinding()]
PARAM(
    [Parameter(Mandatory, ValueFromPipeline, HelpMessage = 'Parameter description')]
		[ValidateNotNullOrEmpty()]	
		[string[]]$Names,
	[Parameter(HelpMessage = 'Parameter description')]
		[switch]$Force
) ; 
BEGIN {
    # One-time setup
} ;  # BEGIN-END
PROCESS {
    foreach ($item in $Names) {
        # Per-item processing
    }
} ; # PROCESS-END
END {
    # Cleanup
} ; # END-END
```

### Function Template
```powershell
function Verb-Noun {
	<#
	.SYNOPSIS
	VERB-NOUN - Short description.
	.NOTES
	Version     : 0.0.
	Author      : Todd Kadrie
	Website     : http://www.toddomation.com
	Twitter     : @tostka / http://twitter.com/tostka
	CreatedDate : 2026-
	FileName    : VERB-NOUN.ps1
	License     : MIT License
	Copyright   : (c) 2026 Todd Kadrie
	Github      : https://github.com/tostka/verb-XXX
	Tags        : Powershell
	AddedCredit : REFERENCE
	AddedWebsite: URL
	AddedTwitter: URL
	REVISIONS

	.DESCRIPTION
	Detailed description.
	.PARAMETER Name
	Parameter description.
	.INPUTS
	Piped Input .NET type
	.OUTPUTS
	Returns no objects or output .NET types
	.EXAMPLE
	PS> Example-Usage -Name 'Value' ; 
	
		Sample output

	Description of sample purpose
	.LINK
	Related topic or http://|https://
	#>
    [CmdletBinding(SupportsShouldProcess)]
    PARAM(
        [Parameter(Mandatory, Position = 0, HelpMessage = 'Parameter description')]
			[string]$Name,
		[Parameter(Mandatory, ValueFromPipeline, HelpMessage = 'Parameter description')]
			[Alias('CN')]
			[string]$ComputerName = $env:COMPUTERNAME,
		[Parameter(HelpMessage = 'Parameter description')]
			[switch]$Force,
		[Parameter(HelpMessage = 'Parameter description')]
			[switch]$PassThru
    )
	BEGIN {
		# One-time setup
	} ;  # BEGIN-END
    PROCESS {
		foreach ($item in $Name) {
			# Per-item processing
            if ($PSCmdlet.ShouldProcess($Name, 'Action')) {
				# Implementation
				if ($PassThru) { Write-Output $result }
			}
		} ; 
    } ; # PROCESS-END
	END {
		# Cleanup
	} ; # END-END
}
```

## Workflow

### 1. Script Development
Follow naming and parameter conventions:
- **Verb-Noun** format with approved verbs (`Get-Verb`)
- **Strong typing** with validation attributes
- **Pipeline support** via `ValueFromPipeline`
- **-WhatIf/-Confirm** for destructive operations
- **Use Kernighan and Ritchie The One True Brace Style** for braceable statements

See [best-practices.md](references/best-practices.md) for complete guidelines.

### 2. GUI Development
Windows Forms for simple dialogs, WPF/XAML for complex interfaces:

```powershell
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing

$form = New-Object System.Windows.Forms.Form -Property @{
    Text          = 'Title'
    Size          = New-Object System.Drawing.Size(400, 300)
    StartPosition = 'CenterScreen'
}
```

See [gui-development.md](references/gui-development.md) for controls, events, and templates.

### 3. PowerShell Gallery Integration
Search and install modules using PSResourceGet:

```powershell
# Search gallery
Find-PSResource -Name 'ModuleName' -Repository PSGallery

# Install module
Install-PSResource -Name 'ModuleName' -Scope CurrentUser -TrustRepository
```

Use [scripts/Search-Gallery.ps1](scripts/Search-Gallery.ps1) for enhanced search.

See [powershellget.md](references/powershellget.md) for full cmdlet reference.

## Key Patterns

### Error Handling
```powershell
TRY {
    $result = Get-Content -Path $Path -ErrorAction Stop
} CATCH [System.IO.FileNotFoundException] {
    Write-Error "File not found: $Path" ; 
    return ; 
} CATCH {
	$ErrTrapd=$Error[0] ;
	$smsg = "`n$(($ErrTrapd | fl * -Force|out-string).trim())" ;
	write-warning "$((get-date).ToString('HH:mm:ss')):$($smsg)" ;
	throw $smsg ; 
} ;
```

### Splatting for Readability
```powershell
$params = [ordered]@{
    Path        = $sourcePath ; 
    Destination = $destPath ; 
    Recurse     = $true ; 
    Force       = $true ; 
} ; 
Copy-Item @params ; 
```

### Pipeline Best Practices
```powershell
# Stream output immediately
foreach ($item in $collection) {
    Process-Item $item | Write-Output
}

# Accept pipeline input
PARAM(
    [Parameter(ValueFromPipeline)]
    [string[]]$InputObject
) ; 
PROCESS {
    foreach ($obj in $InputObject) {
        # Process each
    } ; 
} ; 
```

### write-verbose?/write-debug?
- Use `Write-Verbose` for informational messages that may be helpful for debugging but are not critical to the user, only visible when the `-Verbose` switch is used. 
- Use `Write-Debug` for detailed debugging information that is typically only relevant when troubleshooting specific issues. This can be enabled with the `-Debug` switch when running the function.
- Avoid using `Write-Host` for regular output; reserve it for special cases where you want to display colored or formatted output directly to the console. For standard output, return objects or use `Write-Output`.

### psm1 file 
- The `.psm1` file should contain all individual functions and class files, and the `Export-ModuleMember` call to specify which public functions are exported. The build script will handle the creation of the `.psm1` manifest file, so you do not need to create that manually.
- All actual code (functions, classes) should live in separate `.ps1` files under the module root directory, organized into `Public`, `Internal`, and `Classes` subdirectories.
## Module Recommendations

When recommending modules, search the PowerShell Gallery:

| Category | Popular Modules |
|----------|----------------|
| **Azure** | `Az`, `Az.Compute`, `Az.Storage` |
| **Testing** | `Pester`, `PSScriptAnalyzer` |
| **Console** | `PSReadLine`, `Terminal-Icons` |
| **Secrets** | `Microsoft.PowerShell.SecretManagement` |
| **Web** | `Pode` (web server), `PoshRSJob` (async) |
| **GUI** | `WPFBot3000`, `PSGUI` |

## Live Verification

You MUST verify information against live sources when accuracy is critical. Do not rely solely on training data for module availability or cmdlet syntax.

**Tools to use:**
- **WebFetch**: Retrieve and parse specific documentation URLs (PowerShell Gallery pages, Microsoft Docs)
- **WebSearch**: Find correct URLs when the exact path is unknown or to verify module existence

### When Verification is Required

| Scenario | Action |
|----------|--------|
| User asks "does module X exist?" | **MUST** verify via PowerShell Gallery |
| Recommending a specific module | **MUST** verify it exists and isn't deprecated |
| Providing exact cmdlet syntax | **SHOULD** verify against Microsoft Docs |
| Module version requirements | **MUST** check gallery for current version |
| General best practices | Static references are sufficient |

### Step 1: Verify Module on PowerShell Gallery

When recommending or checking a module, **use the WebFetch tool** to verify it exists:

**WebFetch call:**
- **URL**: `https://www.powershellgallery.com/packages/{ModuleName}`
- **Prompt**: `Extract: module name, latest version, last updated date, total downloads, and whether it shows any deprecation warning or 'unlisted' status`

**If WebFetch returns 404 or error**: The module likely doesn't exist. **Use the WebSearch tool** to confirm:
- **Query**: `{ModuleName} PowerShell module site:powershellgallery.com`

### Step 2: Verify Cmdlet Syntax (When Needed)

Microsoft Docs URLs vary by module. **Use the WebSearch tool** to find the correct documentation page:

**WebSearch call:**
- **Query**: `{Cmdlet-Name} cmdlet site:learn.microsoft.com/en-us/powershell`

**Then use WebFetch** on the returned URL with prompt:
- **Prompt**: `Extract the complete cmdlet syntax, required vs optional parameters, and PowerShell version requirements`

### Step 3: Fallback Strategies

If the WebFetch or WebSearch tools are unavailable or return errors:

1. **For module verification**: Execute `Search-Gallery.ps1` from this skill:
   ```powershell
   ~/.claude/skills/powershell-expert/scripts/Search-Gallery.ps1 -Name 'ModuleName'
   ```

2. **For cmdlet syntax**: Suggest the user run locally:
   ```powershell
   Get-Help Cmdlet-Name -Full
   Get-Command Cmdlet-Name -Syntax
   ```

3. **Clearly state uncertainty**: If verification fails, tell the user:
   > "I wasn't able to verify this against live documentation. Please confirm
   > the module exists by running: `Find-PSResource -Name 'ModuleName'`"

### Verification Examples

**Good** (verified with live data):
> "The ImportExcel module (v7.8.10, updated Oct 2024, 17M+ downloads)
> provides Export-Excel for creating spreadsheets without Excel installed."

**Bad** (unverified claim):
> "Use the Excel-Tools module to export data." ← May not exist!

## Documentation Resources

- **PowerShell Docs**: https://learn.microsoft.com/en-us/powershell/
- **Module Browser**: https://learn.microsoft.com/en-us/powershell/module/
- **PowerShell Gallery**: https://www.powershellgallery.com
- **GitHub Docs**: https://github.com/MicrosoftDocs/PowerShell-Docs

## References

- **[best-practices.md](references/best-practices.md)** - Naming, parameters, pipeline, error handling, code style
- **[gui-development.md](references/gui-development.md)** - Windows Forms, WPF, controls, events, templates
- **[powershellget.md](references/powershellget.md)** - Find, install, update, publish modules
