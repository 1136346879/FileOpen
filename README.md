"# FileOpen"

## 说明

> * 基于腾讯浏览服务，支持多种文件格式，例如doc、excel、ppt、excel、pdf等格式；
> * 支持展示网络文件
> * 支持测试本demo之前先把test文件夹里的文件复制到手机存储，方便测试，路径如下：


## TBS服务接入

参考腾讯TBS官网，地址：https://x5.tencent.com/tbs/guide/sdkInit.html

## 效果图(后面补充)


## 常见问题
- no suport by

出现这个错误提示的原因，首先可能是手机上没有Tbs内核，如果有tbs内核，则可能是内核正在初始化安装，还处于冷启动阶段，这个时候内核还不能使用，打开文件会出现这个错误，可以使用可以按如下方式确定内核是否成功加载并且可用：
方法1：
```
  QbSdk.initX5Environment(this, new QbSdk.PreInitCallback() {
            @Override
            public void onCoreInitFinished() {

            }

            @Override
            public void onViewInitFinished(boolean b) {
              //这里被回调，并且b=true说明内核初始化并可以使用
              //如果b=false,内核会尝试安装，你可以通过下面监听接口获知
            }
        });

       QbSdk.setTbsListener(new TbsListener() {
           @Override
           public void onDownloadFinish(int i) {
              //tbs内核下载完成回调
          }

           @Override
           public void onInstallFinish(int i) {
              //内核安装完成回调，
          }

           @Override
           public void onDownloadProgress(int i) {
                //下载进度监听
           }
       });
```
方法2：
```
QbSdk.preInit(this, new QbSdk.PreInitCallback() {
            @Override
            public void onCoreInitFinished() {

            }

            @Override
            public void onViewInitFinished(boolean b) {

            }
        });
        //tbs内核下载跟踪
        QbSdk.setTbsListener(this.tbsListener);
        //判断是否要自行下载内核
        boolean needDownload = TbsDownloader.needDownload(this, TbsDownloader.DOWNLOAD_OVERSEA_TBS);
        if (needDownload && isNetworkWifi(this)) {
        //isNetworkWifi(this)是我
        //自己写的一个方法，这里我也希望wifi下再下载
            TbsDownloader.startDownload(this);
        }
```
android项目接使用 TBS X5 框架时问题记录（文档加载不出空白问题解决）
最近开发遇到一个棘手的问题
我打开文档后里面的内容变成空白了是怎么回事？
不知到为何，最后跟踪项目代码
发现测试整的这个文档是qq应用内部的文件
我这边打开首先是根据是否有本地路径如有直接打开
如没有，再去根据网络地址在线文档打开
发现出现一个错误，没有权限访问该问题，我的解决法案有两种

1，将其文件copy到我们app可以访问的路径下，直接打开即可
2，直接在线打开（在线打开，也是先把文档下载到本地然后打开）

过了一阵子后，在9.0手机上发现又出现空白问题
这个问题你不注意很难找到原因
android项目接入Tbs-实现项目内部打开office在线文档并解决

最后发现
我们的 x5webview 内核初始化 放在了application中
耳此时应用是没有任何全权限的，导致x5内核初始化失败，
而失败后，你可能不会知道，因为失败后，X5回自动给你切换到系统的webview内核，你也可以顺利的打开webview，你以为一直用的是X5内核，呵呵，其实你懂的
最后发现应用的存储权限没有
导致 x5webview 初始化失败
打开在线文档就会出现空白

解决方法：

X5webview初始化 在splashActivity中
同意存储权限之后就初始化X5内核，就不会出现空白的问题了

  RxPermissions(this)
                    .request(Manifest.permission.WRITE_EXTERNAL_STORAGE,
                            Manifest.permission.READ_EXTERNAL_STORAGE)
                    .subscribe {

                            initX5Webview()
                    }

  /**
     * x5初始化
     */
    private fun initX5Webview(){
        try {
            QbSdk.initX5Environment(applicationContext, object : QbSdk.PreInitCallback {
                override fun onCoreInitFinished() {
                    val map = mutableMapOf<String, Any>()
                    map[TbsCoreSettings.TBS_SETTINGS_USE_SPEEDY_CLASSLOADER] = true
                    QbSdk.initTbsSettings(map)
                    Log.e("x5--","Both success and failure are called back")
                }

                override fun onViewInitFinished(p0: Boolean) {
                    Log.e("x5--","Load the kernel successfully -- $p0")
                }
            })
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
————————————————

## 博客   https://blog.csdn.net/wdx_1136346879/article/details/103640920
