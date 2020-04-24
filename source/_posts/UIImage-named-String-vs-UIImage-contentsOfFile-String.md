---
title: 'UIImage(named: String) vs UIImage(contentsOfFile: String)'
date: 2019-05-20 21:47:00
tags: ["UIImage"]
category: ["iOS"]
---

오늘은 간단하게 이미지 파일 이름으로 이미지를 생성하는 `UIImage(named: String)`과 이미지 파일의 저장 위치를 통해 이미지를 생성하는 `UIImage(contentsOfFile: String)`, 두 생성자의 차이점에 대해 알아보려 한다. 

사실 이 둘의 차이점이 있는지 몰랐는데 우연치 않은 계기를 통해 알게 되어 자기 반성과 함께 공부하고 이를 기록하기로 했다.

애플 문서를 보면 바로 이 둘의 확연한 차이를 확인할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/05/20/UIImage-named-String-vs-UIImage-contentsOfFile-String/%20apple_docs.png)

`UIImage(named: String)`은 **Loading and Caching Images** 섹션에, `UIImage(contentOfFile: String)`은 **Creating and Initializing Image Objects** 섹션에 존재한다. 확실히 이미지를 제공하고 사용하는 방법이 다르다는 것을 의미한다. 

그럼 `UIImage(named: String)`부터 살펴보자.

</br>

### UIImage(named: String)

---

이 생성자는 지정한 이름의 이미지 객체가 **시스템 캐시**에 존재하는지를 검사하고 매인 화면에 가장 적합한 이미지를 반환한다. 만일 지정한 이름과 일치하는 이미지 객체가 캐시에 존재하지 않는다면 해당 이미지 데이터를 디스크 혹은 에셋 카탈로그에서 찾아 로드하여 반환한다. 

시스템은 메모리 공간 확보를 위해 캐싱된 이미지 데이터를 지울 수 있다. 이렇게 캐싱된 이미지를 **지우는 행위**는 **현재는 사용되지 않는 이미지**에 한해서 행해진다. 

iOS 9 그리고 으 이후부터 이 메소드는 **스레드로부터 안전**하다.

만일 **한번만 사용**되며 캐싱되지 않기를 원하는 이미지를 사용하기 위해선 **`UIImage(contentsOfFIle)`**을 통해 이미지를 생성하자. 한번만 사용하는 이미지를 캐시에 저장하지 않음으로 우리는 잠재적으로 앱의 메모리 사용 효율을 개선할 수 있다. 

</br>

### UIImage(contentsOfFile: String)

---

이 생성자 함수는 지정된 경로의 이미지 데이터를 메모리로 불러오고 **"지워질 수 있다"** 라고 표시해둔다. 만일 데이터가 지워지고 다시 불러져야 할 필요가 있다면 이미지 객체가 그 경로로부터 다시 데이터를 불러온다.

</br>



이 둘의 차이는 절대 사소하지 않은 것 같다. 

전자(`UIImage(named: String)`)은 아이콘 이미지와 같이 재사용될 수 있는 이미지들에 사용하는 것이 적합하다. 반면 후자(`UIImage(contentsOfFile: String)`)는 재사용하지 않고 자주 사용되지 않는 이미지들을 불러오는데 적합하다.