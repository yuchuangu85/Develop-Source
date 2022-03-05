<h1 align="center">Gradle-Android</h1>



## 拷贝

```shell
subprojects {
    gradle.projectsEvaluated {
        tasks.withType(JavaCompile) {
            // 放在项目的根目录下
            options.compilerArgs.add('-Xbootclasspath/p:../android.jar')
        }
    }
    def releaseTasks = project.getTasksByName("assembleSmartcom_adult", false)
    def debugTasks = project.getTasksByName("assembleDebug", false)
    copyApkFile(releaseTasks, project)
    copyApkFile(debugTasks, project)
}

def copyApkFile(Set<Task> tasks, Project project) {
    for (task in tasks) {
        // 获取项目根目录，例如：app_health
        def dir = project.getProjectDir().getAbsolutePath()
        println "assemble release dir = $dir"
//        if (file.exists()) {
//            def var = delete(dir + "/build/outputs/apk")
//            def var1 = delete(rootProject.getRootDir().getAbsolutePath() + "/outputs/" + project.getName())
//            println "clear before create $var , $var1 -------"
//        }

        task.doLast {
            println("====: start copy apk to outputs fold")
            // apk输入目录，例如：app_health\build\outputs\apk\smartcom_adult
            // 这个文件夹下有两个文件夹：debug，release
            def fold = new File(dir + "/build/outputs/apk/smartcom_adult/")
            // 获取子文件夹：debug和release文件夹
            def folds = fold.listFiles()
            if (folds != null) {
                println("====: folds:  ${folds.size()}")
                // 遍历文件夹查找文件
                for (foldApk in folds) {
                    if (foldApk != null && foldApk.exists()) {
                        // 要拷贝的apk文件
                        def copyFile = null
                        def copyFileList = foldApk.listFiles()
                        for (file in copyFileList) {
                            if (file != null && file.exists() && file.getAbsolutePath().endsWith(".apk")) {
                                copyFile = file
                            }
                        }
                        if (copyFile != null && copyFile.exists()) {
                            // 对应debug或者release文件
                            def filePath = "smartcom_adult_TM_release"
                            if (copyFile.getAbsolutePath().contains("debug")) {
                                filePath = "smartcom_adult_TM_debug"
                            }
                            copy {
                                def fromPath = copyFile.getAbsolutePath()
                                println("====from: ${fromPath}")
                            }
                        }
                    }
                }
            }
        }
    }
}
```

