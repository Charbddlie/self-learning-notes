重新编译dll

```powershell
$env:PATH = [Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory() 
[AppDomain]::CurrentDomain.GetAssemblies() | ForEach-Object { 
    $path = $_.Location 
    if ($path) { 
        $name = Split-Path $path -Leaf 
        Write-Host -ForegroundColor Yellow "`r`n$name" 
        ngen.exe install $path /nologo 
    } 
    } 
```

