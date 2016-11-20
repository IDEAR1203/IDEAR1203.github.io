---
title: FlowDroid工具的构建与运行
layout: post
---

### 题外话

今天早上收到一封邮件，从我的博客联系到我，问我如何运行flowdroid，着实感到开心。这是我博客搭建起来之后第一次有人联系到我，感觉自己可以做一点事情很开心。本文是以我对该邮件的回复为基础加以整理的。

### 回复邮件正文

您好。

我是从读了这篇论文才开始接触android的，所以对android基础知识、运行机制乃至环境搭建认识非常浅薄。

但我确实把flowdroid运行起来了。

参考资料是官方文档：<https://github.com/secure-software-engineering/soot-infoflow-android/wiki>

该文档对flowdroid的构建分为两节，Obtaining the nightly builds和Building FlowDroid From Source。我都进行了尝试。前者是使用事先构建好的flowdroid运行所必须的jar包，后者是从源码层面自行构建。

采用Obtaining the nightly builds方法时我遇到了android SDK某特定版本找不到的错误，由于对android了解甚少，再者我想从代码层面了解flowdroid，因而我直接放弃了这种方式。

Building FlowDroid From Source。首先从google下载android SDK。然后根据该小节所说的，把`Jasmin`、`Heros`、`Soot`、`soot-infoflow`、`soot-infoflow-android`源码下载下来放在同一个目录下。用eclipse导入已有工程，选择工程的根目录。

导入后效果如下图：

![import project]( {{ site.baseurl }}/images/run-flowdroid/1.png )

然后，你可以在/soot-infoflow-android/test包下找到对droidBench、insecureBank的Junit测试。例如可以尝试运行`soot.jimple.infoflow.android.test.droidBench.CallbackTests.runTestAnonymousClass1()`方法来测试flowdroid在droidbench上的运行效果。

但运行会报错。1是找不到android sdk，2是找不到droidbench所在目录，3找不到`EasyTaintWrapperSource.txt`。通过查看`soot.jimple.infoflow.android.test.droidBench.JUnitTests.analyzeAPKFile(String, boolean)`方法的源码可以找到我们需要配置两个环境变量：`ANDROID_JARS`和`DROIDBENCH`。前者是android.jar所在的目录，该目录在android sdk下，目录结构大概是`Android/sdk/platforms/android-23/android.jar`这样的。后者是droidbench测试工程集的目录。我是OS X Yosemite系统，在家目录下的`.bash_profile`下做相应配置并使之生效。其他系统的环境变量配置也不难。至于`EasyTaintWrapperSource.txt`文件，我发现soot-infoflow项目中存在这个文件，硬拷贝到soot-infoflow-android工程中的，不知道有没有更好的方法。

![environment]( {{ site.baseurl }}/images/run-flowdroid/2.png)

至此，我已经可以运行flowdroid了。

除了官方给的一些测试集合，也可以用我们自己写的一些apk来测试flowdroid。例如我将论文里的示例代码进行了完善，并用flowdroid进行了测试。主体代码如下：

```java
package com.example.wangdongwei.myapplication;

import android.app.Activity;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.EditText;

public class LeakageApp extends Activity {
    private User user = null;
    @Override
    protected void onRestart(){
        super.onRestart();
        EditText usernameText =
                (EditText)findViewById(R.id.username);
        EditText passwordText =
                (EditText)findViewById(R.id.pwdString);
        String uname = usernameText.toString();
        String pwd = passwordText.toString();
        if(!uname.isEmpty() && !pwd.isEmpty())
            this.user = new User(uname, pwd);
    }
    //Callback method in xml file
    public void sendMessage(View view){
        if(user == null) return;
        Password pwd = user.getpwd();
        String pwdString = pwd.getPassword();
        String obfPwd = "";
        //must track primitives
        for(char c : pwdString.toCharArray())
            obfPwd += c + "_";
        String message = "User: " +
                user.getName() + " | PWD: " + obfPwd;
        SmsManager sms = SmsManager.getDefault();
        sms.sendTextMessage("+44 020 7321 0905",
                null, message, null, null);
    }

        @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_leakage_app);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_leakage_app, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
}
```

然后打包成leakageapp.apk，放在`/soot-infoflow-android/testAPKs`包下。在`/soot-infoflow-android/test/soot/jimple/infoflow/android/test/otherAPKs/OtherAPKTests.java`中写了Junit测试方法：

```java
@Test
public void runTest3() throws IOException, XmlPullParserException {
  InfoflowResults res = analyzeAPKFile
  ("testAPKs/leakageapp.apk", false, false, false);
  Assert.assertNotNull(res);
  Assert.assertTrue(res.size() > 0);
}
```

方法上右键->Run As->JUnit Test进行单元测试。结果如下：

```bash
[main] INFO soot.jimple.infoflow.entryPointCreators.AndroidEntryPointCreator - Generated main method:
    public static void dummyMainMethod(java.lang.String[])
    {
        java.lang.String[] $r0;
        int $i0;
        com.example.wangdongwei.myapplication.LeakageApp $r1;
        android.os.Bundle $r2;
        boolean $z0, $z1;
        android.view.View $r3;

        $r0 := @parameter0: java.lang.String[];

        $i0 = 0;

     label1:
        if $i0 == 0 goto label8;

        $r1 = new com.example.wangdongwei.myapplication.LeakageApp;

        specialinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: void <init>()>();

        if $i0 == 1 goto label8;

        $r2 = new android.os.Bundle;

        specialinvoke $r2.<android.os.Bundle: void <init>()>();

        virtualinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: void onCreate(android.os.Bundle)>($r2);

        $r2 = null;

     label2:
        if $i0 == 2 goto label7;

     label3:
        if $i0 == 3 goto label4;

        $z0 = virtualinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: boolean onCreateOptionsMenu(android.view.Menu)>(null);

     label4:
        if $i0 == 4 goto label5;

        $z1 = virtualinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: boolean onOptionsItemSelected(android.view.MenuItem)>(null);

     label5:
        if $i0 == 5 goto label6;

        $r3 = new android.view.View;

        specialinvoke $r3.<android.view.View: void <init>(android.content.Context)>($r1);

        virtualinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: void sendMessage(android.view.View)>($r3);

        $r3 = null;

     label6:
        if $i0 == 6 goto label3;

     label7:
        if $i0 == 7 goto label2;

        if $i0 == 8 goto label8;

        virtualinvoke $r1.<com.example.wangdongwei.myapplication.LeakageApp: void onRestart()>();

        if $i0 == 9 goto label2;

     label8:
        if $i0 == 11 goto label1;

        return;
    }

[Call Graph] For information on where the call graph may be incomplete, use the verbose option to the cg phase.
[Spark] Pointer Assignment Graph in 0.0 seconds.
[Spark] Type masks in 0.0 seconds.
[Spark] Pointer Graph simplified in 0.0 seconds.
[Spark] Propagation in 0.0 seconds.
[Spark] Solution found in 0.0 seconds.
[main] INFO soot.jimple.infoflow.codeOptimization.InterproceduralConstantValuePropagator - Removing side-effect free methods is disabled
[main] INFO soot.jimple.infoflow.Infoflow - Dead code elimination took 0.028132 seconds
[main] INFO soot.jimple.infoflow.Infoflow - Callgraph has 112 edges
[main] WARN soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Static field tracking is disabled, results may be incomplete
[main] WARN soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Using flow-insensitive alias tracking, results may be imprecise
[main] INFO soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Implicit flow tracking is NOT enabled
[main] INFO soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Exceptional flow tracking is enabled
[main] INFO soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Running with a maximum access path length of 5
[main] INFO soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Using path-agnostic result collection
[main] INFO soot.jimple.infoflow.android.InfoflowAndroidConfiguration - Recursive access path shortening is enabled
[main] INFO soot.jimple.infoflow.Infoflow - Looking for sources and sinks...
[main] INFO soot.jimple.infoflow.Infoflow - Source lookup done, found 5 sources and 1 sinks.
[main] INFO soot.jimple.infoflow.Infoflow - Taint wrapper hits: 52
[main] INFO soot.jimple.infoflow.Infoflow - Taint wrapper misses: 61
[main] INFO soot.jimple.infoflow.Infoflow - IFDS problem with 452 forward and 30 backward edges solved, processing 1 results...
[main] INFO soot.jimple.infoflow.data.pathBuilders.ContextInsensitiveSourceFinder - Obtainted 1 connections between sources and sinks
[main] INFO soot.jimple.infoflow.data.pathBuilders.ContextInsensitiveSourceFinder - Building path 1
[main] INFO soot.jimple.infoflow.data.pathBuilders.ContextInsensitiveSourceFinder - Path processing took 0.003869 seconds in total
[main] INFO soot.jimple.infoflow.Infoflow - The sink virtualinvoke $r8.<android.telephony.SmsManager: void sendTextMessage(java.lang.String,java.lang.String,java.lang.String,android.app.PendingIntent,android.app.PendingIntent)>("+44 020 7321 0905", null, $r5, null, null) on line 38 in method <com.example.wangdongwei.myapplication.LeakageApp: void sendMessage(android.view.View)> was called with values from the following sources:
[main] INFO soot.jimple.infoflow.Infoflow - - $r1 = virtualinvoke $r0.<com.example.wangdongwei.myapplication.LeakageApp: android.view.View findViewById(int)>(2131492871) on line 19 in method <com.example.wangdongwei.myapplication.LeakageApp: void onRestart()>
Maximum memory consumption: 526.43924 MB
```

仔细看一下，该结果是说从findViewById到sendTextMessage的source-sink关系使该程序存在taint的风险。

但其实，对于EditText的对象passwordText,直接调用toString是无法得到其文本内容的（正确方式应该是passwordText.getText().toString()），所以这算不算是一个误报呢？这是我没有搞明白的一个点。

我大约是三周前看的这篇论文，因此叙述会有一些疏漏，敬请谅解。

### 原始邮件

> 你好~刚刚看了你对flowdroid的文献理解，最近我也开始进行android动态和静态的分析，不知道是否方便可以交流一二？比如flowdroid，我觉得他网上给的无论是虚拟机下例子还是github下的例子，不知道你是如何跑成的?把所有都扔在一个目录下，然后安装上droidbench进行跑就ok了么？
