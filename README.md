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
package main

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"strconv"
	"strings"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=python"

type extractedJob struct {
	title    string
	location string
	corp     string
}

func main() {
	var jobs []extractedJob //jobs는 구조체의 배열이다
	totalPages := getPages() 

	for i := 1; i < totalPages; i++ {
		extractedJobs := getPage(i) //exxtractedJobs라는 변수에 페이지에서 가져온 변수들을 대입한다.
		jobs = append(jobs, extractedJobs...) //jobs에 extractedJobs의 컨텐트만 더한다 ...을 안붙히면 [[x] [x] [x]]이렇게 되는데 ...을 붙히면 [ x x x]가 된다.
	}
	fmt.Println(jobs)
}

func getPage(page int) []extractedJob {
	var jobs []extractedJob //jobs는 구조체의 배열이다
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
		job := extractJob(card) //카드에 들어있는 구조체를 job이라는 변수에 넣는다.
		jobs = append(jobs, job) //jobs에 job을 더한다
	})
	return jobs
}

func extractJob(card *goquery.Selection) extractedJob {
	title := cleanString(card.Find(".area_job>.job_tit>a").Text())
	location := cleanString(card.Find(".area_job>.job_condition>span>a").Text())
	corp := cleanString(card.Find(".area_corp>.corp_name>a").Text())
	return extractedJob{
		title:    title,
		location: location,
		corp:     corp,
	} // 전에 만들었던 extractedJob 구조체안에 변수를 벨류로 넣는다
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
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

func cleanString(s string) string {
	return strings.Join(strings.Fields(strings.TrimSpace(s)), " ")
} // strings패키지를 이용해서 입력된 문자열의 스페이스를 없애고, 스페이스를 없앤 문자열을 하나로 합친다음, 다시 붙인다 뭔소리지

```
## 5번째 영상
```go
package main

import (
	"encoding/csv"
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=golang"

type extractedJob struct {
	title    string
	location string
	corp     string
}

func main() {
	var jobs []extractedJob
	totalPages := getPages()

	for i := 1; i < totalPages; i++ {
		extractedJobs := getPage(i)
		jobs = append(jobs, extractedJobs...)
	}
	writeJobs(jobs) //writeJobs 함수 실행
	fmt.Println("Done extrated = ", len(jobs))
}

func writeJobs(jobs []extractedJob) { //jobs를 매개변수로 가져온다
	file, err := os.Create("jobs.csv")//os패키지에 Create명령어를 사용해서 jobs.csv파일을 만든다.
	checkErr(err)

	w := csv.NewWriter(file) //jobs.csv파일에 작성할 내용을 w라는 변수로 선언한다.
	defer w.Flush() // 함수가 끝나면 jobs.csv파일에 내용을 작성한다.

	headers := []string{"title", "location", "corp"}//헤더를 만든다.

	wErr := w.Write(headers)//헤더 내용을 jobs.csv파일에 내용을 작성한다.
	checkErr(wErr)

	for _, job := range jobs {
		jobSlice := []string{job.title, job.location, job.corp}
		jwErr := w.Write(jobSlice)
		checkErr(jwErr)
	} // jobs내용을 받아와서 작성한다.
}

func getPage(page int) []extractedJob {
	var jobs []extractedJob
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
		job := extractJob(card)
		jobs = append(jobs, job)
	})
	return jobs
}

func extractJob(card *goquery.Selection) extractedJob {
	title := cleanString(card.Find(".area_job>.job_tit>a").Text())
	location := cleanString(card.Find(".area_job>.job_condition>span>a").Text())
	corp := cleanString(card.Find(".area_corp>.corp_name>a").Text())
	return extractedJob{
		title:    title,
		location: location,
		corp:     corp,
	}
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
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

func cleanString(s string) string {
	return strings.Join(strings.Fields(strings.TrimSpace(s)), " ")
}

```

## 5번째 영상
```go
package main

import (
	"encoding/csv"
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=golang"

type extractedJob struct {
	title    string
	location string
	corp     string
}

func main() {
	var jobs []extractedJob
	totalPages := getPages()

	for i := 1; i < totalPages; i++ {
		extractedJobs := getPage(i)
		jobs = append(jobs, extractedJobs...)
	}
	writeJobs(jobs)
	fmt.Println("Done extrated = ", len(jobs))
}

func writeJobs(jobs []extractedJob) {
	file, err := os.Create("jobs.csv")
	checkErr(err)

	w := csv.NewWriter(file)
	defer w.Flush()

	headers := []string{"title", "location", "corp"}

	wErr := w.Write(headers)
	checkErr(wErr)

	for _, job := range jobs {
		jobSlice := []string{job.title, job.location, job.corp}
		jwErr := w.Write(jobSlice)
		checkErr(jwErr)
	}
}

func getPage(page int) []extractedJob {
	var jobs []extractedJob
	c := make(chan extractedJob) //getPage와 extractJob이 통신하는 채널을 만든다.

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
		go extractJob(card, c) // 함수를 고루틴을 돌린다.
	})

	for i := 0; i < searchCards.Length(); i++ {
		job := <-c
		jobs = append(jobs, job)
	} // searchCards만큼의 내용을 받아온다.
	return jobs
}

func extractJob(card *goquery.Selection, c chan<- extractedJob) {
	title := cleanString(card.Find(".area_job>.job_tit>a").Text())
	location := cleanString(card.Find(".area_job>.job_condition>span>a").Text())
	corp := cleanString(card.Find(".area_corp>.corp_name>a").Text())
	c <- extractedJob{
		title:    title,
		location: location,
		corp:     corp,
	} //extractedJob을 채널에 보낸다
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
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

func cleanString(s string) string {

	return strings.Join(strings.Fields(strings.TrimSpace(s)), " ")
}

```

## 6번째 영상
```go
package main

import (
	"encoding/csv"
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
)

var baseURL string = "https://www.saramin.co.kr/zf_user/search/recruit?&searchword=golang"

type extractedJob struct {
	title    string
	location string
	corp     string
}

func main() {
	var jobs []extractedJob
	c := make(chan []extractedJob) //getPage와 main이 통신하는 채널을 만듦
	totalPages := getPages()

	for i := 0; i < totalPages; i++ {
		go getPage(i, c) //getPage함수를 고루틴으로 돌림
	}

	for i := 0; i < totalPages; i++ {
		extractedJobs := <-c // getPage에서 받은 채널을 extractedJobs변수에 넣음
		jobs = append(jobs, extractedJobs...) //jobs 배열에 컨텐트를 추가함
	}
	writeJobs(jobs)
	fmt.Println("Done extrated = ", len(jobs))
}

func writeJobs(jobs []extractedJob) {
	file, err := os.Create("jobs.csv")
	checkErr(err)

	w := csv.NewWriter(file)
	defer w.Flush()

	headers := []string{"title", "location", "corp"}

	wErr := w.Write(headers)
	checkErr(wErr)

	for _, job := range jobs {
		jobSlice := []string{job.title, job.location, job.corp}
		jwErr := w.Write(jobSlice)
		checkErr(jwErr)
	}
}

func getPage(page int, mainC chan<- []extractedJob) {
	var jobs []extractedJob
	c := make(chan extractedJob)

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
		go extractJob(card, c)
	})

	for i := 0; i < searchCards.Length(); i++ {
		job := <-c
		jobs = append(jobs, job)
	}
	mainC <- jobs
}

func extractJob(card *goquery.Selection, c chan<- extractedJob) {
	title := cleanString(card.Find(".area_job>.job_tit>a").Text())
	location := cleanString(card.Find(".area_job>.job_condition>span>a").Text())
	corp := cleanString(card.Find(".area_corp>.corp_name>a").Text())
	c <- extractedJob{
		title:    title,
		location: location,
		corp:     corp,
	}
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
		log.Fatalln("Request failed with Status: ", res.StatusCode)
	}
}

func cleanString(s string) string {

	return strings.Join(strings.Fields(strings.TrimSpace(s)), " ")
}
```
5번째 영상이랑 같은데 다만 채널을 하나 더 만든것뿐

## 최종적으로 Go 클린 아키텍쳐로 바꾸며 마무리 하겠습니다.
- 공부를 더 한 다음!