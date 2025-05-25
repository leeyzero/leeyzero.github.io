---
title: Gin简明教程
date: 2021-08-29 16:23:26
categories:
- 编程匠艺
tags:
- Web
- Go
- 框架
- Gin
---

Gin是一个用Go语言实现的高性能web框架，其API实现优雅，性能卓越，上手简单，而且比较轻量级，很容易引入到项目中。本文是一篇对Gin框架的学习笔记，不会对Gin框架的用法做面面俱到的，而是对Gin框架做一个整体上的认识与入门。

## 安装

1、首先需要先安装[Go](https://golang.org/)，依赖版本为1.13+，执行下面命令安装Gin：

```shell
$ go get -u github.com/gin-gonic/gin
```

2、在代码中引入Gin包

```Go
import "github.com/gin-gonic/gin"
```

<!--more-->

## 快速入门

1、新建一个项目
```shell
$ mkdir example
$ touch example/main.go
```

2、hello world

```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.String("hello world")
	})
	r.Run() // 默认端口号为8080
}
```

3、测试

在一个终端先运行hello-world服务器程序：

```shell
$ go run main.go
```

在另一个终端发起http请求：

```shell
$ curl http://localhost:8080
```

## 路由

1、HTTP请求方法

Gin中封装了常用的HTTP方法，如GET, POST, PUT, PATCH, DELETE and OPTIONS：

```Go
func main() {
	// 默认路由器
	router := gin.Default()

	router.GET("/get", GetHandler)
	router.POST("/post", PostHandler)
	router.PUT("/put", PutHandler)
	router.DELETE("/delete", DeleteHandler)
	router.PATCH("/patch", PatchHandler)
	router.HEAD("/head", HeadHandler)
	router.OPTIONS("/options", OptionsHandler)

	router.Run("8080")
}
```


2、分组路由

有时我们需要将一组API放在一起，这些方便管理和阅读

```Go
func main() {
	router := gin.Default()

	// Simple group: v1
	v1 := router.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.GET("/read", readEndpoint)
	}

	// Simple group: v2
	v2 := router.Group("/v2")
	{
		v2.POST("/login", loginEndpoint)
		v2.POST("/submit", submitEndpoint)
		v2.GET("/read", readEndpoint)
	}

	router.Run(":8080")
}
```

## 请求参数

1、query参数

query参数是指在uri中问号后面的参数，如`/welcome?name=Jone`中query参数为`name=Jone`，Gin中可以通过Context包中的Query和DefaultQuery获取Query参数，他们的区分是DefautlQuery可以指定默认参数，如下：
```go
func main() {
	router := gin.Default()

	// The request responds to a url matching:  /welcome?firstname=Jane&lastname=Doe
	router.GET("/welcome", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")
		lastname := c.Query("lastname") 
		c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
	})
	router.Run(":8080")
}
```

测试：

```shell
$ curl http://localhost:8080/welcome?firstname=Jane&lastname=Doe
```

2、urlencoded form参数

跟获取query参数类似，Gin中可以通过Context包中的PostForm和DefaultPostForm获取post表单中的参数，如下：

```Go
func main() {
	router := gin.Default()
  
	router.POST("/form_post", func(c *gin.Context) {
		name := c.DefaultPostForm("name", "anonymous")
		message := c.PostForm("message")
		c.String(200, fmt.Sprintf("%s posted %s", name, message))
	})
  
	router.Run(":8080")
}
```

测试：

```shell
$ curl -d'name=John&message=hello' http://localhost:8080/form_post
```

3、路径中参数
使用`:name`匹配uri路径中的参数名为`name`的必选参数；使用`*name`匹配uri路径中的参数名为`name`的可选参数，如下：

```go
func main() {
	router := gin.Default()

	// This handler will match /user/john but will not match /user/ or /user
	router.GET("/user/:name", func(c *gin.Context) {
		name := c.Param("name")
		c.String(http.StatusOK, "Hello %s", name)
	})

	// However, this one will match /user/john/ and also /user/john/send
	// If no other routers match /user/john, it will redirect to /user/john/
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
    c.String(http.StatusOK, name + " is " + action)
	})
 	
  router.Run(":8080")
}
```

测试：

```shell
$ curl http://localhost:8080/user/John
$ curl http://localhost:8080/user/John/send
```

## 绑定

绑定是Gin框架非常有用的特性，可以简单理解为将请求BODY映射到一个内存模型（数据结构）上。支持的格式包括JSON、XML、YAML和标准的urlencoded form参数。

Gin提供两种绑定类型，`MustBind`和`ShouldBind`，对于MustBind，Gin提供了Bind、BindJSON、BindXML、BindQuery、BindYAML、BindHeader；对于ShouldBind，Gin提供跟MustBind配套的API，如ShouldBind、ShouldBindJSON，...。MustBind和ShouldBind的区别是：

如果MustBind绑定失败，请求会被`c.AbortWithError(400, err).SetType(ErrorTypeBind)` 中止，也就是状态码返回400，`Content-Type`被设置为`text/plain; charset=utf-8`；而ShouldBind绑定失败，会返回一个错误，由于程序员决定如何处理。

以下以绑定form和json为例：

```go
type Login struct {
	User     string `form:"user" json:"user" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}

func main() {
	router := gin.Default()

	// Example for binding JSON ({"user": "manu", "password": "123"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var json Login
		if err := c.ShouldBindJSON(&json); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

// Example for binding a HTML form (user=manu&password=123)
	router.POST("/loginForm", func(c *gin.Context) {
		var form Login
		// This will infer what binder to use depending on the content-type header.
		if err := c.ShouldBind(&form); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})
	
	router.Run(":8080")
```

测试：
```shell
curl -v -X POST \
  http://localhost:8080/loginJSON \
  -H 'content-type: application/json' \
  -d '{"user": "manu"}'
  
  curl -v -X POST \
  http://localhost:8080/loginForm \
  -H 'content-type: application/x-www-form-urlencoded' \
  -d 'user=manu&password=123'
```

除了支持绑定POST参数外，Gin框架还支持[绑定query参数](https://github.com/gin-gonic/gin#bind-query-string-or-post-data)、[绑定uri参数](https://github.com/gin-gonic/gin#bind-uri)以及[绑定header](https://github.com/gin-gonic/gin#bind-header)另外，Gin框架还支持[自定义校验](https://github.com/gin-gonic/gin#custom-validators)，在此不一一详述。

## 渲染

Gin框架支持XML、JSON、YAML、和ProtoBuf渲染，以JSON和XML为例：

```go
func main() {
	r := gin.Default()

	// gin.H is a shortcut for map[string]interface{}
	r.GET("/someJSON", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

r.GET("/someXML", func(c *gin.Context) {
		c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})
	
r.Run(":8080")
```

测试：

```shell
$ curl http://localhost:8080/someJSON
$ curl http://localhost:8080/someXML
```

## 上传下载

1、搭建静态文件服务器

```go
func main() {
	router := gin.Default()
	
	router.Static("/assets", "./assets")
	router.StaticFS("/statics", http.Dir("my_file_system"))
	router.StaticFile("/favicon.ico", "./resources/favicon.ico")

	router.Run(":8080")
}
```

测试：

```shell
$ curl http://localhost:8080/statics
```

2、单文件上传

```shell
func main() {
	router := gin.Default()
	
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// single file
		file, _ := c.FormFile("file")
		log.Println(file.Filename)

		// Upload the file to specific dst.
		c.SaveUploadedFile(file, dst)

		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
	})
	
	router.Run(":8080")
}
```

测试：

```shell
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```

3、多文件上传

```shell
func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["upload[]"]

		for _, file := range files {
			log.Println(file.Filename)

			// Upload the file to specific dst.
			c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
	})
	router.Run(":8080")
}
```

测试：

```shell
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

## 中间件
中间件是对Gin框架的扩展，Gin框架内已经定义了一些中间件，使用默认路由器时，会自动加载`Logger`和`Recovery`中间件：

```Go
r := gin.Default()
```

等价于：

```Go
// Creates a router without any middleware by default
	r := gin.New()

	// Global middleware
	// Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
	// By default gin.DefaultWriter = os.Stdout
	r.Use(gin.Logger())

	// Recovery middleware recovers from any panics and writes a 500 if there was one.
	r.Use(gin.Recovery())
```

如果觉得框架定义的中间件满足不了需求，可以自定义中间件，比如，我们可以自定义`Recovery`中间件：

```Go
func main() {
	// Creates a router without any middleware by default
	r := gin.New()

	r.Use(gin.Logger())

	// Recovery middleware recovers from any panics and writes a 500 if there was one.
	r.Use(gin.CustomRecovery(func(c *gin.Context, recovered interface{}) {
		if err, ok := recovered.(string); ok {
			c.String(http.StatusInternalServerError, fmt.Sprintf("error: %s", err))
		}
		c.AbortWithStatus(http.StatusInternalServerError)
	}))

	r.GET("/panic", func(c *gin.Context) {
		// panic with a string -- the custom middleware could save this to a database or report it to the user
		panic("foo")
	})

	r.GET("/", func(c *gin.Context) {
		c.String(http.StatusOK, "ohai")
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

也可以自定义日志格式：

```Go
func main() {
	router := gin.New()

	router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {

		// your custom format
		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
				param.ClientIP,
				param.TimeStamp.Format(time.RFC1123),
				param.Method,
				param.Path,
				param.Request.Proto,
				param.StatusCode,
				param.Latency,
				param.Request.UserAgent(),
				param.ErrorMessage,
		)
	}))
	router.Use(gin.Recovery())

	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	router.Run(":8080")
}
```

## 参考资料

[1] [Gin Web Framework](https://github.com/gin-gonic/gin#gin-web-framework)
