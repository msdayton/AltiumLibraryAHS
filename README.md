# AltiumLibraryAHS

#
# Installing the MySQL ODBC 9.4 Unicode Driver (Windows 64-bit)

## 1. Download the Driver
- Go to the official MySQL Connector/ODBC download page:  
  [https://dev.mysql.com/downloads/connector/odbc/](https://dev.mysql.com/downloads/connector/odbc/)
- Select the latest **MySQL Connector/ODBC 9.4** release (or the version required for your DSN).
- Choose the **Windows (x86, 64-bit), MSI Installer** package.

## 2. Run the Installer
- Double-click the `.msi` file you downloaded.
- When prompted, choose **"Typical" installation** unless you have specific requirements.
- Finish the installation — the Unicode driver file will be installed at:

C:\Program Files\MySQL\MySQL Connector ODBC 9.4\myodbc9w.dll

## 3. Verify the Driver Installation
- Open the **64-bit ODBC Data Source Administrator**:  

C:\Windows\System32\odbcad32.exe

- Go to the **Drivers** tab.
- Confirm you see:

MySQL ODBC 9.4 Unicode Driver

and that the **driver file path** matches the `$driver` value in your PowerShell DSN script.

## 4. Run the DSN Setup Script
- Open **PowerShell as Administrator**.
- Paste and run the DSN creation script provided in this repository.
- After it runs, confirm your DSN appears in the **System DSN** tab of the ODBC Data Source Administrator.

---

## Troubleshooting
- **Architecture Mismatch**:  
If Altium or another application is running as **x64**, make sure you have installed the **x64 ODBC driver**.  
If the app is **32-bit**, install the **x86 ODBC driver** instead.
- **Error 193** (`...driver could not be loaded...`):  
This usually indicates an architecture mismatch between the ODBC Administrator and the driver.


```powershell
# Define DSN parameters
$dsnName   = "ahs-altium-db-library"
$driver    = "C:\Program Files\MySQL\MySQL Connector ODBC 9.4\myodbc9w.dll"
$server    = "ahs-altium-db-library.czenmmkyyzlb.us-west-2.rds.amazonaws.com"
$database  = "AHS_Library"
$uid       = "matthew_librarian"
$pwd       = "YOUR_PASSWORD"
$port      = "3306"

# Preferred OPTION flags:
# 1 = Don't prompt when connecting
# 2 = Enable dynamic cursor
# 16 = Found rows (return matched rows count, not just changed)
# 2048 = Enable automatic reconnect
# Total = 1 + 2 + 16 + 2048 = 2067
$option    = 2067

# Create DSN registry keys
$regPath = "HKLM:\SOFTWARE\ODBC\ODBC.INI\$dsnName"
New-Item -Path $regPath -Force | Out-Null
Set-ItemProperty -Path $regPath -Name "Driver"   -Value $driver
Set-ItemProperty -Path $regPath -Name "SERVER"   -Value $server
Set-ItemProperty -Path $regPath -Name "DATABASE" -Value $database
Set-ItemProperty -Path $regPath -Name "UID"      -Value $uid
Set-ItemProperty -Path $regPath -Name "PWD"      -Value $pwd
Set-ItemProperty -Path $regPath -Name "PORT"     -Value $port
Set-ItemProperty -Path $regPath -Name "OPTION"   -Value $option
Set-ItemProperty -Path $regPath -Name "CHARSET"  -Value "utf8mb4"

# Add to ODBC Data Sources
$odbcSourcesPath = "HKLM:\SOFTWARE\ODBC\ODBC.INI\ODBC Data Sources"
Set-ItemProperty -Path $odbcSourcesPath -Name $dsnName -Value "MySQL ODBC 9.4 Unicode Driver"

Write-Host "System DSN '$dsnName' created successfully."
```
# Setting Up Your Environment for the AHS Altium Database Library
## 1. Download and Install MySQL Workbench 8.0 (Optional but Recommended)
MySQL Workbench provides a convenient GUI for inspecting tables, running queries, and verifying your DSN configuration.

1. Visit the MySQL Workbench 8.0 download page:  
   [https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/)
2. Select the appropriate installer for your operating system (e.g., **Windows (x86, 64-bit), MSI Installer**).
3. Download and run the installer:
   - When prompted, choose **"Typical"** installation (unless you have specific customization needs).
4. After installation, launch MySQL Workbench:
   - Create a new connection using your RDS endpoint:
     - **Hostname**: `ahs-altium-db-library.czenmmkyyzlb.us-west-2.rds.amazonaws.com`
     - **Port**: `3306`
     - **Username**: `matthew_librarian`
     - Provide the same password as used in your DSN setup.
   - Test the connection (should succeed if DSN is set up correctly).

## 2. Configure Altium Database Library (DBLib)

1. In **Altium Designer**, go to:  
   **File → New → Library → Database Library**.

2. In the **Connection** tab:  
   - **Connection String** (use backticks instead of brackets):  
     ```
     Provider=MSDASQL.1;Persist Security Info=False;Data Source=ahs-altium-db-library;Initial Catalog=AHS_Library;Found_rows=1;Big_packets=1;No_prompt=1;
     ```

3. Click **Advanced** and ensure:  
   - **Use Connection String** is checked.  
   - **Read-only** is **unchecked** (if you plan to write back).

4. In the **Table Browser**:  
   - Select your component table, e.g., `AhsComponent`.  
   - Map **Key Field** to **PartName** (database column name: `Part Name`).

5. Save and **Test** the DBLib to confirm the connection works.

