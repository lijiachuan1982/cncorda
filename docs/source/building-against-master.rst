使用 non-release 分支构建 CorDapp
==============================================

It is advisable to develop CorDapps against the most recent Corda stable release. However, you may need to build a CorDapp 
against an unstable non-release branch if your CorDapp uses a very recent feature, or you are using the CorDapp to test a PR 
on the main codebase.

当开发一个 CorDapp 的时候，我们建议使用最新的 Corda stable release，因为被很好的测试过。但是，如果你的 CorDapp 使用了最新的功能的话，你可能需要基于一个非稳定的 non-release 分支来构建你的 CorDapp，或者你是基于主要的 codebase 使用 CorDapp 来测试一个 PR。

To work against a non-release branch, proceed as follows:

1. Clone the `Corda repository <https://github.com/corda/corda>`_

2. Check out the branch or commit of the Corda repository you want to work against

3. Make a note of the ``gradlePluginsVersion`` in the root ``constants.properties`` file of the Corda repository
    
4. Clone the `Corda Gradle Plugins repository <https://github.com/corda/corda-gradle-plugins>`_

5. Check out the tag of the Corda Gradle Plugins repository corresponding to the ``gradlePluginsVersion``

6. Follow the instructions in the readme of the Corda Gradle Plugins repository to install this version of the Corda Gradle plugins locally

7. Open a terminal window in the folder where you cloned the Corda repository

8. Publish Corda to your local Maven repository using the following commands:

  * Unix/Mac OSX: ``./gradlew install``
  * Windows: ``gradlew.bat install``

  .. warning:: If you do modify your local Corda repository after having published it to Maven local, then you must
     re-publish it to Maven local for the local installation to reflect the changes you have made.

  .. warning:: As the Corda repository evolves on a daily basis, two clones of an unstable branch at different points in
     time may differ. If you are using an unstable release and need help debugging an error, then please let us know the
     **commit** you are working from. This will help us ascertain the issue.
     
9. Make a note of the ``corda_release_version`` in the root ``build.gradle`` file of the Corda repository

10. In your CorDapp's root ``build.gradle`` file:

    * Update ``ext.corda_release_version`` to the ``corda_release_version`` noted down earlier
    * Update ``corda_gradle_plugins_version`` to the ``gradlePluginsVersion`` noted down earlier

按照下边的步骤使用 non-release 分支：

1. 克隆 `Corda repository <https://github.com/corda/corda>`_
2. Check out 你想要工作的 Corda repository 的分支
3. 记录一下在 Corda repository 的 ``constants.properties`` 的根 ``gradlePluginsVersion``
4. 克隆 `Corda Gradle Plugins repository <https://github.com/corda/corda-gradle-plugins>`_
5. Check out 相对于 ``gradlePluginsVersion`` 的 Corda Gradle Plugins repository 的 tag
6. 跟随 Corda Gradle Plugins 路径下的 readme 的指导来在本地安装这个版本的 Corda Gradle plugins
7. 在你克隆的 Corda 路径下打开一个 terminal 窗口
8. 使用下边的命令将 Corda 发布到你的本地 Maven repository 中:

  * Unix/Mac OSX: ``./gradlew install``
  * Windows: ``gradlew.bat install``

  .. warning:: 如果你在发布到 Maven 本地之后对你本地的 Corda repository 进行了改动的话，那么你必须要重新发布到你的 Maven 本地，这样才能反映出来你的改动。

  .. warning:: 因为 Corda repository 每天都在更新，不同时间克隆的非稳定版本的分支可能会不一样。如果你在使用 一个非稳定版本并且需要帮助来 debugging 错误的话，请让我们知道你所基于的 **commit**。这会帮助我们确认问题。

9. 记录一下在 Corda repository 的 ``build.gradle`` 文件的根 ``corda_release_version``
10. 在你的 CorDapp 的根 ``build.gradle`` 文件:

    * 更新 ``ext.corda_release_version`` 到你之前记录的 ``corda_release_version``
    * 更新 ``corda_gradle_plugins_version`` 到你之前记录的 ``gradlePluginsVersion``
