## Native Build Quarkus
### 1. 何謂 Native Build 
所謂的 Native Build 指的是說我們可以把 Application 打包成類似 ``.exe`` 檔案，不需要依靠開啟 JVM 來執行，這整個 Application 已經包含了要運行所需要的環境 (包含已經縮小化的 JVM ) 與檔案了，採用 Native 運行可以降低啟動時間、 記憶體消耗且可最小化 disk footprint，但缺點是需要花較長的時間建立。

### 2. 需要的套件
1. GraalVM : https://www.graalvm.org/downloads/
    - 於官方網站下載檔案
    - 解壓縮檔案並放置於任意路徑 (自己要記得)
    - 將 ``GRAALVM_HOME`` 此變數設定到環境變數中，其值為放置GraalVM的路徑。
    <img src="https://github.com/coco40725/Kotlin_Note/blob/main/%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A/img/1.png?raw=true" width=70%>

    <img src="https://github.com/coco40725/Kotlin_Note/blob/main/%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A/img/2.png?raw=true" width=70%>

    - 最後在 cmd 打 ``gu `` 測試該指令是存在的

2. native-image
    - GraalVM 安裝完畢後，接著可直接安裝 native-image
    - 開啟 cmd
    - cd 至$GRAALVM_HOME/bin
    - 在 cmd 打 ``gu install native-image``
    - 在 cmd 打 ``native-image --version``，有出現版本代表成功

### 3. 於 Quarkus Build Native (win10 目前失敗)
- 開啟 terminal 
- cd 至 專案下
- ``./gradlew build "-Dquarkus.package.type=native"``
 (參考 https://github.com/quarkusio/quarkus/issues/22320)，但只能build到一半

錯誤結果:
```kotlin
io.quarkus.builder.BuildException: Build failure: Build failed due to errors
        [error]: Build step io.quarkus.deployment.pkg.steps.NativeImageBuildStep#build threw an exception: io.quarkus.deployment.pkg.steps.NativeImageBuildStep$ImageGenerationFailureException: Image generation failed. Exit code: 1
        at io.quarkus.deployment.pkg.steps.NativeImageBuildStep.imageGenerationFailed(NativeImageBuildStep.java:422)
        at io.quarkus.deployment.pkg.steps.NativeImageBuildStep.build(NativeImageBuildStep.java:263)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:568)
        at io.quarkus.deployment.ExtensionLoader$3.execute(ExtensionLoader.java:909)
        at io.quarkus.builder.BuildContext.run(BuildContext.java:281)
        at org.jboss.threads.ContextHandler$1.runWith(ContextHandler.java:18)
        at org.jboss.threads.EnhancedQueueExecutor$Task.run(EnhancedQueueExecutor.java:2449)
        at org.jboss.threads.EnhancedQueueExecutor$ThreadBody.run(EnhancedQueueExecutor.java:1478)
        at java.base/java.lang.Thread.run(Thread.java:833)
        at org.jboss.threads.JBossThread.run(JBossThread.java:501)
```
- 可能解決方案:
    1. 修改 Docker 記憶體大小: (https://stackoverflow.com/questions/72954579/quarkus-native-image-building-throws-error-image-generation-failed)
        - Increase the Docker memory resource from default 2GB to 8GB. (Docker Desktop -> Settings icon on top right -> Resources -> memory 2GB to 8GB apply and restart.)

        - Then run the command mvn clean install -Dnative -Dquarkus.native.container-build=true