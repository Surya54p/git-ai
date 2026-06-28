function git-ai {
    # Hanya mengambil diff dari file kode, abaikan semua file lain yang bukan teks
    $diff = git diff --cached -- "*.ts" "*.js" "*.tsx" "*.jsx" "*.json" "*.css" "*.html" | Out-String
    
    if (-not $diff.Trim()) {
        Write-Host "Tidak ada perubahan file kode di staging area!" -ForegroundColor Yellow
        return
    }
    
    $apiKey = ""
    $url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.5-flash:generateContent?key=$apiKey"
    
    $prompt = "Buatkan pesan commit git Conventional (type: subject) untuk perubahan kode berikut. Berikan output teksnya saja:`n$diff"
    
    $body = @{ contents = @{ parts = @{ text = $prompt } } } | ConvertTo-Json -Depth 5
    
    try {
        $response = Invoke-RestMethod -Uri $url -Method Post -Body $body -ContentType "application/json"
        $commitMessage = $response.candidates[0].content.parts[0].text
        
        Write-Host "`n--- PESAN COMMIT ---`n" -ForegroundColor Cyan
        Write-Host $commitMessage.Trim() -ForegroundColor Green
        
        $confirm = Read-Host "`nApakah ingin commit dengan pesan ini? (y/n)"
        if ($confirm -eq 'y') { git commit -m "$($commitMessage.Trim())" }
    } catch {
        Write-Host "Gagal memanggil API. Cek koneksi atau API Key." -ForegroundColor Red
    }
}