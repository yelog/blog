# 叶落阁博客

访问地址: [叶落阁](https://www.yelog.org)

# 本地启动

```bash
# 下载 blog 源码
git clone git@github.com:yelog/blog.git
# 进入 blog
cd blog
# 下载 3-hexo 主题
git clone git@github.com:yelog/hexo-theme-3-hexo.git themes/3-hexo
# 安装依赖, 如果没有 pnpm , 直接使用 npm install
pnpm install
# 启动
hexo g && hexo s
```

