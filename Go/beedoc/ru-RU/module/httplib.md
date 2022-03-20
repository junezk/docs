---
name: Модуль Httplib 
sort: 4
---

# Запросы с клиента

Также как и Curl, httplib используется для эмуляции http запросов с клиента. Также как и jQuery, он поддердивает цепочки вызовов (fluent interface). Он прост в использовании. Может быть установен следующей командой:

	go get github.com/astaxie/beego/httplib

## Базовое использование

Импортируйте пакет:

	import (
		"github.com/astaxie/beego/httplib"
	)	

Инициализируйте request метод и путь:

	req := httplib.Get("http://beego.me/")

Отправьте запрос и извлеките данные из запроса:

	str, err := res.String()
	if err != nil {
		t.Fatal(err)
	}
	fmt.Println(str)
	
## Методы

httplib поддерживает следующие методы:

- `Get(url string)`
- `Post(url string)`
- `Put(url string)`
- `Delete(url string)`
- `Head(url string)`

## Отладочный вывод

Включите вывод отладочной информации:

	req.Debug(true)

Затем это должно вывести отладочную информацию:

	httplib.Get("http://beego.me/").Debug(true).Response()

	// Output
	GET / HTTP/0.0
	Host: beego.me
	User-Agent: beegoServer

## HTTPS запрос

Если запрашиваема схема https, мы должны установить TLS клиент:

	req.SetTLSClientConfig(&tls.Config{InsecureSkipVerify: true})

[Узнайте больше о настройках TLS](http://gowalker.org/crypto/tls#Config)

## Установка таймаутов

Можем установить таймауты и считывания данных по таймауту:

	req.SetTimeout(connectTimeout, readWriteTimeout)

Это функция работает с объектом запроса, а значит мы можем сделать так:

	httplib.Get("http://beego.me/").SetTimeout(100 * time.Second, 30 * time.Second).Response()
	
## Установить параметры запроса

Для PUT или POST запросов, мы можем отправить параметры. Параметры могут быть установлены так:

	req := httplib.Post("http://beego.me/")
	req.Param("username","astaxie")
	req.Param("password","123456")

## Отправка больших данных

Чтобы симулировать загрузку файлов или отправить большые данные вы можете использовать `Body` функцию:

	req := httplib.Post("http://beego.me/")
	bt,err:=ioutil.ReadFile("hello.txt")
	if err!=nil{
		log.Fatal("read file err:",err)
	}
	req.Body(bt)

## Установить заголовоки

Вы можете эмулировать HEADER заголовки, например:

	Accept-Encoding:gzip,deflate,sdch
	Host:beego.me
	User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36

Можете использовать `Header` функцию:

	req := httplib.Post("http://beego.me/")
	req.Header("Accept-Encoding","gzip,deflate,sdch")
	req.Header("Host","beego.me")
	req.Header("User-Agent","Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36")

## Загрузка файла

У PostFile функции первый параметр является имя формы, второй параметр имя файла или путь до файла который вы собираетесь отправить. 

```
b:=httplib.Post("http://beego.me/")
b.Param("username","astaxie")
b.Param("password","123456")
b.PostFile("uploadfile1", "httplib.pdf")
b.PostFile("uploadfile2", "httplib.txt")
str, err := b.String()
if err != nil {
    t.Fatal(err)
}
```

## Получение ответа 

Как мы можем получить ответ после запроса? Ниже варианты:

|Метод                           |Тип                      |Описание                                                   |
|--------------------------------|-------------------------|-----------------------------------------------------------|
|`req.Response()`                |`(*http.Response, error)`|Объект`http.Response`. Из него вы сможете получить данные. |
|`req.Bytes()`                   |`([]byte, error)`        |Возвращает необработанное тело ответа.                     |
|`req.String()`                  |`(string, error)`        |Возвращает необработанное тело ответа.                     |
|`req.ToFile(filename string)`   |`error`                  |Сохраняем тело ответа в файл.                              |
|`req.ToJSON(result interface{})`|`error`                  |Разбираем JSON ответ в результирующий объект.              |
|`req.ToXml(result interface{})` |`error`                  |Разбираем XML ответ в результирующий объект.               |
