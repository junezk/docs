---
root: true
name: FAQ
sort: 99
---

# FAQ

1. Can't find the template files or configuration files or nil pointer error?

    It may be because you used `go run main.go` to run your application. `go run` will compile the file and put it into a tmp folder to run it. But Beego needs the static files, templates and config files. So you need to use `go build` and run the application by `./app`. Or you can use `bee run app` to run your application.

2. Can Beego be used for production?

    Yes. Beego has been used in production. E.g.: SNDA's CDN system, 360 search API, Bmob mobile cloud API, weico backend API etc. They are all high concurrence and high performance applications. 

3. Will the future upgrades affect the API I am using right now?

    Beego is keeping the stable API since version 0.1. Many applications upgraded to the latest Beego easily. We will try to keep the API stable in the future.

4. Will Beego keep developing?

    Many people are worried about open source projects that stop developing. We have four people who are contributing to the code. We can keep making Beego better and better.

5. Why I got "github.com/beego/beego/v2" package not found error?

    In BeegoV2, we are using go mod. So you must enable go module feature in your environment. In general, you should set `GO111MODULE=on`.

6. Why I always got i/o timeout when I run `go get github.com/beego/beego/v2`?

    It means that your network has some problem. Sometimes it was caused by the firewall. If you are in China, this is a common case,
    and you could set `GOPROXY`, for example: `export GORPOXY=https://goproxy.cn"`
