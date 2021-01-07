# gin-annotation 
gin-annotation is a powerful cli tool to enable gin annotation

![image](https://raw.githubusercontent.com/1-st/logos/master/gin-annotation/logo.png)
## features
* Using code generating technology by operating golang [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
* Routing group
* enable Aspect Oriented Programming for gin by middleware annotation 

## quick start
1. Installation.
```shell
go get github.com/1-st/gin-annotation
```
2. Write your HandlerFunc anywhere.
  
> source code files see dir: *_example/simple*
```go
// controller/hello.go
package controller

/* Hello a simple controller
[
	method:GET,
	group:/api,
	path:/hello-world,
	need:auth
]
*/
func HelloWorld(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, map[string]string{
		"msg": "hello, world",
	})
}
```
```go
// middleware/auth.go
package middleware

/* Auth a simple middleware
[
	id:auth
]
*/
func Auth(ctx *gin.Context) {
	ctx.Next()
}

/* Log the first middleware in group /api
[
	id:log,
	groups:/api@1
]
*/
func Log(ctx *gin.Context){
	fmt.Println(ctx.ClientIP())
}
```
3. Run gin-annotation at dir: *_example/simple*:
```sh
$ gin-annotation ./controller ./middleware
$ ls
controller main.go route.entry.go(!!!new file)
```
> tips: the name of new file is decided by environment variable <font color=#ee00ee>GIN_ANNOTATION_FILE</font> 
, default is route.entry.go 

4. Look at the generated routing file
```go
package main

import (
"gin-annotation/_example/simple/controller"
"gin-annotation/_example/simple/middleware"
"github.com/gin-gonic/gin"
)

func Route(e *gin.Engine) {
	api := e.Group("/api", middleware.Log)
	{
		v1 := api.Group("/v1")
		{
			v1.GET("/hello-world", middleware.Auth, controller.HelloWorld)
		}
	}
}
``` 
5. The last step , call Route() at main.go
```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	e := gin.Default()
	Route(e)
	_ = e.Run("ip:port")
}
```
