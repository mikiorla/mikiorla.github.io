## Browser Cookies Explained

### What Are Browser Cookies?

Browser cookies are small text files stored by your browser when you visit a website. They enable websites to store session information, preferences, and tracking data to enhance the user experience.

### Key Functions:

- **Session Management:** Maintain login sessions, shopping carts.
- **Personalization:** Store user preferences like language or theme.
- **Tracking & Analytics:** Monitor website visits and behavior.
- **Security:** Manage session authentication.

### Types of Cookies:

| Type                | Description                                    |
| ------------------- | ---------------------------------------------- |
| Session Cookies     | Temporary, deleted on browser close.           |
| Persistent Cookies  | Stored on disk with an expiration date.        |
| First-party Cookies | Set by the visited website.                    |
| Third-party Cookies | Set by domains other than the visited website. |
| Secure Cookies      | Only transmitted over HTTPS.                   |
| HttpOnly Cookies    | Not accessible via JavaScript.                 |

### Cookie Security Risks:

| Attack Technique                  | Description                                     | Example Impact               |
| --------------------------------- | ----------------------------------------------- | ---------------------------- |
| Session Hijacking                 | Stealing session cookies to impersonate a user. | Unauthorized account access. |
| XSS (Cross-site Scripting)        | Injecting scripts to read cookies.              | Session theft.               |
| CSRF (Cross-site Request Forgery) | Forcing users to perform unwanted actions.      | Unauthorized actions.        |
| MITM (Man-in-the-Middle)          | Intercepting cookies on unsecured networks.     | Credential compromise.       |
| Tracking & Privacy Invasion       | Tracking users without consent.                 | Privacy breach.              |
| Cookie Poisoning                  | Modifying cookie data to manipulate sessions.   | Privilege escalation.        |

---

## Where Are Cookies Stored?

### Chrome Cookie Storage:

| OS      | Location                                                                          |
| ------- | --------------------------------------------------------------------------------- |
| Windows | C:\Users\<YourUser>\AppData\Local\Google\Chrome\User Data\Default\Network\Cookies |
| macOS   | \~/Library/Application Support/Google/Chrome/Default/Network/Cookies              |
| Linux   | \~/.config/google-chrome/Default/Network/Cookies                                  |

Format: SQLite database (encrypted).

### Decrypting Chrome Cookies (v80+):

- **Key Storage:** Chrome stores the AES key in `Local State` (Base64 + DPAPI encryption).
- **Encryption:** Cookies are AES-GCM encrypted in the database.

---

## PowerShell Script: Legacy DPAPI Decryption

```powershell
$cookieDb = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies"
Add-Type -AssemblyName System.Data.SQLite
$connection = New-Object System.Data.SQLite.SQLiteConnection("Data Source=$cookieDb;Version=3;Read Only=True;")
$connection.Open()
$command = $connection.CreateCommand()
$command.CommandText = "SELECT host_key, name, encrypted_value FROM cookies"
$reader = $command.ExecuteReader()

while ($reader.Read()) {
    $host = $reader.GetString(0)
    $name = $reader.GetString(1)
    $encryptedBytes = $reader.GetValue(2)

    try {
        $decryptedBytes = [System.Security.Cryptography.ProtectedData]::Unprotect($encryptedBytes, $null, [System.Security.Cryptography.DataProtectionScope]::CurrentUser)
        $cookieValue = [System.Text.Encoding]::UTF8.GetString($decryptedBytes)
        Write-Output "$host`t$name`t$cookieValue"
    }
    catch {
        Write-Warning "Failed to decrypt cookie for $name on $host"
    }
}
$reader.Close()
$connection.Close()
```

---

## PowerShell Script: Chrome v80+ AES-GCM Decryption

```powershell
$localStatePath = "$env:LOCALAPPDATA\Google\Chrome\User Data\Local State"
$cookieDbPath = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies"

$localStateContent = Get-Content -Path $localStatePath -Raw
$localStateJson = ConvertFrom-Json $localStateContent
$encryptedKeyBase64 = $localStateJson.os_crypt.encrypted_key
$encryptedKey = [System.Convert]::FromBase64String($encryptedKeyBase64)
$encryptedKey = $encryptedKey[5..($encryptedKey.Length - 1)]
$masterKey = [System.Security.Cryptography.ProtectedData]::Unprotect($encryptedKey, $null, [System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Add-Type -AssemblyName System.Data.SQLite
$connection = New-Object System.Data.SQLite.SQLiteConnection("Data Source=$cookieDbPath;Version=3;Read Only=True;")
$connection.Open()
$command = $connection.CreateCommand()
$command.CommandText = "SELECT host_key, name, encrypted_value FROM cookies"
$reader = $command.ExecuteReader()

while ($reader.Read()) {
    $host = $reader.GetString(0)
    $name = $reader.GetString(1)
    $encryptedBytes = $reader.GetValue(2)

    if ($encryptedBytes[0] -eq 0x76 -and $encryptedBytes[1] -eq 0x31 -and $encryptedBytes[2] -eq 0x30) {
        $nonce = $encryptedBytes[3..14]
        $cipherText = $encryptedBytes[15..($encryptedBytes.Length - 1)]

        try {
            $aes = [System.Security.Cryptography.AesGcm]::new($masterKey)
            $plainBytes = New-Object byte[] ($cipherText.Length - 16)
            $tag = $cipherText[-16..-1]
            $cipher = $cipherText[0..($cipherText.Length - 17)]

            $aes.Decrypt($nonce, $cipher, $tag, $plainBytes, $null)
            $cookieValue = [System.Text.Encoding]::UTF8.GetString($plainBytes)

            Write-Output "$host`t$name`t$cookieValue"
        }
        catch {
            Write-Warning "Failed to decrypt AES cookie for $name on $host"
        }
    } else {
        try {
            $cookieValue = [System.Text.Encoding]::UTF8.GetString([System.Security.Cryptography.ProtectedData]::Unprotect($encryptedBytes, $null, [System.Security.Cryptography.DataProtectionScope]::CurrentUser))
            Write-Output "$host`t$name`t$cookieValue"
        }
        catch {
            Write-Warning "Failed to decrypt legacy cookie for $name on $host"
        }
    }
}

$reader.Close()
$connection.Close()
```

---

### Security Recommendations:

- Use Secure, HttpOnly, and SameSite cookie attributes.
- Always use HTTPS.
- Periodically clear cookies and invalidate sessions.
- Encrypt sensitive cookie contents.
- Sanitize any data derived from cookies.

\--- End of Document ---

