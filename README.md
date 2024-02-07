# info-powershell

## list file which extension is .myd and save only filename to ./list.txt

``` powershell
Get-ChildItem -Path . -Filter "*.myd" | ForEach-Object { $_.BaseName } | Out-File -File ./list.txt -Encoding UTF8

```
