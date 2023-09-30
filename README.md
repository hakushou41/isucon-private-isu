## 概要

isucon13の練習として、private-isuを実施する
https://github.com/catatsuy/private-isu#private-isu

## 期日

10/17 3-shakeの感想戦まで

## 前提

golang使用

docker-compose.yaml mysqlについて、デフォルトのcpu1,memory:1gだと
初回ベンチマークがタイムアウトするため、cpu2,memory:2gのベンチマークを初期値とした。

# やったこと

## 初回ベンチ

```
❯ docker run --network host -i private-isu-benchmarker /opt/go/bin/benchmarker -t http://host.docker.internal -u /opt/go/userdata
{"pass":true,"score":2553,"success":2450,"fail":0,"messages":[]}
```
