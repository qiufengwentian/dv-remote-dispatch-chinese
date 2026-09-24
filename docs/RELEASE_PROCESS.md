1. 在 info.json 与 RemoteDispatch\RemoteDispatch.csproj 中更新版本号。
2. 在 CHANGELOG.md 中更新版本号、变更内容与发布日期。
3. 提交并推送到 trunk 分支。
4. 为该提交打上新版本号标签（例如 v1.2.3）并推送该标签。
5. 构建 mod 的 Release 版本。
6. 在 GitHub 上用新标签创建 Release，并将构建产物作为附件上传。
7. 在 resources/repository.json 中填入新版本号与新 Release 的下载地址。
8. 提交并推送更新后的 repository.json 到 trunk 分支。
