# vs2015 配置

## 注释

```shell
1. 注释：先 CTRL + K，然后 CTRL + C 
2. 取消注释：先 CTRL + K ，然后 CTRL + U
```

## 加入外部动态链接库
```shell
1. 再项目右键 Properties -> VC++ Directories 中，对 Excutable Directories, Include Directories,
   Library Directories 设置对应的值
2. Properties -> Linker -> Input -> Additional Dependencies 中加入对应的动态链接库的文件名
```

## vs2015 更改现有的项目

```shell
  1. 更改 oldName.sln 为 NewName.sln , 并将 sln 同一层级的 oldName 文件夹 更名为 NewName 文件夹
  2. 将通过 notapad++ 打开  NewName.sln 文件，将 oldName 全部替换为 NewName
  3. 进入 Newname 文件夹, 将 oldName.vcxproj , oldName.vcxproj.filters, oldName.vcxproj.user 都更名为
     NewName.vcxproj , NewName.vcxproj.filters, NewName.vcxproj.user, 
     打开 NewName.vcxproj , NewName.vcxproj.filters, NewName.vcxproj.use， 将 oldName 全部替换为 NewName
  4. 需要重新在 Configuration Properties -> c/c++ -> General 的 Additional Include Directories 与老的项目比较
     
  如果是动态链接库
  注意对外的头文件
      #if (defined _WIN32 || defined _WIN64)
      #   ifdef DRIVER_FIRE_ANALYZER_EXPORTS
      #       define IVMS_EXTERN extern "C" __declspec(dllexport)
      #   else
      #       define IVMS_EXTERN extern "C" __declspec(dllimport)
      #   endif
      #   define IVMS_API __stdcall
      #else
      #   ifdef __linux__
      #       define IVMS_EXTERN extern "C"
      #   else
      #       define IVMS_EXTERN
      #   endif
      #   define IVMS_API
      #endif
      中 DRIVER_FIRE_ANALYZER_EXPORTS 与 NewName.vcxproj 中得 DRIVER_FIRE_ANALYZER_EXPORTS 一一对应
```