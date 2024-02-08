# info-powershell

## list file which extension is .myd and save only filename to ./list.txt

```powershell
Get-ChildItem -Path . -Filter "*.myd" | ForEach-Object { $_.BaseName } | Out-File -File ./list.txt -Encoding UTF8
```

## copy myd myi and frm to mysql docker

```powershell
$containerId = docker ps -qf "name=sql-mysql-1"

Get-Content .\list.txt | ForEach-Object {
    docker exec -i $containerId mysql -uroot -proot -e "DROP DATABASE IF EXISTS $_;" 2>$null
    docker exec -i $containerId mysql -uroot -proot -e "CREATE DATABASE IF NOT EXISTS $_;" 2>$null
    docker cp "$_.myd" sql-mysql-1:/var/lib/mysql/$_
    docker cp "$_.myi" sql-mysql-1:/var/lib/mysql/$_
    docker cp "$_.frm" sql-mysql-1:/var/lib/mysql/$_
}
```

## reinit database with table
```powershell
$containerName = "upbeat_cerf"
$containerId = docker ps -qf "name=$containerName"
docker exec $containerId mkdir -p sql

Get-Content .\list.txt | ForEach-Object {
    $dbNameStripped = $_.Substring(0, [Math]::Min(30, $_.Length))
    docker exec -i $containerId mysql -uroot -proot -e "DROP DATABASE IF EXISTS $dbNameStripped;" 2>$null
    docker exec -i $containerId mysql -uroot -proot -e "CREATE DATABASE IF NOT EXISTS $dbNameStripped;" 2>$null
    docker cp "$_.sql" "${containerName}:/sql/$dbNameStripped.sql"

    $tableNames = @()
    $columnNames = @()

    Get-Content -Path "$_.sql" | ForEach-Object {
        if ($_ -match "^INSERT INTO") {
            $pattern = "INSERT INTO (\w+)\s*\((.*?)\)"
            if ($_ -match $pattern) {
                $tableName = $matches[1]
                $tableNames += $tableName

                $columns = $matches[2] -split ','
                $columns = $columns | ForEach-Object { $_.Trim() }
                $columnNames += $columns
            }
        }
    }

    $tableNames = $tableNames | Select-Object -Unique
    $columnNames = $columnNames | Select-Object -Unique

    foreach ($tableName in $tableNames) {
        $columns = ($columnNames -join ' VARCHAR(255), ')
    
        $command = "docker exec -i $containerId mysql -uroot -proot -e `"use $dbNameStripped; create table $tableName ($columns VARCHAR(255))`""
        Invoke-Expression $command 2>$null
    }
    
    docker exec -i $containerId mysql -uroot -proot -e "use $dbNameStripped;source /sql/$dbNameStripped.sql" 
    "copy and create database name $dbNameStripped"
}

"finish!"
```
