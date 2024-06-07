# GitHub Private Repository の構築の流れ

1. プロジェクトの生成

```shell
pnpm init
```

2. Personal Access Token (classic) を以下の権限を付与して作成

**必要な権限**

- `repo`
- `workflow`
- `read:packages`
- `write:packages`
- `delete:packages`

3. レポジトリの Secrets に Personal Access Token を追加する

Name: DEPLOY_PACKAGE_TOKEN
Secrest: [Personal Access Token]

4. `.npmrc`の作成

```.npmrc
//npm.pkg.github.com/:_authToken=DEPLOY_PACKAGE_TOKEN
@[OrganizationName or GitHubUsername]:registry=https://npm.pkg.github.com
```

5. `package.json`の一部を編集

```json
{
  //"name": "@[OrganizationName or GitHubUsername]/[RepositoryName]"
  "name": "@Myxogastria0808/github-private-package-mytutorial",
  ...
  //誤ってnpmに公開することを防止する
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/"
  },
  //レポジトリの情報を記載
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Myxogastria0808/github-private-package-mytutorial.git"
  },
  ...
}
```

6. `.github/workflows/deploy.yaml`の作成

```yaml
name: deploy-package

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - name: Replace token of .npmrc
        shell: bash
        env:
          DEPLOY_PACKAGE_TOKEN: ${{ secrets.DEPLOY_PACKAGE_TOKEN }}
        run: |
          sed -i "s/DEPLOY_PACKAGE_TOKEN/${DEPLOY_PACKAGE_TOKEN}/" ./.npmrc
      - name: Check .npmrc
        run: |
          cat ./.npmrc
      - name: Publish github private package
        run: npm publish
        env:
          NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 参考サイト

https://zenn.dev/moneyforward/articles/20230620-github-packages

https://qiita.com/marumaru0113/items/21b600c21caf5d9b9775

https://qiita.com/shouki-s/items/5e94f37288c3e796a85e
