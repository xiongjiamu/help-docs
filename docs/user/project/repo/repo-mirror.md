# 仓库镜像[](#repository-mirroring "Permalink")

仓库镜像允许将仓库镜像到外部源或从外部源镜像，它可用于镜像存储库之间的分支，标签和提交。

仓库镜像将自动更新，您也可以最多每 5 分钟手动触发一次更新。

## 概览[](#overview "Permalink")

当您想在 CODEChina 之外的平台使用存储库时，存储库镜像很有用。

我们支持两种仓库镜像方式：

*   推送：用于将 CODEChina 仓库镜像到另一个位置
*   拉取：用于将其他位置的仓库镜像到 CODEChina

至少具有 [Developer](/docs/user/permissions.md)权限的用户也可以强制立即更新，除非：

*   镜像已被更新
*   距离上次更新还未超过 5 分钟

出于[安全原因](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/27166) ，原始仓库的 URL 仅显示给对镜像项目具有 Maintainer 或 Owner 权限的用户。

## 用例[](#use-cases "Permalink")

以下是一些我们建议使用仓库镜像的场景：

*   您迁移到了 CODEChina，但仍需要将项目保留在另一个源中。在这种情况下，您只需将其设置为镜像到 CODEChina（拉取），所有提交，标签和分支的基本历史记录都将在您的 CODEChina 仓库中。
*   您在另一个源中有旧项目，这些旧项目不再使用，但又不想出于归档目的而删除它们。在这种情况下，您可以创建一个推送镜像，以便您的 CODEChina 仓库可以将其更改推送到旧源的项目中。

## 推送到远程仓库[](#pushing-to-a-remote-repository-core "Permalink")

对于现有项目，可以按如下所示设置推送镜像：

1.  导航到项目的**项目设置>仓库**，然后展开**镜像仓库**部分
2.  输入 Git 仓库的 URL
3.  从**镜像方向**下拉菜单中选择**推送** 
4.  如有需要，请从**验证方式**下拉列表中选择一种身份**验证方法** 
5.  如有需要，请选中**只镜像受保护的分支**复选框
6.  如果需要，请选中 **保留分叉的 refs**复选框
7.  单击**镜像仓库**按钮以保存配置

[![Repository mirroring push settings screen](/docs/img/repository_mirroring_push_settings.png)](/docs/img/repository_mirroring_push_settings.png)

启用推送镜像后，为防止镜像发散只有推送会直接提交到镜像仓库。任何时候，所有更改都将最终存储在镜像仓库中：

*   提交被推送到 CODEChina
*   [强制更新](#强制更新)已启动

推送到仓库中的文件的更改至少会自动推送到远程镜像：

*   收到推送后的五分钟内
*   如果启用了**仅受镜像保护的分支**，则在一分钟之内

如果存在分叉的分支，您将在**镜像的仓库**部分看到错误的提示。


### 保留分叉 refs[](#keep-divergent-refs-core "Permalink")

默认情况下，如果远程镜像上的任何 ref 与本地仓库不同，则*整个推送*将失败，并且不会更新任何内容。

比如，如果仓库具有已镜像到远程的`master` ， `develop`和`stable`分支，然后在镜像仓库的`develop`添加了一个新提交 ，则下一次推送尝试将失败，并且将使`master`和`stable`不会同步镜像尽管他们没有分歧的 refs。在解决分歧之前，无法镜像任何分支上的更改。

启用**保持差异 refs**选项后，将跳过`develop`分支，从而可以更新`master`和`stable`。镜像将反映`develop`已偏离并被跳过，并标记为更新失败。

## 创建一个从 CODEChina 到 Github 的镜像[](#setting-up-a-push-mirror-from-CODEChina-to-github-core "Permalink")

要设置从 CODEChina 到 GitHub 的镜像，您需要执行以下步骤：

1.  创建一个[GitHub 个人访问令牌](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) ，并选中`public_repo`复选框
2.  使用以下格式填写**Git 存储库 URL**字段： `https://<your_github_username>@github.com/<your_github_group>/<your_github_project>.git` 
3.  使用您的 GitHub 个人访问令牌填写**密码**字段
4.  单击**镜像仓库**按钮

镜像的仓库将被列出，例如， `https://*****:*****@github.com/<your_github_group>/<your_github_project>.git` 

该仓库将很快进行推送，您也可以通过单击`立即更新`按钮进行强制推送。

## 从远程仓库拉取[](#pulling-from-a-remote-repository-starter "Permalink")

您可以将仓库设置为自动从上游仓库更新分支，标记和提交。

当您感兴趣的仓库位于其他服务器上并且您希望能够在 CODEChina 中浏览其内容时，这个功能会十分有用。

可以通过以下步骤，为现有项目配置镜像拉取仓库：

1.  导航到项目的**项目设置>仓库**，然后展开**镜像仓库**部分
2.  输入镜像 Git 仓库的 URL
3.  从**镜像方向**下拉菜单中选择**拉取** 
4.  如有必要，请从**验证方式**下拉列表中选择一种身份**验证方法**
5.  如有必要，请选中以下框：
    *   **覆盖分叉分支**
    *   **仅镜像保护的分支**
6.  单击**镜像仓库**按钮以保存配置

[![Repository mirroring pull settings screen - upper part](/docs/img/repository_mirroring_pull_settings_upper.png)](/docs/img/repository_mirroring_pull_settings_upper.png)

现在，由于 CODEChina 中的项目已经设置为从上游存仓库中提取更改，所以不应将提交直接推送到 CODEChina 上的仓库中。相反地，现在任何提交都应该推送到上游仓库。推送到上游仓库中的更改将被拉入 CODEChina 仓库中：

*   在一定时间内自动拉取
*   启动[强制更新](#强制更新)时

**警告**：如果您确实在 CODEChina 仓库中更新了分支，则该分支将与上游分叉，并且 CODEChina 将不再自动更新该分支以防止丢失任何更改。另请注意，上游仓库中已删除的分支和标签将不会反映在 CODEChina 的仓库中。

### 工作机制[](#how-it-works "Permalink")

为仓库启用拉取镜像功能后，会将仓库添加到队列中。每分钟一次，Sidekiq cron 作业基于以下内容计划仓库镜像的更新：

*   可用容量，这由 Sidekiq 设置确定， CODEChina 参考了 [Gitlab.com](https://docs.gitlab.com/13.2/ee/user/gitlab_com/index.html#sidekiq) 的默认配置。
*   队列中已经要更新的仓库镜像数，到期时间取决于仓库镜像的最后更新时间以及重试次数。

Sidekiq 可用时，它就会开始处理仓库镜像并对它们进行拉取更新。更新仓库镜像的过程为：

*   成功后，至少需要等待 30 分钟，更新才会重新加入队列
*   失败（例如，分支与上游分支出现分叉），稍后将再次尝试。镜像最多可以发生 14 次故障，然后才不会再次进入队列进行更新

### SSH 认证[](#ssh-authentication "Permalink")

SSH 身份验证是相互的：

*   您必须向服务器证明您有权访问仓库
*   服务器还必须向*您*证明它是谁

您提供凭据作为密码或公共密钥，另一个仓库所在的服务器提供其凭据作为"主机密钥"，其指纹需要手动验证。

如果您通过 SSH 镜像（即使用`ssh://` URL），则可以使用以下方法进行身份验证：

*   基于密码的身份验证，就像通过 HTTPS 一样
*   公钥认证，这通常比密码身份验证更安全，尤其是当其他仓库支持 Deploy Keys 时

可以从下面步骤开始使用 SSH 验证：

1.  导航到项目的**项目设置>仓库，**然后展开**镜像仓库**部分
2.  输入`ssh://` URL 进行镜像

**注意**：目前不支持 SCP 样式的 URL（即`git@example.com:namespace/project.git`，对于这种 URL 格式，需要改写为 `ssh://example.com/namespace/project.git` 的格式)

输入 SSH URL 后将在页面上添加两个按钮：

*   **检测主机密钥**
*   **手动输入主机密钥**

如果单击：

*   **检测主机密钥**按钮，CODEChina 将从服务器获取主机密钥并显示指纹
*   **手动输入主机密钥**按钮，将显示一个字段，您可以在其中粘贴主机密钥

假设您使用了前者，那么现在需要验证指纹是否符合您的期望. gitlhub.com 和其他代码托管站点公开公开其指纹，供您检查：

*   [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/regions.html#regions-fingerprints)
*   [Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/configure-ssh-and-two-step-verification/)
*   [GitHub](https://help.github.com/en/github/authenticating-to-github/githubs-ssh-key-fingerprints)
*   [GitLab.com](https://docs.gitlab.com/13.2/ee/user/gitlab_com/index.html#ssh-host-keys-fingerprints)
*   [Launchpad](https://help.launchpad.net/SSHFingerprints)
*   [Savannah](http://savannah.gnu.org/maintenance/SshAccess/)
*   [SourceForge](https://sourceforge.net/p/forge/documentation/SSH%20Key%20Fingerprints/)

在镜像仓库时，CODEChina 将在连接之前检查至少一个存储的主机密钥是否匹配，这样可以防止将恶意代码注入到您的镜像中，或者防止您的密码被盗。

### SSH 公钥认证[](#ssh-public-key-authentication "Permalink")

要使用 SSH 公钥身份验证，您还需要从**验证方式**下拉菜单中选择该选项。创建镜像后，CODEChina 将会自动生成一个 4096 位 RSA 密钥，可以通过单击**复制 SSH 公共密钥**按钮进行复制。

[![Repository mirroring copy SSH public key to clipboard button](/docs/img/repository_mirroring_copy_ssh_public_key_button.png)](/docs/img/repository_mirroring_copy_ssh_public_key_button.png)

然后，您需要将公共 SSH 密钥添加到另一个仓库的配置中：

*   如果其他仓库托管在 CODEChina 上，则应将公共 SSH 密钥添加为 Deploy Key
*   如果其他仓库位于其他位置，则可能需要将密钥添加到用户的`authorized_keys`文件中，将整个公共 SSH 密钥单独粘贴到文件中并保存。

如果您需要更改密钥，则可以删除并重新添加镜像以生成新密钥。您必须使用新密钥更新另一个仓库，以保持镜像运行。

**注意**：生成的密钥存储在 CODEChina 数据库中，而不存储在文件系统中。因此，无法在 pre-recieve hook 中使用针对镜像的 SSH 公钥身份验证。

### 覆盖分叉分支[](#overwrite-diverged-branches-starter "Permalink")

您可以选择始终使用远程版本更新本地分支，即使它们与远程分支不同。

**警告：**对于镜像分支，启用此选项会导致丢失本地更改。

要使用此选项，请在创建仓库镜像时选中**覆盖分叉分支**复选框。

### 仅镜像保护分支[](#only-mirror-protected-branches-starter "Permalink")

您可以选择仅将受保护的分支从远程仓库拉到 CODEChina，未受保护的分支不会被镜像，并且允许分叉。

要使用此选项，请在创建仓库镜像时选中**仅镜像保护的分支**复选框。

### 同步失败[](#hard-failure-starter "Permalink")

如果镜像过程连续 14 次失败重试，它将被标记为硬盘失败，并将在项目拉取镜像设置页面中可见。

当一个项目同步失败时，它将不再进入镜像队列。 用户可以通过[强制更新](#强制更新)来再次恢复项目镜像。

## 强制更新[](#forcing-an-update-core "Permalink")

当计划将镜像自动更新时，您始终可以使用**项目设置** **仓库**页面的**镜像仓库**部分上的"更新"按钮强制进行更新。

[![Repository mirroring force update user interface](/docs/img/repository_mirroring_force_update.png)](/docs/img/repository_mirroring_force_update.png)

## 故障处理[](#troubleshooting "Permalink")

如果在推送过程中发生错误，则 CODEChina 将在该仓库中显示"错误"信息。然后，将鼠标悬停在突出显示的文本上，即可查看有关错误的详细信息。

[![Repository mirroring error tips](/docs/img/repository_mirroring_error.png)](/docs/img/repository_mirroring_error.png)

### 13:Received RST_STREAM with error code 2 with GitHub[](#13received-rst_stream-with-error-code-2-with-github "Permalink")

如果在镜像到 GitHub 仓库时收到"错误代码为 2 的 13：Received RST_STREAM"，则您的 GitHub 设置可能被设置为阻止推送，以暴露用于提交的电子邮件地址. 可以将您在 GitHub 上的电子邮件地址设置为公开，或者禁用[公开我的电子邮件](https://github.com/settings/emails)设置的" [阻止"命令行推送](https://github.com/settings/emails) 