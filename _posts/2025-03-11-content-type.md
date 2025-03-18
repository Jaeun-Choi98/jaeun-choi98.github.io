---
layout: post
title: 'Go에서 Content-Type 처리'
tags: ['Go', 'http/net']
date: 2025-03-11 11:50:00
---

HTTP 요청과 응답에서 Content-Type 헤더는 전송되는 데이터의 형식을 정의하며, 클라이언트와 서버 간 데이터 처리 방식을 결정합니다. 각 Content-Type은 특정 데이터 형식에 맞춰 처리되고 사용됩니다.

<br>
<br>

## ✅ **1. Content-Type: application/json**

---

📤 요청 예시 (Request Body)

```
POST / HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Content-Length: 33

{
  "name": "Alice",
  "age": 30
}

```

🧑‍💻 net/http 코드

```go
func jsonHandler(w http.ResponseWriter, r *http.Request) {
    var data map[string]interface{}
    body, _ := io.ReadAll(r.Body)
    json.Unmarshal(body, &data)
    fmt.Println("Received JSON:", data)
    w.Write([]byte("OK"))
}

```

🍸 Gin 코드

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func jsonHandler(c *gin.Context) {
    var user User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    fmt.Printf("Parsed User: %+v\\n", user)
    c.JSON(http.StatusOK, user)
}

```

🦍 gorilla/mux 코드

```go
func jsonHandler(w http.ResponseWriter, r *http.Request) {
    var data map[string]interface{}
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Failed to read body", http.StatusBadRequest)
        return
    }
    if err := json.Unmarshal(body, &data); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    fmt.Println("Received JSON:", data)
    w.Write([]byte("OK"))
}

```

<br>

## ✅ **2. Content-Type: application/x-www-form-urlencoded**

---

📤 요청 예시 (Request Body)

```
POST / HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 17

name=Bob&age=25

```

🧑‍💻 net/http 코드

```go
func formHandler(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    name := r.FormValue("name")
    age := r.FormValue("age")
    fmt.Fprintf(w, "Received name=%s, age=%s", name, age)
}

```

🍸 Gin 코드

```go
type FormData struct {
    Name string `form:"name"`
    Age  int    `form:"age"`
}

func formHandler(c *gin.Context) {
    var data FormData
    if err := c.ShouldBind(&data); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusOK, data)
}

```

🦍 gorilla/mux 코드

```go
func formHandler(w http.ResponseWriter, r *http.Request) {
    if err := r.ParseForm(); err != nil {
        http.Error(w, "ParseForm error", http.StatusBadRequest)
        return
    }
    name := r.FormValue("name")
    age := r.FormValue("age")
    fmt.Fprintf(w, "Received name=%s, age=%s", name, age)
}

```

<br>

## ✅ **3. Content-Type: text/plain**

---

📤 요청 예시 (Request Body)

```
POST / HTTP/1.1
Host: localhost:8080
Content-Type: text/plain
Content-Length: 13

Hello, world!

```

🧑‍💻 net/http 코드

```go
func textHandler(w http.ResponseWriter, r *http.Request) {
    body, _ := io.ReadAll(r.Body)
    fmt.Println("Text received:", string(body))
    w.Write([]byte("Text OK"))
}

```

🍸 Gin 코드

```go
func textHandler(c *gin.Context) {
    body, _ := io.ReadAll(c.Request.Body)
    fmt.Println("Text:", string(body))
    c.String(http.StatusOK, "Received: %s", string(body))
}

```

🦍 gorilla/mux 코드

```go
func textHandler(w http.ResponseWriter, r *http.Request) {
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Read error", http.StatusBadRequest)
        return
    }
    fmt.Println("Text received:", string(body))
    w.Write([]byte("Text OK"))
}

```

<br>

## ✅ **4. Content-Type: multipart/form-data (파일 업로드)**

---

📤 요청 예시 (Request Body - 간략화된 예시)

```
POST /upload HTTP/1.1
Host: localhost:8080
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="upload"; filename="sample.txt"
Content-Type: text/plain

(sample.txt 파일 내용)
------WebKitFormBoundary7MA4YWxkTrZu0gW--

```

🧑‍💻 net/http 코드

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    r.ParseMultipartForm(10 << 20) // 10MB
    file, handler, err := r.FormFile("upload")
    if err != nil {
        http.Error(w, "Error uploading file", http.StatusBadRequest)
        return
    }
    defer file.Close()

    fmt.Fprintf(w, "Uploaded File: %s (%d bytes)", handler.Filename, handler.Size)
}

```

🍸 Gin 코드

```go
func uploadHandler(c *gin.Context) {
    file, err := c.FormFile("upload")
    if err != nil {
        c.String(http.StatusBadRequest, "Upload error: %s", err)
        return
    }

    // 저장도 가능: c.SaveUploadedFile(file, "uploads/"+file.Filename)
    c.String(http.StatusOK, "Uploaded: %s (%d bytes)", file.Filename, file.Size)
}

```

🦍 gorilla/mux 코드

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    err := r.ParseMultipartForm(10 << 20)
    if err != nil {
        http.Error(w, "Could not parse multipart form", http.StatusBadRequest)
        return
    }

    file, handler, err := r.FormFile("upload")
    if err != nil {
        http.Error(w, "Error retrieving the file", http.StatusBadRequest)
        return
    }
    defer file.Close()

    fmt.Fprintf(w, "Uploaded File: %s (%d bytes)", handler.Filename, handler.Size)
}

```

## ✅ **5. Content-Type: application/xml**

---

📤 요청 예시 (Request Body)

```
POST / HTTP/1.1
Host: localhost:8080
Content-Type: application/xml
Content-Length: 46

<User>
  <Name>Tom</Name>
  <Age>40</Age>
</User>

```

🧑‍💻 net/http 코드

```go
import (
    "encoding/xml"
    "net/http"
    "io"
    "fmt"
)

type User struct {
    Name string `xml:"Name"`
    Age  int    `xml:"Age"`
}

func xmlHandler(w http.ResponseWriter, r *http.Request) {
    var user User
    body, _ := io.ReadAll(r.Body)
    if err := xml.Unmarshal(body, &user); err != nil {
        http.Error(w, "XML Parse Error", http.StatusBadRequest)
        return
    }
    fmt.Printf("Parsed User: %+v\\n", user)
    w.Write([]byte("XML OK"))
}

```

🍸 Gin 코드

```go
type User struct {
    Name string `xml:"Name"`
    Age  int    `xml:"Age"`
}

func xmlHandler(c *gin.Context) {
    var user User
    if err := c.ShouldBindXML(&user); err != nil {
        c.String(http.StatusBadRequest, "XML Parse Error")
        return
    }
    c.JSON(http.StatusOK, user)
}

```

🦍 gorilla/mux 코드

```go
type User struct {
    Name string `xml:"Name"`
    Age  int    `xml:"Age"`
}

func xmlHandler(w http.ResponseWriter, r *http.Request) {
    var user User
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Read error", http.StatusBadRequest)
        return
    }

    if err := xml.Unmarshal(body, &user); err != nil {
        http.Error(w, "XML Parse Error", http.StatusBadRequest)
        return
    }

    json.NewEncoder(w).Encode(user)
}

```

📌 net/http에서는 XML을 직접 파싱해야 하므로 encoding/xml을 사용하여 구조체에 매핑해야 합니다.

<br>

🔄 여러 Content-Type 동시 지원 (Gin)

```go
func smartHandler(c *gin.Context) {
    contentType := c.ContentType()

    switch contentType {
    case "application/json":
        var jsonData map[string]interface{}
        if err := c.BindJSON(&jsonData); err == nil {
            c.JSON(http.StatusOK, jsonData)
        }
    case "application/x-www-form-urlencoded":
        name := c.PostForm("name")
        c.String(http.StatusOK, "Form name: %s", name)
    case "multipart/form-data":
        file, _ := c.FormFile("upload")
        c.String(http.StatusOK, "Multipart file: %s", file.Filename)
    default:
        c.String(http.StatusUnsupportedMediaType, "Unsupported Content-Type: %s", contentType)
    }
}

```
