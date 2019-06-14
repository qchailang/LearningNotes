破解vscode extjs插件
安装好之后打开文件：LicenseManager.js（路径：你的用户名\.vscode\extensions\Sencha.vscode-extjs-版本号\out\src\LicenseManager.js）
注释掉：
::
 const licenses = this.manager.getProductLicenses()
                .filter(license => license.signature)
                .map(license => this.createLicenseObject(license));
            this.license = licenses.find(l => l.active && l.full) ||
                licenses.find(l => l.active) ||
                licenses[0]; // default to the first license
            if (!this.license || !this.license.active || !this.license.full) {
                statusBarActivation.show();
            }
加入
::
  this.license ={
    active:true,
    full:true,
    data:{}
}

保存后重启Visual Studio Code．