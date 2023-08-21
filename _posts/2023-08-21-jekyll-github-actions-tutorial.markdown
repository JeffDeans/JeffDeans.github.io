layout: post
title:  "通过 GitHub Pages 使用自定义工作流"
date:   2023-08-21 15:21:34 +0800
categories: jekyll update

# 通过 GitHub Pages 使用自定义工作流

## 关于自定义工作流

自定义工作流允许使用 GitHub Actions 生成 GitHub Pages 网站。 你仍然可以通过工作流文件选择要使用的分支，但使用自定义工作流可以执行更多操作。 若要开始使用自定义工作流，必须先为当前存储库启用它们。 有关详细信息，请参阅“[配置 GitHub Pages 站点的发布源](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)”。

## 配置 configure-pages 操作

GitHub Actions 支持通过 `configure-pages` 操作使用 GitHub Pages，这还允许你收集有关网站的不同元数据。 有关详细信息，请参阅 [`configure-pages`](https://github.com/marketplace/actions/configure-github-pages) 一文。

若要使用此操作，请将此代码片段放在所需工作流的 `jobs` 下。

```
- name: Configure GitHub Pages
  uses: actions/configure-pages@v3
```

此操作有助于支持从任何静态站点生成器部署到 GitHub Pages。 若要减少此过程的重复性，可以对一些最广泛使用的静态站点生成器使用入门工作流。 有关详细信息，请参阅“[使用入门工作流程](https://docs.github.com/zh/actions/using-workflows/using-starter-workflows)”。

## 配置 upload-pages-artifact 操作

通过 `upload-pages-artifact` 操作可以打包和上传项目。 GitHub Pages 项目应该是包含单个 `tar` 文件的压缩 `gzip` 存档。 `tar` 文件大小必须低于 10 GB，并且不应包含任何符号或硬链接。 有关详细信息，请参阅 [`upload-pages-artifact`](https://github.com/marketplace/actions/upload-github-pages-artifact) 一文。

若要在当前工作流中使用此操作，请将此代码片段放在 `jobs` 下。

```
- name: Upload GitHub Pages artifact
  uses: actions/upload-pages-artifact@v1
```

## 部署 GiHub Pages项目

`deploy-pages` 操作处理部署项目所需的设置。 为确保功能正常运行，应满足以下要求：

- 作业必须至少具有 `pages: write` 和 `id-token: write` 权限。
- `needs` 参数必须设置为生成步骤的 `id`。 不设置此参数可能会导致独立部署持续搜索尚未创建的项目。
- 必须建立 `environment` 以强制实施分支/环境保护规则。 默认环境为 `github-pages`。
- 若要将页面的 URL 指定为输出，请使用 `url:` 字段。

有关详细信息，请参阅 [`deploy-pages`](https://github.com/marketplace/actions/deploy-github-pages-site) 一文。

```
...

jobs:
  deploy:
    permissions:
      contents: read
      pages: write
      id-token: write
    runs-on: ubuntu-latest
    needs: jekyll-build
    environment:
      name: github-pages
      url: ${{steps.deployment.outputs.page_url}}
    steps:
      - name: Deploy artifact
        id: deployment
        uses: actions/deploy-pages@v1
...
```

## 链接单独的生成和部署文件

可以在单个工作流文件中链接 `build` 和 `deploy` 作业，无需创建两个单独的文件即可获得相同的结果。 若要开始使用工作流文件，可以在 `jobs` 下定义 `build` 和 `deploy` 作业以执行作业。

```
...

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Build with Jekyll
        uses: actions/jekyll-build-pages@v1
        with:
          source: ./
          destination: ./_site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{steps.deployment.outputs.page_url}}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
...
```

在某些情况下，可以选择将所有内容合并到单个作业中，尤其是在不需要生成过程的情况下。 因此，将只专注于部署步骤。

```
...

jobs:
  # Single deploy job no building
  deploy:
    environment:
      name: github-pages
      url: ${{steps.deployment.outputs.page_url}}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Pages
        uses: actions/configure-pages@v3
      - name: Upload Artifact
        uses: actions/upload-pages-artifact@v2
        with:
          # upload entire directory
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2

...
```

可以将作业定义为在不同的运行器上按顺序或并行运行。 有关详细信息，请参阅“[使用作业](https://docs.github.com/zh/actions/using-jobs)”。