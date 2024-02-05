---
title: "Azure Blob Storageに登録されたファイルを自動でAzure AI Searchのインデックスに登録する"
emoji: "🚙"
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
Blob Storageにファイルが登録されたら自動的にインデックスが作成されてAzure AI Searchに登録されるようにしたいです。
具体的には以下の3つを実現したいです。
1. ファイルが新規登録されたらインデックスが新規登録されること
2. ファイルが更新されたらインデックスが更新されること
3. ファイルが削除されたらインデックスが削除されること

つまり、Blob Storageに登録されているファイルとAzure AI Searchのインデックスが同期されるようにしたいです。


# 構築

Azure Blob StorageをデータソースとしてAzure AI Searchでインデックスを作成する手順の説明は他に良いサイトがあるため本記事では詳細は割愛いたします。
以下のページがわかりやすいと思いますのでご参考にしていただければと思います。

[Azure-AI-Search-Workshop - ノーコード全文検索インデックスの作成](https://github.com/nohanaga/Azure-AI-Search-Workshop/blob/main/CreateIndex.md)


# 結果

## 新規追加

新規追加はBlob Storageにファイルを追加して、インデクサーの実行をするだけです。
インデクサーの実行タイミングは最低5分間隔から選択することができます。都度実行することも可能です。

![](https://storage.googleapis.com/zenn-user-upload/360f231e8e65-20240205.png)

## 更新

更新については公式サイトに以下のように記述されています。

> Azure Storage から生成されたインデックス付きコンテンツに対しては、変更検出が自動的に行われます。これは、インデクサーが Azure Storage のオブジェクトとファイルに組み込まれたタイムスタンプを使用して最終更新を追跡しているためです。

[Azure AI Search の Azure Storage のインデクサーを使用した変更検出と削除検出](https://learn.microsoft.com/ja-jp/azure/search/search-howto-index-changed-deleted-blobs)


そこでファイルを更新し、Blob Storageにアップロードし直してみます。
そしてインデクサーを再実行するとインデクスが更新されていることが確認できました。


## 削除

削除については公式サイトに以下のように記述されています。

> 変更の検出は指定されていますが、削除検出は指定されていません。 インデクサーはデータ ソースのオブジェクトの削除を追跡しません。 孤立した検索ドキュメントが含まれないようにするには "論理的な削除" 方法を実装できます。これにより、最初に検索ドキュメントを削除した後に Azure Storage での物理的な削除が行われます。
> 
> 論理的な削除方法を実装するには、2 つの方法があります。
> 
>  - ネイティブ BLOB の論理的な削除。Blob Storage にのみ適用されます
>  - カスタム メタデータを使用した論理的な削除

[Azure AI Search の Azure Storage のインデクサーを使用した変更検出と削除検出](https://learn.microsoft.com/ja-jp/azure/search/search-howto-index-changed-deleted-blobs)

今回は「ネイティブ BLOB の論理的な削除」の方法を試してみます。
以下のように**削除の追跡**にチェックを入れて、**ネイティブ BLOB の論理的な削除**を選択します。

![](https://storage.googleapis.com/zenn-user-upload/5cb6fbe8ada8-20240205.png)


この状態でBlob Storageからファイルを削除すします。
そしてインデクサーを再実行するとインデクスが削除されていることが確認できました。


# まとめ
Azure AI SearchのデータソースとしてBlob Storageを選択することで、ストレージ上のファイルとインデックスが同期できることがわかりました。
Azure AI Searchにはスキルと呼ばれる仕組みを追加してファイルを様々に加工してインデックスを作成することも可能です。次回はスキルについても調査したいと思います。


# 参考にしたサイト

- [Azure AI Search](https://learn.microsoft.com/ja-jp/azure/search/)
- [Azure Cognitive Search の全文検索を重点的に学習するワークショップを公開](https://qiita.com/nohanaga/items/2a90539f7667fa9e486a)
- [Azure-AI-Search-Workshop](https://github.com/nohanaga/Azure-AI-Search-Workshop)