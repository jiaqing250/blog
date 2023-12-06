### LicenseServer

- Install as a jetbrains plugin
- (Ignore if configured)Download[ja-netfilter](https://gitee.com/ja-netfilter/ja-netfilter),then configure javaagent to your ide vmOptions file.
- Open Init License server Action(ctrl+shift+alt+m or in Help Menu) to (important)[Add power param] (Ignore if configured)[add url,dns param]<br />  you can use Add PowerParam method(in License server Action(ctrl+shift+alt+m)) to add power param automatically. You only need to select the power.conf path.then restart ide.
- License server(port may change): [http://localhost:63320/](http://localhost:63320/)
- Default Config params index: [http://localhost:63320/](http://localhost:63320/)
- Port config in Registry(ctrl+alt+shift+/)
- The paid plugin product code will be displayed in the Action (must be downloaded and installed), and this code can be configured according to the rules in `Settings->License server->Activation Code`
- Use generate code in action(Alt+m) to generate the activation code

PS.<br />configure power param only once, unless the power param changes in the file.

The biggest flexibility of this plugin is that you can customize the information in the key, so it means that you can generate all plugin information.

- 安装作为一个jetbrains插件
- (配置过可忽略)下载[ja-netfilter](https://gitee.com/ja-netfilter/ja-netfilter),然后配置javaagent到你的ide vmOptions文件.
- 打开Init License server Action(ctrl+shift+alt+m 或者在help菜单栏里) (重要)[添加power参数] (添加过可忽略)[url参数 dns参数]<br />      添加power参数可以用action(ctrl+shift+alt+m)中的Add PowerParam方法自动添加，只需要选择power.conf文件路径(一般在ja-netfilter/config中).然后重启ide
- 激活服务器地址(端口可能会变): [http://localhost:63320/](http://localhost:63320/)
- 默认配置网页: [http://localhost:63320/](http://localhost:63320/)
- 端口配置在Registry(ctrl+alt+shift+/)
- 在action中会显示付费的product code(必须是已经下载安装了的插件),将这个code按照`Settings->License server->Activation Code`中的规则配置即可
- 用action(Alt+m)中的generate code生成激活码

PS. 一次配置power永久有效,除非power参数在文件里变了.

这个插件的最大灵活性在于你可以自定义key中的信息，所以意味着你可以生成所有插件信息.

Rainbow brackets: [My Plugin](https://github.com/Nasller/plugin-myagent/releases/tag/v1.0.0) 

Download jar then put it to ja-netfilter/plugins folder(or ja-netfilter/plugins-appName folder if you set javaagent like: "-javaagent:/path/to/ja-netfilter.jar=appName"),Restart ide

下载上面的项目放到ja-netfilter/plugins文件夹中,重启ide(如果你是这么设置javaagent的: "-javaagent:/path/to/ja-netfilter.jar=appName" 那么插件目录为plugins-appName)

### 举个激活码激活的栗子🌰

- 点击IDE编辑器顶部的help选项卡 选择证书服务器

![](https://user-images.githubusercontent.com/54784104/215252036-7ca84830-6652-43e4-8dd4-418b61ea6b56.png#id=ElE7g&originHeight=153&originWidth=424&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 证书服务器弹窗大概长这样

![](https://user-images.githubusercontent.com/54784104/215252228-df532d59-bf18-4a65-9d53-3645883c3536.png#id=vq1vn&originHeight=203&originWidth=724&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- PRAINBOWBRACKET 把这个配置到设置的product里面。（彩虹括号）

![](https://user-images.githubusercontent.com/54784104/215252351-69d155fd-34a1-499d-a9b3-8201f60c1371.png#id=g8l3U&originHeight=875&originWidth=1238&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 然后再次打开证书服务器点Jetbrains Code。（复制许可证激活码）

![](https://user-images.githubusercontent.com/54784104/215252487-001a91de-4efc-4c06-a4df-5164d5560654.png#id=WYhSC&originHeight=300&originWidth=751&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 粘贴到插件许可证激活码框里

![](https://user-images.githubusercontent.com/54784104/215252624-22baca7f-5b3f-4a6b-b2c8-5117057c2b86.png#id=ZrhJt&originHeight=591&originWidth=982&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- ojbk 再看相应的插件许可证信息，激活成功😎

![](https://user-images.githubusercontent.com/54784104/215252730-5cbae8ad-88e7-45cb-9185-a516501ece14.png#id=BcXfT&originHeight=591&originWidth=982&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 其余需要许可证激活的插件也是按上面方式操作激活（激活码方式）<br />    只需配置完成所有需要激活的插件，复制一次许可证激活码即可用来激活所有的付费插件

jrebel插件

如果激活出现NO LICENSE FOUND字样，重装jrebel插件 或者 找到plugins\jr-ide-idea\lib\jrebel6这个插件安装目录下，一般会有两个文件jrebel.jar，jrebel-backup.jar。删掉jrebel.jar。再修改jrebel-backup.jar为jrebel.jar，重启服务

如果还出现这个字样那就找我974776824。

![](https://user-images.githubusercontent.com/26132153/218034763-54f12197-f3b1-4020-8319-c47b16d4ee36.png#id=VTBa8&originHeight=103&originWidth=326&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />需要的文件:

[LicenseServer-obfuscate-1.0.0.zip.pdf](https://www.yuque.com/attachments/yuque/0/2023/pdf/2923644/1689149896123-237a2a22-9b7b-45ff-a6bf-7292dd2647d3.pdf?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fpdf%2F2923644%2F1689149896123-237a2a22-9b7b-45ff-a6bf-7292dd2647d3.pdf%22%2C%22name%22%3A%22LicenseServer-obfuscate-1.0.0.zip.pdf%22%2C%22size%22%3A1547510%2C%22ext%22%3A%22pdf%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22u7599f93f-8eba-44b5-8268-746be502e4c%22%2C%22taskType%22%3A%22upload%22%2C%22type%22%3A%22application%2Fpdf%22%2C%22__spacing%22%3A%22both%22%2C%22id%22%3A%22u63164037%22%2C%22margin%22%3A%7B%22top%22%3Atrue%2C%22bottom%22%3Atrue%7D%2C%22card%22%3A%22file%22%7D)<br />需要删除.pdf后缀名<br />来源:<br />[GitHub - Nasller/LicenseServer](https://github.com/Nasller/LicenseServer)
