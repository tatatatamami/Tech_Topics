# Azurite - Azure Storage の強力なローカルエミュレーター

## はじめに

Azure 開発において、クラウドストレージサービスのテストや開発は重要な要素です。しかし、常に Azure 上のリソースを使用するのはコストや効率の面で課題があります。そこで登場するのが **Azurite** - Azure Storage のローカルエミュレーターです。

## Azurite とは

Azurite は、Azure Storage サービスのローカル開発環境を提供するオープンソースのエミュレーターです。Azure Storage の主要な機能をローカル環境で再現し、開発者がクラウドリソースを使用せずにアプリケーションの開発・テストを行えるようにします。

### 主な特徴

- **完全無料**: オープンソースで提供され、Azure 利用料金が発生しません
- **高い互換性**: Azure Storage の REST API をローカルで忠実に再現
- **軽量**: ローカル環境で素早く起動・停止が可能
- **開発効率向上**: ネットワーク遅延なしでの高速テストが可能

## 対応している Azure Storage サービス

Azurite は Azure Storage の以下の主要サービスをサポートしています：

### 1. Blob Storage
- ブロック blob、ページ blob、追加 blob をサポート
- コンテナーの作成、削除、一覧取得
- blob のアップロード、ダウンロード、削除
- blob のメタデータとプロパティ管理
- SAS (Shared Access Signatures) のサポート

### 2. Queue Storage
- メッセージキューの作成、削除
- メッセージの送信、受信、削除
- キューのメタデータ管理
- メッセージの可視性タイムアウト

### 3. Table Storage
- テーブルの作成、削除、一覧取得
- エンティティの挿入、更新、削除、クエリ
- バッチ操作のサポート
- パーティションキーとローキーによるデータ管理

## インストール方法

Azurite は複数の方法でインストール・実行できます。

### 1. npm を使用したインストール

最も一般的で推奨される方法です：

```bash
# グローバルインストール
npm install -g azurite

# またはプロジェクトローカルにインストール
npm install azurite --save-dev
```

### 2. Docker を使用したインストール

コンテナ環境での実行：

```bash
# Docker Hub から公式イメージを取得
docker pull mcr.microsoft.com/azure-storage/azurite

# コンテナとして実行
docker run -p 10000:10000 -p 10001:10001 -p 10002:10002 \
  mcr.microsoft.com/azure-storage/azurite
```

### 3. Visual Studio Code 拡張機能

開発環境に統合された方式：

1. VS Code の拡張機能マーケットプレイスから "Azurite" を検索
2. インストール後、コマンドパレット (Ctrl+Shift+P) から "Azurite: Start" を実行

## 実行方法とコマンドラインオプション

### 基本的な実行方法

```bash
# 全サービス（Blob、Queue、Table）を同時に起動
azurite

# 特定のサービスのみ起動
azurite --blobHost 127.0.0.1 --blobPort 10000
azurite --queueHost 127.0.0.1 --queuePort 10001
azurite --tableHost 127.0.0.1 --tablePort 10002
```

### 主要なコマンドラインオプション

| オプション | 説明 | デフォルト値 |
|------------|------|-------------|
| `--blobHost` | Blob サービスのホスト | 127.0.0.1 |
| `--blobPort` | Blob サービスのポート | 10000 |
| `--queueHost` | Queue サービスのホスト | 127.0.0.1 |
| `--queuePort` | Queue サービスのポート | 10001 |
| `--tableHost` | Table サービスのホスト | 127.0.0.1 |
| `--tablePort` | Table サービスのポート | 10002 |
| `--location` | データ保存場所 | 現在のディレクトリ |
| `--silent` | ログ出力を抑制 | false |
| `--debug` | 詳細なログ出力 | false |

### 実行例

```bash
# カスタムポートとデータ保存場所を指定
azurite --blobPort 10010 --queuePort 10011 --tablePort 10012 \
  --location /path/to/azurite/data --debug

# HTTPS を有効にして実行
azurite --cert /path/to/cert.pem --key /path/to/key.pem
```

## Azure Functions との連携

Azure Functions プロジェクトで Azurite を使用する方法：

### 1. local.settings.json の設定

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet"
  }
}
```

### 2. 接続文字列の設定

Azurite への接続には以下の接続文字列を使用：

```
UseDevelopmentStorage=true
```

または詳細指定：

```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;
```

### 3. C# コード例

```csharp
using Azure.Storage.Blobs;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

public class BlobFunction
{
    private readonly ILogger<BlobFunction> _logger;

    public BlobFunction(ILogger<BlobFunction> logger)
    {
        _logger = logger;
    }

    [Function("BlobTrigger")]
    public void Run([BlobTrigger("samples-workitems/{name}")] 
                   Stream myBlob, string name)
    {
        _logger.LogInformation($"C# Blob trigger function processed blob\n" +
                              $"Name: {name}\n" +
                              $"Size: {myBlob.Length} Bytes");
    }
}
```

## ASP.NET プロジェクトとの連携

### 1. appsettings.Development.json の設定

```json
{
  "ConnectionStrings": {
    "AzureStorage": "UseDevelopmentStorage=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### 2. Startup.cs での DI 設定

```csharp
using Azure.Storage.Blobs;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        var connectionString = Configuration.GetConnectionString("AzureStorage");
        services.AddSingleton(x => new BlobServiceClient(connectionString));
        
        services.AddControllers();
    }
}
```

### 3. コントローラーでの使用例

```csharp
[ApiController]
[Route("api/[controller]")]
public class BlobController : ControllerBase
{
    private readonly BlobServiceClient _blobServiceClient;

    public BlobController(BlobServiceClient blobServiceClient)
    {
        _blobServiceClient = blobServiceClient;
    }

    [HttpPost("upload")]
    public async Task<IActionResult> UploadFile(IFormFile file)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient("uploads");
        await containerClient.CreateIfNotExistsAsync();
        
        var blobClient = containerClient.GetBlobClient(file.FileName);
        await blobClient.UploadAsync(file.OpenReadStream(), overwrite: true);
        
        return Ok(new { blobUrl = blobClient.Uri.ToString() });
    }
}
```

## Azure Storage Explorer との接続

Azure Storage Explorer を使用して Azurite に接続する方法：

### 1. 接続手順

1. Azure Storage Explorer を起動
2. 左パネルで "ローカル & 接続済み" を右クリック
3. "ストレージ エミュレーターに接続" を選択
4. 以下の設定を入力：
   - **表示名**: Azurite Local
   - **Blob エンドポイント**: http://127.0.0.1:10000
   - **Queue エンドポイント**: http://127.0.0.1:10001
   - **Table エンドポイント**: http://127.0.0.1:10002
   - **アカウント名**: devstoreaccount1
   - **アカウント キー**: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==

### 2. 動作確認

接続後、以下の操作が可能になります：

- **Blob コンテナー**: 作成、削除、ファイルのアップロード/ダウンロード
- **Queue**: メッセージの送信、受信、削除
- **Table**: エンティティの追加、編集、削除、クエリ

## 実運用との違いと制限事項

Azurite は優れたローカル開発環境ですが、実際の Azure Storage との違いを理解しておくことが重要です。

### 主な制限事項

#### 1. パフォーマンス特性
- **スループット**: 実際の Azure Storage ほどの高スループットは期待できない
- **同時接続数**: 大量の同時接続には制限がある
- **レスポンス時間**: ネットワーク遅延がないため、実際より高速

#### 2. セキュリティ機能
- **Azure AD 認証**: サポートされていない（SAS トークンは利用可能）
- **ネットワーク ACL**: 仮想ネットワークやファイアウォール設定は無効
- **暗号化**: 保存時暗号化機能は制限的

#### 3. 高可用性機能
- **冗長性**: 地理的冗長性やゾーン冗長性はサポートされない
- **フェイルオーバー**: 自動フェイルオーバー機能は利用不可

#### 4. スケール制限
- **ストレージ容量**: ローカルディスクの容量に依存
- **API レート制限**: 実際の Azure Storage の制限とは異なる

### ベストプラクティス

#### 1. 開発段階での活用
```bash
# 開発環境での推奨起動方法
azurite --silent --location ./azurite-data
```

#### 2. CI/CD での利用
```yaml
# GitHub Actions の例
steps:
  - name: Start Azurite
    run: |
      npm install -g azurite
      azurite --silent --location azurite &
      sleep 5
  
  - name: Run Tests
    run: dotnet test
    env:
      AZURE_STORAGE_CONNECTION_STRING: "UseDevelopmentStorage=true"
```

#### 3. Docker Compose での統合
```yaml
version: '3.8'
services:
  azurite:
    image: mcr.microsoft.com/azure-storage/azurite
    ports:
      - "10000:10000"
      - "10001:10001"  
      - "10002:10002"
    volumes:
      - azurite-data:/opt/azurite/folder

  app:
    build: .
    environment:
      - AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://azurite:10000/devstoreaccount1;QueueEndpoint=http://azurite:10001/devstoreaccount1;TableEndpoint=http://azurite:10002/devstoreaccount1;
    depends_on:
      - azurite

volumes:
  azurite-data:
```

## まとめ

Azurite は Azure Storage 開発において非常に有用なツールです。以下の場面で特に威力を発揮します：

### 適用場面
- **ローカル開発**: コスト効率的な開発環境
- **単体テスト**: 高速で独立したテスト実行
- **CI/CD パイプライン**: 自動化されたテスト環境
- **オフライン開発**: ネットワーク接続に依存しない開発

### 導入時の推奨アプローチ
1. **段階的導入**: まずは単一サービス（Blob など）から開始
2. **テスト戦略**: Azurite でのテストと Azure での統合テストを組み合わせ
3. **本番移行**: 実際の Azure Storage への移行時は設定変更のみで対応

Azure Storage を活用したアプリケーション開発において、Azurite は開発効率と品質向上の両方に貢献する強力なツールといえるでしょう。適切に活用することで、より堅牢で効率的な Azure アプリケーションの開発が可能になります。

## 参考リンク

- [Microsoft Learn - Azurite エミュレーターを使用したローカル Azure Storage 開発](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-use-azurite)
- [Azurite GitHub Repository](https://github.com/Azure/Azurite)
- [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)