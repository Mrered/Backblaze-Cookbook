---
toc:
  depth_from: 2
  depth_to: 6
  ordered: true
output: 
  pdf_document:
    extra_dependencies: ["esint"]
---

<!-- markdownlint-disable MD029 MD033 MD049-->

# Backblaze 的食用方法

[TOC]

## 太长不看

1. 创建一个公开的 B2 存储桶，并记录如下信息，此处以本教程为例；

|       名称       |                值                |
|:----------------:|:--------------------------------:|
|   `存储桶名称`   |          你的存储桶名称          |
|    `Endpoint`    | `s3.us-west-004.backblazeb2.com` |
|      `地区`      |          `us-west-004`           |
|    `友好 URL`    |      `f004.backblazeb2.com`      |
|     `KeyID`      |             `KeyID`              |
| `applicationKey` |         `applicationKey`         |

2. 在 Cloudflare 设置 `CNAME` 解析上述 `友好 URL` ；
3. 在 Cloudflare 部署 `转换规则` ，

```makefile
(http.request.uri.path ne "/file/<你的存储桶名称>" and http.host eq "<上述 CNAME 解析域名>")
```

动态重写到

```makefile
concat("/file/<你的存储桶名称>",http.request.uri.path)
```

4. 在 PicGo 中安装 S3 插件，按下表填写相关信息

## 前言

### 为什么会有此烹饪指南？

1. [Backblaze](https://www.backblaze.com/) 的 [B2 对象存储](https://www.backblaze.com/b2/cloud-storage.html)和 Cloudflare CDN 搭配食用口味奇香，因为 Cloudflare CDN 个人使用完全免费，且上述两家公司均属于 [Bandwidth Alliance](https://www.cloudflare.com/zh-cn/bandwidth-alliance/) 成员，双方数据传输不收费；
2. B2 对象存储因其前 10 GB 存储空间免费和低廉的价格，加上第一条所言，非常划算！

<figure><img src="https://file.uijxmug.top/2023/06/fc54f7b6832167c63289b700a3ab6acb.png" alt="20230621210852" style="max-width:100%; display: block; margin: 0 auto;"></figure>

<figure><img src="https://file.uijxmug.top/2023/06/1faec6b1199c066e58621b7eca60cddd.png" alt="20230621210259" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 什么是对象存储？（本节内容来自 ChatGPT）

对象存储（Object Storage）是一种数据存储架构，用于存储和管理大规模、非结构化数据。与传统的文件系统和块存储相比，对象存储将数据存储为独立的对象，每个对象都具有唯一的标识符（称为键），并通过使用元数据来描述对象的属性和关系。

对象存储的主要特点是：

1. 可扩展性：对象存储具有高度可扩展性，能够处理大规模数据的存储需求。它采用分布式架构，可以将数据分散存储在多个物理设备或服务器上，实现数据的横向扩展。
2. 强大的元数据管理：对象存储使用元数据来描述和组织数据对象，使得数据的管理更加灵活和高效。元数据可以包括对象的属性、标签、权限、访问控制信息等，使得对象的检索和管理更加方便。
3. 高可靠性：对象存储系统通常具备高度可靠性和冗余机制，通过数据的多副本存储和自动数据修复等技术来保证数据的可靠性和可用性。即使发生硬件故障或数据损坏，系统仍能保证数据的完整性。
4. 适应海量数据：对象存储适用于存储海量的非结构化数据，如图像、音频、视频、日志文件等。它提供高效的数据访问和存储管理，可以处理大规模数据的存储和分发需求。

尽管对象存储有很多优点，但也存在一些缺点：

1. 相对较高的延迟：与传统的块存储或文件系统相比，对象存储通常具有较高的读取和写入延迟。这主要是由于对象存储需要访问元数据和进行网络传输等额外的开销所导致的。
2. 不适合频繁的小规模写入：对象存储更适用于大规模数据的批量写入和读取，而对于频繁的小规模写入操作，其性能可能相对较低。这是因为对象存储系统通常需要处理较大的数据块，以提高数据的读取和写入效率。
3. 较高的存储成本：尽管对象存储系统提供了高度可靠性和冗余机制，但相应地也会增加存储成本。存储大规模数据需要购买更多的硬件设备，并为数据的多副本存储付出额外的成本。

需要根据具体的应用场景和需求来评估对象存储的优缺点，并权衡是否适合采用对象存储来满足存储需求。

### 什么是 Backblaze ？（本节内容来自 ChatGPT）

Backblaze是一家知名的云存储服务提供商，提供可靠的备份和存储解决方案。他们的主打产品是Backblaze云备份（Backblaze Cloud Backup）和Backblaze B2云存储（Backblaze B2 Cloud Storage）。

Backblaze云备份是一种面向个人用户和小型企业的在线备份服务。它允许用户将其计算机上的文件自动备份到Backblaze的云存储中，以确保数据的安全性和可恢复性。用户只需安装Backblaze的客户端软件，并设置好备份选项，即可将文件安全地存储在远程服务器上。Backblaze云备份提供了简单易用的界面和功能，包括自动备份、增量备份、文件版本历史记录、文件恢复等。

Backblaze B2云存储是一种面向开发者和企业的弹性对象存储服务。它提供了可扩展、高可靠性和经济高效的云存储解决方案。开发者可以通过使用B2 API或集成了B2兼容性的第三方工具和软件，将大规模的非结构化数据存储到Backblaze的云存储中。Backblaze B2具有灵活的存储选项、高度可靠的数据保护机制和成本效益等优点，适用于各种应用场景，如备份和归档、大规模数据存储、媒体和内容存储等。

总体而言，Backblaze以其简单易用、经济高效以及强大的可靠性而备受用户青睐。他们提供了强大的云存储解决方案，满足了个人用户、小型企业和开发者对数据备份和存储的需求。

### 为什么选择 Backblaze B2 对象存储？（本节内容来自 ChatGPT）

Backblaze在对象存储领域有以下优点，使其成为用户选择的理想之一：

1. 成本效益：Backblaze以其经济高效的定价策略而著名。他们提供透明且竞争力的定价模型，不收取额外的数据传输费用和请求费用。这使得用户能够更好地控制存储成本，并在预算内实现数据存储和备份需求。
2. 简单易用：Backblaze注重用户体验，提供简单易用的界面和工具，使得用户可以轻松设置和管理他们的数据备份和存储。他们的解决方案专注于简化操作流程，无论是个人用户、小型企业还是开发者，都能快速上手并使用。
3. 可靠性和持久性：Backblaze拥有高度可靠的基础设施和数据冗余机制。他们的数据中心分布在多个地理位置，并采用了数据多副本存储和自动修复等技术来确保数据的可靠性和持久性。这意味着用户的数据将受到强大的保护，即使发生硬件故障或数据损坏，也能进行恢复。
4. 扩展性和灵活性：Backblaze B2云存储提供了高度可扩展的存储选项，适用于各种规模的数据存储需求。用户可以根据实际需求灵活地扩展存储容量，而无需担心性能和可用性问题。此外，Backblaze B2支持多种第三方工具和软件的集成，使开发者能够更好地集成和管理他们的应用程序和数据存储。

综上所述，Backblaze以其成本效益、简单易用、可靠性和灵活性而脱颖而出。如果您注重经济性、用户友好性和可靠性，并且需要一个可扩展的对象存储解决方案，那么选择Backblaze可能是一个明智的选择。然而，具体选择供应商还应根据您的具体需求、预算和优先级进行评估和比较。

### 个人使用对象存储能实现什么？（本节内容来自 ChatGPT）

个人用户使用对象存储可以实现以下功能和应用：

1. 数据备份：个人用户可以将自己计算机上的重要文件和数据备份到对象存储中。这样，即使计算机遭遇故障、病毒攻击或意外删除，用户仍然可以轻松恢复其数据。对象存储通常提供自动备份和增量备份功能，使备份过程更加便捷和高效。
2. 文件共享和远程访问：通过对象存储，个人用户可以轻松共享文件和数据，并在不同设备之间实现远程访问。用户可以上传文件到对象存储中，并通过生成共享链接或设置访问权限，与他人共享文件。这对于与朋友、家人或同事分享照片、视频、文档等文件非常有用。
3. 文件存档和长期保存：个人用户可以利用对象存储来存档和长期保存重要的文件和数据。对象存储通常具有高度可靠的数据保护机制和持久性，确保数据的安全性和可用性。这对于存储家庭影片、珍贵照片、重要文档、个人日记等数据具有重要意义。
4. 多媒体存储和流媒体：对象存储可用于存储个人的音乐、视频、播客等多媒体内容。用户可以上传自己的音乐库、个人创作视频或其他媒体文件，并随时通过网络进行播放和分享。对象存储通常提供可靠的数据传输和高带宽支持，适用于流媒体应用。
5. 数据归档和整理：个人用户可以利用对象存储来整理和归档大量的非结构化数据。例如，整理和存储电子邮件附件、社交媒体数据、个人博客、论坛帖子等。对象存储的元数据管理功能可帮助用户对数据进行分类、搜索和管理。

总而言之，个人用户可以使用对象存储来进行数据备份、文件共享、远程访问、文件存档、多媒体存储和流媒体、数据归档和整理等一系列应用。对象存储为个人用户提供了可靠的数据存储和管理平台，以保护和利用他们的重要数据。

### 什么是图床/文件床？（本节内容来自 ChatGPT）

图床（Image Hosting）或文件床（File Hosting）是指用于存储和托管图像或文件的在线服务。它允许用户将图像或文件上传到云存储服务器，并获得一个可访问的直链URL，用于在网页、论坛、社交媒体等地方分享和引用。

图床/文件床的需求主要出于以下几个方面：

1. 避免流量消耗：将图像或文件上传到图床/文件床后，用户可以使用直链URL在网页或社交媒体上引用它们，而不会占用自己的网站或个人服务器的带宽和流量。这对于网站和个人博客来说特别重要，可以提高网站的加载速度和用户体验。
2. 跨平台共享：图床/文件床提供了一个统一的存储和共享平台，可以轻松地在不同的网页、论坛或社交媒体平台上共享图像和文件。这样用户可以方便地在不同的社交媒体上分享图片、在论坛上发布帖子、或在网页上插入图片等。
3. 持久性和可靠性：图床/文件床通常采用可靠的云存储解决方案，具有高度的数据持久性和可靠性。即使用户的网站或个人设备发生故障或数据丢失，上传到图床/文件床的图像和文件仍然可以正常访问和使用。

### 使用对象存储作为图床/文件床有什么优势？（本节内容来自 ChatGPT）

对象存储作为图床/文件床的选择有以下优势：

1. 可扩展性和容量：对象存储提供了高度可扩展的存储选项，可以满足图像和文件的大规模存储需求。它可以轻松处理大量的图像和文件，并且可以根据需要灵活地扩展存储容量。
2. 高速访问和传输：对象存储通常具有高速的数据访问和传输性能，使得图像和文件可以快速加载和下载。这对于在网页上显示图片或提供下载链接的需求非常重要。
3. 强大的可靠性和冗余机制：对象存储系统通常具备高度可靠性和冗余机制，能够保证图像和文件的数据完整性和可用性。它们会自动进行数据多副本存储和数据修复，以应对硬件故障和数据损坏的情况。

虽然GitHub、Gitee或其他支持直链的网盘也可以用作图床/文件床，但相比之下，对象存储在大规模存储、可靠性、可扩展性和性能方面更具优势。对象存储专注于提供高效、经济且可靠的数据存储解决方案，适用于图床/文件床等场景。而GitHub、Gitee或其他网盘服务则更适合代码版本控制、项目管理和协作等用途，它们的设计目标和重点略有不同。选择对象存储作为图床/文件床取决于用户对存储性能、可靠性和扩展性的需求。

## 注册 Backblaze 和创建 B2 存储桶

### 注册 Backblaze

1. 直接进入 [B2 对象存储](https://www.backblaze.com/b2/cloud-storage.html) 页面注册账号，红框位置选择 `US West` ；

<figure><img src="https://file.uijxmug.top/2023/06/d32ab7e75c21874e7a314dea2fe8aba4.png" alt="20230621211830" style="max-width:60%; display: block; margin: 0 auto;"></figure>

2. 在登陆页面将语言改为 `简体中文` 。

<figure><img src="https://file.uijxmug.top/2023/06/dd055373bf13be8a0d5229f92b022e70.png" alt="20230621212012" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 创建 B2 存储桶

1. 点击 `创作一个桶` ；

<figure><img src="https://file.uijxmug.top/2023/06/ce2ed07cc2f83cb8041c3343cc2a0fa6.png" alt="20230621212233" style="max-width:100%; display: block; margin: 0 auto;"></figure>

2. 输入桶的名字需要全球唯一， `桶里面的档案是` 选择 `公众` ，点击下方 `创作一个桶` ；

<figure><img src="https://file.uijxmug.top/2023/06/4db66d1dfe966cbd52b1d026a20df37a.png" alt="20230621212418" style="max-width:100%; display: block; margin: 0 auto;"></figure>

3. 稍等片刻创建成功后，以下图为例，记录如下内容以备后用；

|     名称     |                值                |
|:------------:|:--------------------------------:|
| `存储桶名称` |        你刚刚给它命名了的        |
|  `Endpoint`  | `s3.us-west-004.backblazeb2.com` |
|    `地区`    |          `us-west-004`           |

<figure><img src="https://file.uijxmug.top/2023/06/ed5c9d96e9ac97865c1c8303b09e413f.png" alt="20230621212834" style="max-width:100%; display: block; margin: 0 auto;"></figure>

4. （此步骤可选）点击 `桶设定` ， `桶信息` 填入 `{"cache-control":"max-age=86400"}` ，点击 `更新桶` ；

<figure><img src="https://file.uijxmug.top/2023/06/facd7ed946bc3816c35c14408ad294c0.png" alt="20230621213402" style="max-width:100%; display: block; margin: 0 auto;"></figure>

5. 点击 `CORS 规则` ，按照下图选择，点击 `Update CORS Rules` ；

<figure><img src="https://file.uijxmug.top/2023/06/aa8bb96efaca1238471622a5c61ec8de.png" alt="20230621213808" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 获取友好 URL

1. 点击左侧 `浏览档案` ，进入刚刚创建的存储桶，点击 `上载` 并随便选择一个文件传上去，点击下图❹位置的红圈内的 `i` ；

<figure><img src="https://file.uijxmug.top/2023/06/3d2a4560359d56671c5f234d55e6a87d.png" alt="20230621215807" style="max-width:100%; display: block; margin: 0 auto;"></figure>

2. 以下图为例，记录如下内容以备后用；

|    项目    |           值           |
|:----------:|:----------------------:|
| `友好 URL` | `f004.backblazeb2.com` |

<figure><img src="https://file.uijxmug.top/2023/06/1da0ee834b3ec1260c94c7df3e267637.png" alt="20230621220330" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 创建应用密钥

1. 点击左侧的 `应用密钥` ；

<figure><img src="https://file.uijxmug.top/2023/06/71881a10156bc4d6fff12b53e3286634.png" alt="20230621214301" style="max-width:100%; display: block; margin: 0 auto;"></figure>

2. 向下滑动点击 `添加新的应用程序密钥` ；

<figure><img src="https://file.uijxmug.top/2023/06/12c6072bdcd5ed62281cdaa15b5cdc28.png" alt="20230621214351" style="max-width:100%; display: block; margin: 0 auto;"></figure>

3. 输入 `密钥的名称` ， `允许访问Bucket（s）` 选择刚刚创建的存储桶，==注意红框位置必须和下图一致==，点击 `Create New Key` ；

<figure><img src="https://file.uijxmug.top/2023/06/34e42b25ec5ef9411fd46c0f149f5489.png" alt="20230621214541" style="max-width:100%; display: block; margin: 0 auto;"></figure>

4. ==此内容仅会出现一次==，若操作失误已无法查看，重新创建密钥即可。记录 `KeyID` 和 `applicationKey` 以备后用。

<figure><img src="https://file.uijxmug.top/2023/06/cd9692df780863e373bf48fed33f8ef2.png" alt="20230621214834" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 备用信息汇总

整理一下上面获取到的所有关键信息：

|       名称       |                值                |
|:----------------:|:--------------------------------:|
|   `存储桶名称`   |          你的存储桶名称          |
|    `Endpoint`    | `s3.us-west-004.backblazeb2.com` |
|      `地区`      |          `us-west-004`           |
|    `友好 URL`    |      `f004.backblazeb2.com`      |
|     `KeyID`      |             `KeyID`              |
| `applicationKey` |         `applicationKey`         |

## Cloudflare 中的配置

### 为存储桶开启 Cloudflare CDN 功能

> 此处默认读者已经拥有了自己的 Cloudflare 账号，如没有请自行申请。
> 自行查询如何添加 DNS 解析记录（我在之前的教程中详细写过，此教程不再赘述）

添加一条 `CNAME` 解析记录，目标为 `友好 URL` ， `代理状态` 下面的开关打开。

<figure><img src="https://file.uijxmug.top/2023/06/fc45c1ef7d3069ad6cde13e6f8cc8962.png" alt="20230621222105" style="max-width:100%; display: block; margin: 0 auto;"></figure>

### 使用 Cloudflare 隐藏直链中的存储桶名称

1. 点击左侧 `转换规则` ，再点击右侧 `创建规则` ；

<figure><img src="https://file.uijxmug.top/2023/06/17ee02c9e572860e594eaf397b41dfcb.png" alt="20230621223030" style="max-width:100%; display: block; margin: 0 auto;"></figure>

2. 填写 `规则名称（必需）` ；

<figure><img src="https://file.uijxmug.top/2023/06/d68ddb6c0b449aa8ed06048453859b9d.png" alt="20230621223347" style="max-width:100%; display: block; margin: 0 auto;"></figure>

3. 按照下图填写，其中 `URI 路径` 值为 `/file/<你的存储桶名称>` ， `主机名` 值为 `b2.<你的域名>` ；

<figure><img src="https://file.uijxmug.top/2023/06/01133d7fddba16371bb9bce6707a2fb4.png" alt="20230621223558" style="max-width:100%; display: block; margin: 0 auto;"></figure>

4. 按照下图填写，其中 `重写到...` 选 `Dynamic` ，右侧填写下述内容

```makefile
concat("/file/<你的存储桶名称>",http.request.uri.path)
```

<figure><img src="https://file.uijxmug.top/2023/06/81dbc7351b06c341958e9d6460f9046f.png" alt="20230621223911" style="max-width:100%; display: block; margin: 0 auto;"></figure>

5. 点击右下角 `部署` ，等待片刻完成部署。
