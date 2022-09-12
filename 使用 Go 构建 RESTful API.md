# 前言

使用 Go 实现一个 http 服务器无与伦比的简单，如下：

```go
func main() {
	http.HandleFunc("/", RootFunc)
	err := http.ListenAndServe(":4000", nil)
	if err != nil {
		log.Fatalln(err)
	}
}

func RootFunc(w http.ResponseWriter, r *http.Request) {

	// Changing the header map after a call to WriteHeader (or
	// Write) has no effect unless the modified headers are
	// trailers.
	w.Header().Set("Content-Type", "text/plain; charset=utf-8")

	w.Write([]byte("Language Home"))

	// 该方法必须在最后调用, 否则 w.Header().Set() 不会生效(下文略)
	w.WriteHeader(http.StatusOK)
}
```

这样就实现了一个端口为 4000 的 http 服务器。不过呢，因为 RootFunc 方法中我们限制请求方式，所以任何类型的请求都会响应：

```bash
curl -i -X GET localhost:4000
 # 或
curl -i -X POST localhost:4000
```

**响应示例：**

```http
HTTP/1.1 200 OK
Date: Mon, 12 Sep 2022 02:44:18 GMT
Content-Length: 13
Content-Type: text/plain; charset=utf-8
Connection: close

Language Home
```

那么，我们接下来就来看下如何在方法中限制请求类型及如何处理对应请求类型的参数数据：

# 准备数据

先准备点数据：

```go
type Language struct {
	Id   string `json:"id"`
	Name string `json:"name"`
	URL  string `json:"url"`
}

var langs []Language = []Language{
	{
		Id:   "1",
		Name: "GoLang",
		URL:  "go.dev",
	},
	{
		Id:   "2",
		Name: "Dart",
		URL:  "dart.dev",
	},
	{
		Id:   "3",
		Name: "Flutter",
		URL:  "flutter.dev",
	},
}
```

# Get 列表查询

```go
func main() {
  // ...
  http.HandleFunc("/langs", GetLangs)
  // ...
}

func GetLangs(w http.ResponseWriter, r *http.Request) {

	// Accept Get Method Only
	if r.Method != http.MethodGet {
		w.Header().Set("Content-Type", "text/plain")
		w.Write([]byte("Unsupported Request Method"))
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	json.NewEncoder(w).Encode(&langs)

	w.WriteHeader(http.StatusOK)
}
```

为了只用于 GET 查询，这次在方法中增加了请求方式限制。首先获取当前 http 请求类型（`r.Method`），判断是否为 GET 请求（`http.MethodGet`），如果是其他类型的请求直接响应 MethodNotAllowed。

另外，还有一点需要特别注意的是 `w.WriteHeader()` 必须在最后调用，否则 `w.Header().Set()` 不会生效。

可以看下官网文档说明：https://pkg.go.dev/net/http#ResponseWriter

**下面是 cURL 请求示例：**

```bash
$ curl -i -X GET localhost:4000/langs

...
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Mon, 12 Sep 2022 06:49:31 GMT
Content-Length: 134

[{"id":"1","name":"GoLang","url":"go.dev"},{"id":"2","name":"Dart","url":"dart.dev"},{"id":"3","name":"Flutter","url":"flutter.dev"}]
```

# GET 查询单条数据

```go
func main() {
  // ...
  http.HandleFunc("/getLangById", GetLangById)
  // ...
}

func GetLangById(w http.ResponseWriter, r *http.Request) {

	// Only Get Request
	if r.Method != http.MethodGet {
		w.WriteHeader(http.StatusMethodNotAllowed)
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.Write([]byte("Unsupported Request Method"))
		return
	}

	// 获取URL中的请求参数
	v := r.URL.Query()
	id := v.Get("id")
	if id == "" {
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.Write([]byte("Cannot find \"id\" parameter"))
		w.WriteHeader(http.StatusInternalServerError)
		return
	}

	// 或直接使用 FormValue 获取
	// id := r.FormValue("id")

	for _, lang := range langs {
		if lang.Id == id {

			w.Header().Set("Content-Type", "application/json; charset=utf-8")

			json.NewEncoder(w).Encode(&lang)

			w.WriteHeader(http.StatusOK)
			return
		}
	}

	w.WriteHeader(http.StatusNoContent)
}
```

单条数据查询与 [列表查询](#get-列表查询) 没什么区别，唯一要注意的就是如何获取 URL 中的请求参数。

这里有两种方式：`r.URL.Query()` 或 `r.FormValue()`。

`r.URL.Query()` 方法会返回一个 `map[string][]string` map 数据类型，拿到该 map 之后再根据 key 获取值就好了。

而 `r.FormValue()` 方法的区别是他在内部处理了该 map，因为当我们调用该方式时本质就是根据 Key 获取 map 的值，只不过是内部做了封装处理而已。

**下面是 cURL 请求示例：**

```bash
$ curl -i -X GET localhost:4000/getLangById?id=1

...
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Mon, 12 Sep 2022 06:50:10 GMT
Content-Length: 42

{"id":"1","name":"GoLang","url":"go.dev"}
```

# POST 请求：application/json

`application/json` POST 请求在实际 API 中用的最多，在 Go 中来看下如何处理这类请求的数据：

```go
func main() {
  // ...
  http.HandleFunc("/addLang", AddLang)
  // ...
}

func AddLang(w http.ResponseWriter, r *http.Request) {
	// Only POST Request
	if r.Method != http.MethodPost {
		w.Write([]byte("Unsupported Request Method"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	cType := r.Header.Get("Content-Type")
	contains := strings.Contains(cType, "application/json; charset=utf-8")
	if !contains {
		w.Write([]byte("Only support application/json"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusUnsupportedMediaType)
		return
	}

	// 直接将请求体中的数据转换为结构体
	var lang = new(Language)
	json.NewDecoder(r.Body).Decode(lang)

	// 或者直接获取原始数据
	// bytes, _ := ioutil.ReadAll(r.Body)
	// log.Println(string(bytes))

	// add
	langs = append(langs, *lang)

	// response
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	json.NewEncoder(w).Encode(lang)
	w.WriteHeader(http.StatusOK)
}
```

对应 `application/json` 请求，我们唯一要做的就是获取请求体（`r.Body`）中的数据。获取到原始数据之后，接下来该怎么处理就看业务需求了~

**cURL 请求示例：**

```bash
curl -i \
--request POST 'localhost:4000/addLang' \
--header 'Content-Type: application/json' \
--data-raw '{"id": "4","name": "Java","url": "oracle.com"}'
```

**http 请求示例：**

```http
POST /addLang
Host: localhost:4000
Content-Type: multipart/form-data

{
  "id": "4",
  "name": "Java",
  "url": "oracle.com"
}
```

# POST 请求：application/x-www-form-urlencoded

`application/x-www-form-urlencoded` 用于 Form 表单请求，在前后端还没分离的年代这类请求类型比较常见。不过现在更多是还是使用 `application/application/json`。

看下 Go 如何处理 Form 表单请求：

```go
func main() {
  // ...
  http.HandleFunc("/addLangForm", AddLangForm)
  // ...
}

func AddLangForm(w http.ResponseWriter, r *http.Request) {

	// Only POST Request
	if r.Method != http.MethodPost {
		w.Write([]byte("Unsupported Request Method"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	cType := r.Header.Get("Content-Type")
	contains := strings.Contains(cType, "application/x-www-form-urlencoded")
	if !contains {
		w.Write([]byte("Only support application/x-www-form-urlencoded"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusUnsupportedMediaType)
		return
	}

	id := r.FormValue("id")
	name := r.FormValue("name")
	url := r.FormValue("url")

	// 或者直接使用下面的方式获取 Form 值
	// values := r.PostForm

	var lang = Language{
		Id:   id,
		Name: name,
		URL:  url,
	}

	// response
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	json.NewEncoder(w).Encode(&lang)
	w.WriteHeader(http.StatusOK)
}
```

**cURL 请求示例：**

```bash
curl -i \
--request POST 'localhost:4000/fileUpload' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'id=5' \
--data-urlencode 'name=rust' \
--data-urlencode 'url=rust-lang.org'
```

**http 请求示例：**

```http
POST /addLang HTTP/1.1
Host: localhost:4000
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

id=5&name=rust&url=rust-lang.org
```

响应示例：

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Mon, 12 Sep 2022 06:25:30 GMT
Content-Length: 47
Connection: close

{
  "id": "5",
  "name": "rust",
  "url": "rust-lang.org"
}
```

# POST 请求：multipart/form-data

`multipart/form-data` 请求主要用于带文件上传的 Form 表单请求，因为相比较 `application/x-www-form-urlencoded` 表单请求，`multipart/form-data` 实现文件上传更加高效：

```go
func main() {
  // ...
  http.HandleFunc("/fileUpload", FileUpload)
  // ...
}

func FileUpload(w http.ResponseWriter, r *http.Request) {
	// Only POST Request
	if r.Method != http.MethodPost {
		w.Write([]byte("Unsupported Request Method"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	cType := r.Header.Get("Content-Type")
	contains := strings.Contains(cType, "multipart/form-data")
	if !contains {
		w.Write([]byte("Only support application/json"))
		w.Header().Set("Content-Type", "text/plain; charset=utf-8")
		w.WriteHeader(http.StatusUnsupportedMediaType)
		return
	}

	// Limit your max input length!
	r.ParseMultipartForm(32 << 20)

	username := r.FormValue("username")
	_, header, err := r.FormFile("avatar")
	if err != nil {

	}

	var info = make(map[string]interface{})
	info["username"] = username
	info["filename"] = header.Filename
	info["filesize"] = header.Size

	// response
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	json.NewEncoder(w).Encode(&info)
	w.WriteHeader(http.StatusOK)
}
```

**cURL 请求示例：**

```bash
$ curl -i \
--request POST 'localhost:4000/fileUpload' \
--header 'Content-Type: multipart/form-data' \
--form 'name="bob"' \
--form 'file=@"path/ubuntu_before_init_partition_disk.png"'
```

**http 请求示例：**

```http
POST /fileUpload HTTP/1.1
Host: localhost:4000
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 288

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="name"

bob
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="ubuntu_before_init_partition_disk.png"
Content-Type: image/png

< path/ubuntu_before_init_partition_disk.png
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**响应结果：**

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Mon, 12 Sep 2022 06:11:05 GMT
Content-Length: 87
Connection: close

{
  "filename": "ubuntu_before_init_partition_disk.png",
  "filesize": 39428,
  "username": "Bob"
}
```