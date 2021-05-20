# My Blog

Auto deploy to [https://zaaack.github.io](https://zaaack.github.io) via Github Actions.



## How to deploy hexo site to Github Pages via Github Actions

1. run `ssh-keygen -f ./test` to generate ssh keys
2. put `test.pub` content to [zaaack.github.io's deploy keys](https://github.com/zaaack/zaaack.github.io/settings/keys)
3. put `test` content to [blog's secrets](https://github.com/zaaack/blog/settings/secrets/actions) with name `SSH_PRIVATE_KEY`
