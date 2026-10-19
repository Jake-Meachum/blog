# WangyeBlog

基于 [Hexo](https://hexo.io/zh-cn/) + [安知鱼（Anzhiyu）](https://docs.anheyu.com/) 主题的静态博客，目标是部署到 **阿里云 ESA（边缘安全加速）**，通过公网域名访问。

## 本地开发

需要 Node.js >= 20.19.0。

```bash
npm ci             # 安装依赖
npm run server     # 本地预览 http://localhost:4000
npm run build      # 生成静态文件到 public/
npm run clean      # 清理缓存与 public/
```

新建文章：

```bash
npx hexo new "文章标题"
```

文章位于 `source/_posts/`，主题自定义配置位于 `_config.anzhiyu.yml`。

## 部署到 ESA Pages

项目根目录的 `esa.jsonc` 已按 Hexo 配置好。ESA Pages 检测到该文件后会以它作为唯一构建配置：

```jsonc
{
  "name": "wangyeblog",
  "installCommand": "npm ci",
  "buildCommand": "npm run build",
  "assets": {
    "directory": "./public",
    "notFoundStrategy": "404Page"
  }
}
```

部署步骤：

1. 将本项目推送到 GitHub 仓库（`public/`、`node_modules/` 已在 `.gitignore` 中排除，主题文件随仓库提交）。
2. 登录阿里云控制台，进入 **边缘安全加速 ESA → 边缘计算 → 函数和 Pages**，选择从 GitHub 仓库导入。
3. 选择仓库和分支，确认构建配置来自 `esa.jsonc`，保存并部署。
4. 首次部署完成后会获得 `*.er.aliyun-esa.net` 形式的默认域名，可直接预览。
5. 在 **域名绑定** 中添加自己的域名，按控制台提示配置 CNAME；ESA 可申请免费边缘证书。
6. 若使用中国大陆加速区域，域名需先完成 ICP 备案。

注意事项：

- `esa.jsonc` 存在时，控制台中的构建命令、输出目录等配置会被忽略，修改构建配置需修改该文件并重新推送。
- Hexo 8 要求 Node.js >= 20.19.0，已通过 `package.json` 的 `engines` 字段声明。
- Pages 默认启用尾部斜杠：访问 `/about` 会 301 到 `/about/`。
- 更新主题或修改配置后重新推送主分支，ESA 会自动构建并部署。

参考文档：

- [函数和 Pages 产品概览](https://www.alibabacloud.com/help/zh/edge-security-acceleration/esa/user-guide/what-is-functions-and-pages/)
- [Pages 构建和路由指南](https://help.aliyun.com/zh/edge-security-acceleration/esa/user-guide/build-pages)
- [从 GitHub 仓库创建 Pages](https://www.alibabacloud.com/help/en/edge-security-acceleration/esa/user-guide/connect-pages-to-github)

## 备选方案：OSS + ESA

若不使用 Pages，也可以将 `public/` 同步到 OSS 静态网站托管，再把 OSS Bucket 作为 ESA 的源站并绑定域名加速。Pages 方案无需维护 Bucket，推荐优先使用。
