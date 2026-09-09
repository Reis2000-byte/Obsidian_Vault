
## What is WinGet?  
  
**WinGet (Windows Package Manager)** is Microsoft's command-line package manager for Windows. It allows you to discover, install, update, remove, and configure software directly from Command Prompt, PowerShell, or Windows Terminal. It serves as the client interface to the Windows Package Manager service and is included with modern versions of Windows 10 and Windows 11 through the App Installer package.

## What WinGet Can Do  

- Install applications from trusted repositories. 

- Search for available software packages. 

- Upgrade installed applications.

- Uninstall applications. 

- List installed software.

- Export installed applications to a file. 

- Import and automatically install applications on another computer.

- Manage package sources (repositories). 

- Configure development and workstation environments.

- Automate software deployment through scripts.  

---  

# Basic Syntax  

```powershell  

winget <command> [options]  

```  

### Example  

```powershell  

winget install Microsoft.VisualStudioCode  

```  

---  

# Most Common WinGet Commands  

| Command                       | What It Does                            |
| ----------------------------- | --------------------------------------- |
| `winget search <name>`        | Search for a package                    |
| `winget show <package>`       | Show detailed package information       |
| `winget install <package>`    | Install an application                  |
| `winget list`                 | List installed applications             |
| `winget upgrade`              | Show available updates                  |
| `winget upgrade --all`        | Upgrade all eligible applications       |
| `winget uninstall <package>`  | Remove an application                   |
| `winget export -o apps.json`  | Export installed app list               |
| `winget import -i apps.json`  | Install applications from exported list |
| `winget source list`          | Display configured sources/repositories |
| `winget source update`        | Refresh source metadata                 |
| `winget source add`           | Add a package source                    |
| `winget source remove`        | Remove a package source                 |
| `winget --version`            | Display WinGet version                  |
| `winget --info`               | Display WinGet environment information  |
| `winget settings`             | Open WinGet settings file               |
| `winget pin list`             | Show pinned packages                    |
| `winget pin add <package>`    | Prevent a package from updating         |
| `winget pin remove <package>` | Remove a package pin                    |

---  
# Common Examples  

## Search for an Application  

```powershell  

winget search vscode  

```  
## Install an Application  

```powershell  

winget install Microsoft.VisualStudioCode  

```  
## Install Silently  

```powershell  

winget install Microsoft.VisualStudioCode --silent  

```  
## List Installed Software  
```powershell  

winget list  

```  
## Check for Available Updates  
```powershell  

winget upgrade  

```  
## Update Everything  
```powershell  

winget upgrade --all  

```  
## Uninstall an Application  
```powershell  

winget uninstall Microsoft.VisualStudioCode  

```  
## Export Installed Applications  

```powershell  

winget export -o apps.json  

```  
## Import Applications on a New PC  
```powershell  

winget import -i apps.json  

```  

---  
# Helpful Options  

| Option                        | Description                                |
| ----------------------------- | ------------------------------------------ |
| `--id`                        | Use exact package ID                       |
| `--silent`                    | Install without prompts                    |
| `--force`                     | Force installation or upgrade              |
| `--accept-source-agreements`  | Automatically accept repository agreements |
| `--accept-package-agreements` | Automatically accept package licenses      |
| `--scope machine`             | Install for all users                      |
| `--scope user`                | Install for current user only              |
| `--version <version>`         | Install a specific version                 |

---  
# Administrative Commands  

## Upgrade Everything Automatically  

```powershell  

winget upgrade --all --silent --accept-package-agreements --accept-source-agreements  

```  
## Find a Package's Exact ID  

```powershell  

winget search chrome  

```  
## Install Using Exact Package ID  
```powershell  

winget install --id Google.Chrome  

```  
## Show Detailed Package Information  

```powershell  

winget show Microsoft.VisualStudioCode  

```  

---  
# Best Practices  

1. Use `--id` when installing packages in scripts.  

2. Review available upgrades before running `upgrade --all`.  

3. Export your application list before rebuilding a PC.  

4. Use package pins to prevent critical software from updating unexpectedly.  

5. Run Terminal or PowerShell as Administrator when installing system-wide software.  

---  
# Troubleshooting  
## Update WinGet Sources  

```powershell  

winget source update  

```  
## Reset Sources  
```powershell  

winget source reset --force  

```  
## Check WinGet Version  
```powershell  

winget --version  

```  
## Display Detailed Environment Information  
```powershell  

winget --info  

```  
---  
# Quick Cheat Sheet  

```powershell  

# Search  

winget search <app>  

  

# Install  

winget install <app>  

  

# Install by ID  

winget install --id <packageID>  

  

# Show package details  

winget show <app>  

  

# List installed software  

winget list  

  

# View available updates  

winget upgrade  

  

# Upgrade everything  

winget upgrade --all  

  

# Remove software  

winget uninstall <app>  

  

# Export installed app list  

winget export -o apps.json  

  

# Import app list  

winget import -i apps.json  

  

# View repositories  

winget source list  

  

# Refresh repositories  

winget source update  

  

# Open settings  

winget settings  

  

# Version information  

winget --version  

  

# Detailed environment information  

winget --info  

```  
## One-Liner for Routine Maintenance  

```powershell  

winget upgrade --all --silent --accept-package-agreements --accept-source-agreements  

```  

Updates all eligible packages with minimal user interaction.