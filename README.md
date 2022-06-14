# SQL Server 2019 in Docker, under Ubuntu 22.04 WSL in Windows


## Docker Setup
Go read [Felipe's Guide](https://dev.to/felipecrs/simply-run-docker-on-wsl2-3o8)


## SQL Server

1. Make a data folder
    ```bash
    mkdir -p ./mssql/data
    mkdir ./mssql/log
    ```

2. Write yourself a environment file (here env.local) with the SQL password in:
    ```bash
    SA_PASSWORD=something more secure than this
    ```

3. Run the dockerfile with 
    ```bash
    docker-compose --env-file ./env.local up -d
    ```

4. This will fail, due to the PID of SQL Server not being owner of the data folder above. So have a look what it is:
    ```bash
    docker run -it  mcr.microsoft.com/mssql/server:2019-latest id mssql 
    ```
It's probably 1001

5. Bring down SQL Server
    ```bash
    docker-compose --env-file ./env.local down
    ```

6. And give ownership to SQL Server:
    ```bash
    sudo chown 10001 mssql/log
    ```
7. Bring it up again
    ```bash
    docker-compose --env-file ./env.local up -d
    ```

## ODBC
If you want to test this locally within the container, Microsoft has ODBC [installation instructions](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver16), but they're no good at the time of writing for Ubuntu LTS 22.04. So [AskUbuntu](https://askubuntu.com/questions/1407533/microsoft-odbc-v18-is-not-find-by-apt) has the solution:

```bash
sudo apt-get install odbcinst
sudo curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/21.10/prod impish main" | sudo tee /etc/apt/sources.list.d/mssql-release.list
sudo apt update
sudo ACCEPT_EULA=Y apt-get install msodbcsql18
sudo ACCEPT_EULA=Y apt-get install -y mssql-tools18
```


### Noice
```
sqlcmd -U sa -P ******** -S localhost -d master -C -Q "SELECT @@VERSION"
-------------------------------------------------------------------------
Microsoft SQL Server 2019 (RTM-CU16) (KB5011644) - 15.0.4223.1 (X64)
        Apr 11 2022 16:24:07
        Copyright (C) 2019 Microsoft Corporation
        Developer Edition (64-bit) on Linux (Ubuntu 20.04.4 LTS) <X64>
```

`-C` is to trust the self signed cert that the container uses, check the appropriate box in Management Studio connect or pass `TrustServerCertificate=True` in a **development** connection string

## Connect
You'll need the IP of the WSL instance to connect from Windows

From inside WSL bash:
```bash
ip -4 addr show eth0
6: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    inet 172.24.177.8/20 brd 172.24.191.255 scope global eth0
       valid_lft forever preferred_lft forever
```

or from Windows PowerShell

```powershell
wsl -- ip -o -4 -json addr list eth0 `
| ConvertFrom-Json `
| %{ $_.addr_info.local } `
| ?{ $_ }
```
