# jop-scrapper

노마드코더 Golang강의 JOB SCRAPPER

## 1번째 영상
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

	doc.Find(".pagination") // pagination이라는 클래스를 찾아옴
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
## 2번째 영상
```go
package main

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"strconv"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=python"

func main() {
	totalPages := getPages()

	for i := 1; i < totalPages; i++ {
		getPage(i)
	}
}

func getPage(page int) {
	pageURL := baseURL + "&recruitPage=" + strconv.Itoa(page) // baseURL에 뒤에 엔드포이트를 추가해서 pageURL를 불러오기 strconv이라는 패키지를 사용해 int형인 page변수를 아스키코드로 변환
	fmt.Println("Requesting", pageURL)
}
func getPages() int {
	pages := 0
	res, err := http.Get(baseURL)
	checkErr(err)
	checkCode(res)

	defer res.Body.Close()
	doc, err := goquery.NewDocumentFromReader(res.Body)
	checkErr(err)

	doc.Find(".pagination").Each(func(i int, s *goquery.Selection) {
		pages = s.Find("a").Length() //.pagination안에있는 a태그들의 갯수를 찾아오기
	})
	return pages
}

func checkErr(err error) {
	if err != nil {
		log.Fatalln(err)
	}
}

func checkCode(res *http.Response) {
	if res.StatusCode != 200 {
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

```
## 3번째 영상
```go
package main

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"strconv"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=python"

type extractedJob struct { //extractedJob이라는 구조체를 만듦 
	title    string
	location string
}

func main() {
	totalPages := getPages()

	for i := 1; i < totalPages; i++ {
		getPage(i)
	}
}

func getPage(page int) {
	pageURL := baseURL + "&recruitPage=" + strconv.Itoa(page)
	fmt.Println("Requesting", pageURL)
	res, err := http.Get(pageURL)
	checkErr(err)
	checkCode(res)

	defer res.Body.Close()
	doc, err := goquery.NewDocumentFromReader(res.Body)
	checkErr(err)

	searchCards := doc.Find(".item_recruit")

	searchCards.Each(func(i int, card *goquery.Selection) {
		title := card.Find(".area_job>.job_tit>a").Text()
		location := card.Find(".area_job>.job_condition>span>a").Text()
		corp := card.Find(".area_corp>.corp_name>a").Text() //card안에 들어있는 정보를 받아오고 그걸 텍스트로 변환
		fmt.Println(title)
		fmt.Println(location)
		fmt.Println(corp)
	})
}
func getPages() int {
	pages := 0
	res, err := http.Get(baseURL)
	checkErr(err)
	checkCode(res)

	defer res.Body.Close()
	doc, err := goquery.NewDocumentFromReader(res.Body)
	checkErr(err)

	doc.Find(".pagination").Each(func(i int, s *goquery.Selection) {
		pages = s.Find("a").Length()
	})
	return pages
}

func checkErr(err error) {
	if err != nil {
		log.Fatalln(err)
	}
}

func checkCode(res *http.Response) {
	if res.StatusCode != 200 {
		log.Fatalln("Request failed with Status: ", res.StatusCode) // 문자열을 정리해주는 함수
	}
}

func cleanString(s string) string {
	
}

```

## 4번째 영상
```go

```