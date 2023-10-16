# docker
```
docker-compose exec web bundle install

docker-compose ps -q web # コンテナIDの確認
docker attach コンテナID

# pryでデバッグできる（変数の確認など）
Rails.env
show-source Rails

ls bin/docker-compose-attach

docker-compose exec web ./bin/rails console # Rails consoleでpryを利用可能になる
docker-compose exec web ./bin/rails console -s # サンドボックスモード
[1] pry(main)> show-source Post.has_rich_text

```

# モデルの相関についての確認（action_text_rich_textsテーブル）
```
post = Post.find(3)
content = post.content

# articleモデルを作成（複数のモデルにhas_rich_textが利用されているケースを確認するため）
docker-compose exec web ./bin/rails generate model article
docker-compose exec web ./bin/rails db:migrate

article = Article.create
statement = article.statement
statement.save!

# ポリモーフィックアソシエーション
show-source ActionText::RichText
# belongs_to :record, polymorphic: true, touch: true
```

# モデルの相関について（active_storage_attachments）
```
show-source ActionText::RichText
# has_many_attached :embeds

post = Post.find(3)
content = post.content
content.embeds_attachments # rich_textが保持する画像データ
attachments = content.embeds_attachments
attachment = attachments.first

show-source ActiveStorage::Attachment # 多対多の関係を確認
  belongs_to :record, polymorphic: true, touch: true
  belongs_to :blob, class_name: "ActiveStorage::Blob"

blob = content.embeds_blobs.first
open storage/gs/tu/gstuw6kr899e67mnl1zq4oag6zrs -a preview # 画像ファイルを確認 
```

# Validation
```
validates :title, length: { maximun: 32 }

docker-compose exec web ./bin/rails console -s

post = Post.new
post.title = "aaaaaaaaaassssssssssddddddddddffffffffffffffffffffffffff"
post.valid?
post.errors
post.title.chop
post.title.chop!


reload!
```

# 国際化
https://guides.rubyonrails.org/i18n.html#configure-the-i18n-module

# バリデーション 投稿の長さ
```
post = Post.find(4)

```

# 添付ファイルのサイズ　　バリデーション
```
docker-compose exec web ./bin/rails console -s
show-source ActionText::RichText
content = post.content
content.body.attachables
attachable = content.body.attachables.grep(ActiveStorage::Blob).first
attachable.byte_size
attachable.byte_size / (1024 * 1_000)
attachable.byte_size / (1024 * 1_000.0)
```


# 添付ファイル数のバリデーション
```
docker-compose exec web ./bin/rails console -s
post = Post.find(3) 
show-source ActionText::RichText
post.content.body .attachables
post.content.body .attachables.grep(ActiveStorage::Blob)
post.content.body .attachables.grep(ActiveStorage::Blob).count
```

# ゴミデータをぶっ壊す！
```
ActiveStorage::Blob.count
blob_ids = ActiveStorage::Blob.pluck(:id)
ActiveStorage::Attachment
ActiveStorage::Attachment.pluck(:blob_id)
_blob_ids = ActiveStorage::Attachment.pluck(:blob_id)
unreferenced_blob_ids = blob_ids - _blob_ids
ActiveStorage::Blob.where(id: unreferenced_blob_ids)
ActiveStorage::Blob.where(id: unreferenced_blob_ids).delete_all
```