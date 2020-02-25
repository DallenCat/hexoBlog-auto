
---

title: 测试语雀能否同步到github

urlname: cn6z5u

date: 2020-02-22 13:54:26 +0800

tags: []

---

```javascript
 "yuqueConfig": {
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "u46795",
    "repo": "blog",
    "mdNameFormat": "title",
    "postPath": "source/_posts/yuque"
  },
```
<!--more-->

```javascript
 "scripts": {
    "sync": "yuque-hexo sync",
    "clean:yuque": "yuque-hexo clean",
    "deploy": "npm run sync && hexo clean && hexo g -d",
  },
```


