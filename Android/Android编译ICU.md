# ICU
ICU是International Component for Unicode的简称，是一套稳定成熟、功能强大、轻便易用和跨平台支持Unicode的开发包。ICU使得开发人员在C/C++和Java上开发全球化软件产品更容易，ICU是由IBM发布和维护，并且是开放源代码的。

ICU可以根据客户端的语言环境给客户返回最接近语言的字符串，也就是说客户端可能与服务器端的语言环境不一致，不能只根据服务器端的语言来返回字符串。而且将来单独增加或维护资源文件，不需要重新生成可执行文件或动态链接库。为了提高重用性，最好将所有资源信息统一管理，不是每个模块各自维护管理。

ICU的官方地址是：http://site.icu-project.org/

## Android下编译
在Android中，ICU的地址是 external/icu/icu4c/source/，修改完成后需要在该目录下编译并且替换icudtXXX.dat文件。具体步骤如下：
``` shell
执行icuConfigureRun命令，生成make文件:
~/android/external/icu/icu4c/source/# ./runConfigureICU Linux
执行make命令：
~/android/external/icu/icu4c/source/# make INCLUDE_UNI_CORE_DATA=1
替换dat文件：
~/android/external/icu/icu4c/source/# cp -rf data/out/tmp/icudt58l.dat stubdata/
生成新的系统镜像：
~/android/# make clean
~/android/# make -j8
```
