---
title: (演講筆記)Getting Started with AWS Lambda and the Serverless Cloud
tags: [talks, aws, lambda, serverless]
---

# 迷思

- 只能用簡單的演算法嗎？
    - 不， function 可以很複雜，簡單的是基礎建設

- 資料有辦法 persistent 嗎？

# 優點

不用管 server

scale-out

沒人需要運算資源的時候就不用付錢

# Pricing

免費：一百萬個 requests, 400000 GBs 每個月

# Using AWS Lambda

Stateless DynamoDB, S3 or ElasticCache


# API Gateway

不只可以接到 lambda functions

也可以接到 EC2

好處：

- 統一的 API 格式

- DDoS protection

- Authenticate and authorize requests

# Use cases

# Reference

- [來源](https://www.youtube.com/watch?v=g5PNX-8MRt0)
