# document-loaders 模块组

## 模块职责
**文档加载器集合** — 从各种数据源 (云存储、网页、代码仓库) 加载文档，统一转换为 Document 对象。

## 统一架构模式

所有加载器遵循相同的两层架构:

```
DocumentSource (接口)
  ├── inputStream() → InputStream
  └── metadata() → Metadata

DocumentLoader.load(source, parser)
  ├── source.inputStream()
  ├── parser.parse(inputStream) → Document
  └── 附加 source.metadata() 到 Document
```

## 子模块列表

| 子模块 | 数据源 | SDK/客户端 | 关键类 |
|--------|--------|-----------|--------|
| `langchain4j-document-loader-amazon-s3` | AWS S3 | AWS SDK v2 S3Client | `AmazonS3DocumentLoader`, `AmazonS3Source` |
| `langchain4j-document-loader-azure-storage-blob` | Azure Blob | Azure BlobServiceClient | `AzureBlobStorageDocumentLoader`, `AzureBlobStorageSource` |
| `langchain4j-document-loader-github` | GitHub 仓库 | Kohsuke GitHub API | `GitHubDocumentLoader`, `GitHubSource` |
| `langchain4j-document-loader-google-cloud-storage` | GCS | GCP Storage SDK | `GoogleCloudStorageDocumentLoader`, `GcsSource` |
| `langchain4j-document-loader-tencent-cos` | 腾讯云 COS | Tencent COSClient | `TencentCosDocumentLoader`, `TencentCosSource` |
| `langchain4j-document-loader-selenium` | 网页 | Selenium WebDriver | `SeleniumDocumentLoader` |
| `langchain4j-document-loader-playwright` | 网页 | Playwright Browser | `PlaywrightDocumentLoader` |

## 核心执行流程

```
// 以 S3 为例
AmazonS3DocumentLoader loader = AmazonS3DocumentLoader.builder()
    .region("us-east-1")
    .build();

// 加载单个文档
Document doc = loader.loadDocument("my-bucket", "path/to/file.pdf", parser);

// 批量加载 (按前缀)
List<Document> docs = loader.loadDocuments("my-bucket", "path/to/prefix/", parser);
```

### 完整流程

```
用户调用 loader.loadDocument(bucket, key, parser)
  │
  ├── 创建 Source: AmazonS3Source(bucket, key, s3Client)
  │   └── s3Client.getObject(bucket, key) → InputStream
  │       └── Metadata: fileName, source (S3 URI)
  │
  ├── DocumentLoader.load(source, parser)
  │   ├── source.inputStream()
  │   ├── parser.parse(inputStream)  ← 使用指定的 DocumentParser
  │   └── document.metadata().mergeFrom(source.metadata())
  │
  └── 返回 Document
```

## 云存储加载器共性

- 使用 Builder 模式配置认证信息
- 支持 `loadDocument(单个)` 和 `loadDocuments(批量)` 方法
- 批量加载: 按前缀列出所有对象，逐个加载
- 自动附加元数据 (文件名、来源 URI)

## 网页加载器

- **Selenium**: 等待 `ExpectedCondition` 满足后获取页面源码
- **Playwright**: 等待 `DOMCONTENTLOADED` 后获取页面内容
- 两者都实现 `AutoCloseable` 用于清理浏览器资源
