# github配置ssh密钥
到`~/.ssh`找到`id_dsa.pub`文件，内涵本机公钥
```bash
$ cd ~/.ssh
$ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```
通过`cat`命令打印`id_dsa.pub`
```bash
$cat id_dsa.pub
```
将公钥拷贝到[github配置网站](https://github.com/settings/keys)
点击`New SSH key`将公钥导入。