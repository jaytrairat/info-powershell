# info-powershell

## list file which extension is .myd and save only filename to ./list.txt

``` powershell
Get-ChildItem -Path . -Filter "*.myd" | ForEach-Object { $_.BaseName } | Out-File -File ./list.txt -Encoding UTF8
```

## copy myd myi and frm to mysql docker

```powershell
$containerId = docker ps -qf "name=sql-mysql-1"

Get-Content .\list.txt | ForEach-Object {
    docker exec -i $containerId mysql -uroot -proot -e "DROP DATABASE IF EXISTS $_;"
    docker exec -i $containerId mysql -uroot -proot -e "CREATE DATABASE IF NOT EXISTS $_;"
    docker cp "$_.myd" sql-mysql-1:/var/lib/mysql/$_
    docker cp "$_.myi" sql-mysql-1:/var/lib/mysql/$_
    docker cp "$_.frm" sql-mysql-1:/var/lib/mysql/$_
}
```
