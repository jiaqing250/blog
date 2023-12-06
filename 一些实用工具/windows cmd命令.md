### 1、查看本机已连接的wifi密码
```javascript
foreach ($item in (netsh wlan show profile|where{ $_ -match "文件"})){netsh wlan show profile name=($item -replace "    所有用户配置文件 : ","") key=clear|where{ $_ -match "    名称|关键内容"}}
```
