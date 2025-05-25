---
title: 使用Go实现批量文件上传
date: 2021-11-28 12:20:24
categories:
- 编程匠艺
tags:
- Web
- Go
---

在现在生活中，我们经常在使用文件上传功能，但我们很少会自己去做一个文件上传的服务，了解文件的上传原理有助于我们开发出更健壮的程序。利用Go语言以及其标准库的能力，用很少的代码量就可以实现一个文件上传服务器。本文以小工具的方式讲解如何实现一个简单的文件上传服务器，并假设读者对Go、HTTP、HTML有一定了解。

完整代码参考：[github.com/leeyzero/go-tools/uploadserver.go](https://github.com/leeyzero/go-tools/blob/main/uploadserver.go)

## Server端

文件上传一般使用multipart/form-data上传上传，Go语言标准库其进行了良好的封装，感兴趣的可以直接查询其原码实现。对于文件上传服务器，需要搭建一个简单的服务端框架，幸运的是Go语言http包已经为我们实现了，我们只需要实现一个handler即可，

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    ...
}

func main() {
    http.HandleFunc("/upload", uploadHandler)
    http.ListenAndServe(":8080", nil)
}
```

在uploadHandler中，只需要进行以下三个步骤：
1、解析multiform data
2、遍历multiform data中的文件
3、将文件保存至指定位置

<!--more-->

其中第1步，Go语言的http.Request包已经提供了ParseMultipartform API，我们只需要实现2、3步即可。如下：

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    err := r.ParseMultipartForm(50<<20)
    if err != nil {
        fmt.Fprintln(w, err)
        return
    }

    files := r.MultipartForm.File["files"]
    for i, _ := range files {
        file, err := files[i].Open()
        if err != nil {
            fmt.Fprintln(w, err)
            return
        }
        defer file.Close()

        dst, err := os.Create("/tmp/" + files[i].Filename)
        if err != nil {
            fmt.Fprintln(w, err)
            return
        }
        defer dst.Close()

        _, err = io.Copy(dst, file)
        if err != nil {
            fmt.Fprintln(w, err)
            return
        }

        fmt.Fprintf(w, "upload %s success\n", files[i].Filename)
    }
}
```

在上面的代码中，我们硬编码了一些常量，如监听的端口号，保存上传文件的路径以及上传文件最大允许使用的内存，可以通过环境变量配置出来：

```go
var (
    gAddr      string
    gTargetDir string
    gMaxMemory int64
)

func TryGetEnvString(key string, defVal string) string {
    strVal := os.Getenv(key)
    if strVal == "" {
        strVal = defVal
    }

    return strVal
}

func TryGetEnvInt64(key string, defVal int64) int64 {
    strVal := os.Getenv(key)
    if strVal == "" {
        return defVal
    }

    intVal, err := strconv.ParseInt(strVal, 10, 64)
    if err != nil || intVal <= 0 {
        return defVal
    }

    return intVal
}

func init() {
    gAddr = TryGetEnvString("ADDR", ":8080")
    gTargetDir = TryGetEnvString("TARGET_DIR", "/tmp")
    gMaxMemory = TryGetEnvInt64("MAX_MEMORY", 50<<20)
}
```

启动上传服务器，其中监听的端口、上传存储的路径、上传文件使用的最大内存可以分别通过ADDR、TARGET_DIR、MAX_MEMORY环境变量进行配置。如下：

```shell
ADDR=:8888 go run uploadserver.go
```

## Client端

如果比较喜欢命令行工具，curl是个不错的选择，比如现在要上传test1.txt test2.txt文件：

```shell
> curl -F 'files=@test.txt' -F 'files=@test2.txt' 'http://localhost:8080/upload'
```

也可以使用html写一个简单的表单界面，利用浏览器进行上传，如下upload.html文件：

```html
<!DOCTYPE html>
<html>

<head>
    <title>Go upload</title>
</head>

<body>
    <form action="http://localhost.com:8080/upload" method="post" enctype="multipart/form-data">
        <label for="file">选择文件:</label>
        <input type="file" name="files" id="file" multiple><br>
        <input type="submit" name="submit" value="上传">
    </form>
</body>

</html>
```

将上述upload.html文件用浏览器打开，选择完文件后，点击上传即可将所选文件上传至服务器的指定位置。

## 总结

本文介绍了如何使用Go实现一个简单的文件上传服务器，Go语言http包已经帮我们做了很多工作，我们只需要解析multiform data并从遍历文件列表，逐步将文件保存到指定的目录即可。标准库给我们封装好用的API的同时，隐藏了multiform-data协议后面的很多细节，对于这些细节，我们不需要面面俱到，但一定要有所了解，这有助于知其然，也知其所以然，参数资料给出进一步的阅读材料。

## 参考资料

[1] [Go http package](https://pkg.go.dev/net/http)
[2] [HTML input tag](https://www.runoob.com/tags/tag-input.html)
[3] [RFC2388 multipart/form-data](https://datatracker.ietf.org/doc/html/rfc2388)
