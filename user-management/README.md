# 共同GPUサーバーへのユーザーの一括追加/削除

## 使い方


### ユーザーの追加

1. 追加したいユーザーの公開鍵を取得してください。
2. 公開鍵を入手したら、`list-users/`配下の`add-users-list.yaml`を編集してユーザー情報を追加してください。

add-users-list.yaml
```
user_info:
  - user_name: "test"
    public_key: "公開鍵"
  - user_name: "test2"
    public_key: "公開鍵"
```
3. [actions]→[Add Users]のGithub Actionを手動でトリガーしてください。
4. ワークフローがエラーなく完了したら、全てのGPUサーバーへのユーザー追加が完了です。

### ユーザーの削除

1. 追加したいユーザー名を取得してください。
2. `list-users/`配下の`delete-users-list.yaml`を編集してユーザー情報を追加してください。

delete-users-list.yaml
```
user_info:
  - user_name: "test"
  - user_name: "test2"
```
3. [actions]→[Delete Users]のGithub Actionを手動でトリガーしてください。誤作動防止のため"delete"と入力しないとワークフローが走らないように設定されています。
4.  ワークフローがエラーなく完了したら、全てのGPUサーバーにおけるユーザーの削除が完了です。

*削除のワークフローを実行すると、ユーザーのホームディレクトリも削除されてしまうので、十分注意してください。


[How to set up from scratch](how-to-setup.md)
