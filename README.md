# Personal site

https://gohugo.io/hosting-and-deployment/hosting-on-github/g

```
hugo
cd public && git add --all && git commit -m "Generate site" && cd ..
git push origin source
git push origin master
```

## [Hugo](https://github.com/gohugoio/hugo)

### Create new posts
```
hugo new posts/my-first-post.md
```

### Start the Hugo server
```
hugo server -D
```

### Build static pages
```
hugo
```