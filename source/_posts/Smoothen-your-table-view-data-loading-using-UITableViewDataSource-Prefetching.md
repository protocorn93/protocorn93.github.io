---
title: UITableViewDataSource Prefetching
subtitle: Smoothen your table view data loading
date: 2018-09-19 11:06:37
tags: ["iOS","Swift","UITableView", "UITableViewDataSourcePrefetching"]
category: ["iOS", "번역"]
---

만일 테이블 뷰를 갖고 있는 뷰 컨트롤러가 있고 웹 API를 통해 이 테이블 뷰 위에 100개 행에 해당하는 데이터를 불러와야 한다면 당신은 어떻게 접근하겠는가?

이 글은 이미 당신인 `URLSession`과 JSON 파싱 그리고 테이블 뷰와 델리게이트를 안다는 가정을 하고 시작한다.

> **목차**
>
> 1. Load all data at once 
> 2. Load data by batch when user reach the bottom
> 3. Introducing UITableViewDataSource Prefetching
> 4. How prefetching works
> 5. Example - Just in time data loading using Prefetch
> 6. Summary 

---

#### 1. Load all 100 data at once 

가장 간단한 방법은 `viewDidLoad`에서 `URLSession`을 통해 100개의 데이터를 받아오고 테이블 뷰의 데이터 소스를 갱신 후 테이블 뷰를 리로드하는 것입니다. 

```swift
// 모든 데이터를 (100개)를 한 번에 불러온다.
URLSession(configuration: URLSessionConfiguration.default).dataTask(with: URL(string: "https://jsonplaceholder.typicode.com/posts")!) { data, response, error in
  // HTTP response에 데이터가 있음을 보장.
  guard let data = data else {
    print("No data")
    return
  }
  
  // JSONDecoder를 사용해 JSON을 Post 구조체 배열로 파싱한다.
  guard let posts = try? JSONDecoder().decode([Post].self, from: data) else {
    print("Error: Couldn't decode data into post model")
    return
  }
  
  // 데이터 소스
  self.postArray = posts
  
  // UI 갱신은 반드시 메인 스레드에서 일어나야 한다.
  DispatchQueue.main.async {
    self.postTableView.reloadData()
  }
}.resume()
```

이 방법은 매우 직관적이지만 100개 행의 데이터를 한번에 받아오는 것은 상대적으로 많은 시간과 메모리 사용량을 필요로 한다. 그리고 몇몇의 경우에는 데이터를 한번에 받아오는 것이 불가능 할 수 있다. (페이스북 타임라인의 경우 사용자 계정이 활성화된 후 모든 정보를 아이폰 메모리에 가져오기엔 너무 많다.)

---

#### 2. Load data by batch when user reaches bottom

좀 더 심화된 방법은 데이터 20묶음을 불러오고 유저가 테이블 뷰의 바닥에 다다랐을 때 다음 20 묶음 데이터를 불러오는 것처럼 묶음 단위로 불러오는 것이다. 이 방법은 페이스북이나 트위터가 그들의 타임라인을 불러올 때 사용하는 방법과 유사하다. 

> 영상을 보면 묶음 단위 로딩이 무엇인지 단번에 알 수 있을 것이다. 
>
> [묶음 단위 로딩 예제 영상](https://iosimage.s3.amazonaws.com/2018/22-prefetch/batch.mp4)

묶음 단위 로딩 코드는 아래와 비슷할 것이다. 

```swift
class BatchViewController: UIViewController, UITableViewDataSource{

  @IBOutlet weak var coinTableView: UITableView!
  
  var coinArray : [Coin] = []
  
  let baseURL = "https://api.coinmarketcap.com/v2/ticker/?"
  
  // 한 묶음 당 15개씩 불러온다.
  let itemsPerBatch = 15
  
  // 현재 행에 해당하는 데이터베이스
  var currentRow : Int = 1
  
  // 현재 행으로 계산 된 URL
  var url : URL {
    return URL(string: "\(baseURL)start=\(currentRow)&limit=\(itemsPerBatch)")!
  }
  
  // ... skipped viewDidLoad stuff
    
  // MARK : - Tableview data source
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    // 마지막 행에서 불러오는 데이터를 보여주기 위해 +1
    return self.coinArray.count + 1
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // 바닥에 다다르면 다음 묶음을 불러온다.
    if indexPath.row == self.coinArray.count {
      let cell = tableView.dequeueReusableCell(withIdentifier: loadingCellIdentifier, for: indexPath) as! LoadingTableViewCell
      loadNextBatch()
      return cell
    }

    // 바닥에 다다르지 않은 상태에선 평소처럼 보여준다.
    let cell = tableView.dequeueReusableCell(withIdentifier: coinCellIdentifier , for: indexPath) as! CoinTableViewCell
    
    let coin = coinArray[indexPath.row]
    cell.configureCell(with: coin)
    
    return cell
  }
  
  // MARK : - Batch
  func loadNextBatch() {
    URLSession(configuration: URLSessionConfiguration.default).dataTask(with: url) { data, response, error in
      
      // JSONDecoder를 사용해 JSON을 Car 구조체 배열로 파싱한다.
      guard let coinList = try? JSONDecoder().decode(CoinList.self, from: data!) else {
        print("Error: Couldn't decode data into coin list")
        return
      }
      
      // 튜플을 갖는 배열 ie. [(key : ID, value : Coin)]
      let coinTupleArray = coinList.data.sorted {$0.value.rank < $1.value.rank}
      for coinTuple in coinTupleArray {
        self.coinArray.append(coinTuple.value)
      }
      
      // 현재 행을 증가
      self.currentRow += self.itemsPerBatch
      
      // UI 갱신은 반드시 메인 스레드에서 일어나야 한다.
      DispatchQueue.main.async {
        self.coinTableView.reloadData()
      }
    }.resume()
  }
}
```

이 방법은 꽤나 합리적이고 한 번에 15개의 데이터만 불러오고 테이블 뷰의 바닥에 다다랐을 때만 추가로 불러오기 때문에 메모리 사용에 부담도 덜을 수 있다. 사소한 단점이라면 바닥에 다다랐을 때 사용자가 데이터를 불러오는 행위를 기다려야 한다는 것이다. 만일 화면에 보이기 전 데이터를 불러올 수 있다면 어떨까? 추가적인 데이터를 불러오는 것을 기다리는 것보다 낫지 않을까?

---

#### 3. Introducing UITableViewDataSourcePrefetching

`UITableView`와 `UICollectionView`에 사용할 수 있는`UITableViewDataSourcePrefetching`  프로토콜은 iOS 10부터 사용가능하다. 델리게이트 함수인 `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])`는 행이 디스플레이 영역에 다다랐을 때 호출될 거이며 이 곳에서 웹 API를 호출하여 데이터를 받아올 수 있다. 

<img src="https://iosimage.s3.amazonaws.com/2018/22-prefetch/prefetchRows.png">

`UITableViewDataSourcePrefetching` 프로토콜에는 다음과 같은 두 함수가 있다. 

```swift
protocol UITableViewDataSourcePrefetching {
  // This is called when the rows are near to the visible area
  public func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])
  
  // This is called when the rows move further away from visible area, eg: user scroll in an opposite direction
  optional public func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath])
}
```

`UITableViewDataSource`의 `tableView.dataSource`와 유사하게 테이블 뷰에는 프리페치 데이터 소스를 위한 `prefetchDataSource` 프로퍼티가 존재한다.

```swift
// similar to tableview.dataSource = self
tableView.prefetchDataSource = self
```

---

#### 4. How prefetching works

다음은 애플이 정확하게 언급하지 않은 프리페치로 몇 행의 데이터가 불러오는지에 대한 저자(원작자)가 개인적으로 관찰한 것을 정리한 내용이다. 

1. `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])` 함수는 스크롤을 하지 않는 이상 화면에 보여지지 않는 행을 불러오지 않는다. 
2. 초기 보여질 수 있는 행이 가시화되면 `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])`가 호출되고 `indexPath` 변수는 대략 10개의 보여질 수 있는 영역 근처의 행들을 포함하게 된다.
3. 스크롤 속도에 따라 프리페치 함수의 `indexPath` 변수가 포함하게 되는 행의 갯수는 달라진다. 보통의 스크롤 속도에는 보통 1개의 행을 포함한다. 만일 빠른 속도로 스크롤하게 되면 다수의 행을 포함하게 된다.

다음은 이에 대한 코드와 영상 링크다.

```swift
func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
		
  print("prefetching row of \(indexPaths)")
}
	
func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
		
  print("cancel prefetch row of \(indexPaths)")
}
```

[관련 영상](https://iosimage.s3.amazonaws.com/2018/22-prefetch/fetchDemo.mp4)

---

#### 5. Example - Just in time data loading using Prefetch

이 예제에서는 프리페치를 사용하여 [Hacker News API](https://github.com/HackerNews/API)가 제공하는 top 100 스토리를 불러올 것이다. 이미 [Top Stories API](https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty)로부터 뉴스 아이디를 얻었다 가정하자. 우리는 뉴스를 뉴스 아이디를 사용하여 다음과 같이 정렬하여 보여줄 것이다. 

```swift
let newsID = [17549050, 17549099, 17548768, 17546915, 17534858, ... ]
```

뷰 컨트롤러에서 우리는 API로부터 가져온 뉴스 객체를 배열을 사용하여 저장할 것이고 이를 테이블 뷰의 데이터 소스로 사용할 것이다. 우리는 또한 API 호출을 위한 `URLSessionDataTask`의 참조를 유지하기 위해 또 다른 배열을 갖는다.

```swift
class PrefetchViewController: UIViewController {	
  // 테이블 뷰 위에 보여질 데이터 소스
  // 100개의 nil로 먼저 배열을 채우고 이후에 각 인덱스/위치에 해당하는 뉴스 객체를 불러왔을 때 이 객체로 nil을 대체
  var newsArray : [News?] = [News?](repeating: nil, count: 100)
	
  // 각각의 뉴스를 불러오는 작업(API를 호출하는 행위)를 저장
  var dataTasks : [URLSessionDataTask] = []
}
```

`viewDidLoad()`에서 테이블 뷰의 데이터 소스와 프리페치 데이터 소스를 설정한다. 

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  newsTableView.dataSource = self
  newsTableView.prefetchDataSource = self
}
```

다음은 API로부터 뉴스 데이터를 가져오는 함수다. 우리는 `URLSessionDataTask`를 만들어 데이터를 받아올 것이며 이 작업이 시작된 뒤 이전에 정의한 `dataTasks` 배열에 이 작업을 추가할 것이다. 

```swift
// 'index' 파라미터는 테이블 뷰 행의 인덱스를 의미한다.
// 이 행에 해당하는 뉴스 데이터를 가져오고 보여줄 것이다.
func fetchNews(ofIndex index: Int) {
  let newsID = newsIDs[index]
  let url = URL(string: "https://hacker-news.firebaseio.com/v0/item/\(newsID).json")!
    
  // 특정 뉴스 url에 대한 데이터 작업이 존재한다면, 이는 이전 혹은 현재 이미 그것을 불러왔다는 의미다. 
  // 함수를 반환함으로써 재다운로드하는 것을 막는다.
  if dataTasks.index(where: { task in
    task.originalRequest?.url == url
  }) != nil {
    return
  }
  
  let dataTask = URLSession.shared.dataTask(with: url) { data, response, error in
    guard let data = data else {
      print("No data")
      return
    }
    
    // JSONDecoder를 사용해 JSON을 Car 구조체 배열로 파싱한다.
    guard let news = try? JSONDecoder().decode(News.self, from: data) else {
      print("Error: Couldn't decode data into news")
      return
    }
    
    // 초기에 'nil'로 되어있던 값이 받아온 뉴스 데이터로 대체되는 것은 테이블 뷰에 보여질 뉴스가 로드되었다는 것을 의미한다.                                             
    self.newsArray[index] = news
    
	// 메인 스레드에서 UI 갱신
    DispatchQueue.main.async {
      let indexPath = IndexPath(row: index, section: 0)
      // API를 호출하여 받아온 뉴스 데이터의 행이 화면에서 보여질 수 있는 행의 영역안인지 검사.
      // 'indexPathForVisibleRows?'가 옵셔널인 이유는 더 이상 보여질 행이 없을 경우 'nil'을 반환하기 때문
      // 'indexPathsForVisibleRows'가 nil일 경우 '?? false'를 통해 false를 반환한다.
      if self.newsTableView.indexPathsForVisibleRows?.contains(indexPath) ?? false {
        // 만일 행이 보여질 수 있으면 (현재는 화면에서 비어있음을 의미), 받아온 데이터를 fade 애니메이션과 함께 refresh한다.
        self.newsTableView.reloadRows(at: [IndexPath(row: index, section: 0)], with: .fade)
      }
    }
  }
  
  // 뉴스를 받아오는 작업을 수행하고 이를 dataTasks 배열에 추가한다.
  dataTask.resume()
  dataTasks.append(dataTask)
}
```

테이블 뷰에 보여질 행이 없다면 `indexPathsForVisibleRows?`는 `nil`을 반환할 것이고 그렇기 때문에 `self.newsTableView.indexPathsForVisibleRows?.contains(indexPath)`는 옵셔널 체이닝으로 `indexPathsForVisibleRows?`가 `nil`을 반환할 경우 전체가 `nil`이 된다. 뒤에 이어지는 `?? false`는 `nil`이 반환되면 `false`를 반환하는 것을 의미한다. 

특정 뉴스 데이터가 로드된 후 이 데이터를 보여줄 행이 보여질 수 있는 행의 영역(현재는 빈칸)에 있다면 우리는 뉴스 데이터와 함께 행을 갱신하거나 리로드할 것이다. 

네트워크 호출을 최적화하기 위해 사용자의 스크롤로 인해 보여질 행으로부터 멀어졌을 경우 그 행의 데이터를 받아오는 `dataTask`를 취소하기 위한 함수인 `cancelFetchNews`를 추가해줄 것이다. 이 함수는 `cancelPrefetchingForRowsAt` 델리게이트 메소드 안에서 사용된다.

```swift
// 'index' 파라미터는 테이블 뷰 행의 인덱스를 의미한다.
func cancelFetchNews(ofIndex index: Int) {
  let newsID = newsIDs[index]
  let url = URL(string: "https://hacker-news.firebaseio.com/v0/item/\(newsID).json")!
  
  // 특정 뉴스를 불러오는 dataTask의 인덱스를 가져온다.
  // 특정 뉴스에 대한 작업이 존재하지 않는다면 취소할 필요 없다.
  guard let dataTaskIndex = dataTasks.index(where: { task in
    task.originalRequest?.url == url
  }) else {
    return
  }
  
  let dataTask =  dataTasks[dataTaskIndex]

  // dataTask를 취소하고 dataTasks 배열로부터 삭제한다.
  // 그래야 다음에 불러오는 것을 완료하지 못하고 취소된 뉴스를 불러오기 위한 새로운 dataTask가 만들어질 수 있다.
  dataTask.cancel()
  dataTasks.remove(at: dataTaskIndex)
}
```

이제 `fetchNews` 함수를 `cellForRowAtIndexPath` 메소드 안에서 호출해줄 것이고, 화면 위의 셀이 자신이 보여줄 데이터를 갖고 있지 않다면 API로부터 데이터를 받아올 것이다. 

```swift
extension PrefetchViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return newsArray.count
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: newsCellIdentifier, for: indexPath) as! NewsTableViewCell
    
    if let news = newsArray[indexPath.row] {
      cell.configureCell(with: news)
    } else {
      // if the news havent loaded (nil havent got replaced), reset all the label
      // 뉴스를 모두 불러오지 못했다면 모든 label을 리셋한다.
      cell.truncateCell()
      
      // fetch the news from API
      self.fetchNews(ofIndex: indexPath.row)
    }
    
    return cell
  }
}
```

`configureCell`은 저자가 작성한 셀이 레이블에 뉴스 데이터를 할당해주는 사용자 정의 메소드이다. `truncateCell`은 레이블의 텍스트를 빈칸으로 설정하는 메소드다.

다음은 이 글의 중요한 부분으로 `fetchNews` 함수를 호출하여 보여지는 행의 영역에 가깝지만 아직 보이지 않는 행에 대한 뉴스 데이터를 받아올 것이다. 그렇기 때문에 사용자는 로딩 딜레이가 없는 것 같은 환상을 보게 될 것이다. (데이터는 행이 화면에 보여지기 전 이미 로드되어 있다.)

```swift
extension PrefetchViewController : UITableViewDataSourcePrefetching {
  func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {

    // API로부터 프리페치 될 행의 뉴스를 받아온다.
    for indexPath in indexPaths {
      self.fetchNews(ofIndex: indexPath.row)
    }
  }
  
  func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) { 
   
    // 사용자 스크롤로 인해 프리페치 될 행으로부터 멀어지면 API로부터 뉴스를 받아오는 작업을 취소
    for indexPath in indexPaths {
      self.cancelFetchNews(ofIndex: indexPath.row)
    }
  }
}
```

우리는 또한 `cancelPrefetchingForRowAt` 델리게이트 메소드 안에서 로딩 중인 행으로부터 멀어지면  `cancelFetchNews`를 호출할 것이다. 이런 방법으로 우리는 그 순간에 필요하지 않은 데이터의 로딩을 취소함으로써 네트워크 호출을 최적화할 수 있다.

[예제 결과 영상](https://iosimage.s3.amazonaws.com/2018/22-prefetch/smoothtrim.mp4)

---

#### 6. Summary

애플이 제공하는 프리페치 메소드를 활용한다면 lazy한 데이터 로드가 가능하고 행이 보여지기 바로 전 데이터를 불러오며 이는 불러오는 과정을 보여주지 않음으로 보다 나은 사용자 경험을 제공한다.

백엔드 개발자나 회사가 설정한 HTTP API 제한으로 프리페치를 사용하는 것이 가능하지 않을 수도 있다. 이번 포스팅에서 언급된 방법들을 활용해 테이블 뷰에서 가장 좋은 사용자 경험을 제공할 수 있는 방법을 자유롭게 선택하여 사용해보자

---

참고자료


1. [Smoothen your table view data loading using UITableViewDataSourcePrefetching](https://fluffy.es/prefetching/#loadall)