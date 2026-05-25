# UTF-8 → Shift-JIS 変換BAT

## 処理内容

このBATは、`input` フォルダ内のファイルを読み込み、  
UTF-8 から Shift-JIS に変換して `output` フォルダへ出力します。

出力ファイル名は、入力ファイル名と同じです。

変換後、元ファイルは以下のフォルダへ移動します。

```text
backup/yyyyMMdd/
```

`input` フォルダ自体は削除しません。  
処理後、`input` フォルダの中は空になります。

---

## フォルダ構成

```text
convert_tool/
├─ convert.bat
├─ convert.ps1
├─ template/
│  └─ blank_sjis.txt
├─ input/
│  └─ 変換したいファイル.csv
├─ output/
└─ backup/
```

---

## convert.bat

```bat
@echo off
setlocal

cd /d "%~dp0"

powershell -NoProfile -ExecutionPolicy Bypass -File "%~dp0convert.ps1"

pause
endlocal
```

---

## convert.ps1

```powershell
$ErrorActionPreference = "Stop"

$baseDir = Split-Path -Parent $MyInvocation.MyCommand.Path

$inputDir  = Join-Path $baseDir "input"
$outputDir = Join-Path $baseDir "output"
$backupRootDir = Join-Path $baseDir "backup"
$templateFile = Join-Path $baseDir "template\blank_sjis.txt"

$today = Get-Date -Format "yyyyMMdd"
$backupDir = Join-Path $backupRootDir $today

# Shift-JIS
$sjis = [System.Text.Encoding]::GetEncoding(932)

if (!(Test-Path $inputDir)) {
    Write-Host "input フォルダが存在しません。"
    exit 1
}

if (!(Test-Path $templateFile)) {
    Write-Host "テンプレートファイルが存在しません: $templateFile"
    exit 1
}

New-Item -ItemType Directory -Force -Path $outputDir | Out-Null
New-Item -ItemType Directory -Force -Path $backupDir | Out-Null

$files = Get-ChildItem -Path $inputDir -File

if ($files.Count -eq 0) {
    Write-Host "input フォルダにファイルがありません。"
    exit 0
}

foreach ($file in $files) {

    $outputFile = Join-Path $outputDir $file.Name

    # テンプレートをコピーして出力ファイルを作成
    Copy-Item -Path $templateFile -Destination $outputFile -Force

    # UTF-8として読み込み
    $text = [System.IO.File]::ReadAllText($file.FullName, [System.Text.Encoding]::UTF8)

    # BOMがある場合は削除
    $text = $text.TrimStart([char]0xFEFF)

    # Shift-JISで書き込み
    [System.IO.File]::WriteAllText($outputFile, $text, $sjis)

    # backup/yyyyMMdd に元ファイルを移動
    $backupFile = Join-Path $backupDir $file.Name

    if (Test-Path $backupFile) {
        $timestamp = Get-Date -Format "HHmmssfff"
        $name = [System.IO.Path]::GetFileNameWithoutExtension($file.Name)
        $ext  = [System.IO.Path]::GetExtension($file.Name)
        $backupFile = Join-Path $backupDir "${name}_${timestamp}${ext}"
    }

    Move-Item -Path $file.FullName -Destination $backupFile

    Write-Host "変換完了: $($file.Name)"
}

Write-Host "すべての処理が完了しました。"
Write-Host "出力先: $outputDir"
Write-Host "退避先: $backupDir"
Write-Host "input フォルダは残しています。"
```

---

## template/blank_sjis.txt

空ファイルでOKです。

ただし、Shift-JISの空ファイルとして保存しておく想定です。

---

## 使い方

1. `input` フォルダに変換したいファイルを置く
2. `convert.bat` をダブルクリックする
3. `output` フォルダに同じファイル名で変換後ファイルが出力される
4. 元ファイルは `backup/yyyyMMdd` に移動される
5. `input` フォルダ自体は残る
6. 処理後、`input` フォルダの中は空になる

---

## 処理後のイメージ

```text
convert_tool/
├─ input/
│  └─ 空
├─ output/
│  └─ 変換後ファイル.csv
└─ backup/
   └─ 20260525/
      └─ 元ファイル.csv
```

---

## 注意点

- 出力ファイル名は入力ファイル名と同じです
- BOMは削除されます
- 出力文字コードは Shift-JIS です
- 日本語ヘッダー行もそのまま変換されます
- `input` フォルダ自体は削除されません
- `input` フォルダ内のファイルは `backup/yyyyMMdd` に移動されます
- 同名ファイルが backup にある場合は、時刻付きで退避します









# UTF-8 → Shift-JIS 変換BAT  
# ヘッダー行削除版

## 処理内容

このBATは、`input` フォルダ内のファイルを読み込み、  
**1行目のヘッダー行を削除**したうえで、UTF-8 から Shift-JIS に変換します。

変換後のファイルは `output` フォルダへ出力します。

出力ファイル名は、入力ファイル名と同じです。

変換後、元ファイルは以下のフォルダへ移動します。

```text
backup/yyyyMMdd/
```

`input` フォルダ自体は削除しません。  
処理後、`input` フォルダの中は空になります。

---

## フォルダ構成

```text
convert_tool/
├─ convert_header_delete.bat
├─ convert_header_delete.ps1
├─ template/
│  └─ blank_sjis.txt
├─ input/
│  └─ 変換したいファイル.csv
├─ output/
└─ backup/
```

---

## convert_header_delete.bat

```bat
@echo off
setlocal

cd /d "%~dp0"

powershell -NoProfile -ExecutionPolicy Bypass -File "%~dp0convert_header_delete.ps1"

pause
endlocal
```

---

## convert_header_delete.ps1

```powershell
$ErrorActionPreference = "Stop"

$baseDir = Split-Path -Parent $MyInvocation.MyCommand.Path

$inputDir  = Join-Path $baseDir "input"
$outputDir = Join-Path $baseDir "output"
$backupRootDir = Join-Path $baseDir "backup"
$templateFile = Join-Path $baseDir "template\blank_sjis.txt"

$today = Get-Date -Format "yyyyMMdd"
$backupDir = Join-Path $backupRootDir $today

# Shift-JIS
$sjis = [System.Text.Encoding]::GetEncoding(932)

if (!(Test-Path $inputDir)) {
    Write-Host "input フォルダが存在しません。"
    exit 1
}

if (!(Test-Path $templateFile)) {
    Write-Host "テンプレートファイルが存在しません: $templateFile"
    exit 1
}

New-Item -ItemType Directory -Force -Path $outputDir | Out-Null
New-Item -ItemType Directory -Force -Path $backupDir | Out-Null

$files = Get-ChildItem -Path $inputDir -File

if ($files.Count -eq 0) {
    Write-Host "input フォルダにファイルがありません。"
    exit 0
}

foreach ($file in $files) {

    $outputFile = Join-Path $outputDir $file.Name

    # テンプレートをコピーして出力ファイルを作成
    Copy-Item -Path $templateFile -Destination $outputFile -Force

    # UTF-8として全行読み込み
    $lines = [System.IO.File]::ReadAllLines($file.FullName, [System.Text.Encoding]::UTF8)

    if ($lines.Count -eq 0) {
        Write-Host "空ファイルのためスキップ: $($file.Name)"
        continue
    }

    # 1行目のBOMを削除
    $lines[0] = $lines[0].TrimStart([char]0xFEFF)

    # 1行目のヘッダー行を削除
    $bodyLines = $lines | Select-Object -Skip 1

    # Shift-JISで書き込み
    [System.IO.File]::WriteAllLines($outputFile, $bodyLines, $sjis)

    # backup/yyyyMMdd に元ファイルを移動
    $backupFile = Join-Path $backupDir $file.Name

    if (Test-Path $backupFile) {
        $timestamp = Get-Date -Format "HHmmssfff"
        $name = [System.IO.Path]::GetFileNameWithoutExtension($file.Name)
        $ext  = [System.IO.Path]::GetExtension($file.Name)
        $backupFile = Join-Path $backupDir "${name}_${timestamp}${ext}"
    }

    Move-Item -Path $file.FullName -Destination $backupFile

    Write-Host "変換完了: $($file.Name)"
}

Write-Host "すべての処理が完了しました。"
Write-Host "出力先: $outputDir"
Write-Host "退避先: $backupDir"
Write-Host "input フォルダは残しています。"
```

---

## template/blank_sjis.txt

空ファイルでOKです。

ただし、Shift-JISの空ファイルとして保存しておく想定です。

---

## 使い方

1. `input` フォルダに変換したいファイルを置く
2. `convert_header_delete.bat` をダブルクリックする
3. 1行目のヘッダー行が削除される
4. UTF-8 から Shift-JIS に変換される
5. `output` フォルダに同じファイル名で出力される
6. 元ファイルは `backup/yyyyMMdd` に移動される
7. `input` フォルダ自体は残る
8. 処理後、`input` フォルダの中は空になる

---

## 処理後のイメージ

```text
convert_tool/
├─ input/
│  └─ 空
├─ output/
│  └─ 変換後ファイル.csv
└─ backup/
   └─ 20260525/
      └─ 元ファイル.csv
```

---

## 注意点

- 1行目のヘッダー行は削除されます
- 2行目以降だけが出力されます
- 出力ファイル名は入力ファイル名と同じです
- BOMは削除されます
- 出力文字コードは Shift-JIS です
- 日本語を含むヘッダー行でも削除されます
- `input` フォルダ自体は削除されません
- `input` フォルダ内のファイルは `backup/yyyyMMdd` に移動されます
- 同名ファイルが backup にある場合は、時刻付きで退避します