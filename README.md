# 编译记录

在编译ROS的package，使用`catkin build`时，经常出现warning，提示"/usr/lib/x86_64-linux-gnu"下的libopencv_core.so.3.2.0与目前安装的opencv_xxx.so.x.x.x冲突。一直没有在意。直到今天，在一个全新工控机安装了ros-melodic后又安装了opencv330，搜索opencv的lib时发现在/usr/lib/x86_64-linux-gnu路径下有opencv320的内容，手贱删掉了。然后ROS的编译就fail。

经查，ros-melodic安装时自带了opencv320的内容，主要是cv_bridge这个模块需要依赖opencv_core/imgproc/imgcodes这几个，就安装在那个路径下。我手贱删了，自然编译不过去。

那怎么办，自然想到了重新编译cv_bridge.so这个库。

参考链接：https://blog.csdn.net/qinqinxiansheng/article/details/120219388

其中的方法2是修改ros cv_bridge的cmake，查找最新的opencv。
```bash
sudo vim /opt/ros/kinetic/share/cv_bridge/cmake/cv_bridgeConfig.cmake
# 修改关于3.2.0的描述
set(libraries "cv_bridge;/usr/local/lib/libopencv_core.so.3.2.0;/usr/local/lib/libopencv_imgproc.so.3.2.0;/usr/local/lib/libopencv_imgcodecs.so.3.2.0")
```
修改完，可以成功编译了，但运行时还报错。毕竟cv_bridge编译时依赖的是320，而不是现在的330。所以只好重新编译源码。

## 编译源码
下载这个repo，cmake 三连编译。  
编译时，遇到了找不到`boost_python37`这个以来，我瞎jb改了CMakeLists里面的Find，改成当前电脑安装的（find一下）boost_python版本就行。

这时候编译时会提示，找到的opencv版本是3.3.0。编辑即可。

然后替换：/opt/ros/melodic/lib/cv_bridge.so这个文件，换成新编译的cv_bridge。完美解决问题。

特此记录！
