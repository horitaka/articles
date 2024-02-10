---
title: "Azure Blob Storageに登録されたファイルのメタ情報をAzure AI Searchのフィールドに登録する"
emoji: "ℹ️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "rag", "openai", "generativeai"]
published: true
---

# はじめに
近年盛り上がりを見せているGenerative AIのシステムを構築するためにRAG(Retrieval Augmented Generation)アーキテクチャが採用されることが多いです。RAGを構築するためには、データとそのデータを検索するための仕組みが必要です。
データは頻繁に更新されるため、その更新に追従して検索インデックスも更新する必要があります。その運用を楽にできるような仕組みとしてAzure Blob Storageに登録されたファイルを元にインデックスを生成する方法を調査しました。


# 使用するサービス
- Azure AI Search (旧 Azure Cognitive Search)
- Azure Blob Storage

# やりたいこと
Blob Storageに登録されたファイルのメタ情報がAI Searchのフィールドに自動的に反映されるようにしたいと考えております。
例えば、ファイルにカテゴリーや作成者の情報がメタ情報として登録されている場合にAI Searchのフィールドにこれら情報が反映されるようにしたいというケースがあります。カテゴリーなどの情報を検索時のフィルターに使用する場合を想定しております。



# メタデータの登録

Azureの公式サイトには以下のような記述があります。

> ファイルに関する追加情報を含める場合は、別のストアを使用せずに、メタデータを BLOB に直接関連付けることができます。 組み込みの Blob Storage 検索インデクサーは、このメタデータを読み取って、検索インデックスに配置することもできます。 これにより、ユーザーはファイル コンテンツと共にメタデータを検索できます。

[Azure Cognitive Search を使用してファイル コンテンツとメタデータのインデックスを作成する](https://learn.microsoft.com/ja-jp/azure/architecture/ai-ml/architecture/search-blob-metadata#searching-file-metadata)


今回はcategoryというメタデータをAI Searchのフィールドに登録するケースを考えます。

作成したストレージコンテナーのメタデータとしてcategoryというキー名のメタデータを作成します。
![](https://storage.googleapis.com/zenn-user-upload/541274d9d953-20240210.png)

そして、各ファイルのメタデータとしてcategoryというキーにして、具体的なカテゴリーを設定します。
![](https://storage.googleapis.com/zenn-user-upload/c46e93f9778a-20240210.png)


BlobストレージをデータソースとしてAI Searchのインデクサーを作成します。

categoryというフィールドが作成されていることが確認できました。
これにより、categoryを例えば検索のフィルターとして使用することができます。
![](https://storage.googleapis.com/zenn-user-upload/efa02a93bfb1-20240210.png)


# まとめ
Blob Storageにメタデータを設定し、AI SearchのデータソースとしてBlobストレージを選択してインデクサーを構成することでファイルのメタデータがAI Searchのフィールドにも反映できることを確認しました。

AI Searchの利用に関しては、こちらの記事もよければご覧ください。
[Azure Blob Storageに登録されたファイルを自動でAzure AI Searchのインデックスに登録する](https://zenn.dev/horitaka/articles/azure-ai-search-blobstorage1)


# 参考にしたサイト

- [Azure AI Search](https://learn.microsoft.com/ja-jp/azure/search/)
- [Azure Cognitive Search の全文検索を重点的に学習するワークショップを公開](https://qiita.com/nohanaga/items/2a90539f7667fa9e486a)
- [Azure-AI-Search-Workshop](https://github.com/nohanaga/Azure-AI-Search-Workshop)
- [Azure Cognitive Search を使用してファイル コンテンツとメタデータのインデックスを作成する](https://learn.microsoft.com/ja-jp/azure/architecture/ai-ml/architecture/search-blob-metadata)
