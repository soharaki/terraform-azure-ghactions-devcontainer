# Terraform and AzureをVS Codeのコンテナ開発環境で開発できる優れものや

この環境をフォークしてgit cloneしたらすぐ始めよっか。   
ちょいちょいいじったらThe [VS Code Remote Development (Remote - Containers)](https://code.visualstudio.com/docs/remote/containers)はすぐ使えるよーなるで。

- こんなことできるようになるよ!!
  - TerraformによるAzure管理 
    - `terraformとAzureCliツールはすでにdevcontainerの中にあるから安心しいや`
  - GitHub Actionsによる自動実行
    - `Azureは一度作ったら終わりじゃない。繰り返し繰り返し壊して作りなおすのじゃ`

# 始め方

## 必要なもの

下記のツールを準備したってや

1. Visual Studio Code : https://code.visualstudio.com/download
1. VS Code Extension : [VS Code Remote Development (Remote - Containers)](https://code.visualstudio.com/docs/remote/containers)
1. Docker Desktop (in any OS) : https://www.docker.com/get-started

これが全てや!

## ほな、始めよっか
1. Docker Desktopが動いていることを確認してな
1. GitHubのリポジトリをフォークしてな 
   (AzureServicePrincipalの鍵とか重要なものをぎょーさん扱うさかい、プライベートリポジトリにすることをお勧めするで).
1. ローカル環境にcloneしてや
    ```sh
    git clone <your repository>
    ```
1. クローンしたリポジトリフォルダをVs codeで開いて、左下のRemote Developmentアイコンをクリックや(あるいはコマンドラインから同じこともできるでー)  
    ![VS Code Remote Development](docs/images/launch-vscode-remotecontainer-01.png)
    ```cmd
    REM Windowsだといつも俺はこうやるで
    REM 1. クローンしたリポジトリフォルダをExplorerで普通に開く
    REM 2. 上のフォルダパスの入力欄にcmdと打ち込んでコマンドプロンプトを開く
    REM 3. 下記の通りコマンドを打ち込むんじゃー。最後の.忘れんといてな
    REM 3. `C:\Users\soharaki\SomeRepository>code -r .`
    ```
    
1. 起動したメニュー欄から `Remote-Containers: Reopen in Containers...` を選択する
    ![Reopen in containers](docs/images/vscode-remote-menu-reopenincontainer.png)
1. VS Codeのターミナルを開いてlets tryや!  
    ```sh
    $ terraform -v
    Terraform v0.12.29
    
    $ az --version
    azure-cli                          2.9.1
    ...
    ```
    
    ドヤァ

    FYI: terraformの構文チェックを行う `tflint` やビルドを良い感じにしてくれる `terragrunt` もインストールしとるで。  
    詳しいことは [Dockerfile](.devcontainer/Dockerfile) を参照な。

# 実践編

## Terraform backends on Azure Storage
手始めに [Terraform backends on Azure Storage](https://www.terraform.io/docs/backends/types/azurerm.html)を作ってみよっか

1. ターミナルを開いてや。いまあんたはリポジトリのディレクトリの直下におるはずや
    ```sh
    $ pwd
    /workspace/<your repository name>
    ```
1. Azureにログインしましょ. ログインページを開いて次の通り入力してや (the part `************` in below).
    ```sh
    $ az login
    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code ************ to authenticate.
    ```
    もしもログインに成功したら, JSONの形でAzureのサブスクリプション一覧が出るで。

1. 適切なサブスクリプションを選んで、`id` プロパティに GUID を指定して実行じゃ
    ```sh
    $ az account set --subscription <your subscription GUID>
    ```
1. Go to the below directory.
    ```sh
    $ cd 00-create-azurerm-backend
    ```
1. Run the `terraform init`
    ```sh
    $ terraform init

    Initializing the backend...

    Initializing provider plugins...

    Terraform has been successfully initialized!

    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.

    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.
   ```
1. `terraform plan`を実行や。その際にストレージアカウントの名前を聞かれるで。   
　　世界でオンリーワンの名前を準備してトライな。

    ```sh
    $ terraform plan

    var.backend_storage_account_name
    Storage account name for terraform backend

    Enter a value: ****
    ```
    Okay if you see this terraform plan output.
    ```
    ...
    ...
    Plan: 3 to add, 0 to change, 0 to destroy.
    ```

    Notice: If you are not login to Azure, you will see this error message. Please go back to `az login`.
    ```
    Error: Error building AzureRM Client: Authenticating using the Azure CLI is only supported as a User (not a Service Principal).
    ```
1. `terraform apply`を実行や。はたまたストレージアカウントの名前を聞かれるで。  
   さっき入力した名前、それを使うんじゃ。

    ```sh
    $ terraform apply

    var.backend_storage_account_name
    Storage account name for terraform backend

    Enter a value: ****
    ```
    You will be asked the confirmation message. Entry `yes` if okay.
    ```sh
    ...
    ...
    Plan: 3 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions?
    Terraform will perform the actions described above.
    Only 'yes' will be accepted to approve.

    Enter a value: yes
    ```
    Wait a little...
    ```
    azurerm_resource_group.rg: Creating...
    azurerm_resource_group.rg: Creation complete after 0s [id=/subscriptions/****GUID****/resourceGroups/terraform-rg]
    azurerm_storage_account.strg: Creating...
    azurerm_storage_account.strg: Still creating... [10s elapsed]
    azurerm_storage_account.strg: Still creating... [20s elapsed]
    azurerm_storage_account.strg: Creation complete after 20s [id=/subscriptions/****GUID****/resourceGroups/terraform-rg/providers/Microsoft.Storage/storageAccounts/****your storage account name****]
    azurerm_storage_container.strg-container: Creating...
    azurerm_storage_container.strg-container: Creation complete after 0s [id=https://********.blob.core.windows.net/tfstate]
    ```

    Finished!
    ```
    Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
    ```
1. ストレージアカウントが作成されたかチェックやー

    ```sh
    $ az group show --name terraform-rg --out table
    (result)
    
    $ az storage account show --name '<replace by yours>' --out table
    (result)
    ```

# GitHub Actions編

(少し待っててな)

# カスタマイズ編

カスタマイズ? ほんならこれらのファイルを確認や。重要なことは全て書いてある。[VSコードドキュメント](https://code.visualstudio.com/docs/remote/containers)もみるとええで。

- [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json)
- [.devcontainer/Dockerfile](.devcontainer/Dockerfile)

## dotfilesによる自分専用の環境を作りたい

ドットファイルの仕組みでお好きなように自分専用環境作れるでー。

[.devcontainer/devcontainer.json](.devcontainer/devcontainer.json)
```json
    "settings": {
        // ...
        // dotfiles
        "dotfiles.repository": "hoisjp/terraform-azure-ghactions-devcontainer", // change here to your repository.
        "dotfiles.targetPath": "~/.devcontainer/dotfiles",
        "dotfiles.installCommand": "~/.devcontainer/dotfiles/install.sh"
    },
```

- [Personalizing with dotfile repositories](https://code.visualstudio.com/docs/remote/containers#_personalizing-with-dotfile-repositories)

# 参考文献や

## VS Code docs

- Developing inside a Container : https://code.visualstudio.com/docs/remote/containers

## Terraform for Azure

- https://docs.microsoft.com/en-us/azure/developer/terraform/
- Terraform - Azure Provider : https://www.terraform.io/docs/providers/azurerm/index.html
- [Terraform - Azure Provider - GitHub Repos](https://github.com/terraform-providers/terraform-provider-azurerm)
- [Terraform - Azure Provider - GitHub Repos - Examples](https://github.com/terraform-providers/terraform-provider-azurerm/tree/master/examples)

## 個人的なメモ
terraformに関する必要な開発環境がここには入っている。
拡張先としては、例えばterraformの静的テスト確認が(コマンドはあるので)簡単に確認する方法とか
そのあたりだろうか。

これをJava+Quarkus環境に応用しようとしたら何が必要なのだろう。
mavenキャッシュに関する問題はDockerをコマンドラインの実行用ツールとしてのみ使えばキャッシュ先はローカルになるのか？

大阪弁、また同僚にレビューしてもらわないとな...
もういっそのこと厳格さが要求されないrunbookは全部口語体でいいのに。