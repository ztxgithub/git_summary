# 工具简介

## source insight 
``` shell
    1. 破解版 source insight 4.0.exe 替换安装路径下的source insight 4.0.exe，然后运行SI4，
       在弹出的对话框中选择第三项并将下载的文件 si4.pediy.lic选中并“Next”即可破解.
       
    2. source insight 完全卸载
            (1)、清除注册表信息：
                “win ”+ R  或者  “开始” -> “运行”，输入“regedit”，回车；
                在弹出的注册表管理器中，选择“编辑”-> “查找”->“source insight”，
                或按照下述路径展开：HKEY_CURRENT_USER -> software -> Source Dynamics -> Source Insight;
                将该项下面的source insight 需要清除的对应版本项目选中，右键“删除“。
            (2)、删除全局配置信息：
                在 ./user/document/source insight 3.0/4.0 下的所有文件及该文件夹
                  注意此处的路径可能不同  也可能是:“库”->“用户”(也可能是你的名字) -> 文档 -> source insight3.0/4.0
                  或者  你上次安装的时候所指定的其他位置
    3. 未正确显示 Project Windows 窗口
            在 View 菜单下找到 Panels 级联菜单下的 Project Windows，把它勾选上, 此时工程文件的列表框就正确显示出来
```