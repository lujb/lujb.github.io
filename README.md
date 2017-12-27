## Init
```bash
git clone https://github.com/lujb/lujb.github.io
cd lujb.github.io
git checkout source
mkdir themes
git clone https://github.com/lujb/hexo-theme-vexo themes/vexo
```

## Writing
```bash
hexo new post hello_post
hexo new page hello_post
hexo new draft hello_draft
hexo publish hello_draft
```

## Deploy
```bash
hexo deploy --generate
```
or
```bash
hexo generate
hexo deploy
```

## Commit
```bash
git push origin source
```