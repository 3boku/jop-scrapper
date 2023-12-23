# jop-scrapper

노마드코더 Golang강의 JOB SCRAPPER

## 1번쨰 영상
```go
package main

import (
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=python"

func main() {
	getPages()
}

func getPages() int {
	res, err := http.Get(baseURL) //http패키지의 Get매소드를 사용해 baseURL에 있는 html파일을 로드해옴
	checkErr(err)  //error를 체크하는 함수
	checkCode(res) // res를 체크하는 함수

	defer res.Body.Close() //defer을 사용해 함수가 끝나면 res.Body를 닫아줌
	doc, err := goquery.NewDocumentFromReader(res.Body) //goquery 패키지를 사용해 로드해온 html파일에 body를 읽어옴
	checkErr(err)

	doc.Find(".ClassName") //클래스네임 클래스를 찾아옴
	return 0
}

func checkErr(err error) {
	if err != nil {
		log.Fatalln(err)
	}
}

func checkCode(res *http.Response // 포인터로 되어있는 http.Response를 매개변수를 설정함) {
	if res.StatusCode != 200 {
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

```