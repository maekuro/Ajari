# nft-vc

## Setup

## セットアップ手順

### 前提条件

* **Homebrew:** パッケージマネージャー Homebrew がインストールされている必要があります。
* **nodenv:** Node.js のバージョン管理ツールとして nodenv を使用します。

### 手順

1. **Node.js のインストールとバージョン設定**

   * nodenv を使って Node.js 18.0.0 をインストールし、グローバルバージョンに設定します。

     ```bash
     /usr/local/bin/nodenv install 18.0.0
     /usr/local/bin/nodenv global 18.0.0
     ```

   * 正しく設定されたか確認します。

     ```bash
     node -v 
     ```

     * `v18.0.0` のように表示されればOKです。

2. **yarn のインストール**

   * `npm` を使って `yarn` をグローバルにインストールします。 `nodenv` を使用している場合は、`nodenv exec` を使って `nodenv` の環境下で実行する必要があります。

     ```bash
     nodenv exec npm install -g yarn
     ```

3. **Python 環境の準備**

   * `pyenv` を使って Python 3.9.13 をインストールし、ローカルバージョンに設定します。

     ```bash
     /usr/local/bin/pyenv install 3.9.13
     /usr/local/bin/pyenv local 3.9.13
     ```

   * `Ajari` プロジェクトのルートディレクトリに移動します。

     ```bash
     cd /Users/matsudahidehiko/git/fan-mily/Ajari # 適宜パスを修正
     ```

   * 仮想環境を作成し、アクティブ化します。

     ```bash
     /Users/matsudahidehiko/git/fan-mily/Ajari % python -m venv venv
     /Users/matsudahidehiko/git/fan-mily/Ajari % source venv/bin/activate
     ```

   * `pip` を最新版にアップグレードします。

     ```bash
     (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % pip install --upgrade pip
     ```

4. **必要なパッケージのインストール**

   * `coincurve` と `pysha3` をビルドするために必要なパッケージをインストールします。

     ```bash
     brew install automake libtool pango
     ```

   * `requirements.txt` に記載された Python パッケージをインストールします。 `openssl` 関連のエラーが発生する場合は、環境変数を設定してインストールします。

     ```bash
     (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % env LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" pip install -r requirements.txt
     ```

   * `coincurve` と `pysha3` のビルドでエラーが発生した場合は、以下の手順で手動ビルドを試みてください。

     1. `coincurve` のビルド

        * `coincurve` のソースコードをダウンロードします。（すでにダウンロード済みの場合は不要です）

          ```bash
          /Users/matsudahidehiko/git/fan-mily/Ajari % cd .. 
          /Users/matsudahidehiko/git/fan-mily % git clone https://github.com/ofek/coincurve.git
          ```

        * `coincurve` のディレクトリに移動し、ビルドディレクトリを作成して移動します。

          ```bash
          /Users/matsudahidehiko/git/fan-mily % cd coincurve
          /Users/matsudahidehiko/git/fan-mily/coincurve % mkdir build
          /Users/matsudahidehiko/git/fan-mily/coincurve % cd build
          ```

        * `CMake` を使ってビルドの設定を行い、ビルドとインストールを実行します。

          ```bash
          (venv) /Users/matsudahidehiko/git/fan-mily/coincurve/build % export OPENSSL_ROOT_DIR=$(brew --prefix openssl)
          (venv) /Users/matsudahidehiko/git/fan-mily/coincurve/build % cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local 
          (venv) /Users/matsudahidehiko/git/fan-mily/coincurve/build % make
          (venv) /Users/matsudahidehiko/git/fan-mily/coincurve/build % make install
          ```

     2. `pysha3` のビルド

        * 仮想環境内で `python3-config --includes` を実行し、出力結果から `pystrhex.h` が含まれるパスを確認します。

          ```bash
          (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % python3-config --includes
          ```

        * 確認したパスを `CFLAGS` に設定し、`pip install` を再実行します。

          ```bash
          (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % export CFLAGS="-I<取得した include パス> -I/opt/homebrew/opt/openssl@3/include" 
          (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % cd coincurve/build 
          (venv) /Users/matsudahidehiko/git/fan-mily/Ajari/coincurve/build % env LDFLAGS="-L$(brew --prefix openssl)/lib" pip install -r ../../requirements.txt
          ```

5. **cert-issuer の動作確認**

   * `Ajari` プロジェクトのルートディレクトリに移動し、仮想環境をアクティブにした状態で、`cert-issuer` コマンドが正しくインストールされたか確認します。

     ```bash
     (venv) /Users/matsudahidehiko/git/fan-mily/Ajari % cert-issuer -h
     ```

     * ヘルプが表示されれば、セットアップ完了です。

**補足**

* 上記の手順は、macOS 環境を想定しています。
* `requirements.txt` の内容は、実際のプロジェクトに合わせて修正してください。
* 各コマンドの実行前に、仮想環境がアクティブになっていることを確認してください。
* エラーが発生した場合は、エラーメッセージをよく読み、適切な対処を行ってください。

これで、`cert-issuer` のセットアップが完了し、利用できるようになるはずです。不明な点等ございましたら、お気軽にご質問ください。

## Issuer Setup

1. issuer の情報を did:web メソッドで resolve できるようにする。

- Ethereum の秘密鍵を指定して JWK 形式の公開鍵に変換する

```sh
$ node scripts/convert-did-public-key.js ./keys/wallet-private.dev.key
{"kty":"EC","crv":"K-256","x":"XyZpmS5rwy23Dqm3iYNGn_A_p5JYXOSbyp_ev1Uss7E","y":"XyZvZO81bLTeH0-XmjQlrhAWOC79utg3aC5BV5amAsI"}
```

- `./keys/wallet-private.dev.key` に秘密鍵を配置して下さい

2. did.json を作成する (上記の公開鍵情報を含める)

- 参考: `hostings/staging/public/.well-known/did.json`

3. IssuerProfile ファイルを作成する

- 参考: `hostings/staging/public/blockcerts.json`

4. RevocationList ファイルを作成する

- 参考: `hostings/staging/public/blockcerts_revocation_list.json`

5. 手順 2,3,4 で作成したファイルをホスティングする。

- 参考: `hostings/staging/README.md`

## 証明書発行ワークフロー

### VC 発行フロー

1. GoogleForm から JSON に変換

```sh
$ node scripts/convert-members.js ./tmp/form.csv > ./tmp/members.json
```

2. VC 用の画像生成

```sh
$ node scripts/generate-vc-image.js ./tmp/members.json
```

3. 署名なし VC を生成

```sh
$ node scripts/generate-unsigned-vc.js ./tmp/members.json
```

4. VC 発行

```sh
# dev (goerli)
$ cert-issuer -c cert-issuer.dev.ini --chain ethereum_goerli --goerli_rpc_url $GOERLI_ALCHEMY_URL

# prd (mainnet)
$ cert-issuer -c cert-issuer.prd.ini --chain ethereum_mainnet --ethereum_rpc_url $MAINNET_ALCHEMY_URL
```

5. 発行済み VC を IPFS にアップロード

```sh
$ node scripts/bulk-upload-to-ipfs.js ./tmp/members.json vc
```

### NFT 発行フロー

1. NFT 用の画像生成

```sh
$ node scripts/generate-nft-image.js ./tmp/members.json
```

2. NFT 用の画像を IPFS にアップロード

```sh
$ node scripts/bulk-upload-to-ipfs.js ./tmp/members.json nft
```

3. コントラクトのデプロイ

```sh
$ npx hardhat run scripts/deploy.js --network $NODE_ENV
Compiled 1 Solidity file successfully
deployed to: 0x1234567890123456789012345678901234567890
```

環境変数 `CONTRACT_ADDRESS` に上記のアドレスをセットします。

4. ソースコードのアップロード

```sh
$ npx hardhat verify $CONTRACT_ADDRESS --constructor-args arguments.js --network $NODE_ENV
```

※ エラー発生時に `$ npx hardhat clean` で解消するケースを確認している。
※ エラー発生時でも verify に成功しているケースを確認しているため、etherscan で CONTRACT_ADDRESS を検索するとヒント見つかるかも。

5. NFT bulk mint

```sh
# dry-run
$ node scripts/bulk-mint.js ./tmp/members.json

# mint
$ node scripts/bulk-mint.js ./tmp/members.json --dry-run=false
```

## Test

### Test for scripts

```sh
$ npm test
```

※ 一部の Utility 関数のみテスト定義している

### Test for Smart Contract

local でノードを起動

```sh
$ npx hardhat node
```

テスト実行

```sh
$ npm run test_contracts
# OR
$ npx hardhat test --network localhost
```

テストネット環境でテスト実行

```sh
$ npx hardhat test --network goerli
```

※ ガス代結構かかるので注意。
※ 一部ケースは NG になる `ethers.getSigners()` で 3 アカウント分のセットアップが必要なため。
