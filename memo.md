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
```