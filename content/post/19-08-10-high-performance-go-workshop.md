+++
date          = "2019-09-03T20:07:00+09:00"
draft         = true
title         = "[번역] 고성능 Go Workshop"
tags          = ["Go", "Golang", "High Performance"]
categories    = ["Programming"]
slug          = "high-performance-go-workshop"
toc           = true
socialsharing = true
nocomment     = false
+++

{{< notice note >}}
원글은 [High Performance Go Workshop](https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html)입니다. 학습 차원에서 허락없이 번역하였습니다. 저작권 등의 문제가 있을 경우, 이메일로 연락주시길 바랍니다.
{{< /notice >}}

## 개요

이 워크샵의 목표는 Go 애플리케이션에서 성능 문제를 진단하고 해결하는 데 필요한 도구를 제공하는 것입니다.

하루 동안 작은 것부터 작업할 것입니다. 벤치마크를 작성하는 방법을 배우고, 작은 코드 조각을 프로파일링하는 등 그런 다음 한걸음 나와 엑스큐션 트레이서, 가비지 콜렉터, 실행 중인 애플리케이션 트레이싱에 대해 얘기하고자 합니다. 남는 시간에 질문을 하며 자신의 코드로 실험해 볼 수 있을 것입니다.

> 이 프리젠테이션의 최신 버전은 다음 사이트에서 제공됩니다.
> http://bit.ly/dotgo2019


## 일정

다음은 오늘의 개략적인 일정입니다.

|시작 |설명|
|-----|---|
|09:00|환영 인사 그리고 소개|
|09:30|벤치마킹|
|10:45|휴식 (15분)|
|11:00|성능 측정 그리고 프로파일링|
|12:00|점심 (90분) |
|13:30|컴파일러 최적화|
|14:30|엑스큐션 트레이서|
|15:30|휴식 (15분)|
|15:45|메모리 그리고 가비지 콜렉터|
|16:15|팁과 트릭|
|16:30|연습|
|16:45|최종 질문 및 결론|
|17:00|맺음말|

## 환영인사

안녕하세요, 환영합니다 🎉

이 워크샵은 Go 애플리케이션에서 성능 문제를 진단하고 해결하는 데 필요한 도구를 제공하려는 것이 목적입니다. 

하루 동안 작은 것부터 작업할 것입니다. 벤치마크를 작성하는 방법을 배우고, 작은 코드 조각을 프로파일링하는 등 그런 다음 한걸음 나와 엑스큐션 트레이서, 가비지 콜렉터, 실행 중인 애플리케이션 트레이싱에 대해 얘기하고자 합니다. 남는 시간에 질문을 하며 자신의 코드로 실험해 볼 수 있을 것입니다.

### 강사
- Dave Cheney dave@cheney.net

## 라이센스 및 내용물

이 워크숍은 [데이브 체니(Dave Cheney)](https://twitter.com/davecheney)와 [프란체스코 캄포이(Francesc Campoy)](https://twitter.com/francesc)의 합작품입니다.

이 프레젠테이션은 [Creative Commons At Attion-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/) 라이센스에 따라 라이센스가 부여됩니다.

## 전제 조건 

오늘 다운로드해야 할 몇 가지 소프트웨어들이 있습니다.

### 워크샵 저장소

[https://github.com/davecheney/high-performance-go-workshop](https://github.com/davecheney/high-performance-go-workshop)에서 이 문서에 대한 소스와 및 코드 샘플을 다운로드하세요.

### 노트북, 전원 공급 장치 등
워크샵 자료는 Go 1.12를 대상으로 합니다.

[Go 1.12 다운로드](https://golang.org/dl/)

> Go 1.13으로 이미 업그레이드했다면 좋습니다. Go 마이너 릴리즈마다 항상 최적화 선택에 따라 약간의 변경 사항이 있는데, 진행하면서 언급하겠습니다. 

### Graphviz

pprof 섹션에서 그래피즈 도구 모음과 함께 제공되는 `dot` 프로그램이 필요합니다.

- Linux: `[sudo] apt-get install graphviz`
- OSX:
	- MacPorts: `sudo port install graphviz`
	- Homebrew: `brew install graphviz`
- [Windows](https://graphviz.gitlab.io/download/#Windows) (테스트 안됨)

### 구글 크롬

엑스큐선 트레이서 (execution tracer) 섹션에서 구글 크롬이 필요합니다. Safari, Edge, Firefox 또는 IE 4.01에서는 작동하지 않습니다. 배터리에게 미안하다고 말해주세요. (역자주, 미국식 농담??)

[구글 크롬 다운로드](https://www.google.com/chrome/)

### 프로파일링과 최적화를 위한 자신의 코드

오늘 남는 시간에는 배운 도구로 실험을 해보는 공개 세션이 될 것입니다.

### 한 가지 더

이건 강의가 아니라 대화입니다. 질문을 할 수 있는 많은 휴식 시간이 있을 겁니다.

만약 여러분이 무언가를 이해하지 못하거나, 여러분이 듣고 있는 것이 정확하지 않다고 생각된다면, 질문하세요.

## 1. 마이크로 프로세서의 과거, 현재, 그리고 미래 

고성능 코드를 작성하는 것에 대한 워크샵입니다. 다른 워크샵에서는 디커플된 설계와 유지보수성에 대해 이야기하지만, 오늘은 성능에 대해 이야기하려고 합니다.

저는 오늘 컴퓨터의 진화의 역사에 대해 어떻게 생각하는지 그리고 왜 고성능 소프트웨어를 작성하는 것이 중요하다고 생각하는지에 대한 짧은 강의로 시작하고 싶습니다.

실제로 소프트웨어는 하드웨어에서 실행되므로 고성능 코드 작성에 대해 이야기하려면 먼저 코드를 실행하는 하드웨어에 대해 이야기해야 합니다.

### 1.1 기계적 공감

![](https://dave.cheney.net/high-performance-go-workshop/images/image-20180818145606919.png)

요즘 대중적으로 사용되는 용어가 있는데, 마틴 톰슨(Martin Thompson)이나 빌 케네디(Bill Kennedy) 같은 사람들이 "기계적 공감"에 대해 이야기하는 것을 들을 수 있습니다.

"기계적 공감"이라는 이름은 세계 3대 포뮬라1 챔피언인 위대한 경주 자동차 드라이버인 재키 스튜어트(Jackie Stewart)로부터 유래했습니다. 그는 최고의 운전자는 기계가 어떻게 작동하는지 충분히 이해하고 있어서 기계와 조화를 이룰 수 있다고 믿었습니다.

훌륭한 경주용 자동차 드라이버가 되기 위해서는 훌륭한 정비사가 될 필요는 없지만, 자동차 작동 방식에 대한 호기심 이상의 이해가 필요합니다.

소프트웨어 엔지니어들도 마찬가지라고 생각합니다. 저는 이 자리에 계신 여러분 중 누구도 전문적인 CPU 디자이너가 되지는 않을 것이라고 생각합니다. 하지만 그렇다고 해서 CPU 디자이너들이 직면하고 있는 문제들을 무시할 수는 없습니다.

### 1.2 10의 6승

다음과 같은 일반적인 인터넷 밈이 있습니다.

![](https://dave.cheney.net/high-performance-go-workshop/images/jalopnik.png)

물론 말도 안 되는 소리이지만, 이는 컴퓨터 업계에서 얼마나 많은 변화가 일어났는지를 잘 보여주고 있습니다.

이 자리에 모인 소프트웨어 저자들은 모두 무어의 법칙의 혜택을 받아왔기 때문에, 40년 동안 18개월마다 칩에서 사용할 수 있는 트랜지스터의 수가 두 배로 증가했습니다. 평생동안 툴이 10의 6승 만큼 개선된 산업 분야는 없습니다.

하지만 이 모든 것이 변하고 있습니다.

### 1.3 컴퓨터는 여전히 더 빨라지고 있나요?

그래서 근본적인 질문은, 위의 그림에서와 같은 통계와 마주치게 됩니다. 컴퓨터가 여전히 더 빨라지고 있는지 물어봐야 할까요?

컴퓨터가 여전히 빨라지고 있다면, 코드 성능에 신경 쓸 필요가 없습니다. 조금만 기다리면 하드웨어 제조업체가 성능 문제를 해결할테니깐요.

#### 1.3.1 데이터를 살펴봅시다.

다음은 컴퓨터 아키텍처, 존 해네시(John L. Hennessy)의 정량적 접근(A Quantitative Approach, David A. Patterson)과 같은 교과서에서 볼 수 있는 고전적인 데이터입니다. 이 그래프는 5판에서 가져왔습니다.

![](https://community.cadence.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-01-06/2313.processorperf.jpg)

5판에서 헤네시(Hennessey)와 패터슨(Patterson)은 세 가지 컴퓨팅 성능 시대가 있다고 주장합니다.

- 첫 번째는 1970년대와 80년대 초였으며, 형성기였습니다. 오늘날 우리가 알고 있는 마이크로프로세서는 실제로 존재하지 않았습니다. 컴퓨터는 개별 트랜지스터나 소형 집적회로에서 제작되었습니다. 재료 과학의 비용, 크기, 그리고 이해의 한계가 제한 요인이었습니다.

- 80년대 중반부터 2004년까지 추세선은 명확합니다. 컴퓨터 정수 성능이 매년 평균 52% 향상되었습니다. 컴퓨터 전력은 2년마다 두 배씩 증가했기 때문에 사람들은 트랜지스터 다이의 수를 두 배로 늘려가며 무어의 법칙과 컴퓨터 성능을 혼용하였습니다.

- 그리고 나서 우리는 컴퓨터 성능의 세 번째 시대에 도달했습니다. 느려집니다. 총 변화율은 연간 22%입니다.

이전 그래프는 2012년까지만 올라갔지만, 다행히도 2012년 [제프 프레싱(Jeff Preshing)](http://preshing.com/20120208/a-look-back-at-single-threaded-cpu-performance/)은 [Spec 웹 사이트를 스크래치하고 자신만의 그래프를 만드는 도구](https://github.com/preshing/analyze-spec-benchmarks)를 작성했습니다.

![](https://dave.cheney.net/high-performance-go-workshop/images/int_graph.png)

이것은 1995년부터 2017년까지 Spec 데이터를 사용한 것과 동일한 그래프입니다.

2012년 데이터에서 살펴본 단계 변화보다는 단일 코어 성능이 한계에 근접하고 있다고 말씀드리고 싶습니다. 부동 소수점보다 숫자가 더 좋지만, 이 자리에 있는 비즈니스 애플리케이션 라인의 업무를 수행하는 저희에게 있어, 그다지 관련이 없을 것입니다.

#### 1.3.2 네, 컴퓨터는 여전히 더 빨라지고 있어요, 천천히

> 무어의 법칙 끝에서 가장 먼저 기억해야 할 것은 고든 무어(Gordon Moore)가 내게 얘기한 것입니다. 그는 "모든 지수가 끝나고 있다"고 말했습니다. — 존 헤네시


이것은 헤네시(Hennessy)가 Google Next 18에서 그의 Turing Award 강연에서 인용한 것입니다. 그의 주장은 네, CPU 성능은 여전히 향상되고 있습니다. 그러나 단일 쓰레드 정수(Integer) 성능은 여진히 연간 약 2~3% 개선되고 있습니다. 이런 속도라면 정수의 성능을 두 배로 끌어올리기 위해 20년 간의 복합 성장(Compound Growth)을 해야할 것입니다. 이를 2년마다 성능이 2배씩 증가한 90년대의 잘 나가던(Go-Go) 시대와 비교해야 합니다.

왜 이런 일이 일어나는 걸까요?

### 1.4 클럭 속도

![](https://dave.cheney.net/high-performance-go-workshop/images/stuttering.png)

2015년도의 그래프가 이를 잘 보여줍니다. 상단 라인은 트랜지스터 다이의 수를 나타냅니다. 이것은 1970년대 이후로 거의 선형적인 추세선상에서 지속되어 왔습니다. 로그/린 그래프이기 때문에 이 선형 계열은 지수 성장을 나타냅니다.

하지만, 중간 라인을 보면, 10년 동안 클럭 속도가 증가하지 않았으며 2004년경에 CPU 속도가 정체된 것을 알 수 있습니다.

하단 그래프는 열 방출 전력(thermal dissipation power)을 보여 줍니다. 즉, 열로 변환되는 전력은 동일한 패턴을 따릅니다. 클럭 속도와 CPU 열 방출은 서로 관련이 있습니다.

### 1.5 열

CPU가 열을 발생시키는 이유는 무엇일까요? 고체 상태 장치(Solid State Device)이고 움직이는 구성 요소가 없으므로 마찰과 같은 효과는 (직접적으로) 관련이 없습니다.

이 다이어그램은 [TI에서 제작한 훌륭한 데이터 시트](http://www.ti.com/lit/an/scaa035b/scaa035b.pdf)에서 가져온 것입니다. 이 모델에서 N타입 장치의 스위치는 양극 전압으로 끌어 당겨집니다. P타입 디바이스는 양극 전압에서 쫓겨납니다.

![](https://dave.cheney.net/high-performance-go-workshop/images/cmos-inverter.png)

이 방, 여러분의 책상, 그리고 여러분의 주머니에 있는 모든 트랜지스터에 사용되는 CMOS 기기의 전력 소비량은 세 가지 요인의 조합입니다.

- 정적 전력(static power). 트랜지스터가 정적인 경우 즉, 상태를 변경하지 않으면 트랜지스터를 통해 접지로 누설되는 소량의 전류가 있습니다. 트랜지스터가 작을수록 누수가 많아집니다. 온도가 높아지면 누수가 증가합니다. 수 십억 개의 트랜지스터가 있으면 1분이라도 누수가 발생합니다!

- 동적 전력(dynamic power). 트랜지스터가 한 상태에서 다른 상태로 전이될 때, 트랜지스터는 게이트에 연결된 다양한 커패시턴스를 충전하거나 방전해야 합니다. 트랜지스터당 동적 전력은 제곱한 전압 곱하기 커패시턴스와 변화 빈도입니다. 전압을 낮추면 트랜지스터가 소비하는 전력이 감소하지만 전압이 낮으면 트랜지스터가 느리게 전환됩니다.

- 크라우바 또는 단락 전류(crowbar, or short circuit current) 우리는 트랜지스터를 원자 상태에서 하나의 상태 또는 다른 상태를 차지하는 디지털 장치로 생각합니다. 실제로 트랜지스터는 아날로그 장치입니다. 스위치로서 트랜지스터는 대부분 오프(off) 상태에서 시작하여 온(on) 상태로 전이 또는 전환됩니다. 이러한 전이 또는 전환 시간은 매우 빠릅니다. 현대 프로세서에서는 피코 초의 단위이지만, Vcc에서 접지로의 저항 경로가 있을 때 여전히 어느 정도의 시간이 걸립니다. 트랜지스터의 전환 속도가 빠를수록 더 많은 열이 방출됩니다.

### 1.6 Dennard 스케일링의 종말

다음에 무슨 일이 일어났는지 이해하려면 1974년에 로버트 H. 데너드(Robert H. Dennard)가 공동 저술한 논문을 살펴봐야 합니다. 데너드의 스케일링 법칙에 따르면 트랜지스터가 작아질수록 전력 밀도는 일정하게 유지됩니다. 트랜지스터가 작아질수록 전압도 낮아지고, 게이트 커패시턴스도 낮아지며, 스위치 속도도 빨라져 동적 전력량을 줄일 수 있습니다.

그래서 어떻게 되었을까요?

![](http://semiengineering.com/wp-content/uploads/2014/04/Screen-Shot-2014-04-14-at-8.49.48-AM.png)

결과는 별로 좋지 않습니다. 트랜지스터의 게이트 길이가 몇 개의 실리콘 원자의 너비에 접근함에 따라 트랜지스터 크기, 전압 및 중요한 누수 사이의 관계가 끊어졌습니다.

[1999년 Micro-32 컨퍼런스](https://pdfs.semanticscholar.org/6a82/1a3329a60def23235c75b152055c36d40437.pdf)에서 클럭 속도를 높이고 트랜지스터 크기를 줄이는 추세를 따랐다면, 프로세서 시대의 트랜지스터 접합부가 원자로 코어 온도에 근접할 것이라고 가정했습니다. 분명히 바보 같은 짓이었죠. 펜티엄 4는 단일 코어, 고주파, 소비자 CPU [라인의 종지부를 찍었습니다](https://arstechnica.com/uncategorized/2004/10/4311-2/).

다시 그래프로 돌아와서, 클럭 속도가 멈춘 이유는 CPU의 속도가 CPU를 냉각시키는 능력을 초과했기 때문입니다. 2006년까지 트랜지스터 크기의 감소로 전력 효율은 더 이상 개선되지 않았습니다.

이제 CPU 피처 크기 감소는 주로 전력 소비를 줄이는 데 목적이 있다는 것을 알게 되었습니다. 전력 소비량을 줄인다고 해서 단순히 지구를 살리는 재활용과 같은 "친환경"을 의미하는 것이 아닙니다. 주된 목표는 전력 소비를 유지하고, 그러한 열 손실로, [CPU를 손상시키는 수준을 낮추는](https://en.wikipedia.org/wiki/Electromigration#Practical_implications_of_electromigration) 것입니다.

![](https://dave.cheney.net/high-performance-go-workshop/images/stuttering.png)

하지만, 그래프의 한 부분에서 보듯이, 트랜지스터 다이 수는 계속 증가하고 있습니다. 주어진 동일한 영역에 더 많은 트랜지스터를 집적하는 CPU 피처 크기의 행진은 긍정적인 효과와 부정적인 효과 모두를 지닙니다.

또한, 다음 인서트에서도 볼 수 있듯이 트랜지스터당 비용은 5년 전까지 계속 하락했습니다. 그 이후 트랜지스터당 비용이 다시 상승하기 시작했습니다.

![](https://whatsthebigdata.files.wordpress.com/2016/08/moores-law.png)

더 작은 트랜지스터를 만드는 데, 더 많은 비용이 들 뿐만 아니라 더 어려워지고 있습니다. 2016년의 이 보고서는 칩 제조사들이 2013년 격게 될 일의 예측했었습니다. 2년 후 그들의 모든 예측이 빗나갔습니다. 이 보고서의 최신 버전은 없지만, 이러한 추세를 되돌릴 수 있을 것 같은 징후는 보이지 않습니다.

인텔, TSMC, AMD, 그리고 삼성은 새로운 공장을 짓고 새로운 프로세스 툴링을 구매해야 하기 때문에 수 십억 달러의 비용이 듭니다. 그래서 다이당 트랜지스터 수가 계속 상승하며, 단가가 상승하기 시작했습니다.

{{< notice note >}}
나노미터 단위로 측정했던 게이트 길이(gate length)라는 용어도 모호해졌습니다. 다양한 제조사들은 트랜지스터 크기를 다양한 방법으로 측정하여, 납품없이 경쟁사보다 적은 수의 트랜지스터를 시연할 수 있습니다. 이는 CPU 제조사의 비일반회계(Non-GAAP) 수익 보고 모델입니다.
{{< /notice >}}

### 1.7 더 많은 코어

![](https://i.redd.it/y5cdp7nhs2uy.jpg)

열 및 주파수 한계에 도달하면 단일 코어 속도를 2배 빠르게 실행할 수 없습니다. 그러나 다른 코어를 추가하면 소프트웨어에서 지원할 수 있는 경우 처리 용량을 두 배로 제공할 수 있습니다.

실제로 CPU의 코어 수는 열 방출에 의해 좌우됩니다. 데나드 스케일링의 종말은 CPU 클럭 속도가 얼마나 뜨거운 지에 따라 1~4Ghz 사이의 임의의 숫자라는 것을 의미합니다. 벤치마킹에 대해 이야기할 때 이것을 잠깐 보게 될 것입니다.

### 1.8 암달의 법칙

CPU는 점점 더 빨라지지는 않지만, 하이퍼 스레딩과 다중 코어로 점점 더 확장되고 있습니다. 모바일 부품에 듀얼 코어, 데스크톱 부품에 쿼드 코어, 서버 부품에 수십 개의 코어. 이것이 컴퓨터 성능의 미래일까요? 불행히도 그렇지 않습니다.

IBM/360의 설계자인 진 암달(Gene Amdahl)의 이름을 딴 암달의 법칙은 리소스가 개선된 시스템에서 기대할 수 있는 고정 워크로드에서 작업 실행의 대기 시간을 이론적으로 가속화하는 공식입니다.

![](https://upload.wikimedia.org/wikipedia/commons/e/ea/AmdahlsLaw.svg)

암달의 법칙에 따르면 프로그램의 최대 속도는 프로그램의 순차적인 부분에 의해 제한됩니다. 실행의 95%를 병렬로 실행할 수 있는 프로그램을 작성하는 경우, 수천 개의 프로세서를 사용하더라도 프로그램 실행의 최대 속도는 20배로 제한됩니다.

우리가 매일 작업하는 프로그램에 대해 생각해봅시다. 얼마나 많이 병렬로 실행할 수 있을까요?

### 1.9 동적 최적화

클럭 속도가 지연되고, 추가 코어를 투척해도 소득이 없는 문제점이 있을 때, 속도 향상은 어디서 오는 것일까요? 속도는 칩 자체의 아키텍처를 개선한 결과입니다. 이들은 [Nehalem](https://en.wikipedia.org/wiki/List_of_Intel_CPU_microarchitectures#Pentium_4_/_Core_Lines), [Sandy Bridge](https://en.wikipedia.org/wiki/List_of_Intel_CPU_microarchitectures#Pentium_4_/_Core_Lines), 및 [Skylake](https://en.wikipedia.org/wiki/List_of_Intel_CPU_microarchitectures#Pentium_4_/_Core_Lines)와 같은 이름의 5~7년 짜리의 큰 프로젝트입니다.

지난 20년 동안의 성능 향샹은 아키텍처 개선에서 비롯되었습니다.

#### 1.9.1 비순차적 명령어 처리 (Out of order execution)

슈퍼 스칼라라고 하는 비순차적 실행은 CPU가 실행 중인 코드에서 소위 명령어 수준 병렬 처리를 추출하는 방법입니다. 최신 CPU는 하드웨어 수준에서 SSA를 효과적으로 수행하여 작업 간의 데이터 종속성을 식별하고 가능한 경우 독립적인 명령을 병렬로 실행합니다.

그러나 모든 코드에 내재된 병렬 처리량에는 제한이 있습니다. 엄청나게 힘도 듭니다. 대부분의 최신 CPU는 파이프라인의 각 단계에서 각 실행 장치를 다른 모든 장치에 연결하는데 n제곱의 비용이 발생하기 때문에 코어당 6개의 실행 장치로 정착했습니다.

#### 1.9.2 추측 실행 (Speculative execution)

가장 작은 마이크로 컨트롤러들을 모으면, 모든 CPU는 fetch/decode/execute/commit 명령 사이클의 일부와 겹치는 명령 파이프라인(instruction pipeline)을 활용합니다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Fivestagespipeline.png/800px-Fivestagespipeline.png)

명령어 파이프라인의 문제는 분기 명령어이며 평균 5-8 명령어마다 발생합니다. CPU가 분기에 도달하면 분기를 넘어 추가적인 명령을 실행할 수 없으며 프로그램 카운터도 분기할 위치를 알 때까지 파이프 라인 채우기를 시작할 수 없습니다. 추측 실행은 CPU가 분기 명령이 여전히 처리되고 있는 중에도 어느 경로로 분기해야 할지 "추측"할 수 있게 합니다!

CPU가 분기를 올바르게 예측하면 명령 파이프라인을 가득 채울 수 있습니다. CPU가 올바른 분기를 예측하지 못하고 오류로 인식하면 아키텍처 상태에 대한 변경 사항을 롤백해야 합니다. 우리 모두가 Spectre 스타일의 취약점을 배우고 있기 때문에, 때때로 이 롤백은 기대만큼 원활하지 않은 경우가 있습니다.

분기 예측률이 낮을 때, 추측 실행이 매우 어려울 수 있습니다. 분기가 잘못 예측된 경우, CPU 역추적은 잘못 예측한 지점까지 역추적해야 할 뿐만 아니라 잘못된 분기에서 소비된 에너지가 낭비됩니다.

엄청난 수의 트랜지스터와 전력을 희생하며 이러한 모든 최적화로 단일 쓰레드 성능이 개선되었습니다.

{{< notice note >}}
클리프 클릭(Cliff Click)은 비순차적 및 추론적 실행은 캐시 미스를 조기에 시작하여 관찰된 캐시 대기 시간을 줄이는데 가장 유용하다고 논증하는 [훌륭한 프리젠테이션](https://www.youtube.com/watch?v=OFgxAFdxYAQ)을 제공합니다.
{{</ notice >}}

### 1.10 현대 CPU는 벌크 동작에 대해 최적화되었습니다.

> 현대의 프로세서는 니트로 연료가 함유된 재미있는 자동차와 비슷하며 단거리 경주에서 뛰어납니다. 불행히도 현대의 프로그래밍 언어는 Monte Carlo와 비슷하지만 트위스트와 턴으로 가득합니다. — 데이비드 언가(David Ungar)

이는 영향력 있는 컴퓨터 과학자이자 SELF 프로그래밍 언어의 개발자인 데이비드 언가의 인용문으로 온라인에서 찾은 아주 오래된 프리젠테이션에서 언급되었습니다.

따라서 최신 CPU는 대량 전송 및 대량 작업에 최적화되어 있습니다. 모든 레벨에서 작업 설정 비용은 대량 작업을 권장합니다. 예를 들어, 다음과 같습니다.

- 메모리는 바이트당 로드되지 않고 여러 캐시 라인마다 로드되므로 정렬이 이전 컴퓨터에서만큼 문제가 되지않는 이유입니다.
- MMX 및 SSE와 같은 벡터 명령을 사용하면 프로그램이 해당 형식으로 표현될 수 있는 동시에 다중 데이터 항목에 대해 단일 명령을 실행할 수 있습니다.

### 1.11 현대 프로세서는 메모리 용량이 아닌 메모리 지연 시간으로 제한됩니다.

CPU라는 토지 상황이 그다지 나쁘지 않았다면, 메모리라는 집 쪽에서 들려오는 뉴스는 크게 나아지지 않습니다.

서버에 연결된 물리적 메모리가 기하학적으로 증가했습니다. 1980년대에 저의 첫 컴퓨터는 킬로바이트의 메모리를 가지고 있었습니다. 고등학교를 다닐 때 저는 386에 1.8메가바이트의 램을 가지고 모든 에세이를 썼습니다. 이제 수십 기가바이트 또는 수백 기가바이트의 램을 가진 서버를 찾는 것이 일반적이며, 클라우드 공급업체들은 테라바이트의 램을 도입하고 있습니다.

그러나 프로세서 속도와 메모리 액세스 시간 사이의 간극은 계속 증가하고 있습니다.

![](https://www.extremetech.com/wp-content/uploads/2018/01/mem_gap.png)

하지만, 메모리 대기 중에 손실된 프로세서 싸이클의 경우, 메모리가 CPU 속도의 증가에 보조를 맞추지 못했기 때문에 물리적 메모리는 그 어느때보다 여전히 멀리 떨어져 있습니다.

따라서 대부분의 최신 프로세서는 커패시티가 아닌 메모리 레이턴시로 제한됩니다.

![](https://pbs.twimg.com/media/BmBr2mwCIAAhJo1.png)

### 1.12 캐시가 주변의 모든 것을 지배합니다.

![](https://www.extremetech.com/wp-content/uploads/2014/08/latency.png)

수십 년 동안 프로세서/메모리 캡에 대한 솔루션은 캐시를 추가하는 것이 었습니다. 캐시는 CPU에 더 가까이, 이제는 직접 통합된 작은 고속 메모리입니다.

그러나;

- L1은 수십 년 동안 코어 당 32kb로 고정되었습니다.

- L2는 가장 큰 인텔 부품에서 최대 512kb까지 천천히 상승했습니다.

- L3는 지금 4-32mb 범위로 측정되지만, 액세스 시간은 가변적입니다.

![](https://i3.wp.com/computing.llnl.gov/tutorials/linux_clusters/images/E5v4blockdiagram.png)

캐시는 [CPU 다이에서 물리적으로 크기 때문에](http://www.itrs.net/Links/2000UpdateFinal/Design2000final.pdf) 크기가 제한되어 많은 전력을 소비합니다. 캐시 미스률을 절반으로 줄이려면 캐시 크기를 4배로 늘려야합니다.

### 1.13 공짜 점심은 끝났다.

2005년 C++위원회 리더인 허브 서터(Herb Sutter)는 [공짜 점심은 끝났다](http://www.gotw.ca/publications/concurrency-ddj.htm)라는 제목의 기사를 썼습니다. 그의 글에서 서터는 제가 다룰 모든 점에 대해 논의했으며 미래 프로그래머는 더 이상 느린 프로그램 또는 느린 프로그래밍 언어를 수정하기 위해 더 빠른 하드웨어에 의존할 수 없을 것이라고 주장했습니다.

10년이 지난 지금, 허브 서터가 옳았다는 것에는 의심의 여지가 없습니다. 메모리가 느리고 캐시가 너무 작고, CPU 클럭 속도가 거꾸로 가고 있으며 단일 쓰레드 CPU의 단순한 세계는 오래 전에 사라졌습니다.

무어의 법칙은 여전히 유효하지만, 이 자리에 있는 우리 모두에게 공짜 점심은 끝이 났습니다.


### 1.14 결론

> 제가 인용하려는 숫자들은 2010년까지 될 것입니다. 30GHz, 100억 개의 트랜지스터, 초당 1테라 인스트럭션이 그 숫자들이죠. — [팻 겔싱어(Pat Gelsinger), 인텔의 전설적인 CTO, 2002년 4월](https://www.cnet.com/news/intel-cto-chip-heat-becoming-critical-issue/)

재료 과학의 획기적인 발전없이는 CPU 성능의 연간 증가율 52%의 시대로 돌아올 가능성이 거의 없다는 것이 분명합니다. 일반적인 합의는 재료 과학 자체가 아니라 트랜지스터 사용 방법에 결함이 있다는 것입니다. 실리콘으로 표현된 순차적 명령흐름의 논리적 모델은 값비싼 최종 게임으로 귀결됩니다.

온라인에 많은 프리젠테이션이 이 점을 반복하고 있습니다. 미래에는 컴퓨터가 오늘날처럼 프로그래밍되지 않을 것입니다. 일부는 수백 개의 매우 멍청하고 일관성이 없는 프로세서를 갖춘 그래픽 카드처럼 보일 것이라고 주장합니다. 다른 사람들은 VLIW(Very Long Instruction Word) 컴퓨터가 우세할 것이라고 주장합니다. 현재 순차 프로그래밍 언어는 이러한 종류의 프로세서와 호환되지 않을 것이라는 데 모두 동의합니다.
 
제 생각에는 이러한 예측이 정확하다는 것입니다. 이 시점에서 우리를 구해 온 하드웨어 제조업체에 대한 전망은 어둡습니다. 그러나 오늘날 우리가 쓰는 하드웨어에 작성하는 프로그램을 최적화할 수 있는 광범위한 방법이 있습니다. 릭 허드슨(Rick Hudson)은 GopherCon 2015에서 오늘날의 하드웨어와 무관하지 않게 동작하는 소프트웨어의 "유연한 순환"에 다시 참여하는 것에 대해 말했습니다.

앞서 보여드린 그래프를 보면, 2015년부터 2018년까지 정수의 성능은 기껏해야 5~8% 향상되고 메모리 대기 시간은 이보다 적은 것으로 나타났는데, Go팀은 가비지 콜렉션 일시 중지 시간을 2자리 수의 크기로 줄였습니다. Go 1.11 프로그램은 Go 1.6을 사용하는 동일한 하드웨어의 동일한 프로그램보다 GC 대기 시간이 훨씬 뛰어납니다. 이 중 어느 것도 하드웨어에서 비롯된 것이 아닙니다.

따라서, 오늘날의 오늘날 하드웨어에서 최상의 성능을 얻으려면 다음과 같은 프로그래밍 언어가 필요합니다.

- 해석된 프로그래밍 언어가 CPU 분기 예측 변수 및 추론 실행과 제대로 상호 작용하지 않기 때문에 해석되지 않고 컴파일됩니다.

- 효율적인 코드를 작성할 수 있는 언어가 필요하며, 모든 숫자가 이상적인 부동 소수인 척 하기보다는 비트와 바이트, 정수의 길이에 대해 효과적으로 얘기할 수 있어야 합니다.

- 프로그래머가 메모리에 대해 효과적으로 이야기하고 구조체와 자바 객체를 생각할 수 있는 언어가 필요합니다. 왜냐하면 포인터 추적이 CPU 캐시에 압력을 가하고 캐시 미스가 수백 번의 사이클을 태우기 때문입니다.

- 애플리케이션 성능에 따라 여러 코어로 확장되는 프로그래밍 언어는 캐시를 얼마나 효율적으로 사용하고 여러 코어에서 작업을 얼마나 효율적으로 병렬화할 수 있는지에 따라 결정됩니다.

분명히 우리는 Go에 대해 이야기하러 여기에 왔습니다. 그리고 저는 Go가 방금 설명한 많은 특징들을 물려받았다고 믿습니다.

#### 1.14.1 우리와 무슨 상관이 있나요?

> 최적화는 세 가지 뿐입니다. 적게 하세요. 덜 자주하세요. 더 빨리 하세요.
> 가장 큰 이득은 첫 번째에서 비롯되지만, 우리는 모든 시간을 세 번째에서 소비합니다. — Michael Fromberger

이 강의의 요점은 프로그램이나 시스템의 성능에 대해 이야기 할 때 전적으로 소프트웨어에 있음을 설명하는 것입니다. 더 빠른 하드웨어가 하루를 구하기를 기다리는 것은 어리석은 일입니다.

그러나 좋은 소식이 있습니다. 소프트웨어에서 할 수 있는 많은 개선 사항들이 있습니다. 그리고 그것이 오늘 이야기하고자 하는 것입니다.

#### 1.14.2 더 읽을 거리 

- [The Future of Microprocessors, Sophie Wilson](https://www.youtube.com/watch?v=zX4ZNfvw1cw) JuliaCon 2018
- [50 Years of Computer Architecture: From Mainframe CPUs to DNN TPUs, David Patterson](https://www.youtube.com/watch?v=HnniEPtNs-4)
- [The Future of Computing, John Hennessy](https://web.stanford.edu/~hennessy/Future%20of%20Computing.pdf)
- [The future of computing: a conversation with John Hennessy](https://www.youtube.com/watch?v=Azt8Nc-mtKM) (Google I/O '18)

## 2. 벤치마킹

> 두 번 측정하고 한 번 잘라라 — 고대 속담

코드의 성능을 향상시키기 전에 먼저 현재 성능을 알아야 합니다.

이 섹션에서는 Go 테스트 프레임워크를 사용하여 유용한 벤치마크를 구성하는 방법에 중점을 두고 위험요소을 피하기 위한 실용적인 팁을 제공합니다.

### 2.1 벤치마킹 기본 규칙

벤치마킹하기 전에 반복 가능한 결과를 얻으려면 안정적인 환경이 있어야 합니다.

- 컴퓨터는 유휴 상태이어야 합니다. 공용 하드웨어에서 프로파일링하지 말고 긴 벤치마크가 실행되기를 기다리는 동안 웹을 탐색하지 마세요.

- 절전 및 열 스케일링에 주의하세요. 이들은 현대 노트북에서는 거의 피할 수 없습니다.

- 가상머신 및 공유 클라우드 호스팅을 피하세요. 일관된 측정을 하기에는 노이즈가 꽤 있을 수 있습니다.

여유가 있다면 전용 성능 테스트 하드웨어를 구입하세요. 랙을 세팅하고, 모든 전원 관리 및 열 스케일링을 비활성화한 다음, 시스템의 소프트웨어를 업데이트하지 마세요. 마지막 사항은 시스템 관리 관점에서는 좋지 않은 조언이지만 소프트웨어 업데이트로 커널 또는 라이브러리의 성능을 변경하는 경우, (Spectre 패치를 생각해 보시면), 이전의 모든 벤치마킹 결과가 무효화되어 버립니다.

우리 모두를 위해, 사전 및 사후 샘플을 가지고 일관된 결과를 얻으려면 여러 번 실행하시길 바랍니다. 

### 2.2. 벤치마킹에 테스트 패키지 사용

테스트 패키지는 벤치마크 작성을 지원합니다. 다음과 같은 간단한 기능이 있다면,

```golang
func Fib(n int) int {
	switch n {
	case 0:
		return 0
	case 1:
		return 1
	default:
		return Fib(n-1) + Fib(n-2)
	}
}
```

테스트 패키지를 사용하여 다음 형태로 함수의 벤치마크를 작성할 수 있습니다.

```golang
func BenchmarkFib20(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Fib(20) // run the Fib function b.N times
	}
}
```

{{< notice tip >}}
벤치마크 함수는 `_test.go` 파일에 테스트 함수와 나란히 있습니다.
{{</ notice >}}

벤치마크는 테스트와 유사하지만 유일한 차이점은 `*testing.T` 대신 `* testing.B`를 사용한다는 것입니다. 이 두 가지 유형 모두 `Errorf()`, `Fatalf()` 및 `FailNow()`와 같은 대중들이 좋아하는 `testing.TB` 인터페이스를 구현합니다.

#### 2.2.1 패키지의 벤치마크 실행

벤치마크에서 테스트 패키지를 사용할 때, `go test` 하위 명령을 통해 실행됩니다. 그러나 기본적으로 `go test`만 호출하면 벤치마크는 제외됩니다.

패키지에서 벤치마크를 명시적으로 실행하려면 `-bench` 플래그를 사용하세요. `-bench`는 실행하려는 벤치마크 이름과 일치하는 정규식을 사용하므로 패키지에서 모든 벤치마크를 호출하는 가장 일반적인 방법은 `-bench=.`입니다. 다음은 예제입니다.

```sh
% go test -bench=. ./examples/fib/
goos: darwin
goarch: amd64
BenchmarkFib20-8           30000             40865 ns/op
PASS
ok      _/Users/dfc/devel/high-performance-go-workshop/examples/fib     1.671s
```

{{< notice tip >}}
`go test`는 벤치마크와 일치하기 전에 패키지의 모든 테스트를 실행합니다. 따라서, 패키지에 많은 테스트가 있거나 실행하는 데 시간이 오래 걸리면 `go test`의 `-run` 플래그에 아무것도 일치하지 않는 정규식을 줘서 테스트에서 제외시킬 수 있습니다. 

```sh
go test -run=^$	
```
{{< /notice >}}

#### 2.2.2 벤치마크 동작 방식

각 벤치마크 함수는 `b.N`에 다른 값으로 호출됩니다. 이는 벤치마크가 실행해야 하는 반복 횟수입니다.

`b.N`은 1에서 시작합니다. 벤치마크 함수가 1초 미만으로 완료되는 경우(기본값), 그 다음 `b.N`이 증가되고 벤치마크 함수가 다시 실행됩니다.

`b.N`은 대략적으로 1, 2, 3, 5, 10, 20, 30, 50, 100 등의 순서로 증가합니다. 벤치마크 프레임워크는 꾀를 쓰려고 합니다. 작은 `b.N` 값이 비교적 빠르게 완료되는 경우에는 반복 횟수가 더 빠르게 증가하죠.

위의 예를 살펴보면 `BenchmarkFib20-8`은 약 30,000회의 루프 반복이 1초 이상 걸린다는 것을 발견했습니다. 여기에서 벤치마크 프레임워크는 작업당 평균 시간이 40865ns임을 계산했습니다.
	
{{< notice tip >}}
`-8` 접미사는 이 테스트를 실행하는 데 사용된 `GOMAXPROCS` 값과 관련이 있습니다. 이 숫자는 `GOMAXPROCS`와 마찬가지로, 시작시 Go 프로세스에서 보이는 CPU 수로 기본 설정됩니다. 벤치마크를 실행할 값 목록을 가져오는 `-cpu` 플래그를 사용하여 이 값을 변경할 수 있습니다.

```sh
	% go test -bench=. -cpu=1,2,4 ./examples/fib/
	goos: darwin
	goarch: amd64
	BenchmarkFib20             30000             39115 ns/op
	BenchmarkFib20-2           30000             39468 ns/op
	BenchmarkFib20-4           50000             40728 ns/op
	PASS
	ok      _/Users/dfc/devel/high-performance-go-workshop/examples/fib     5.531s
```
	
여기에서는 1, 2, 4개의 코어로 벤치마크를 실행하고 있습니다. 이 경우 이 벤치마크는 완전히 순차적이므로 플래그가 결과에 거의 영향을 미치지 않습니다.
{{< /notice >}}

#### 2.2.3 벤치마크 정확도 개선

`fib` 함수는 약간 억지스런 예제입니다. TechPower 웹서버 벤치마크를 작성하지 않는 한, 피보나치 시퀀스에서 20번째 숫자를 얼마나 빨리 계산할 수 있는지에 대한 비즈니스가 결정되지는 않을 것입니다. 그러나 이 벤치마크는 올바른 벤치마크의 신뢰할만한 예제를 보여줍니다.

특히, 벤치마크를 수만 번 반복하여 실행하면 작업당 좋은 평균값을 얻을 수 있습니다. 벤치마크가 100회 또는 10회 반복으로 실행되는 경우, 해당 실행의 평균은 표준편차가 높을 수 있습니다. 벤치마크가 수백만 또는 수십억 번 반복되는 경우 평균은 매우 정확할 수 있지만, 코드 레이아웃 및 정렬에 차이가 있을 수 있습니다.

반복 횟수를 늘리려면, `-benchtime` 플래그로 벤치마크 시간을 늘릴 수 있습니다. 예를 들면,

```sh
% go test -bench=. -benchtime=10s ./examples/fib/
goos: darwin
goarch: amd64
BenchmarkFib20-8          300000             39318 ns/op
PASS
ok      _/Users/dfc/devel/high-performance-go-workshop/examples/fib     20.066s
```

반환되는 데 10초 이상 걸리는 `b.N` 값에 도달할 때까지 동일한 벤치마크를 실행합니다. 10배 더 길어질수록 총 반복 횟수는 10배 더 큽니다. 결과가 크게 달라지지 않았는데, 이는 우리가 예상한 것입니다.

> 총 시간이 10이 아닌 20초인 이유는 무엇일까요?

마이크로 또는 나노초 범위에서 작업당 시간을 발생시키는 수백만 또는 수십억 회 반복 벤치마크가 있는 경우 열 스케일링, 메모리 로컬성, 백그라운드 처리, gc 활동 등으로 인해 벤치마크 수가 불안정할 수 있습니다.


작업당 시간이 두 자리 또는 한 자리 수의 나노초 단위로 측정되는 경우, 명령어 순서 변경 및 코드 정렬의 상대적인 효과가 벤치마크 시간에 영향을 미칩니다.

이를 해결하려면 `-count` 플래그로 벤치마크를 여러 번 실행합니다.

```sh
% go test -bench=Fib1 -count=10 ./examples/fib/
goos: darwin
goarch: amd64
BenchmarkFib1-8         2000000000               1.99 ns/op
BenchmarkFib1-8         1000000000               1.95 ns/op
BenchmarkFib1-8         2000000000               1.99 ns/op
BenchmarkFib1-8         2000000000               1.97 ns/op
BenchmarkFib1-8         2000000000               1.99 ns/op
BenchmarkFib1-8         2000000000               1.96 ns/op
BenchmarkFib1-8         2000000000               1.99 ns/op
BenchmarkFib1-8         2000000000               2.01 ns/op
BenchmarkFib1-8         2000000000               1.99 ns/op
BenchmarkFib1-8         1000000000               2.00 ns/op
```

`Fib (1)`의 벤치마크는 +/- 2% 차이로 약 2 나노초가 걸립니다.

Go 1.12의 새로운 기능인 `-benchtime` 플래그는 반복 횟수를 취합니다. 예를 들면, `-benchtime=20x`은 코드를 정확하게 `benchtime` 번 실행합니다.

> 10x, 20x, 50x, 100x 및 300x의 `-benchtime`으로 위의 `fib` 벤치를 실행해보세요. 어떤가요?

{{< notice tip >}}
`go test`가 적용하는 기본값을 특정 패키지에 맞게 조정해야 하는 경우, 벤치마크를 실행하려는 모든 사람이 동일한 설정으로 수행할 수 있도록 `Makefile`에서 해당 설정을 코드화하는 것이 좋습니다.
{{< /notice >}}

### 2.3 benchstat과 벤치마크 비교

이전 섹션에서는 더 많은 데이터를 평균화하기 위해 벤치마크를 두 번 이상 실행할 것을 제안했습니다. 이는 챕터의 시작 부분에서 언급한 전원 관리, 백그라운드 프로세스 및 발열 관리의 영향으로 인한 모든 벤치마크에 유용합니다.

러스 콕스(Russ Cox)가 [benchstat](https://godoc.org/golang.org/x/perf/cmd/benchstat)라고 부르는 도구를 소개하겠습니다.

```sh
% go get golang.org/x/perf/cmd/benchstat
```

`benchstat`는 일련의 벤치마크 실행을 수행하고 얼마나 안정적인지 알려줍니다. 다음은 배터리 전원에 대한 Fib(20)의 예제입니다.

```sh
% go test -bench=Fib20 -count=10 ./examples/fib/ | tee old.txt
goos: darwin
goarch: amd64
BenchmarkFib20-8           50000             38479 ns/op
BenchmarkFib20-8           50000             38303 ns/op
BenchmarkFib20-8           50000             38130 ns/op
BenchmarkFib20-8           50000             38636 ns/op
BenchmarkFib20-8           50000             38784 ns/op
BenchmarkFib20-8           50000             38310 ns/op
BenchmarkFib20-8           50000             38156 ns/op
BenchmarkFib20-8           50000             38291 ns/op
BenchmarkFib20-8           50000             38075 ns/op
BenchmarkFib20-8           50000             38705 ns/op
PASS
ok      _/Users/dfc/devel/high-performance-go-workshop/examples/fib     23.125s
% benchstat old.txt
name     time/op
Fib20-8  38.4µs ± 1%
```

`benchstat`에 따르면 평균은 샘플 전체에서 +/- 1% 오차로 38.8 마이크로 초입니다. 이는 배터리 전원에 아주 좋습니다.

- 첫 번째 실행이 운영 체제에서 전원을 절약하기 위해 CPU를 클럭 다운시켰기 때문에 가장 느립니다.

- 다음 두 번의 실행이 가장 빠릅니다. 왜냐하면 운영 체제가 일시적인 작업 스파이크가 아니라고 판단함으로써, 다시 슬립 상태로 돌아갈 수 있기를 바라며 가능한 한 빨리 작업을 수행할 수 있도록 클럭 속도를 높였기 때문입니다.

- 나머지 실행은 열 생산을 위한 운영 체제와 바이오스 트레이드 전력 소비량입니다.

### 2.3.1 Fib 개선

두 벤치마크 세트 간의 성능 델타를 결정하는 것은 지루하고 오류가 발생하기 쉽습니다. Benchstat가 이를 도와줄 수 있습니다.

{{< notice tip >}}
벤치마크 실행에서 출력을 저장하는 것이 유용하지만, 이를 생성한 바이너리를 저장할 수도 있습니다. 이를 통해 이전 반복 벤치마크를 다시 실행할 수 있습니다. 이렇게 하려면, `-c` 플래그를 사용하여 테스트 바이너리를 저장합니다. 종종 이 이진데이터의 이름을 .test에서 .golden로 바꿉니다.

```sh
% go test -c
% mv fib.test fib.golden
```
{{< /notice >}}

이전 `Fib` 함수는 피보나치 시리즈의 0번째와 1번째 숫자의 값을 하드코딩했습니다. 그 후 코드는 재귀적으로 호출됩니다. 이후에 재귀 비용에 대해 이야기할테지만, 현재로서는, 특히 알고리즘이 기하급수적인 시간을 사용하기 때문에 비용이 든다고 가정하겠습니다.

이를 간단하게 해결할 수 있는 방법은 피보나치 시리즈에서 또다른 수를 하드코딩하여 각 재귀 호출의 깊이를 하나씩 줄이는 것입니다.

```golang
func Fib(n int) int {
	switch n {
	case 0:
		return 0
	case 1:
		return 1
	case 2:
		return 1
	default:
		return Fib(n-1) + Fib(n-2)
	}
}
```

새 버전을 비교하기 위해, 새로운 테스트 바이너리를 컴파일하고 둘 다 벤치마킹하고 `benchstat`를 사용하여 출력을 비교합니다.

```sh
% go test -c
% ./fib.golden -test.bench=. -test.count=10 > old.txt
% ./fib.test -test.bench=. -test.count=10 > new.txt
% benchstat old.txt new.txt
name     old time/op  new time/op  delta
Fib20-8  44.3µs ± 6%  25.6µs ± 2%  -42.31%  (p=0.000 n=10+10)
```

벤치마크를 비교할 때, 확인해야 할 세 가지가 있습니다.

- 이전 샘플과 새 샘플의 ±오차. 1-2% 변화가 양호하고, 3-5%는 괜찮으며, 5% 보다 크면 일부 샘플은 신뢰할 수 없는 것으로 간주됩니다. 한 쪽의 변화가 큰 벤치마크를 비교할 때, 개선이 이루지지 않는 것을 볼 수 있으므로 주의해야 합니다.
- p값. 0.05보다 낮은 p값은 양호하고 0.05보다 크면 벤치마크가 통계적으로 유의하지 않을 수 있음을 의미합니다.
- 누락된 샘플. benchstat는 이전 및 새 샘플 중 유효하다고 생각되는 샘플의 개수를 보고합니다. 때로는 `-count = 10`을 수행하더라도 9개만 보고될 수 있습니다. 10% 이하의 거부율(rejection rate)은 괜찮지만, 10%보다 높으면 설정이 불안정하고 너무 적은 수의 샘플을 비교하고 있는 것일 수 있습니다.

### 2.4 벤치마킹 시작 비용 피하기

때로는 벤치마크를 실행할마다 셋업 비용이 한번씩 발생할 수 있습니다. `b.ResetTimer()`은 셋업에서 발생한 시간을 무시할 때 사용합니다.

```golang
func BenchmarkExpensive(b *testing.B) {
        boringAndExpensiveSetup() // (1)
        b.ResetTimer() 
        for n := 0; n < b.N; n++ {
                // function under test
        }
}
```

(1) - 벤치마크 타이머를 리셋합니다.

루프가 반복될 때마다 값비싼 셋업 로직이 있는 경우, 벤치마크 타이머를 일시 중지하려면 `b.StopTimer()` 및 `b.StartTimer()`를 사용하세요.

```golang
func BenchmarkComplicated(b *testing.B) {
        for n := 0; n < b.N; n++ {
                b.StopTimer() // (1)
                complicatedSetup()
                b.StartTimer() // (2)
                // function under test
        }
}
```

(1) - 벤치마크 타이머를 중지합니다.<br/>
(2) - 타이머를 재개합니다.

### 2.5 벤치마킹 할당

할당 횟수와 크기는 벤치마크 시간과 밀접한 상관 관계가 있습니다. 테스트중인 코드에서 진행된 할당수를 기록하도록 `testing` 프레임워크에 지시할 수 있습니다.

```golang
func BenchmarkRead(b *testing.B) {
        b.ReportAllocs()
        for n := 0; n < b.N; n++ {
                // function under test
        }
}
```

다음은 `bufio` 패키지의 벤치마크를 사용한 예입니다.

```sh
% go test -run=^$ -bench=. bufio
goos: darwin
goarch: amd64
pkg: bufio
BenchmarkReaderCopyOptimal-8            20000000               103 ns/op
BenchmarkReaderCopyUnoptimal-8          10000000               159 ns/op
BenchmarkReaderCopyNoWriteTo-8            500000              3644 ns/op
BenchmarkReaderWriteToOptimal-8          5000000               344 ns/op
BenchmarkWriterCopyOptimal-8            20000000                98.6 ns/op
BenchmarkWriterCopyUnoptimal-8          10000000               131 ns/op
BenchmarkWriterCopyNoReadFrom-8           300000              3955 ns/op
BenchmarkReaderEmpty-8                   2000000               789 ns/op            4224 B/op          3 allocs/op
BenchmarkWriterEmpty-8                   2000000               683 ns/op            4096 B/op          1 allocs/op
BenchmarkWriterFlush-8                  100000000               17.0 ns/op             0 B/op          0 allocs/op
```

{{< notice tip >}}
`go test -benchmem` 플래그를 사용해서 테스트 프레임워크가 모든 벤치마크 실행에 대한 할당 통계를 보고하도록 할 수도 있습니다.

```sh
	% go test -run=^$ -bench=. -benchmem bufio
	goos: darwin
	goarch: amd64
	pkg: bufio
	BenchmarkReaderCopyOptimal-8            20000000                93.5 ns/op            16 B/op          1 allocs/op
	BenchmarkReaderCopyUnoptimal-8          10000000               155 ns/op              32 B/op          2 allocs/op
	BenchmarkReaderCopyNoWriteTo-8            500000              3238 ns/op           32800 B/op          3 allocs/op
	BenchmarkReaderWriteToOptimal-8          5000000               335 ns/op              16 B/op          1 allocs/op
	BenchmarkWriterCopyOptimal-8            20000000                96.7 ns/op            16 B/op          1 allocs/op
	BenchmarkWriterCopyUnoptimal-8          10000000               124 ns/op              32 B/op          2 allocs/op
	BenchmarkWriterCopyNoReadFrom-8           500000              3219 ns/op           32800 B/op          3 allocs/op
	BenchmarkReaderEmpty-8                   2000000               748 ns/op            4224 B/op          3 allocs/op
	BenchmarkWriterEmpty-8                   2000000               662 ns/op            4096 B/op          1 allocs/op
	BenchmarkWriterFlush-8                  100000000               16.9 ns/op             0 B/op          0 allocs/op
	PASS
	ok      bufio   20.366s
```
{{< /notice >}}

### 2.6 컴파일러 최적화 주의

이 예제는 [이슈 14813](https://github.com/golang/go/issues/14813#issue-140603392)에서 가져온 것입니다.

```golang
const m1 = 0x5555555555555555
const m2 = 0x3333333333333333
const m4 = 0x0f0f0f0f0f0f0f0f
const h01 = 0x0101010101010101

func popcnt(x uint64) uint64 {
	x -= (x >> 1) & m1
	x = (x & m2) + ((x >> 2) & m2)
	x = (x + (x >> 4)) & m4
	return (x * h01) >> 56
}

func BenchmarkPopcnt(b *testing.B) {
	for i := 0; i < b.N; i++ {
		popcnt(uint64(i))
	}
}
```


이 함수가 얼마나 빨리 벤치마킹할 것이라고 생각하시나요? 알아봅시다.

```golang
% go test -bench=. ./examples/popcnt/
goos: darwin
goarch: amd64
BenchmarkPopcnt-8       2000000000               0.30 ns/op
PASS
```

0.3 나노초입니다. 이는 기본적으로 한 클럭 사이클입니다. CPU에 클럭 틱당 몇 개의 인플라이트 (in flight) 명령어가 있다고 가정하더라도 이 숫자는 터무니없이 낮은 것으로 보입니다. 어떻게 된 걸까요?

무슨 일이 일어났는지 이해하기 위해서, 벤치마크 내의 `popcnt` 함수를 살펴봐야 합니다. `popcnt`는 리프 함수 (다른 함수를 호출하지 않음) 이므로 컴파일러는 이를 인라인화할 수 있습니다.

함수가 인라인화되어 있으므로 이제 컴파일러에서 사이드 이펙트가 없음을 알 수 있습니다. `popcnt`는 전역 변수의 상태에 영향을 주지 않습니다. 따라서 호출을 없앱니다. 컴파일러가 보는 것은 다음과 같습니다.

```golang
func BenchmarkPopcnt(b *testing.B) {
	for i := 0; i < b.N; i++ {
		// 최적화되어 사라짐
	}
}
```

테스트해 본 모든 버전의 Go 컴파일러에서 루프가 계속 생성됩니다. 그러나 Intel CPU는 루프, 특히 빈 루프를 최적화하는 데 정말 좋습니다.

#### 2.6.1 연습, 어셈블리 보기

계속하기 전에, 우리가 본 것을 확인하기 위해 어셈블리를 살펴봅시다.

```sh
% go test -gcflags=-S
```
	
> `gcflags = -l -S`를 사용해서 인라인을 비활성화합니다. 어셈블리 출력에 어떻게 영향을 미칠까요?

{{< notice note >}}
**최적화는 좋은 것입니다**
실제 코드를 빠르게 만드는 똑같은 최적화를 제거해야 합니다. 이는 불필요한 계산을 제거함으로써, 관찰할 수 없는 사이드 이펙트가 있는 벤치마크를 제거하는 것과 동일합니다. 
 
이는 Go 컴파일러가 개선됨에 따라 더 흔해지겠죠.
{{< /notice >}}

#### 2.6.2 벤치마크 고치기

벤치마크 동작을 위해 인라인을 비활성화하는 것은 비현실적입니다. 우리는 최적화를 바탕으로 코드를 작성하려고합니다.

이 벤치마크를 수정하려면 컴파일러에서 `BenchmarkPopcnt`의 바디가 전역 상태를 변경하지 않음을 증명할 수 없도록 해야합니다.

```golang
var Result uint64

func BenchmarkPopcnt(b *testing.B) {
	var r uint64
	for i := 0; i < b.N; i++ {
		r = popcnt(uint64(i))
	}
	Result = r
}
```

이처럼 컴파일러가 루프 바디를 최적화하지 않도록 권장합니다.

첫째, `popcnt`를 호출한 결과는 `r`에 저장하는데 사용합니다. 둘째, 벤치마크가 끝나면 `r`이 `BenchmarkPopcnt` 범위 내에서 로컬로 선언되었으므로, `r`의 결과는 프로그램의 다른 부분에 보여지지 않으며, 최종 조치로 `r`의 값을 패키지 퍼블릭 변수 `Result`에 할당합니다.

`Result`는 퍼블릭이기 때문에 컴파일러는 이 패키지를 임포트하는 다른 패키지가 시간이 지남에 따라 변경되는 `Result` 값을 볼 수 없다는 것을 증명할 수 없으므로, 할당으로 이어지는 모든 작업을 최적화 할 수 없습니다.

`Result`에 직접 할당하면 어떻게 될까요? 이것이 벤치마크 시간에 영향을 줄까요? `popcnt`의 결과를 `_`에 할당하면 어떨까요?

{{< notice warning >}}
이전 `Fib` 벤치마크에서 이러한 예방 조치를 취하지 않았습니다. 그렇게 했어야 했을까요?
{{< /notice >}}

### 2.7 벤치마크 실수들

`for` 루프는 벤치마크 동작에 중요합니다.

다음은 두 가지 잘못된 벤치마크입니다. 무엇이 잘못되었는지 설명할 수 있을까요?

```golang
func BenchmarkFibWrong(b *testing.B) {
	Fib(b.N)
}
```

```golang
func BenchmarkFibWrong2(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Fib(n)
	}
}
```

이 벤치마크를 실행해보세요, 어떤가요?

### 2.8 프로파일링 벤치마크

`testing` 패키지는 CPU, 메모리 및 블록 프로파일 생성을 지원합니다.

- `-cpuprofile=$FILE`은 CPU 프로파일을 `$FILE`에 씁니다.

- `-memprofile=$FILE`, 메모리 프로파일을 `$FILE`에 쓰고, `-memprofilerate=N`은 프로파일 속도를 1 / N으로 조정합니다.

- `-blockprofile=$FILE`, `$FILE`에 블록 프로파일을 씁니다.

이 플래그 중 하나를 사용하면 바이너리도 유지됩니다.

```sh
% go test -run=XXX -bench=. -cpuprofile=c.p bytes
% go tool pprof c.p
```

### 2.9 토론

질문 있으세요?

휴식 시간이겠군요.

## 3. 성능 측정 및 프로파일링

이전 섹션에서는 병목 현상이 발생한 위치를 미리 알고 있을 때, 유용한 개별 함수를 벤치마킹하는 방법에 대해 살펴 보았습니다. 하지만, 종종 여러분은 질문하는 입장이 될 것입니다.

> 이 프로그램은 왜 이렇게 오래 걸리는 거죠?

이처럼 고수준의 질문에 답을 하려면 전체 프로그램을 프로파일링하는 것이 유용합니다. 이번 섹션에서는 Go에 내장된 프로파일링 도구를 사용하여 프로그램내부 동작을 조사할 것입니다.

### 3.1 pprof

오늘 우리가 다룰 첫 번째 도구는 pprof입니다. [pprof](https://github.com/google/pprof)는 [Google Perf Tools](https://github.com/gperftools/gperftools) 도구 모음에서 유래했으며, 최초 공개 버전 이후로 Go 런타임에 통합되었습니다.

`pprof`는 두 부분으로 구성됩니다.

- `runtime/pprof` 패키지는 모든 Go 프로그램에 내장되어 있습니다.
- `go tool pprof`는 프로파일을 조사합니다.

### 3.2 프로파일링 타입

`pprof`는 여러 타입의 프로파일링을 지원합니다. 오늘은 이들 중 세 가지에 대해 설명하겠습니다.

- CPU 프로파일링.
- 메모리 프로파일링.
- 블락 (또는 블락킹) 프로파일링.
- 뮤텍스 경합 프로파일링.

#### 3.2.1 CPU 프로파일링

CPU 프로파일링은 가장 일반적인 유형의 프로파일이며 가장 명확합니다.

CPU 프로파일링이 활성화되면, 런타임은 10ms마다 자체적으로 중단되고 현재 실행중인 고루틴의 스택 트래이싱을 기록합니다.

프로파일이 완료되면 가장 핫한 코드 경로를 결정하기 위해 프로파일을 분석할 수 있습니다.

함수가 프로파일에 더 많이 나타날수록, 코드 경로가 총 런타임의 백분율로 더 많은 시간이 소요됩니다.

#### 3.2.2 메모리 프로파일링

메모리 프로파일링은 힙 할당이 이루어질 때 스택 트레이싱을 기록합니다.

스택 할당은 사용 가능(free)한 것으로 가정되며 메모리 프로파일에서 추적되지 않습니다.

CPU 프로파일링과 같은 메모리 프로파일링은 기본적으로 1000개 할당마다 메모리 프로파일링 샘플 1을 기본으로 합니다. 이 비율은 변경할 수 있습니다.

메모리 프로파일링은 샘플 기반이며 사용하지 않는 할당을 추적하기 때문에 메모리 프로파일링을 사용하여 애플리케이션의 전체 메모리 사용량을 결정하는 것은 어렵습니다.
                                                                                                                                                                                                                                                                                                                                                   *개인적인 의견*: 메모리 누수를 찾는 데 유용한 메모리 프로파일링을 찾지 못했습니다. 애플리케이션에서 사용중인 메모리 양을 확인하는 더 좋은 방법이 있습니다. 나중 프레젠테이션에서 이에 대해 논의할 것입니다.

#### 3.2.3 블록 프로파일링

블록 프로파일링은 Go만의 고유한 특징입니다.

블록 프로파일은 CPU 프로파일과 유사하지만, 고루틴이 공유 자원을 기다리는 데 소요된 시간을 기록합니다.

이는 애플리케이션의 동시성 병목 현상을 파악하는 데 유용할 수 있습니다.

블록 프로파일링은 많은 수의 고루틴이 진행할 수 있지만, 블록된 때를 보여줄 수 있습니다. 블럭킹에는 다음이 포함됩니다.

- 언버퍼드 채널에서 전송 또는 수신합니다.
- 풀(full) 채널로 전송하고 빈(empty) 채널에서 수신합니다.
- 다른 고루틴이 잠군 `sync.Mutex`를 `Lock`하려고 합니다.

블록 프로파일링은 매우 전문화된 도구이므로, 모든 CPU 및 메모리 사용 병목 현상을 제거했다고 확신하기 전에는 사용해서는 안 됩니다.

#### 3.2.4 뮤텍스 프로파일링

뮤텍스 프로파일링은 블록 프로파일링과 유사하지만 뮤텍스 경합으로 인한 지연을 유발하는 작업에만 집중합니다.

이 타입의 프로필에 대한 경험이 많지 않지만, 이를 보여주는 예제를 작성했습니다. 곧 그 예제를 살펴 보겠습니다.

### 3.3 한 번에 한 프로파일

프로파일링은 공짜가 아닙니다.

프로파일링은 프로그램 성능에 괜찮지만, 프로그램에 측정 가능한 영향을 미칩니다. 특히, 메모리 프로파일 샘플 속도를 증가시키는 경우에 그러합니다.

대부분의 도구는 한 번에 여러 프로파일링을 활성화하지 못하게 합니다.

{{< notice warning >}}
한 번에 둘 이상의 프로파일을 활성화하지 마세요. 
 
여러 프로파일을 동시에 활성화하면, 서로의 상호 작용을 보게 되고, 그 결과를 버리게 됩니다.
{{< /notice >}}

### 3.4 프로파일 수집

Go 런타임의 프로파일링 인터페이스는 `runtime/pprof` 패키지에 있습니다. `runtime/pprof`는 매우 저수준의 도구이며, 역사적 이유로 여러 유형의 프로파일링에 대한 인터페이스가 통일되어 있지 않습니다.

이전 섹션에서 보았듯이, `pprof` 프로파일링은 `testing` 패키지에 내장되어 있지만, `testing.B` 벤치마크의 컨텍스트 내에 프로파일링하려는 코드를 배치하는 것이 불편하거나 어려운 경우가 있습니다. 그래서 `runtime/pprof` API를 직접 사용해야 합니다.

몇 년 전에 저는 기존 애플리케이션을 보다 쉽게 프로파일링할 수 있도록 [작은 패키지][0]를 작성했습니다.

```golang
import "github.com/pkg/profile"

func main() {
	defer profile.Start().Stop()
	// ...
}
```

이 섹션에서는 프로파일링 패키지를 사용합니다. 나중에 `runtime/pprof` 인터페이스를 직접 사용하는 것에 대해 이야기할 것입니다.

### 3.5 pprof 프로파일 분석

pprof가 측정할 수 있는 것과 프로파일을 생성하는 방법에 대해 말씀드렸으니 pprof를 사용하여 프로파일을 분석하는 방법에 대해 살펴 보겠습니다.

분석은 `go pprof` 하위 명령으로 수행합니다.

```sh
go tool pprof /path/to/your/profile
```

이 도구는 텍스트, 그래픽, 심지어 프레임 그래프 등 프로파일링 데이터를 여러가지 다르게 표현할 수 있습니다.

{{< notice tip >}}
Go를 한동안 사용했다면 `pprof`에 두 가지 아규먼트가 있다고 들었을 것입니다. Go 1.9부터 프로파일 파일에는 프로파일을 렌더링하는 데 필요한 모든 정보가 포함되어 있습니다. 더 이상 프로파일을 생성한 바이너리가 필요하지 않습니다. 🎉
{{< /notice >}}

#### 3.5.1 더 읽을거리

- [Profiling Go programs](http://blog.golang.org/profiling-go-programs) (Go Blog)
- [Debugging performance issues in Go programs](https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs)

#### 3.5.2 CPU 프로파일링 (연습문제)

단어 개수를 세는 프로그램을 작성해봅시다.

```golang
package main

import (
	"fmt"
	"io"
	"log"
	"os"
	"unicode"

	"github.com/pkg/profile"
)

func readbyte(r io.Reader) (rune, error) {
	var buf [1]byte
	_, err := r.Read(buf[:])
	return rune(buf[0]), err
}

func main() {
	defer profile.Start().Stop()

	f, err := os.Open(os.Args[1])
	if err != nil {
		log.Fatalf("could not open file %q: %v", os.Args[1], err)
	}

	words := 0
	inword := false
	for {
		r, err := readbyte(f)
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Fatalf("could not read file %q: %v", os.Args[1], err)
		}
		if unicode.IsSpace(r) && inword {
			words++
			inword = false
		}
		inword = unicode.IsLetter(r)
	}
	fmt.Printf("%q: %d words\n", os.Args[1], words)
}
```

허먼 멜빌(Herman Melville)의 [고전 모비딕](https://www.gutenberg.org/ebooks/2701)(구텐베르크 프로젝트에서 발췌)에 몇 개의 단어가 있는지 살펴보겠습니다.

```golang
% go build && time ./words moby.txt
"moby.txt": 181275 words

real    0m2.110s
user    0m1.264s
sys     0m0.944s
```

`wc -w`의 유닉스 명령과 비교합니다.

```golang
% time wc -w moby.txt
215829 moby.txt

real    0m0.012s
user    0m0.009s
sys     0m0.002s
```

그러니까 숫자가 같지 않네요. wc는 나의 간단한 프로그램과 하는 것처럼 단어를 다르게 생각하기 때문에 약 19% 더 높습니다. 이는 중요하지 않습니다. 두 프로그램 모두 전체 파일을 입력으로 사용하고 단일 패스 카운트에서 한 단어에서 비단어로의 전환 횟수를 계산합니다.

`pprof`를 사용하여 이러한 프로그램의 실행 시간이 다른 이유를 살펴 보겠습니다.

#### 3.5.3 CPU 프로파일링 추가

먼저, `main.go` 파일을 수정하여 프로파일링을 활성화 합니다.

```golang
import (
        "github.com/pkg/profile"
)

func main() {
        defer profile.Start().Stop()
        // ...
```

이제 프로그램을 실행하면 `cpu.pprof` 파일이 생성됩니다.

```sh
% go run main.go moby.txt
2018/08/25 14:09:01 profile: cpu profiling enabled, /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile239941020/cpu.pprof
"moby.txt": 181275 words
2018/08/25 14:09:03 profile: cpu profiling disabled, /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile239941020/cpu.pprof
```

이제 `go tool pprof`를 사용하여 분석할 수 있는 프로파일이 있죠.

```sh
% go tool pprof /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile239941020/cpu.pprof
Type: cpu
Time: Aug 25, 2018 at 2:09pm (AEST)
Duration: 2.05s, Total samples = 1.36s (66.29%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 1.42s, 100% of 1.42s total
      flat  flat%   sum%        cum   cum%
     1.41s 99.30% 99.30%      1.41s 99.30%  syscall.Syscall
     0.01s   0.7%   100%      1.42s   100%  main.readbyte
         0     0%   100%      1.41s 99.30%  internal/poll.(*FD).Read
         0     0%   100%      1.42s   100%  main.main
         0     0%   100%      1.41s 99.30%  os.(*File).Read
         0     0%   100%      1.41s 99.30%  os.(*File).read
         0     0%   100%      1.42s   100%  runtime.main
         0     0%   100%      1.41s 99.30%  syscall.Read
         0     0%   100%      1.41s 99.30%  syscall.read
```

`top` 커맨드는 가장 많이 사용하는 커맨드입니다. 이 프로그램은 99%의 시간을 `syscall.Syscall`에, 그리고 미미한 부분을 `main.readbyte`에 쓰고 있음을 알 수 있습니다.

`web` 명령으로 이 호출을 시각화할 수도 있습니다. 프로파일 데이터에서 방향 그래프를 생성합니다. 내부적으로는 Graphviz의 `dot`명령을 사용합니다.

그러나 Go 1.10 (아마도 1.11)에서 Go는 네이티브하게 http 서버를 지원하는 `pprof` 버전을 제공합니다.

```sh
% go tool pprof -http=:8080 /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile239941020/cpu.pprof
```
	
웹 브라우저를 엽니다.

- 그래프 모드
- 플레임 그래프 (Flame Graph) 모드

그래프에서는 가장 많은 CPU 시간을 소비하는 박스가 가장 큽니다. `sys call.SysCall`은 프로그램이 소요한 총 시간의 99.3%입니다. `syscall.Syscall`로 이어지는 일련의 박스는 순간적인 호출자를 나타냅니다. 여러 개의 코드 경로가 동일한 함수에 수렴하는 경우 둘 이상이 있을 수 있습니다. 화살표의 크기는 박스의 자식이 소요한 시간을 나타냅니다. `main.readByte` 이후 부터는 이 그래프의 암에 사용된 1.41초 중 거의 0을 차지한다는 것을 알 수 있습니다.

질문: 왜 우리 버전이 wc보다 훨씬 느린지 짐작할 수 있을까요?

#### 3.5.4 현재 버전 개선하기

프로그램이 느린 이유는 Go의 `syscall.Syscall`이 느리기 때문이 아닙니다. 일반적으로 syscall은 비싼 작업이기 때문입니다. (그리고 Spectre 계열의 취약점이 더 많이 발견될수록 비용이 더 많이 듭니다).

`readbyte`를 호출할 때마다 버퍼 크기가 1인 syscall.Read가 발생합니다. 따라서 프로그램에서 실행되는 syscall 수는 입력의 크기와 같습니다. `pprof` 그래프에서 입력을 읽는 것이 다른 모든 것을 지배한다는 것을 알 수 있습니다.

```golang
func main() {
	defer profile.Start(profile.MemProfile, profile.MemProfileRate(1)).Stop()
	// defer profile.Start(profile.MemProfile).Stop()

	f, err := os.Open(os.Args[1])
	if err != nil {
		log.Fatalf("could not open file %q: %v", os.Args[1], err)
	}

	b := bufio.NewReader(f)
	words := 0
	inword := false
	for {
		r, err := readbyte(b)
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Fatalf("could not read file %q: %v", os.Args[1], err)
		}
		if unicode.IsSpace(r) && inword {
			words++
			inword = false
		}
		inword = unicode.IsLetter(r)
	}
	fmt.Printf("%q: %d words\n", os.Args[1], words)
}
```

입력 파일과 `readbyte` 사이에 `bufio.Reader`를 삽입하고

> 이 수정된 프로그램의 시간을 `wc`와 비교해보세요. 얼마나 가까워졌나요? 프로필을 보고 뭐가 또 있는지 살펴보세요.

#### 3.5.5 메모리 프로파일링

새로운 `words` 프로파일은 `readbyte` 함수 안에 무언가가 할당되어 있음을 나타냅니다. `pprof`를 사용하여 조사할 수 있습니다.

```golang
defer profile.Start(profile.MemProfile).Stop()
```
	

다음 평소와 같이 프로그램을 수행하면,

```sh
% go run main2.go moby.txt
2018/08/25 14:41:15 profile: memory profiling enabled (rate 4096), /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile312088211/mem.pprof
"moby.txt": 181275 words
2018/08/25 14:41:15 profile: memory profiling disabled, /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile312088211/mem.pprof
```

<div><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%px" height="100%px"><script type="text/ecmascript">
/**
 *  SVGPan library 1.2.2
 * ======================
 *
 * Given an unique existing element with id "viewport" (or when missing, the
 * first g-element), including the library into any SVG adds the following
 * capabilities:
 *
 *  - Mouse panning
 *  - Mouse zooming (using the wheel)
 *  - Object dragging
 *
 * You can configure the behaviour of the pan/zoom/drag with the variables
 * listed in the CONFIGURATION section of this file.
 *
 * Known issues:
 *
 *  - Zooming (while panning) on Safari has still some issues
 *
 * Releases:
 *
 * 1.2.2, Tue Aug 30 17:21:56 CEST 2011, Andrea Leofreddi
 *	- Fixed viewBox on root tag (#7)
 *	- Improved zoom speed (#2)
 *
 * 1.2.1, Mon Jul  4 00:33:18 CEST 2011, Andrea Leofreddi
 *	- Fixed a regression with mouse wheel (now working on Firefox 5)
 *	- Working with viewBox attribute (#4)
 *	- Added "use strict;" and fixed resulting warnings (#5)
 *	- Added configuration variables, dragging is disabled by default (#3)
 *
 * 1.2, Sat Mar 20 08:42:50 GMT 2010, Zeng Xiaohui
 *	Fixed a bug with browser mouse handler interaction
 *
 * 1.1, Wed Feb  3 17:39:33 GMT 2010, Zeng Xiaohui
 *	Updated the zoom code to support the mouse wheel on Safari/Chrome
 *
 * 1.0, Andrea Leofreddi
 *	First release
 *
 * This code is licensed under the following BSD license:
 *
 * Copyright 2009-2017 Andrea Leofreddi &lt;a.leofreddi@vleo.net&gt;. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification, are
 * permitted provided that the following conditions are met:
 *
 *    1. Redistributions of source code must retain the above copyright
 *       notice, this list of conditions and the following disclaimer.
 *    2. Redistributions in binary form must reproduce the above copyright
 *       notice, this list of conditions and the following disclaimer in the
 *       documentation and/or other materials provided with the distribution.
 *    3. Neither the name of the copyright holder nor the names of its
 *       contributors may be used to endorse or promote products derived from
 *       this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY COPYRIGHT HOLDERS AND CONTRIBUTORS ''AS IS'' AND ANY EXPRESS
 * OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
 * AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL COPYRIGHT HOLDERS OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 * The views and conclusions contained in the software and documentation are those of the
 * authors and should not be interpreted as representing official policies, either expressed
 * or implied, of Andrea Leofreddi.
 */

"use strict";

/// CONFIGURATION
/// ====&gt;

var enablePan = 1; // 1 or 0: enable or disable panning (default enabled)
var enableZoom = 1; // 1 or 0: enable or disable zooming (default enabled)
var enableDrag = 0; // 1 or 0: enable or disable dragging (default disabled)
var zoomScale = 0.2; // Zoom sensitivity

/// &lt;====
/// END OF CONFIGURATION

var root = document.documentElement;

var state = 'none', svgRoot = null, stateTarget, stateOrigin, stateTf;

setupHandlers(root);

/**
 * Register handlers
 */
function setupHandlers(root){
	setAttributes(root, {
		"onmouseup" : "handleMouseUp(evt)",
		"onmousedown" : "handleMouseDown(evt)",
		"onmousemove" : "handleMouseMove(evt)",
		//"onmouseout" : "handleMouseUp(evt)", // Decomment this to stop the pan functionality when dragging out of the SVG element
	});

	if(navigator.userAgent.toLowerCase().indexOf('webkit') &gt;= 0)
		window.addEventListener('mousewheel', handleMouseWheel, false); // Chrome/Safari
	else
		window.addEventListener('DOMMouseScroll', handleMouseWheel, false); // Others
}

/**
 * Retrieves the root element for SVG manipulation. The element is then cached into the svgRoot global variable.
 */
function getRoot(root) {
	if(svgRoot == null) {
		var r = root.getElementById("viewport") ? root.getElementById("viewport") : root.documentElement, t = r;

		while(t != root) {
			if(t.getAttribute("viewBox")) {
				setCTM(r, t.getCTM());

				t.removeAttribute("viewBox");
			}

			t = t.parentNode;
		}

		svgRoot = r;
	}

	return svgRoot;
}

/**
 * Instance an SVGPoint object with given event coordinates.
 */
function getEventPoint(evt) {
	var p = root.createSVGPoint();

	p.x = evt.clientX;
	p.y = evt.clientY;

	return p;
}

/**
 * Sets the current transform matrix of an element.
 */
function setCTM(element, matrix) {
	var s = "matrix(" + matrix.a + "," + matrix.b + "," + matrix.c + "," + matrix.d + "," + matrix.e + "," + matrix.f + ")";

	element.setAttribute("transform", s);
}

/**
 * Dumps a matrix to a string (useful for debug).
 */
function dumpMatrix(matrix) {
	var s = "[ " + matrix.a + ", " + matrix.c + ", " + matrix.e + "\n  " + matrix.b + ", " + matrix.d + ", " + matrix.f + "\n  0, 0, 1 ]";

	return s;
}

/**
 * Sets attributes of an element.
 */
function setAttributes(element, attributes){
	for (var i in attributes)
		element.setAttributeNS(null, i, attributes[i]);
}

/**
 * Handle mouse wheel event.
 */
function handleMouseWheel(evt) {
	if(!enableZoom)
		return;

	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var delta;

	if(evt.wheelDelta)
		delta = evt.wheelDelta / 360; // Chrome/Safari
	else
		delta = evt.detail / -9; // Mozilla

	var z = Math.pow(1 + zoomScale, delta);

	var g = getRoot(svgDoc);

	var p = getEventPoint(evt);

	p = p.matrixTransform(g.getCTM().inverse());

	// Compute new scale matrix in current mouse position
	var k = root.createSVGMatrix().translate(p.x, p.y).scale(z).translate(-p.x, -p.y);

        setCTM(g, g.getCTM().multiply(k));

	if(typeof(stateTf) == "undefined")
		stateTf = g.getCTM().inverse();

	stateTf = stateTf.multiply(k.inverse());
}

/**
 * Handle mouse move event.
 */
function handleMouseMove(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(state == 'pan' &amp;&amp; enablePan) {
		// Pan mode
		var p = getEventPoint(evt).matrixTransform(stateTf);

		setCTM(g, stateTf.inverse().translate(p.x - stateOrigin.x, p.y - stateOrigin.y));
	} else if(state == 'drag' &amp;&amp; enableDrag) {
		// Drag mode
		var p = getEventPoint(evt).matrixTransform(g.getCTM().inverse());

		setCTM(stateTarget, root.createSVGMatrix().translate(p.x - stateOrigin.x, p.y - stateOrigin.y).multiply(g.getCTM().inverse()).multiply(stateTarget.getCTM()));

		stateOrigin = p;
	}
}

/**
 * Handle click event.
 */
function handleMouseDown(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(
		evt.target.tagName == "svg"
		|| !enableDrag // Pan anyway when drag is disabled and the user clicked on an element
	) {
		// Pan mode
		state = 'pan';

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	} else {
		// Drag mode
		state = 'drag';

		stateTarget = evt.target;

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	}
}

/**
 * Handle mouse button release event.
 */
function handleMouseUp(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	if(state == 'pan' || state == 'drag') {
		// Quit pan mode
		state = '';
	}
}
</script><g id="viewport" transform="scale(0.5,0.5) translate(0,0)"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 393)">
<title>unnamed</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-393 616,-393 616,4 -4,4"></polygon>
<g id="clust1" class="cluster">
<title>cluster_L</title>
<polygon fill="none" stroke="#000000" points="8,-303 8,-381 464,-381 464,-303 8,-303"></polygon>
</g>
<!-- Type: inuse_space -->
<g id="node1" class="node">
<title>Type: inuse_space</title>
<polygon fill="#f8f8f8" stroke="#000000" points="456,-373 16,-373 16,-311 456,-311 456,-373"></polygon>
<text text-anchor="start" x="24" y="-356.2" font-family="Times,serif" font-size="16.00" fill="#000000">Type: inuse_space</text>
<text text-anchor="start" x="24" y="-338.2" font-family="Times,serif" font-size="16.00" fill="#000000">Time: Mar 23, 2019 at 6:14pm (CET)</text>
<text text-anchor="start" x="24" y="-320.2" font-family="Times,serif" font-size="16.00" fill="#000000">Showing nodes accounting for 368.72kB, 100% of 368.72kB total</text>
</g>
<!-- N1 -->
<g id="node1" class="node">
<title>N1</title>
<g id="a_node1"><a xlink:title="main.readbyte (368.72kB)">
<polygon fill="#edd5d5" stroke="#b20000" points="612,-173 424,-173 424,-87 612,-87 612,-173"></polygon>
<text text-anchor="middle" x="518" y="-149.8" font-family="Times,serif" font-size="24.00" fill="#000000">main</text>
<text text-anchor="middle" x="518" y="-123.8" font-family="Times,serif" font-size="24.00" fill="#000000">readbyte</text>
<text text-anchor="middle" x="518" y="-97.8" font-family="Times,serif" font-size="24.00" fill="#000000">368.72kB (100%)</text>
</a>
</g>
</g>
<!-- NN1_0 -->
<g id="NN1_0" class="node">
<title>NN1_0</title>
<g id="a_NN1_0"><a xlink:title="368.72kB">
<polygon fill="#f8f8f8" stroke="#000000" points="545,-36 495,-36 491,-32 491,0 541,0 545,-4 545,-36"></polygon>
<polyline fill="none" stroke="#000000" points="541,-32 491,-32 "></polyline>
<polyline fill="none" stroke="#000000" points="541,-32 541,0 "></polyline>
<polyline fill="none" stroke="#000000" points="541,-32 545,-36 "></polyline>
<text text-anchor="middle" x="518" y="-16.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N1&#45;&gt;NN1_0 -->
<g id="edge1" class="edge">
<title>N1-&gt;NN1_0</title>
<g id="a_edge1"><a xlink:title="368.72kB">
<path fill="none" stroke="#000000" d="M518,-86.6979C518,-73.0814 518,-58.4077 518,-46.1319"></path>
<polygon fill="#000000" stroke="#000000" points="521.5001,-46.0482 518,-36.0482 514.5001,-46.0482 521.5001,-46.0482"></polygon>
</a>
</g>
<g id="a_edge1-label"><a xlink:title="368.72kB">
<text text-anchor="middle" x="547.5" y="-57.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 368.72kB</text>
</a>
</g>
</g>
<!-- N2 -->
<g id="node2" class="node">
<title>N2</title>
<g id="a_node2"><a xlink:title="runtime.main (368.72kB)">
<polygon fill="#edd5d5" stroke="#b20000" points="562,-360 474,-360 474,-324 562,-324 562,-360"></polygon>
<text text-anchor="middle" x="518" y="-349.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="518" y="-340.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="518" y="-331.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 368.72kB (100%)</text>
</a>
</g>
</g>
<!-- N3 -->
<g id="node3" class="node">
<title>N3</title>
<g id="a_node3"><a xlink:title="main.main (368.72kB)">
<polygon fill="#edd5d5" stroke="#b20000" points="562,-260 474,-260 474,-224 562,-224 562,-260"></polygon>
<text text-anchor="middle" x="518" y="-249.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="518" y="-240.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="518" y="-231.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 368.72kB (100%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N3 -->
<g id="edge3" class="edge">
<title>N2-&gt;N3</title>
<g id="a_edge3"><a xlink:title="runtime.main -> main.main (368.72kB)">
<path fill="none" stroke="#b20000" stroke-width="6" d="M518,-323.6585C518,-308.7164 518,-287.3665 518,-270.2446"></path>
<polygon fill="#b20000" stroke="#b20000" stroke-width="6" points="523.2501,-270.2252 518,-260.2253 512.7501,-270.2253 523.2501,-270.2252"></polygon>
</a>
</g>
<g id="a_edge3-label"><a xlink:title="runtime.main -> main.main (368.72kB)">
<text text-anchor="middle" x="547.5" y="-281.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 368.72kB</text>
</a>
</g>
</g>
<!-- N3&#45;&gt;N1 -->
<g id="edge2" class="edge">
<title>N3-&gt;N1</title>
<g id="a_edge2"><a xlink:title="main.main -> main.readbyte (368.72kB)">
<path fill="none" stroke="#b20000" stroke-width="6" d="M518,-223.5055C518,-212.5063 518,-197.9346 518,-183.5757"></path>
<polygon fill="#b20000" stroke="#b20000" stroke-width="6" points="523.2501,-183.2119 518,-173.212 512.7501,-183.212 523.2501,-183.2119"></polygon>
</a>
</g>
<g id="a_edge2-label"><a xlink:title="main.main -> main.readbyte (368.72kB)">
<text text-anchor="middle" x="547.5" y="-194.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 368.72kB</text>
</a>
</g>
</g>
</g>
</g></svg>
</div>

할당이 `readbyte`에서 비롯된 것으로 의심했던 것과 같네요. 이는 그렇게 복잡하지 않았죠, readbyte는 3줄짜리라서요.

> pprof를 사용해서 할당 위치를 판별해보세요.

```golang
func readbyte(r io.Reader) (rune, error) {
        var buf [1]byte // (1)
        _, err := r.Read(buf[:])
        return rune(buf[0]), err
}
```
(1) 여기서 할당이 일어납니다.

다음 섹션에서 왜 이런 일이 발생하는지 자세히 설명하겠지만, 현재는 보여지는 것은 `readbyte`에 대한 모든 호출이 새 1바이트 길이의 배열을 할당하고 해당 배열이 힙에 할당되고 있다는 것입니다.

> 이것을 피할 수 있는 방법은 무엇일까요? 그 방법들을 CPU와 메모리 프로파일링을 사용해서 입증해봅시다.

**alloc 오브젝트와 inuse 오브젝트**
메모리 프로파일은 두 가지로 구분되며, `go tool pprof` 플래그의 이름을 따라 명명되었습니다.

- `-alloc_objects`는 각 할당이 이루어진 호출 사이트를 리포팅합니다.
- `-inuse_objects`는 프로파일 끝까지 도달하게 되는 경우, 할당이 이루어진 호출 사이트를 리포팅합니다.

다음은 이를 증명해보기 위해, 다수의 메모리를 제어된 방식으로 할당하도록 고안된 프로그램입니다.

```golang
const count = 100000

var y []byte

func main() {
	defer profile.Start(profile.MemProfile, profile.MemProfileRate(1)).Stop()
	y = allocate()
	runtime.GC()
}

// allocate allocates count byte slices and returns the first slice allocated.
func allocate() []byte {
	var x [][]byte
	for i := 0; i < count; i++ {
		x = append(x, makeByteSlice())
	}
	return x[0]
}

// makeByteSlice returns a byte slice of a random length in the range [0, 16384).
func makeByteSlice() []byte {
	return make([]byte, rand.Intn(2^14))
}
```

프로그램은 `profile` 패키지의 어노테이션을 달고, 메모리 프로파일 속도를 1로 설정합니다. 즉, 모든 할당에 대해 스택 트레이싱을 기록합니다. 이로 인해 프로그램 속도가 많이 느려지지만, 잠시 후 이유를 알게 됩니다.

```sh
% go run main.go
2018/08/25 15:22:05 profile: memory profiling enabled (rate 1), /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile730812803/mem.pprof
2018/08/25 15:22:05 profile: memory profiling disabled, /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile730812803/mem.pprof
```

할당된 객체의 그래프를 살펴 보겠습니다. 이 그래프는 기본값이며 프로파일 동안 모든 객체의 할당으로 이어지는 호출 그래프를 보여줍니다.

```sh
% go tool pprof -http=:8080 /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile891268605/mem.pprof
```

<div>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%px" height="100%px">
<script type="text/ecmascript">
/**
 *  SVGPan library 1.2.2
 * ======================
 *
 * Given an unique existing element with id "viewport" (or when missing, the
 * first g-element), including the library into any SVG adds the following
 * capabilities:
 *
 *  - Mouse panning
 *  - Mouse zooming (using the wheel)
 *  - Object dragging
 *
 * You can configure the behaviour of the pan/zoom/drag with the variables
 * listed in the CONFIGURATION section of this file.
 *
 * Known issues:
 *
 *  - Zooming (while panning) on Safari has still some issues
 *
 * Releases:
 *
 * 1.2.2, Tue Aug 30 17:21:56 CEST 2011, Andrea Leofreddi
 *	- Fixed viewBox on root tag (#7)
 *	- Improved zoom speed (#2)
 *
 * 1.2.1, Mon Jul  4 00:33:18 CEST 2011, Andrea Leofreddi
 *	- Fixed a regression with mouse wheel (now working on Firefox 5)
 *	- Working with viewBox attribute (#4)
 *	- Added "use strict;" and fixed resulting warnings (#5)
 *	- Added configuration variables, dragging is disabled by default (#3)
 *
 * 1.2, Sat Mar 20 08:42:50 GMT 2010, Zeng Xiaohui
 *	Fixed a bug with browser mouse handler interaction
 *
 * 1.1, Wed Feb  3 17:39:33 GMT 2010, Zeng Xiaohui
 *	Updated the zoom code to support the mouse wheel on Safari/Chrome
 *
 * 1.0, Andrea Leofreddi
 *	First release
 *
 * This code is licensed under the following BSD license:
 *
 * Copyright 2009-2017 Andrea Leofreddi &lt;a.leofreddi@vleo.net&gt;. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification, are
 * permitted provided that the following conditions are met:
 *
 *    1. Redistributions of source code must retain the above copyright
 *       notice, this list of conditions and the following disclaimer.
 *    2. Redistributions in binary form must reproduce the above copyright
 *       notice, this list of conditions and the following disclaimer in the
 *       documentation and/or other materials provided with the distribution.
 *    3. Neither the name of the copyright holder nor the names of its
 *       contributors may be used to endorse or promote products derived from
 *       this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY COPYRIGHT HOLDERS AND CONTRIBUTORS ''AS IS'' AND ANY EXPRESS
 * OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
 * AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL COPYRIGHT HOLDERS OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 * The views and conclusions contained in the software and documentation are those of the
 * authors and should not be interpreted as representing official policies, either expressed
 * or implied, of Andrea Leofreddi.
 */

"use strict";

/// CONFIGURATION
/// ====&gt;

var enablePan = 1; // 1 or 0: enable or disable panning (default enabled)
var enableZoom = 1; // 1 or 0: enable or disable zooming (default enabled)
var enableDrag = 0; // 1 or 0: enable or disable dragging (default disabled)
var zoomScale = 0.2; // Zoom sensitivity

/// &lt;====
/// END OF CONFIGURATION

var root = document.documentElement;

var state = 'none', svgRoot = null, stateTarget, stateOrigin, stateTf;

setupHandlers(root);

/**
 * Register handlers
 */
function setupHandlers(root){
	setAttributes(root, {
		"onmouseup" : "handleMouseUp(evt)",
		"onmousedown" : "handleMouseDown(evt)",
		"onmousemove" : "handleMouseMove(evt)",
		//"onmouseout" : "handleMouseUp(evt)", // Decomment this to stop the pan functionality when dragging out of the SVG element
	});

	if(navigator.userAgent.toLowerCase().indexOf('webkit') &gt;= 0)
		window.addEventListener('mousewheel', handleMouseWheel, false); // Chrome/Safari
	else
		window.addEventListener('DOMMouseScroll', handleMouseWheel, false); // Others
}

/**
 * Retrieves the root element for SVG manipulation. The element is then cached into the svgRoot global variable.
 */
function getRoot(root) {
	if(svgRoot == null) {
		var r = root.getElementById("viewport") ? root.getElementById("viewport") : root.documentElement, t = r;

		while(t != root) {
			if(t.getAttribute("viewBox")) {
				setCTM(r, t.getCTM());

				t.removeAttribute("viewBox");
			}

			t = t.parentNode;
		}

		svgRoot = r;
	}

	return svgRoot;
}

/**
 * Instance an SVGPoint object with given event coordinates.
 */
function getEventPoint(evt) {
	var p = root.createSVGPoint();

	p.x = evt.clientX;
	p.y = evt.clientY;

	return p;
}

/**
 * Sets the current transform matrix of an element.
 */
function setCTM(element, matrix) {
	var s = "matrix(" + matrix.a + "," + matrix.b + "," + matrix.c + "," + matrix.d + "," + matrix.e + "," + matrix.f + ")";

	element.setAttribute("transform", s);
}

/**
 * Dumps a matrix to a string (useful for debug).
 */
function dumpMatrix(matrix) {
	var s = "[ " + matrix.a + ", " + matrix.c + ", " + matrix.e + "\n  " + matrix.b + ", " + matrix.d + ", " + matrix.f + "\n  0, 0, 1 ]";

	return s;
}

/**
 * Sets attributes of an element.
 */
function setAttributes(element, attributes){
	for (var i in attributes)
		element.setAttributeNS(null, i, attributes[i]);
}

/**
 * Handle mouse wheel event.
 */
function handleMouseWheel(evt) {
	if(!enableZoom)
		return;

	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var delta;

	if(evt.wheelDelta)
		delta = evt.wheelDelta / 360; // Chrome/Safari
	else
		delta = evt.detail / -9; // Mozilla

	var z = Math.pow(1 + zoomScale, delta);

	var g = getRoot(svgDoc);

	var p = getEventPoint(evt);

	p = p.matrixTransform(g.getCTM().inverse());

	// Compute new scale matrix in current mouse position
	var k = root.createSVGMatrix().translate(p.x, p.y).scale(z).translate(-p.x, -p.y);

        setCTM(g, g.getCTM().multiply(k));

	if(typeof(stateTf) == "undefined")
		stateTf = g.getCTM().inverse();

	stateTf = stateTf.multiply(k.inverse());
}

/**
 * Handle mouse move event.
 */
function handleMouseMove(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(state == 'pan' &amp;&amp; enablePan) {
		// Pan mode
		var p = getEventPoint(evt).matrixTransform(stateTf);

		setCTM(g, stateTf.inverse().translate(p.x - stateOrigin.x, p.y - stateOrigin.y));
	} else if(state == 'drag' &amp;&amp; enableDrag) {
		// Drag mode
		var p = getEventPoint(evt).matrixTransform(g.getCTM().inverse());

		setCTM(stateTarget, root.createSVGMatrix().translate(p.x - stateOrigin.x, p.y - stateOrigin.y).multiply(g.getCTM().inverse()).multiply(stateTarget.getCTM()));

		stateOrigin = p;
	}
}

/**
 * Handle click event.
 */
function handleMouseDown(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(
		evt.target.tagName == "svg"
		|| !enableDrag // Pan anyway when drag is disabled and the user clicked on an element
	) {
		// Pan mode
		state = 'pan';

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	} else {
		// Drag mode
		state = 'drag';

		stateTarget = evt.target;

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	}
}

/**
 * Handle mouse button release event.
 */
function handleMouseUp(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	if(state == 'pan' || state == 'drag') {
		// Quit pan mode
		state = '';
	}
}
</script><g id="viewport" transform="scale(0.5,0.5) translate(0,0)"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 510)">
<title>unnamed</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-510 573,-510 573,4 -4,4"></polygon>
<g id="clust1" class="cluster">
<title>cluster_L</title>
<polygon fill="none" stroke="#000000" points="8,-402 8,-498 432,-498 432,-402 8,-402"></polygon>
</g>
<!-- Type: alloc_objects -->
<g id="node1" class="node">
<title>Type: alloc_objects</title>
<polygon fill="#f8f8f8" stroke="#000000" points="423.5,-490 16.5,-490 16.5,-410 423.5,-410 423.5,-490"></polygon>
<text text-anchor="start" x="24.5" y="-473.2" font-family="Times,serif" font-size="16.00" fill="#000000">Type: alloc_objects</text>
<text text-anchor="start" x="24.5" y="-455.2" font-family="Times,serif" font-size="16.00" fill="#000000">Time: Mar 23, 2019 at 1:08pm (GMT)</text>
<text text-anchor="start" x="24.5" y="-437.2" font-family="Times,serif" font-size="16.00" fill="#000000">Showing nodes accounting for 43837, 99.83% of 43910 total</text>
<text text-anchor="start" x="24.5" y="-419.2" font-family="Times,serif" font-size="16.00" fill="#000000">Dropped 66 nodes (cum &lt;= 219)</text>
</g>
<!-- N1 -->
<g id="node1" class="node">
<title>N1</title>
<g id="a_node1"><a xlink:title="main.makeByteSlice (43806)">
<polygon fill="#edd5d5" stroke="#b20000" points="569,-173 397,-173 397,-87 569,-87 569,-173"></polygon>
<text text-anchor="middle" x="483" y="-149.8" font-family="Times,serif" font-size="24.00" fill="#000000">main</text>
<text text-anchor="middle" x="483" y="-123.8" font-family="Times,serif" font-size="24.00" fill="#000000">makeByteSlice</text>
<text text-anchor="middle" x="483" y="-97.8" font-family="Times,serif" font-size="24.00" fill="#000000">43806 (99.76%)</text>
</a>
</g>
</g>
<!-- NN1_0 -->
<g id="NN1_0" class="node">
<title>NN1_0</title>
<g id="a_NN1_0"><a xlink:title="43806">
<polygon fill="#f8f8f8" stroke="#000000" points="510,-36 460,-36 456,-32 456,0 506,0 510,-4 510,-36"></polygon>
<polyline fill="none" stroke="#000000" points="506,-32 456,-32 "></polyline>
<polyline fill="none" stroke="#000000" points="506,-32 506,0 "></polyline>
<polyline fill="none" stroke="#000000" points="506,-32 510,-36 "></polyline>
<text text-anchor="middle" x="483" y="-16.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N1&#45;&gt;NN1_0 -->
<g id="edge1" class="edge">
<title>N1-&gt;NN1_0</title>
<g id="a_edge1"><a xlink:title="43806">
<path fill="none" stroke="#000000" d="M483,-86.6979C483,-73.0814 483,-58.4077 483,-46.1319"></path>
<polygon fill="#000000" stroke="#000000" points="486.5001,-46.0482 483,-36.0482 479.5001,-46.0482 486.5001,-46.0482"></polygon>
</a>
</g>
<g id="a_edge1-label"><a xlink:title="43806">
<text text-anchor="middle" x="502.5" y="-57.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 43806</text>
</a>
</g>
</g>
<!-- N2 -->
<g id="node2" class="node">
<title>N2</title>
<g id="a_node2"><a xlink:title="runtime.main (43856)">
<polygon fill="#edd5d5" stroke="#b20000" points="524.5,-468 441.5,-468 441.5,-432 524.5,-432 524.5,-468"></polygon>
<text text-anchor="middle" x="483" y="-457.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="483" y="-448.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="483" y="-439.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 43856 (99.88%)</text>
</a>
</g>
</g>
<!-- N4 -->
<g id="node4" class="node">
<title>N4</title>
<g id="a_node4"><a xlink:title="main.main (43856)">
<polygon fill="#edd5d5" stroke="#b20000" points="524.5,-359 441.5,-359 441.5,-323 524.5,-323 524.5,-359"></polygon>
<text text-anchor="middle" x="483" y="-348.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="483" y="-339.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="483" y="-330.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 43856 (99.88%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N4 -->
<g id="edge2" class="edge">
<title>N2-&gt;N4</title>
<g id="a_edge2"><a xlink:title="runtime.main -> main.main (43856)">
<path fill="none" stroke="#b20000" stroke-width="5" d="M483,-431.5096C483,-414.4952 483,-389.0055 483,-369.4076"></path>
<polygon fill="#b20000" stroke="#b20000" stroke-width="5" points="487.3751,-369.2172 483,-359.2173 478.6251,-369.2173 487.3751,-369.2172"></polygon>
</a>
</g>
<g id="a_edge2-label"><a xlink:title="runtime.main -> main.main (43856)">
<text text-anchor="middle" x="502.5" y="-380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 43856</text>
</a>
</g>
</g>
<!-- N3 -->
<g id="node3" class="node">
<title>N3</title>
<g id="a_node3"><a xlink:title="main.allocate (43837)">
<polygon fill="#edd5d5" stroke="#b20000" points="525.5,-272 440.5,-272 440.5,-224 525.5,-224 525.5,-272"></polygon>
<text text-anchor="middle" x="483" y="-260.8" font-family="Times,serif" font-size="9.00" fill="#000000">main</text>
<text text-anchor="middle" x="483" y="-250.8" font-family="Times,serif" font-size="9.00" fill="#000000">allocate</text>
<text text-anchor="middle" x="483" y="-240.8" font-family="Times,serif" font-size="9.00" fill="#000000">31 (0.071%)</text>
<text text-anchor="middle" x="483" y="-230.8" font-family="Times,serif" font-size="9.00" fill="#000000">of 43837 (99.83%)</text>
</a>
</g>
</g>
<!-- N3&#45;&gt;N1 -->
<g id="edge4" class="edge">
<title>N3-&gt;N1</title>
<g id="a_edge4"><a xlink:title="main.allocate -> main.makeByteSlice (43806)">
<path fill="none" stroke="#b20000" stroke-width="5" d="M483,-223.8362C483,-212.1166 483,-197.5399 483,-183.3966"></path>
<polygon fill="#b20000" stroke="#b20000" stroke-width="5" points="487.3751,-183.2217 483,-173.2218 478.6251,-183.2218 487.3751,-183.2217"></polygon>
</a>
</g>
<g id="a_edge4-label"><a xlink:title="main.allocate -> main.makeByteSlice (43806)">
<text text-anchor="middle" x="502.5" y="-194.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 43806</text>
</a>
</g>
</g>
<!-- N4&#45;&gt;N3 -->
<g id="edge3" class="edge">
<title>N4-&gt;N3</title>
<g id="a_edge3"><a xlink:title="main.main -> main.allocate (43837)">
<path fill="none" stroke="#b20000" stroke-width="5" d="M483,-322.6262C483,-311.1524 483,-296.019 483,-282.3808"></path>
<polygon fill="#b20000" stroke="#b20000" stroke-width="5" points="487.3751,-282.3283 483,-272.3284 478.6251,-282.3284 487.3751,-282.3283"></polygon>
</a>
</g>
<g id="a_edge3-label"><a xlink:title="main.main -> main.allocate (43837)">
<text text-anchor="middle" x="502.5" y="-293.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 43837</text>
</a>
</g>
</g>
</g>
</g></svg>
</div>

당연하게 할당량의 99% 이상이 `makeByteSlice` 내부에 있습니다. 이제 `-inuse_objects`를 사용하여 동일한 프로파일을 살펴보겠습니다.

```sh
% go tool pprof -http=:8080 /var/folders/by/3gf34_z95zg05cyj744_vhx40000gn/T/profile891268605/mem.pprof
```
	
<div>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%px" height="100%px">
<script type="text/ecmascript">
/**
 *  SVGPan library 1.2.2
 * ======================
 *
 * Given an unique existing element with id "viewport" (or when missing, the
 * first g-element), including the library into any SVG adds the following
 * capabilities:
 *
 *  - Mouse panning
 *  - Mouse zooming (using the wheel)
 *  - Object dragging
 *
 * You can configure the behaviour of the pan/zoom/drag with the variables
 * listed in the CONFIGURATION section of this file.
 *
 * Known issues:
 *
 *  - Zooming (while panning) on Safari has still some issues
 *
 * Releases:
 *
 * 1.2.2, Tue Aug 30 17:21:56 CEST 2011, Andrea Leofreddi
 *	- Fixed viewBox on root tag (#7)
 *	- Improved zoom speed (#2)
 *
 * 1.2.1, Mon Jul  4 00:33:18 CEST 2011, Andrea Leofreddi
 *	- Fixed a regression with mouse wheel (now working on Firefox 5)
 *	- Working with viewBox attribute (#4)
 *	- Added "use strict;" and fixed resulting warnings (#5)
 *	- Added configuration variables, dragging is disabled by default (#3)
 *
 * 1.2, Sat Mar 20 08:42:50 GMT 2010, Zeng Xiaohui
 *	Fixed a bug with browser mouse handler interaction
 *
 * 1.1, Wed Feb  3 17:39:33 GMT 2010, Zeng Xiaohui
 *	Updated the zoom code to support the mouse wheel on Safari/Chrome
 *
 * 1.0, Andrea Leofreddi
 *	First release
 *
 * This code is licensed under the following BSD license:
 *
 * Copyright 2009-2017 Andrea Leofreddi &lt;a.leofreddi@vleo.net&gt;. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification, are
 * permitted provided that the following conditions are met:
 *
 *    1. Redistributions of source code must retain the above copyright
 *       notice, this list of conditions and the following disclaimer.
 *    2. Redistributions in binary form must reproduce the above copyright
 *       notice, this list of conditions and the following disclaimer in the
 *       documentation and/or other materials provided with the distribution.
 *    3. Neither the name of the copyright holder nor the names of its
 *       contributors may be used to endorse or promote products derived from
 *       this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY COPYRIGHT HOLDERS AND CONTRIBUTORS ''AS IS'' AND ANY EXPRESS
 * OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
 * AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL COPYRIGHT HOLDERS OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 * The views and conclusions contained in the software and documentation are those of the
 * authors and should not be interpreted as representing official policies, either expressed
 * or implied, of Andrea Leofreddi.
 */

"use strict";

/// CONFIGURATION
/// ====&gt;

var enablePan = 1; // 1 or 0: enable or disable panning (default enabled)
var enableZoom = 1; // 1 or 0: enable or disable zooming (default enabled)
var enableDrag = 0; // 1 or 0: enable or disable dragging (default disabled)
var zoomScale = 0.2; // Zoom sensitivity

/// &lt;====
/// END OF CONFIGURATION

var root = document.documentElement;

var state = 'none', svgRoot = null, stateTarget, stateOrigin, stateTf;

setupHandlers(root);

/**
 * Register handlers
 */
function setupHandlers(root){
	setAttributes(root, {
		"onmouseup" : "handleMouseUp(evt)",
		"onmousedown" : "handleMouseDown(evt)",
		"onmousemove" : "handleMouseMove(evt)",
		//"onmouseout" : "handleMouseUp(evt)", // Decomment this to stop the pan functionality when dragging out of the SVG element
	});

	if(navigator.userAgent.toLowerCase().indexOf('webkit') &gt;= 0)
		window.addEventListener('mousewheel', handleMouseWheel, false); // Chrome/Safari
	else
		window.addEventListener('DOMMouseScroll', handleMouseWheel, false); // Others
}

/**
 * Retrieves the root element for SVG manipulation. The element is then cached into the svgRoot global variable.
 */
function getRoot(root) {
	if(svgRoot == null) {
		var r = root.getElementById("viewport") ? root.getElementById("viewport") : root.documentElement, t = r;

		while(t != root) {
			if(t.getAttribute("viewBox")) {
				setCTM(r, t.getCTM());

				t.removeAttribute("viewBox");
			}

			t = t.parentNode;
		}

		svgRoot = r;
	}

	return svgRoot;
}

/**
 * Instance an SVGPoint object with given event coordinates.
 */
function getEventPoint(evt) {
	var p = root.createSVGPoint();

	p.x = evt.clientX;
	p.y = evt.clientY;

	return p;
}

/**
 * Sets the current transform matrix of an element.
 */
function setCTM(element, matrix) {
	var s = "matrix(" + matrix.a + "," + matrix.b + "," + matrix.c + "," + matrix.d + "," + matrix.e + "," + matrix.f + ")";

	element.setAttribute("transform", s);
}

/**
 * Dumps a matrix to a string (useful for debug).
 */
function dumpMatrix(matrix) {
	var s = "[ " + matrix.a + ", " + matrix.c + ", " + matrix.e + "\n  " + matrix.b + ", " + matrix.d + ", " + matrix.f + "\n  0, 0, 1 ]";

	return s;
}

/**
 * Sets attributes of an element.
 */
function setAttributes(element, attributes){
	for (var i in attributes)
		element.setAttributeNS(null, i, attributes[i]);
}

/**
 * Handle mouse wheel event.
 */
function handleMouseWheel(evt) {
	if(!enableZoom)
		return;

	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var delta;

	if(evt.wheelDelta)
		delta = evt.wheelDelta / 360; // Chrome/Safari
	else
		delta = evt.detail / -9; // Mozilla

	var z = Math.pow(1 + zoomScale, delta);

	var g = getRoot(svgDoc);

	var p = getEventPoint(evt);

	p = p.matrixTransform(g.getCTM().inverse());

	// Compute new scale matrix in current mouse position
	var k = root.createSVGMatrix().translate(p.x, p.y).scale(z).translate(-p.x, -p.y);

        setCTM(g, g.getCTM().multiply(k));

	if(typeof(stateTf) == "undefined")
		stateTf = g.getCTM().inverse();

	stateTf = stateTf.multiply(k.inverse());
}

/**
 * Handle mouse move event.
 */
function handleMouseMove(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(state == 'pan' &amp;&amp; enablePan) {
		// Pan mode
		var p = getEventPoint(evt).matrixTransform(stateTf);

		setCTM(g, stateTf.inverse().translate(p.x - stateOrigin.x, p.y - stateOrigin.y));
	} else if(state == 'drag' &amp;&amp; enableDrag) {
		// Drag mode
		var p = getEventPoint(evt).matrixTransform(g.getCTM().inverse());

		setCTM(stateTarget, root.createSVGMatrix().translate(p.x - stateOrigin.x, p.y - stateOrigin.y).multiply(g.getCTM().inverse()).multiply(stateTarget.getCTM()));

		stateOrigin = p;
	}
}

/**
 * Handle click event.
 */
function handleMouseDown(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(
		evt.target.tagName == "svg"
		|| !enableDrag // Pan anyway when drag is disabled and the user clicked on an element
	) {
		// Pan mode
		state = 'pan';

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	} else {
		// Drag mode
		state = 'drag';

		stateTarget = evt.target;

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	}
}

/**
 * Handle mouse button release event.
 */
function handleMouseUp(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	if(state == 'pan' || state == 'drag') {
		// Quit pan mode
		state = '';
	}
}
</script><g id="viewport" transform="scale(0.5,0.5) translate(0,0)"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 2029)">
<title>unnamed</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-2029 1378,-2029 1378,4 -4,4"></polygon>
<g id="clust1" class="cluster">
<title>cluster_L</title>
<polygon fill="none" stroke="#000000" points="8,-1939 8,-2017 372,-2017 372,-1939 8,-1939"></polygon>
</g>
<!-- Type: inuse_objects -->
<g id="node1" class="node">
<title>Type: inuse_objects</title>
<polygon fill="#f8f8f8" stroke="#000000" points="363.5,-2009 16.5,-2009 16.5,-1947 363.5,-1947 363.5,-2009"></polygon>
<text text-anchor="start" x="24.5" y="-1992.2" font-family="Times,serif" font-size="16.00" fill="#000000">Type: inuse_objects</text>
<text text-anchor="start" x="24.5" y="-1974.2" font-family="Times,serif" font-size="16.00" fill="#000000">Time: Mar 23, 2019 at 1:08pm (GMT)</text>
<text text-anchor="start" x="24.5" y="-1956.2" font-family="Times,serif" font-size="16.00" fill="#000000">Showing nodes accounting for 60, 100% of 60 total</text>
</g>
<!-- N1 -->
<g id="node1" class="node">
<title>N1</title>
<g id="a_node1"><a xlink:title="runtime.malg (24)">
<polygon fill="#eddbd5" stroke="#b22a00" points="639,-823 503,-823 503,-737 639,-737 639,-823"></polygon>
<text text-anchor="middle" x="571" y="-799.8" font-family="Times,serif" font-size="24.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="571" y="-773.8" font-family="Times,serif" font-size="24.00" fill="#000000">malg</text>
<text text-anchor="middle" x="571" y="-747.8" font-family="Times,serif" font-size="24.00" fill="#000000">24 (40.00%)</text>
</a>
</g>
</g>
<!-- NN1_0 -->
<g id="NN1_0" class="node">
<title>NN1_0</title>
<g id="a_NN1_0"><a xlink:title="24">
<polygon fill="#f8f8f8" stroke="#000000" points="598,-682 548,-682 544,-678 544,-646 594,-646 598,-650 598,-682"></polygon>
<polyline fill="none" stroke="#000000" points="594,-678 544,-678 "></polyline>
<polyline fill="none" stroke="#000000" points="594,-678 594,-646 "></polyline>
<polyline fill="none" stroke="#000000" points="594,-678 598,-682 "></polyline>
<text text-anchor="middle" x="571" y="-662.1" font-family="Times,serif" font-size="8.00" fill="#000000">384B</text>
</a>
</g>
</g>
<!-- N1&#45;&gt;NN1_0 -->
<g id="edge1" class="edge">
<title>N1-&gt;NN1_0</title>
<g id="a_edge1"><a xlink:title="24">
<path fill="none" stroke="#000000" d="M571,-736.8059C571,-721.8626 571,-705.5141 571,-692.1016"></path>
<polygon fill="#000000" stroke="#000000" points="574.5001,-692.0573 571,-682.0573 567.5001,-692.0574 574.5001,-692.0573"></polygon>
</a>
</g>
<g id="a_edge1-label"><a xlink:title="24">
<text text-anchor="middle" x="580" y="-707.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 24</text>
</a>
</g>
</g>
<!-- N2 -->
<g id="node2" class="node">
<title>N2</title>
<g id="a_node2"><a xlink:title="runtime.allocm (21)">
<polygon fill="#eddbd5" stroke="#b23000" points="686,-1235 566,-1235 566,-1151 686,-1151 686,-1235"></polygon>
<text text-anchor="middle" x="626" y="-1217.4" font-family="Times,serif" font-size="17.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1198.4" font-family="Times,serif" font-size="17.00" fill="#000000">allocm</text>
<text text-anchor="middle" x="626" y="-1179.4" font-family="Times,serif" font-size="17.00" fill="#000000">7 (11.67%)</text>
<text text-anchor="middle" x="626" y="-1160.4" font-family="Times,serif" font-size="17.00" fill="#000000">of 21 (35.00%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N1 -->
<g id="edge32" class="edge">
<title>N2-&gt;N1</title>
<g id="a_edge32"><a xlink:title="runtime.allocm -> runtime.malg (7)">
<path fill="none" stroke="#b27b4a" d="M594.8039,-1150.7397C582.3903,-1129.9153 571,-1103.7828 571,-1078 571,-1078 571,-1078 571,-892 571,-872.8047 571,-851.7589 571,-833.1441"></path>
<polygon fill="#b27b4a" stroke="#b27b4a" points="574.5001,-833.0798 571,-823.0799 567.5001,-833.0799 574.5001,-833.0798"></polygon>
</a>
</g>
<g id="a_edge32-label"><a xlink:title="runtime.allocm -> runtime.malg (7)">
<text text-anchor="middle" x="576.5" y="-979.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- NN2_0 -->
<g id="NN2_0" class="node">
<title>NN2_0</title>
<g id="a_NN2_0"><a xlink:title="7">
<polygon fill="#f8f8f8" stroke="#000000" points="653,-1096 603,-1096 599,-1092 599,-1060 649,-1060 653,-1064 653,-1096"></polygon>
<polyline fill="none" stroke="#000000" points="649,-1092 599,-1092 "></polyline>
<polyline fill="none" stroke="#000000" points="649,-1092 649,-1060 "></polyline>
<polyline fill="none" stroke="#000000" points="649,-1092 653,-1096 "></polyline>
<text text-anchor="middle" x="626" y="-1076.1" font-family="Times,serif" font-size="8.00" fill="#000000">1kB</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;NN2_0 -->
<g id="edge2" class="edge">
<title>N2-&gt;NN2_0</title>
<g id="a_edge2"><a xlink:title="7">
<path fill="none" stroke="#000000" d="M626,-1150.8312C626,-1135.9603 626,-1119.6133 626,-1106.1809"></path>
<polygon fill="#000000" stroke="#000000" points="629.5001,-1106.117 626,-1096.117 622.5001,-1106.1171 629.5001,-1106.117"></polygon>
</a>
</g>
<g id="a_edge2-label"><a xlink:title="7">
<text text-anchor="middle" x="631.5" y="-1121.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- N34 -->
<g id="node34" class="node">
<title>N34</title>
<g id="a_node34"><a xlink:title="runtime.mcommoninit (7)">
<polygon fill="#ede5df" stroke="#b27b4a" points="708.5,-1001 641.5,-1001 641.5,-965 708.5,-965 708.5,-1001"></polygon>
<text text-anchor="middle" x="675" y="-990.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="675" y="-981.1" font-family="Times,serif" font-size="8.00" fill="#000000">mcommoninit</text>
<text text-anchor="middle" x="675" y="-972.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 7 (11.67%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N34 -->
<g id="edge33" class="edge">
<title>N2-&gt;N34</title>
<g id="a_edge33"><a xlink:title="runtime.allocm -> runtime.mcommoninit (7)">
<path fill="none" stroke="#b27b4a" d="M645.0396,-1150.6853C651.3632,-1135.044 657.8435,-1116.9853 662,-1100 669.2881,-1070.2177 672.4987,-1035.1607 673.9079,-1011.2394"></path>
<polygon fill="#b27b4a" stroke="#b27b4a" points="677.4115,-1011.2597 674.4368,-1001.091 670.421,-1010.8953 677.4115,-1011.2597"></polygon>
</a>
</g>
<g id="a_edge33-label"><a xlink:title="runtime.allocm -> runtime.mcommoninit (7)">
<text text-anchor="middle" x="675.5" y="-1074.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- N3 -->
<g id="node3" class="node">
<title>N3</title>
<g id="a_node3"><a xlink:title="runtime.mstart (17)">
<polygon fill="#eddcd5" stroke="#b23800" points="452.5,-1996 381.5,-1996 381.5,-1960 452.5,-1960 452.5,-1996"></polygon>
<text text-anchor="middle" x="417" y="-1985.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="417" y="-1976.1" font-family="Times,serif" font-size="8.00" fill="#000000">mstart</text>
<text text-anchor="middle" x="417" y="-1967.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 17 (28.33%)</text>
</a>
</g>
</g>
<!-- N8 -->
<g id="node8" class="node">
<title>N8</title>
<g id="a_node8"><a xlink:title="runtime.systemstack (14)">
<polygon fill="#edddd5" stroke="#b23f00" points="468,-1888 366,-1888 366,-1820 468,-1820 468,-1888"></polygon>
<text text-anchor="middle" x="417" y="-1872.8" font-family="Times,serif" font-size="14.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="417" y="-1857.8" font-family="Times,serif" font-size="14.00" fill="#000000">systemstack</text>
<text text-anchor="middle" x="417" y="-1842.8" font-family="Times,serif" font-size="14.00" fill="#000000">3 (5.00%)</text>
<text text-anchor="middle" x="417" y="-1827.8" font-family="Times,serif" font-size="14.00" fill="#000000">of 14 (23.33%)</text>
</a>
</g>
</g>
<!-- N3&#45;&gt;N8 -->
<g id="edge27" class="edge">
<title>N3-&gt;N8</title>
<g id="a_edge27"><a xlink:title="runtime.mstart -> runtime.systemstack (14)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M417,-1959.9693C417,-1943.8642 417,-1919.6041 417,-1898.3586"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="420.5001,-1898.2201 417,-1888.2201 413.5001,-1898.2202 420.5001,-1898.2201"></polygon>
</a>
</g>
<g id="a_edge27-label"><a xlink:title="runtime.mstart -> runtime.systemstack (14)">
<text text-anchor="middle" x="426" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 14</text>
</a>
</g>
</g>
<!-- N36 -->
<g id="node36" class="node">
<title>N36</title>
<g id="a_node36"><a xlink:title="runtime.mstart1 (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="561.5,-1872 498.5,-1872 498.5,-1836 561.5,-1836 561.5,-1872"></polygon>
<text text-anchor="middle" x="530" y="-1861.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="530" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">mstart1</text>
<text text-anchor="middle" x="530" y="-1843.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N3&#45;&gt;N36 -->
<g id="edge49" class="edge">
<title>N3-&gt;N36</title>
<g id="a_edge49"><a xlink:title="runtime.mstart -> runtime.mstart1 (3)">
<path fill="none" stroke="#b2a085" d="M433.4312,-1959.9693C452.6793,-1938.8475 484.7101,-1903.6986 506.5618,-1879.7198"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="509.4003,-1881.8012 513.549,-1872.0524 504.2264,-1877.0863 509.4003,-1881.8012"></polygon>
</a>
</g>
<g id="a_edge49-label"><a xlink:title="runtime.mstart -> runtime.mstart1 (3)">
<text text-anchor="middle" x="486.5" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N4 -->
<g id="node4" class="node">
<title>N4</title>
<g id="a_node4"><a xlink:title="runtime.gcBgMarkWorker (8)">
<polygon fill="#ede4dd" stroke="#b2713b" points="828,-2012 680,-2012 680,-1944 828,-1944 828,-2012"></polygon>
<text text-anchor="middle" x="754" y="-1993.6" font-family="Times,serif" font-size="18.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="754" y="-1973.6" font-family="Times,serif" font-size="18.00" fill="#000000">gcBgMarkWorker</text>
<text text-anchor="middle" x="754" y="-1953.6" font-family="Times,serif" font-size="18.00" fill="#000000">8 (13.33%)</text>
</a>
</g>
</g>
<!-- NN4_0 -->
<g id="NN4_0" class="node">
<title>NN4_0</title>
<g id="a_NN4_0"><a xlink:title="8">
<polygon fill="#f8f8f8" stroke="#000000" points="781,-1872 731,-1872 727,-1868 727,-1836 777,-1836 781,-1840 781,-1872"></polygon>
<polyline fill="none" stroke="#000000" points="777,-1868 727,-1868 "></polyline>
<polyline fill="none" stroke="#000000" points="777,-1868 777,-1836 "></polyline>
<polyline fill="none" stroke="#000000" points="777,-1868 781,-1872 "></polyline>
<text text-anchor="middle" x="754" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N4&#45;&gt;NN4_0 -->
<g id="edge3" class="edge">
<title>N4-&gt;NN4_0</title>
<g id="a_edge3"><a xlink:title="8">
<path fill="none" stroke="#000000" d="M754,-1943.7882C754,-1924.3725 754,-1900.3726 754,-1882.0717"></path>
<polygon fill="#000000" stroke="#000000" points="757.5001,-1882.0327 754,-1872.0327 750.5001,-1882.0328 757.5001,-1882.0327"></polygon>
</a>
</g>
<g id="a_edge3-label"><a xlink:title="8">
<text text-anchor="middle" x="759.5" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 8</text>
</a>
</g>
</g>
<!-- N5 -->
<g id="node5" class="node">
<title>N5</title>
<g id="a_node5"><a xlink:title="runtime.schedule (18)">
<polygon fill="#eddcd5" stroke="#b23600" points="661.5,-1753 590.5,-1753 590.5,-1717 661.5,-1717 661.5,-1753"></polygon>
<text text-anchor="middle" x="626" y="-1742.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1733.1" font-family="Times,serif" font-size="8.00" fill="#000000">schedule</text>
<text text-anchor="middle" x="626" y="-1724.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 18 (30.00%)</text>
</a>
</g>
</g>
<!-- N39 -->
<g id="node39" class="node">
<title>N39</title>
<g id="a_node39"><a xlink:title="runtime.resetspinning (15)">
<polygon fill="#edddd5" stroke="#b23c00" points="661.5,-1644.5 590.5,-1644.5 590.5,-1608.5 661.5,-1608.5 661.5,-1644.5"></polygon>
<text text-anchor="middle" x="626" y="-1633.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">resetspinning</text>
<text text-anchor="middle" x="626" y="-1615.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 15 (25.00%)</text>
</a>
</g>
</g>
<!-- N5&#45;&gt;N39 -->
<g id="edge25" class="edge">
<title>N5-&gt;N39</title>
<g id="a_edge25"><a xlink:title="runtime.schedule -> runtime.resetspinning (15)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M626,-1716.5945C626,-1699.6581 626,-1674.2853 626,-1654.7773"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="629.5001,-1654.6337 626,-1644.6337 622.5001,-1654.6338 629.5001,-1654.6337"></polygon>
</a>
</g>
<g id="a_edge25-label"><a xlink:title="runtime.schedule -> runtime.resetspinning (15)">
<text text-anchor="middle" x="635" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 15</text>
</a>
</g>
</g>
<!-- N42 -->
<g id="node42" class="node">
<title>N42</title>
<g id="a_node42"><a xlink:title="runtime.stoplockedm (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="742.5,-1644.5 679.5,-1644.5 679.5,-1608.5 742.5,-1608.5 742.5,-1644.5"></polygon>
<text text-anchor="middle" x="711" y="-1633.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="711" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">stoplockedm</text>
<text text-anchor="middle" x="711" y="-1615.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N5&#45;&gt;N42 -->
<g id="edge51" class="edge">
<title>N5-&gt;N42</title>
<g id="a_edge51"><a xlink:title="runtime.schedule -> runtime.stoplockedm (3)">
<path fill="none" stroke="#b2a085" d="M640.4191,-1716.5945C654.1786,-1699.0308 675.0459,-1672.3943 690.5259,-1652.6346"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="693.382,-1654.6642 696.7939,-1644.6337 687.8716,-1650.3473 693.382,-1654.6642"></polygon>
</a>
</g>
<g id="a_edge51-label"><a xlink:title="runtime.schedule -> runtime.stoplockedm (3)">
<text text-anchor="middle" x="682.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N6 -->
<g id="node6" class="node">
<title>N6</title>
<g id="a_node6"><a xlink:title="github.com/pkg/profile.Start.func8 (9)">
<polygon fill="#ede3db" stroke="#b2662c" points="942.5,-2017 853.5,-2017 853.5,-1939 942.5,-1939 942.5,-2017"></polygon>
<text text-anchor="middle" x="898" y="-2002.6" font-family="Times,serif" font-size="13.00" fill="#000000">profile</text>
<text text-anchor="middle" x="898" y="-1988.6" font-family="Times,serif" font-size="13.00" fill="#000000">Start</text>
<text text-anchor="middle" x="898" y="-1974.6" font-family="Times,serif" font-size="13.00" fill="#000000">func8</text>
<text text-anchor="middle" x="898" y="-1960.6" font-family="Times,serif" font-size="13.00" fill="#000000">2 (3.33%)</text>
<text text-anchor="middle" x="898" y="-1946.6" font-family="Times,serif" font-size="13.00" fill="#000000">of 9 (15.00%)</text>
</a>
</g>
</g>
<!-- NN6_0 -->
<g id="NN6_0" class="node">
<title>NN6_0</title>
<g id="a_NN6_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="853,-1872 803,-1872 799,-1868 799,-1836 849,-1836 853,-1840 853,-1872"></polygon>
<polyline fill="none" stroke="#000000" points="849,-1868 799,-1868 "></polyline>
<polyline fill="none" stroke="#000000" points="849,-1868 849,-1836 "></polyline>
<polyline fill="none" stroke="#000000" points="849,-1868 853,-1872 "></polyline>
<text text-anchor="middle" x="826" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N6&#45;&gt;NN6_0 -->
<g id="edge4" class="edge">
<title>N6-&gt;NN6_0</title>
<g id="a_edge4"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M875.2188,-1938.7656C864.3037,-1919.9674 851.5059,-1897.9268 841.6965,-1881.0328"></path>
<polygon fill="#000000" stroke="#000000" points="844.6333,-1879.1204 836.5852,-1872.23 838.5798,-1882.6354 844.6333,-1879.1204"></polygon>
</a>
</g>
<g id="a_edge4-label"><a xlink:title="1">
<text text-anchor="middle" x="869.5" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN6_1 -->
<g id="NN6_1" class="node">
<title>NN6_1</title>
<g id="a_NN6_1"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="925,-1872 875,-1872 871,-1868 871,-1836 921,-1836 925,-1840 925,-1872"></polygon>
<polyline fill="none" stroke="#000000" points="921,-1868 871,-1868 "></polyline>
<polyline fill="none" stroke="#000000" points="921,-1868 921,-1836 "></polyline>
<polyline fill="none" stroke="#000000" points="921,-1868 925,-1872 "></polyline>
<text text-anchor="middle" x="898" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">96B</text>
</a>
</g>
</g>
<!-- N6&#45;&gt;NN6_1 -->
<g id="edge5" class="edge">
<title>N6-&gt;NN6_1</title>
<g id="a_edge5"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M898,-1938.7656C898,-1920.4896 898,-1899.1488 898,-1882.4524"></path>
<polygon fill="#000000" stroke="#000000" points="901.5001,-1882.23 898,-1872.23 894.5001,-1882.23 901.5001,-1882.23"></polygon>
</a>
</g>
<g id="a_edge5-label"><a xlink:title="1">
<text text-anchor="middle" x="903.5" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N12 -->
<g id="node12" class="node">
<title>N12</title>
<g id="a_node12"><a xlink:title="os/signal.Notify (7)">
<polygon fill="#ede5df" stroke="#b27b4a" points="1008.5,-1769 913.5,-1769 913.5,-1701 1008.5,-1701 1008.5,-1769"></polygon>
<text text-anchor="middle" x="961" y="-1753.8" font-family="Times,serif" font-size="14.00" fill="#000000">signal</text>
<text text-anchor="middle" x="961" y="-1738.8" font-family="Times,serif" font-size="14.00" fill="#000000">Notify</text>
<text text-anchor="middle" x="961" y="-1723.8" font-family="Times,serif" font-size="14.00" fill="#000000">3 (5.00%)</text>
<text text-anchor="middle" x="961" y="-1708.8" font-family="Times,serif" font-size="14.00" fill="#000000">of 7 (11.67%)</text>
</a>
</g>
</g>
<!-- N6&#45;&gt;N12 -->
<g id="edge31" class="edge">
<title>N6-&gt;N12</title>
<g id="a_edge31"><a xlink:title="github.com/pkg/profile.Start.func8 -> os/signal.Notify (7)">
<path fill="none" stroke="#b27b4a" d="M915.7452,-1938.7387C922.2314,-1923.1877 929.1577,-1904.9936 934,-1888 944.2602,-1851.9925 951.3319,-1810.0505 955.6563,-1779.1959"></path>
<polygon fill="#b27b4a" stroke="#b27b4a" points="959.1386,-1779.5623 957.0182,-1769.1819 952.2025,-1778.619 959.1386,-1779.5623"></polygon>
</a>
</g>
<g id="a_edge31-label"><a xlink:title="github.com/pkg/profile.Start.func8 -> os/signal.Notify (7)">
<text text-anchor="middle" x="954.5" y="-1850.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- N7 -->
<g id="node7" class="node">
<title>N7</title>
<g id="a_node7"><a xlink:title="runtime.mcall (15)">
<polygon fill="#edddd5" stroke="#b23c00" points="661.5,-1996 590.5,-1996 590.5,-1960 661.5,-1960 661.5,-1996"></polygon>
<text text-anchor="middle" x="626" y="-1985.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1976.1" font-family="Times,serif" font-size="8.00" fill="#000000">mcall</text>
<text text-anchor="middle" x="626" y="-1967.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 15 (25.00%)</text>
</a>
</g>
</g>
<!-- N38 -->
<g id="node38" class="node">
<title>N38</title>
<g id="a_node38"><a xlink:title="runtime.park_m (15)">
<polygon fill="#edddd5" stroke="#b23c00" points="661.5,-1872 590.5,-1872 590.5,-1836 661.5,-1836 661.5,-1872"></polygon>
<text text-anchor="middle" x="626" y="-1861.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">park_m</text>
<text text-anchor="middle" x="626" y="-1843.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 15 (25.00%)</text>
</a>
</g>
</g>
<!-- N7&#45;&gt;N38 -->
<g id="edge22" class="edge">
<title>N7-&gt;N38</title>
<g id="a_edge22"><a xlink:title="runtime.mcall -> runtime.park_m (15)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M626,-1959.9693C626,-1939.5822 626,-1906.1268 626,-1882.2614"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="629.5001,-1882.0524 626,-1872.0524 622.5001,-1882.0525 629.5001,-1882.0524"></polygon>
</a>
</g>
<g id="a_edge22-label"><a xlink:title="runtime.mcall -> runtime.park_m (15)">
<text text-anchor="middle" x="635" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 15</text>
</a>
</g>
</g>
<!-- NN8_0 -->
<g id="NN8_0" class="node">
<title>NN8_0</title>
<g id="a_NN8_0"><a xlink:title="2">
<polygon fill="#f8f8f8" stroke="#000000" points="372,-1753 322,-1753 318,-1749 318,-1717 368,-1717 372,-1721 372,-1753"></polygon>
<polyline fill="none" stroke="#000000" points="368,-1749 318,-1749 "></polyline>
<polyline fill="none" stroke="#000000" points="368,-1749 368,-1717 "></polyline>
<polyline fill="none" stroke="#000000" points="368,-1749 372,-1753 "></polyline>
<text text-anchor="middle" x="345" y="-1733.1" font-family="Times,serif" font-size="8.00" fill="#000000">64B</text>
</a>
</g>
</g>
<!-- N8&#45;&gt;NN8_0 -->
<g id="edge6" class="edge">
<title>N8-&gt;NN8_0</title>
<g id="a_edge6"><a xlink:title="2">
<path fill="none" stroke="#000000" d="M396.3677,-1819.8994C385.2882,-1801.5875 371.8181,-1779.3243 361.4517,-1762.1909"></path>
<polygon fill="#000000" stroke="#000000" points="364.218,-1760.0018 356.0468,-1753.2578 358.2289,-1763.6255 364.218,-1760.0018"></polygon>
</a>
</g>
<g id="a_edge6-label"><a xlink:title="2">
<text text-anchor="middle" x="391.5" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 2</text>
</a>
</g>
</g>
<!-- NN8_1 -->
<g id="NN8_1" class="node">
<title>NN8_1</title>
<g id="a_NN8_1"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="444,-1753 394,-1753 390,-1749 390,-1717 440,-1717 444,-1721 444,-1753"></polygon>
<polyline fill="none" stroke="#000000" points="440,-1749 390,-1749 "></polyline>
<polyline fill="none" stroke="#000000" points="440,-1749 440,-1717 "></polyline>
<polyline fill="none" stroke="#000000" points="440,-1749 444,-1753 "></polyline>
<text text-anchor="middle" x="417" y="-1733.1" font-family="Times,serif" font-size="8.00" fill="#000000">48B</text>
</a>
</g>
</g>
<!-- N8&#45;&gt;NN8_1 -->
<g id="edge7" class="edge">
<title>N8-&gt;NN8_1</title>
<g id="a_edge7"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M417,-1819.8994C417,-1802.0962 417,-1780.5581 417,-1763.6304"></path>
<polygon fill="#000000" stroke="#000000" points="420.5001,-1763.2578 417,-1753.2578 413.5001,-1763.2579 420.5001,-1763.2578"></polygon>
</a>
</g>
<g id="a_edge7-label"><a xlink:title="1">
<text text-anchor="middle" x="422.5" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N37 -->
<g id="node37" class="node">
<title>N37</title>
<g id="a_node37"><a xlink:title="runtime.newproc.func1 (11)">
<polygon fill="#ede0d7" stroke="#b2500e" points="515.5,-1648.5 444.5,-1648.5 444.5,-1604.5 515.5,-1604.5 515.5,-1648.5"></polygon>
<text text-anchor="middle" x="480" y="-1638.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="480" y="-1629.1" font-family="Times,serif" font-size="8.00" fill="#000000">newproc</text>
<text text-anchor="middle" x="480" y="-1620.1" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
<text text-anchor="middle" x="480" y="-1611.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 11 (18.33%)</text>
</a>
</g>
</g>
<!-- N8&#45;&gt;N37 -->
<g id="edge29" class="edge">
<title>N8-&gt;N37</title>
<g id="a_edge29"><a xlink:title="runtime.systemstack -> runtime.newproc.func1 (11)">
<path fill="none" stroke="#b2500e" d="M433.4212,-1819.8796C440.2591,-1804.5961 447.7997,-1786.1792 453,-1769 464.2403,-1731.8675 471.7309,-1687.8128 475.9136,-1658.6195"></path>
<polygon fill="#b2500e" stroke="#b2500e" points="479.3994,-1658.9627 477.3058,-1648.5768 472.4658,-1658.0015 479.3994,-1658.9627"></polygon>
</a>
</g>
<g id="a_edge29-label"><a xlink:title="runtime.systemstack -> runtime.newproc.func1 (11)">
<text text-anchor="middle" x="478" y="-1731.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 11</text>
</a>
</g>
</g>
<!-- N9 -->
<g id="node9" class="node">
<title>N9</title>
<g id="a_node9"><a xlink:title="runtime.newm (21)">
<polygon fill="#eddbd5" stroke="#b23000" points="661.5,-1340.5 590.5,-1340.5 590.5,-1304.5 661.5,-1304.5 661.5,-1340.5"></polygon>
<text text-anchor="middle" x="626" y="-1329.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1320.6" font-family="Times,serif" font-size="8.00" fill="#000000">newm</text>
<text text-anchor="middle" x="626" y="-1311.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 21 (35.00%)</text>
</a>
</g>
</g>
<!-- N9&#45;&gt;N2 -->
<g id="edge20" class="edge">
<title>N9-&gt;N2</title>
<g id="a_edge20"><a xlink:title="runtime.newm -> runtime.allocm (21)">
<path fill="none" stroke="#b23000" stroke-width="2" d="M626,-1304.4936C626,-1289.1443 626,-1266.2909 626,-1245.2637"></path>
<polygon fill="#b23000" stroke="#b23000" stroke-width="2" points="629.5001,-1245.1288 626,-1235.1288 622.5001,-1245.1289 629.5001,-1245.1288"></polygon>
</a>
</g>
<g id="a_edge20-label"><a xlink:title="runtime.newm -> runtime.allocm (21)">
<text text-anchor="middle" x="635" y="-1256.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 21</text>
</a>
</g>
</g>
<!-- N10 -->
<g id="node10" class="node">
<title>N10</title>
<g id="a_node10"><a xlink:title="runtime.ensureSigM.func1 (5)">
<polygon fill="#ede8e3" stroke="#b28f68" points="825.5,-1648.5 762.5,-1648.5 762.5,-1604.5 825.5,-1604.5 825.5,-1648.5"></polygon>
<text text-anchor="middle" x="794" y="-1638.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="794" y="-1629.1" font-family="Times,serif" font-size="8.00" fill="#000000">ensureSigM</text>
<text text-anchor="middle" x="794" y="-1620.1" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
<text text-anchor="middle" x="794" y="-1611.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 5 (8.33%)</text>
</a>
</g>
</g>
<!-- N30 -->
<g id="node30" class="node">
<title>N30</title>
<g id="a_node30"><a xlink:title="runtime.LockOSThread (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="827,-1548 761,-1548 761,-1512 827,-1512 827,-1548"></polygon>
<text text-anchor="middle" x="794" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="794" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">LockOSThread</text>
<text text-anchor="middle" x="794" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N10&#45;&gt;N30 -->
<g id="edge47" class="edge">
<title>N10-&gt;N30</title>
<g id="a_edge47"><a xlink:title="runtime.ensureSigM.func1 -> runtime.LockOSThread (3)">
<path fill="none" stroke="#b2a085" d="M794,-1604.1184C794,-1590.4086 794,-1572.7427 794,-1558.0969"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="797.5001,-1558.066 794,-1548.066 790.5001,-1558.066 797.5001,-1558.066"></polygon>
</a>
</g>
<g id="a_edge47-label"><a xlink:title="runtime.ensureSigM.func1 -> runtime.LockOSThread (3)">
<text text-anchor="middle" x="799.5" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N32 -->
<g id="node32" class="node">
<title>N32</title>
<g id="a_node32"><a xlink:title="runtime.chansend1 (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="908.5,-1548 845.5,-1548 845.5,-1512 908.5,-1512 908.5,-1548"></polygon>
<text text-anchor="middle" x="877" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="877" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">chansend1</text>
<text text-anchor="middle" x="877" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1 (1.67%)</text>
</a>
</g>
</g>
<!-- N10&#45;&gt;N32 -->
<g id="edge65" class="edge">
<title>N10-&gt;N32</title>
<g id="a_edge65"><a xlink:title="runtime.ensureSigM.func1 -> runtime.chansend1 (1)">
<path fill="none" stroke="#b2aea3" d="M813.2505,-1604.1184C825.6148,-1589.7431 841.72,-1571.0183 854.6492,-1555.9862"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="857.594,-1557.9298 861.4614,-1548.066 852.287,-1553.3652 857.594,-1557.9298"></polygon>
</a>
</g>
<g id="a_edge65-label"><a xlink:title="runtime.ensureSigM.func1 -> runtime.chansend1 (1)">
<text text-anchor="middle" x="845.5" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N40 -->
<g id="node40" class="node">
<title>N40</title>
<g id="a_node40"><a xlink:title="runtime.selectgo (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="989.5,-1548 926.5,-1548 926.5,-1512 989.5,-1512 989.5,-1548"></polygon>
<text text-anchor="middle" x="958" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="958" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">selectgo</text>
<text text-anchor="middle" x="958" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1 (1.67%)</text>
</a>
</g>
</g>
<!-- N10&#45;&gt;N40 -->
<g id="edge66" class="edge">
<title>N10-&gt;N40</title>
<g id="a_edge66"><a xlink:title="runtime.ensureSigM.func1 -> runtime.selectgo (1)">
<path fill="none" stroke="#b2aea3" d="M825.6208,-1607.8938C852.0775,-1592.3264 889.9577,-1570.0371 918.4241,-1553.2871"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="920.4655,-1556.1469 927.3092,-1548.0589 916.9155,-1550.1138 920.4655,-1556.1469"></polygon>
</a>
</g>
<g id="a_edge66-label"><a xlink:title="runtime.ensureSigM.func1 -> runtime.selectgo (1)">
<text text-anchor="middle" x="890.5" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N11 -->
<g id="node11" class="node">
<title>N11</title>
<g id="a_node11"><a xlink:title="runtime.startm (18)">
<polygon fill="#eddcd5" stroke="#b23600" points="661.5,-1451.5 590.5,-1451.5 590.5,-1415.5 661.5,-1415.5 661.5,-1451.5"></polygon>
<text text-anchor="middle" x="626" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">startm</text>
<text text-anchor="middle" x="626" y="-1422.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 18 (30.00%)</text>
</a>
</g>
</g>
<!-- N11&#45;&gt;N9 -->
<g id="edge21" class="edge">
<title>N11-&gt;N9</title>
<g id="a_edge21"><a xlink:title="runtime.startm -> runtime.newm (18)">
<path fill="none" stroke="#b23600" stroke-width="2" d="M626,-1415.1706C626,-1397.7373 626,-1371.2482 626,-1351.0489"></path>
<polygon fill="#b23600" stroke="#b23600" stroke-width="2" points="629.5001,-1350.8566 626,-1340.8566 622.5001,-1350.8567 629.5001,-1350.8566"></polygon>
</a>
</g>
<g id="a_edge21-label"><a xlink:title="runtime.startm -> runtime.newm (18)">
<text text-anchor="middle" x="635" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 18</text>
</a>
</g>
</g>
<!-- NN12_0 -->
<g id="NN12_0" class="node">
<title>NN12_0</title>
<g id="a_NN12_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="916,-1644.5 866,-1644.5 862,-1640.5 862,-1608.5 912,-1608.5 916,-1612.5 916,-1644.5"></polygon>
<polyline fill="none" stroke="#000000" points="912,-1640.5 862,-1640.5 "></polyline>
<polyline fill="none" stroke="#000000" points="912,-1640.5 912,-1608.5 "></polyline>
<polyline fill="none" stroke="#000000" points="912,-1640.5 916,-1644.5 "></polyline>
<text text-anchor="middle" x="889" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">144B</text>
</a>
</g>
</g>
<!-- N12&#45;&gt;NN12_0 -->
<g id="edge8" class="edge">
<title>N12-&gt;NN12_0</title>
<g id="a_edge8"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M938.4163,-1700.9676C928.1706,-1685.5279 916.2329,-1667.5385 906.6577,-1653.1091"></path>
<polygon fill="#000000" stroke="#000000" points="909.4829,-1651.0365 901.0372,-1644.6395 903.6502,-1654.907 909.4829,-1651.0365"></polygon>
</a>
</g>
<g id="a_edge8-label"><a xlink:title="1">
<text text-anchor="middle" x="930.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN12_1 -->
<g id="NN12_1" class="node">
<title>NN12_1</title>
<g id="a_NN12_1"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="988,-1644.5 938,-1644.5 934,-1640.5 934,-1608.5 984,-1608.5 988,-1612.5 988,-1644.5"></polygon>
<polyline fill="none" stroke="#000000" points="984,-1640.5 934,-1640.5 "></polyline>
<polyline fill="none" stroke="#000000" points="984,-1640.5 984,-1608.5 "></polyline>
<polyline fill="none" stroke="#000000" points="984,-1640.5 988,-1644.5 "></polyline>
<text text-anchor="middle" x="961" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N12&#45;&gt;NN12_1 -->
<g id="edge9" class="edge">
<title>N12-&gt;NN12_1</title>
<g id="a_edge9"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M961,-1700.9676C961,-1686.1105 961,-1668.8926 961,-1654.7575"></path>
<polygon fill="#000000" stroke="#000000" points="964.5001,-1654.6394 961,-1644.6395 957.5001,-1654.6395 964.5001,-1654.6394"></polygon>
</a>
</g>
<g id="a_edge9-label"><a xlink:title="1">
<text text-anchor="middle" x="966.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN12_2 -->
<g id="NN12_2" class="node">
<title>NN12_2</title>
<g id="a_NN12_2"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1060,-1644.5 1010,-1644.5 1006,-1640.5 1006,-1608.5 1056,-1608.5 1060,-1612.5 1060,-1644.5"></polygon>
<polyline fill="none" stroke="#000000" points="1056,-1640.5 1006,-1640.5 "></polyline>
<polyline fill="none" stroke="#000000" points="1056,-1640.5 1056,-1608.5 "></polyline>
<polyline fill="none" stroke="#000000" points="1056,-1640.5 1060,-1644.5 "></polyline>
<text text-anchor="middle" x="1033" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">48B</text>
</a>
</g>
</g>
<!-- N12&#45;&gt;NN12_2 -->
<g id="edge10" class="edge">
<title>N12-&gt;NN12_2</title>
<g id="a_edge10"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M983.5837,-1700.9676C993.8294,-1685.5279 1005.7671,-1667.5385 1015.3423,-1653.1091"></path>
<polygon fill="#000000" stroke="#000000" points="1018.3498,-1654.907 1020.9628,-1644.6395 1012.5171,-1651.0365 1018.3498,-1654.907"></polygon>
</a>
</g>
<g id="a_edge10-label"><a xlink:title="1">
<text text-anchor="middle" x="1008.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N28 -->
<g id="node28" class="node">
<title>N28</title>
<g id="a_node28"><a xlink:title="os/signal.Notify.func1 (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1127.5,-1552 1064.5,-1552 1064.5,-1508 1127.5,-1508 1127.5,-1552"></polygon>
<text text-anchor="middle" x="1096" y="-1541.6" font-family="Times,serif" font-size="8.00" fill="#000000">signal</text>
<text text-anchor="middle" x="1096" y="-1532.6" font-family="Times,serif" font-size="8.00" fill="#000000">Notify</text>
<text text-anchor="middle" x="1096" y="-1523.6" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
<text text-anchor="middle" x="1096" y="-1514.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 4 (6.67%)</text>
</a>
</g>
</g>
<!-- N12&#45;&gt;N28 -->
<g id="edge41" class="edge">
<title>N12-&gt;N28</title>
<g id="a_edge41"><a xlink:title="os/signal.Notify -> os/signal.Notify.func1 (4)">
<path fill="none" stroke="#b29877" d="M1008.7726,-1707.9196C1030.2694,-1693.3971 1054.108,-1673.6132 1069,-1650 1085.745,-1623.4486 1092.1621,-1587.8226 1094.5909,-1562.2983"></path>
<polygon fill="#b29877" stroke="#b29877" points="1098.1033,-1562.2794 1095.4061,-1552.0337 1091.1253,-1561.7251 1098.1033,-1562.2794"></polygon>
</a>
</g>
<g id="a_edge41-label"><a xlink:title="os/signal.Notify -> os/signal.Notify.func1 (4)">
<text text-anchor="middle" x="1092.5" y="-1622.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N13 -->
<g id="node13" class="node">
<title>N13</title>
<g id="a_node13"><a xlink:title="os/signal.signal_enable (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1145.5,-1352 1046.5,-1352 1046.5,-1293 1145.5,-1293 1145.5,-1352"></polygon>
<text text-anchor="middle" x="1096" y="-1336" font-family="Times,serif" font-size="15.00" fill="#000000">signal</text>
<text text-anchor="middle" x="1096" y="-1319" font-family="Times,serif" font-size="15.00" fill="#000000">signal_enable</text>
<text text-anchor="middle" x="1096" y="-1302" font-family="Times,serif" font-size="15.00" fill="#000000">4 (6.67%)</text>
</a>
</g>
</g>
<!-- NN13_0 -->
<g id="NN13_0" class="node">
<title>NN13_0</title>
<g id="a_NN13_0"><a xlink:title="4">
<polygon fill="#f8f8f8" stroke="#000000" points="1123,-1211 1073,-1211 1069,-1207 1069,-1175 1119,-1175 1123,-1179 1123,-1211"></polygon>
<polyline fill="none" stroke="#000000" points="1119,-1207 1069,-1207 "></polyline>
<polyline fill="none" stroke="#000000" points="1119,-1207 1119,-1175 "></polyline>
<polyline fill="none" stroke="#000000" points="1119,-1207 1123,-1211 "></polyline>
<text text-anchor="middle" x="1096" y="-1191.1" font-family="Times,serif" font-size="8.00" fill="#000000">96B</text>
</a>
</g>
</g>
<!-- N13&#45;&gt;NN13_0 -->
<g id="edge11" class="edge">
<title>N13-&gt;NN13_0</title>
<g id="a_edge11"><a xlink:title="4">
<path fill="none" stroke="#000000" d="M1096,-1292.7901C1096,-1271.4241 1096,-1242.774 1096,-1221.6536"></path>
<polygon fill="#000000" stroke="#000000" points="1099.5001,-1221.3737 1096,-1211.3738 1092.5001,-1221.3738 1099.5001,-1221.3737"></polygon>
</a>
</g>
<g id="a_edge11-label"><a xlink:title="4">
<text text-anchor="middle" x="1101.5" y="-1256.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N14 -->
<g id="node14" class="node">
<title>N14</title>
<g id="a_node14"><a xlink:title="runtime.acquireSudog (2)">
<polygon fill="#edebe9" stroke="#b2a794" points="981,-1347.5 893,-1347.5 893,-1297.5 981,-1297.5 981,-1347.5"></polygon>
<text text-anchor="middle" x="937" y="-1333.1" font-family="Times,serif" font-size="13.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="937" y="-1319.1" font-family="Times,serif" font-size="13.00" fill="#000000">acquireSudog</text>
<text text-anchor="middle" x="937" y="-1305.1" font-family="Times,serif" font-size="13.00" fill="#000000">2 (3.33%)</text>
</a>
</g>
</g>
<!-- NN14_0 -->
<g id="NN14_0" class="node">
<title>NN14_0</title>
<g id="a_NN14_0"><a xlink:title="2">
<polygon fill="#f8f8f8" stroke="#000000" points="964,-1211 914,-1211 910,-1207 910,-1175 960,-1175 964,-1179 964,-1211"></polygon>
<polyline fill="none" stroke="#000000" points="960,-1207 910,-1207 "></polyline>
<polyline fill="none" stroke="#000000" points="960,-1207 960,-1175 "></polyline>
<polyline fill="none" stroke="#000000" points="960,-1207 964,-1211 "></polyline>
<text text-anchor="middle" x="937" y="-1191.1" font-family="Times,serif" font-size="8.00" fill="#000000">96B</text>
</a>
</g>
</g>
<!-- N14&#45;&gt;NN14_0 -->
<g id="edge12" class="edge">
<title>N14-&gt;NN14_0</title>
<g id="a_edge12"><a xlink:title="2">
<path fill="none" stroke="#000000" d="M937,-1297.2237C937,-1275.3646 937,-1243.7999 937,-1221.1423"></path>
<polygon fill="#000000" stroke="#000000" points="940.5001,-1221.1214 937,-1211.1214 933.5001,-1221.1215 940.5001,-1221.1214"></polygon>
</a>
</g>
<g id="a_edge12-label"><a xlink:title="2">
<text text-anchor="middle" x="942.5" y="-1256.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 2</text>
</a>
</g>
</g>
<!-- N15 -->
<g id="node15" class="node">
<title>N15</title>
<g id="a_node15"><a xlink:title="runtime.main (6)">
<polygon fill="#ede7e1" stroke="#b28559" points="1185.5,-1996 1118.5,-1996 1118.5,-1960 1185.5,-1960 1185.5,-1996"></polygon>
<text text-anchor="middle" x="1152" y="-1985.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="1152" y="-1976.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="1152" y="-1967.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 6 (10.00%)</text>
</a>
</g>
</g>
<!-- N20 -->
<g id="node20" class="node">
<title>N20</title>
<g id="a_node20"><a xlink:title="main.main (6)">
<polygon fill="#ede7e1" stroke="#b28559" points="1185.5,-1872 1118.5,-1872 1118.5,-1836 1185.5,-1836 1185.5,-1872"></polygon>
<text text-anchor="middle" x="1152" y="-1861.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="1152" y="-1852.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="1152" y="-1843.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 6 (10.00%)</text>
</a>
</g>
</g>
<!-- N15&#45;&gt;N20 -->
<g id="edge36" class="edge">
<title>N15-&gt;N20</title>
<g id="a_edge36"><a xlink:title="runtime.main -> main.main (6)">
<path fill="none" stroke="#b28559" d="M1152,-1959.9693C1152,-1939.5822 1152,-1906.1268 1152,-1882.2614"></path>
<polygon fill="#b28559" stroke="#b28559" points="1155.5001,-1882.0524 1152,-1872.0524 1148.5001,-1882.0525 1155.5001,-1882.0524"></polygon>
</a>
</g>
<g id="a_edge36-label"><a xlink:title="runtime.main -> main.main (6)">
<text text-anchor="middle" x="1157.5" y="-1909.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 6</text>
</a>
</g>
</g>
<!-- N16 -->
<g id="node16" class="node">
<title>N16</title>
<g id="a_node16"><a xlink:title="github.com/pkg/profile.Start (5)">
<polygon fill="#ede8e3" stroke="#b28f68" points="1190.5,-1765 1113.5,-1765 1113.5,-1705 1190.5,-1705 1190.5,-1765"></polygon>
<text text-anchor="middle" x="1152" y="-1751.4" font-family="Times,serif" font-size="12.00" fill="#000000">profile</text>
<text text-anchor="middle" x="1152" y="-1738.4" font-family="Times,serif" font-size="12.00" fill="#000000">Start</text>
<text text-anchor="middle" x="1152" y="-1725.4" font-family="Times,serif" font-size="12.00" fill="#000000">1 (1.67%)</text>
<text text-anchor="middle" x="1152" y="-1712.4" font-family="Times,serif" font-size="12.00" fill="#000000">of 5 (8.33%)</text>
</a>
</g>
</g>
<!-- NN16_0 -->
<g id="NN16_0" class="node">
<title>NN16_0</title>
<g id="a_NN16_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1179,-1644.5 1129,-1644.5 1125,-1640.5 1125,-1608.5 1175,-1608.5 1179,-1612.5 1179,-1644.5"></polygon>
<polyline fill="none" stroke="#000000" points="1175,-1640.5 1125,-1640.5 "></polyline>
<polyline fill="none" stroke="#000000" points="1175,-1640.5 1175,-1608.5 "></polyline>
<polyline fill="none" stroke="#000000" points="1175,-1640.5 1179,-1644.5 "></polyline>
<text text-anchor="middle" x="1152" y="-1624.6" font-family="Times,serif" font-size="8.00" fill="#000000">48B</text>
</a>
</g>
</g>
<!-- N16&#45;&gt;NN16_0 -->
<g id="edge13" class="edge">
<title>N16-&gt;NN16_0</title>
<g id="a_edge13"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1152,-1704.7767C1152,-1689.236 1152,-1670.3988 1152,-1655.0981"></path>
<polygon fill="#000000" stroke="#000000" points="1155.5001,-1654.6795 1152,-1644.6796 1148.5001,-1654.6796 1155.5001,-1654.6795"></polygon>
</a>
</g>
<g id="a_edge13-label"><a xlink:title="1">
<text text-anchor="middle" x="1157.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N24 -->
<g id="node24" class="node">
<title>N24</title>
<g id="a_node24"><a xlink:title="github.com/pkg/profile.Start.func2 (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1246.5,-1552 1183.5,-1552 1183.5,-1508 1246.5,-1508 1246.5,-1552"></polygon>
<text text-anchor="middle" x="1215" y="-1541.6" font-family="Times,serif" font-size="8.00" fill="#000000">profile</text>
<text text-anchor="middle" x="1215" y="-1532.6" font-family="Times,serif" font-size="8.00" fill="#000000">Start</text>
<text text-anchor="middle" x="1215" y="-1523.6" font-family="Times,serif" font-size="8.00" fill="#000000">func2</text>
<text text-anchor="middle" x="1215" y="-1514.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 4 (6.67%)</text>
</a>
</g>
</g>
<!-- N16&#45;&gt;N24 -->
<g id="edge38" class="edge">
<title>N16-&gt;N24</title>
<g id="a_edge38"><a xlink:title="github.com/pkg/profile.Start -> github.com/pkg/profile.Start.func2 (4)">
<path fill="none" stroke="#b29877" d="M1166.3474,-1704.8138C1173.5611,-1688.7909 1182.0176,-1668.6139 1188,-1650 1197.3344,-1620.9567 1204.659,-1586.9587 1209.31,-1562.5803"></path>
<polygon fill="#b29877" stroke="#b29877" points="1212.8089,-1562.9083 1211.197,-1552.4369 1205.927,-1561.628 1212.8089,-1562.9083"></polygon>
</a>
</g>
<g id="a_edge38-label"><a xlink:title="github.com/pkg/profile.Start -> github.com/pkg/profile.Start.func2 (4)">
<text text-anchor="middle" x="1205.5" y="-1622.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N17 -->
<g id="node17" class="node">
<title>N17</title>
<g id="a_node17"><a xlink:title="log.(*Logger).Output (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1253.5,-1359 1176.5,-1359 1176.5,-1286 1253.5,-1286 1253.5,-1359"></polygon>
<text text-anchor="middle" x="1215" y="-1345.4" font-family="Times,serif" font-size="12.00" fill="#000000">log</text>
<text text-anchor="middle" x="1215" y="-1332.4" font-family="Times,serif" font-size="12.00" fill="#000000">(*Logger)</text>
<text text-anchor="middle" x="1215" y="-1319.4" font-family="Times,serif" font-size="12.00" fill="#000000">Output</text>
<text text-anchor="middle" x="1215" y="-1306.4" font-family="Times,serif" font-size="12.00" fill="#000000">1 (1.67%)</text>
<text text-anchor="middle" x="1215" y="-1293.4" font-family="Times,serif" font-size="12.00" fill="#000000">of 4 (6.67%)</text>
</a>
</g>
</g>
<!-- NN17_0 -->
<g id="NN17_0" class="node">
<title>NN17_0</title>
<g id="a_NN17_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1242,-1211 1192,-1211 1188,-1207 1188,-1175 1238,-1175 1242,-1179 1242,-1211"></polygon>
<polyline fill="none" stroke="#000000" points="1238,-1207 1188,-1207 "></polyline>
<polyline fill="none" stroke="#000000" points="1238,-1207 1238,-1175 "></polyline>
<polyline fill="none" stroke="#000000" points="1238,-1207 1242,-1211 "></polyline>
<text text-anchor="middle" x="1215" y="-1191.1" font-family="Times,serif" font-size="8.00" fill="#000000">144B</text>
</a>
</g>
</g>
<!-- N17&#45;&gt;NN17_0 -->
<g id="edge14" class="edge">
<title>N17-&gt;NN17_0</title>
<g id="a_edge14"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1215,-1285.7369C1215,-1265.2637 1215,-1240.1681 1215,-1221.2481"></path>
<polygon fill="#000000" stroke="#000000" points="1218.5001,-1221.1726 1215,-1211.1727 1211.5001,-1221.1727 1218.5001,-1221.1726"></polygon>
</a>
</g>
<g id="a_edge14-label"><a xlink:title="1">
<text text-anchor="middle" x="1220.5" y="-1256.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N25 -->
<g id="node25" class="node">
<title>N25</title>
<g id="a_node25"><a xlink:title="log.(*Logger).formatHeader (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-1100 1246.5,-1100 1246.5,-1056 1309.5,-1056 1309.5,-1100"></polygon>
<text text-anchor="middle" x="1278" y="-1089.6" font-family="Times,serif" font-size="8.00" fill="#000000">log</text>
<text text-anchor="middle" x="1278" y="-1080.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Logger)</text>
<text text-anchor="middle" x="1278" y="-1071.6" font-family="Times,serif" font-size="8.00" fill="#000000">formatHeader</text>
<text text-anchor="middle" x="1278" y="-1062.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N17&#45;&gt;N25 -->
<g id="edge44" class="edge">
<title>N17-&gt;N25</title>
<g id="a_edge44"><a xlink:title="log.(*Logger).Output -> log.(*Logger).formatHeader (3)">
<path fill="none" stroke="#b2a085" d="M1232.2632,-1285.6734C1238.9178,-1270.285 1246.0919,-1252.0481 1251,-1235 1263.1677,-1192.7354 1270.6372,-1142.3973 1274.5501,-1110.3247"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1278.0504,-1110.5266 1275.7424,-1100.1862 1271.0983,-1109.7089 1278.0504,-1110.5266"></polygon>
</a>
</g>
<g id="a_edge44-label"><a xlink:title="log.(*Logger).Output -> log.(*Logger).formatHeader (3)">
<text text-anchor="middle" x="1273.5" y="-1189.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N18 -->
<g id="node18" class="node">
<title>N18</title>
<g id="a_node18"><a xlink:title="runtime.newproc1 (11)">
<polygon fill="#ede0d7" stroke="#b2500e" points="515.5,-1548 444.5,-1548 444.5,-1512 515.5,-1512 515.5,-1548"></polygon>
<text text-anchor="middle" x="480" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="480" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">newproc1</text>
<text text-anchor="middle" x="480" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 11 (18.33%)</text>
</a>
</g>
</g>
<!-- N18&#45;&gt;N1 -->
<g id="edge30" class="edge">
<title>N18-&gt;N1</title>
<g id="a_edge30"><a xlink:title="runtime.newproc1 -> runtime.malg (10)">
<path fill="none" stroke="#b25b1d" d="M480,-1511.8587C480,-1492.4632 480,-1460.8082 480,-1433.5 480,-1433.5 480,-1433.5 480,-892 480,-869.0789 491.6223,-848.2363 506.5599,-830.9141"></path>
<polygon fill="#b25b1d" stroke="#b25b1d" points="509.4473,-832.9439 513.6265,-823.2081 504.2882,-828.2128 509.4473,-832.9439"></polygon>
</a>
</g>
<g id="a_edge30-label"><a xlink:title="runtime.newproc1 -> runtime.malg (10)">
<text text-anchor="middle" x="489" y="-1121.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 10</text>
</a>
</g>
</g>
<!-- N22 -->
<g id="node22" class="node">
<title>N22</title>
<g id="a_node22"><a xlink:title="runtime.allgadd (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="572,-1457 508,-1457 508,-1410 572,-1410 572,-1457"></polygon>
<text text-anchor="middle" x="540" y="-1443.4" font-family="Times,serif" font-size="12.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="540" y="-1430.4" font-family="Times,serif" font-size="12.00" fill="#000000">allgadd</text>
<text text-anchor="middle" x="540" y="-1417.4" font-family="Times,serif" font-size="12.00" fill="#000000">1 (1.67%)</text>
</a>
</g>
</g>
<!-- N18&#45;&gt;N22 -->
<g id="edge67" class="edge">
<title>N18-&gt;N22</title>
<g id="a_edge67"><a xlink:title="runtime.newproc1 -> runtime.allgadd (1)">
<path fill="none" stroke="#b2aea3" d="M491.2855,-1511.8491C499.3446,-1498.8874 510.4037,-1481.1007 519.948,-1465.7503"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="523.0504,-1467.3891 525.3583,-1457.0487 517.1058,-1463.6929 523.0504,-1467.3891"></polygon>
</a>
</g>
<g id="a_edge67-label"><a xlink:title="runtime.newproc1 -> runtime.allgadd (1)">
<text text-anchor="middle" x="519.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N19 -->
<g id="node19" class="node">
<title>N19</title>
<g id="a_node19"><a xlink:title="time.LoadLocationFromTZData (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1357,-322 1199,-322 1199,-258 1357,-258 1357,-322"></polygon>
<text text-anchor="middle" x="1278" y="-307.6" font-family="Times,serif" font-size="13.00" fill="#000000">time</text>
<text text-anchor="middle" x="1278" y="-293.6" font-family="Times,serif" font-size="13.00" fill="#000000">LoadLocationFromTZData</text>
<text text-anchor="middle" x="1278" y="-279.6" font-family="Times,serif" font-size="13.00" fill="#000000">2 (3.33%)</text>
<text text-anchor="middle" x="1278" y="-265.6" font-family="Times,serif" font-size="13.00" fill="#000000">of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- NN19_0 -->
<g id="NN19_0" class="node">
<title>NN19_0</title>
<g id="a_NN19_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1233,-207 1183,-207 1179,-203 1179,-171 1229,-171 1233,-175 1233,-207"></polygon>
<polyline fill="none" stroke="#000000" points="1229,-203 1179,-203 "></polyline>
<polyline fill="none" stroke="#000000" points="1229,-203 1229,-171 "></polyline>
<polyline fill="none" stroke="#000000" points="1229,-203 1233,-207 "></polyline>
<text text-anchor="middle" x="1206" y="-187.1" font-family="Times,serif" font-size="8.00" fill="#000000">224B</text>
</a>
</g>
</g>
<!-- N19&#45;&gt;NN19_0 -->
<g id="edge15" class="edge">
<title>N19-&gt;NN19_0</title>
<g id="a_edge15"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1255.0208,-257.7653C1245.3237,-244.1624 1234.1698,-228.516 1224.9538,-215.588"></path>
<polygon fill="#000000" stroke="#000000" points="1227.5795,-213.2417 1218.9248,-207.1306 1221.8796,-217.305 1227.5795,-213.2417"></polygon>
</a>
</g>
<g id="a_edge15-label"><a xlink:title="1">
<text text-anchor="middle" x="1247.5" y="-228.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN19_1 -->
<g id="NN19_1" class="node">
<title>NN19_1</title>
<g id="a_NN19_1"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1305,-207 1255,-207 1251,-203 1251,-171 1301,-171 1305,-175 1305,-207"></polygon>
<polyline fill="none" stroke="#000000" points="1301,-203 1251,-203 "></polyline>
<polyline fill="none" stroke="#000000" points="1301,-203 1301,-171 "></polyline>
<polyline fill="none" stroke="#000000" points="1301,-203 1305,-207 "></polyline>
<text text-anchor="middle" x="1278" y="-187.1" font-family="Times,serif" font-size="8.00" fill="#000000">4kB</text>
</a>
</g>
</g>
<!-- N19&#45;&gt;NN19_1 -->
<g id="edge16" class="edge">
<title>N19-&gt;NN19_1</title>
<g id="a_edge16"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1278,-257.7653C1278,-244.8164 1278,-230.0157 1278,-217.4709"></path>
<polygon fill="#000000" stroke="#000000" points="1281.5001,-217.1305 1278,-207.1306 1274.5001,-217.1306 1281.5001,-217.1305"></polygon>
</a>
</g>
<g id="a_edge16-label"><a xlink:title="1">
<text text-anchor="middle" x="1283.5" y="-228.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N23 -->
<g id="node23" class="node">
<title>N23</title>
<g id="a_node23"><a xlink:title="time.byteString (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="1374,-134 1308,-134 1308,-87 1374,-87 1374,-134"></polygon>
<text text-anchor="middle" x="1341" y="-120.4" font-family="Times,serif" font-size="12.00" fill="#000000">time</text>
<text text-anchor="middle" x="1341" y="-107.4" font-family="Times,serif" font-size="12.00" fill="#000000">byteString</text>
<text text-anchor="middle" x="1341" y="-94.4" font-family="Times,serif" font-size="12.00" fill="#000000">1 (1.67%)</text>
</a>
</g>
</g>
<!-- N19&#45;&gt;N23 -->
<g id="edge69" class="edge">
<title>N19-&gt;N23</title>
<g id="a_edge69"><a xlink:title="time.LoadLocationFromTZData -> time.byteString (1)">
<path fill="none" stroke="#b2aea3" d="M1293.2347,-257.6914C1300.0805,-242.5006 1307.9587,-224.0226 1314,-207 1321.315,-186.3884 1327.9569,-162.7483 1332.8227,-144.0251"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="1336.2537,-144.7347 1335.3362,-134.1797 1329.4713,-143.0031 1336.2537,-144.7347"></polygon>
</a>
</g>
<g id="a_edge69-label"><a xlink:title="time.LoadLocationFromTZData -> time.byteString (1)">
<text text-anchor="middle" x="1330.5" y="-185.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N20&#45;&gt;N16 -->
<g id="edge37" class="edge">
<title>N20-&gt;N16</title>
<g id="a_edge37"><a xlink:title="main.main -> github.com/pkg/profile.Start (5)">
<path fill="none" stroke="#b28f68" d="M1152,-1835.9265C1152,-1819.9073 1152,-1795.9351 1152,-1775.3731"></path>
<polygon fill="#b28f68" stroke="#b28f68" points="1155.5001,-1775.2981 1152,-1765.2981 1148.5001,-1775.2981 1155.5001,-1775.2981"></polygon>
</a>
</g>
<g id="a_edge37-label"><a xlink:title="main.main -> github.com/pkg/profile.Start (5)">
<text text-anchor="middle" x="1157.5" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 5</text>
</a>
</g>
</g>
<!-- N27 -->
<g id="node27" class="node">
<title>N27</title>
<g id="a_node27"><a xlink:title="main.allocate (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="1286.5,-1753 1223.5,-1753 1223.5,-1717 1286.5,-1717 1286.5,-1753"></polygon>
<text text-anchor="middle" x="1255" y="-1742.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="1255" y="-1733.1" font-family="Times,serif" font-size="8.00" fill="#000000">allocate</text>
<text text-anchor="middle" x="1255" y="-1724.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1 (1.67%)</text>
</a>
</g>
</g>
<!-- N20&#45;&gt;N27 -->
<g id="edge62" class="edge">
<title>N20-&gt;N27</title>
<g id="a_edge62"><a xlink:title="main.main -> main.allocate (1)">
<path fill="none" stroke="#b2aea3" d="M1167.6435,-1835.9265C1184.961,-1815.9188 1213.0171,-1783.5045 1232.6483,-1760.8238"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="1235.3112,-1763.0953 1239.2093,-1753.2436 1230.0184,-1758.5142 1235.3112,-1763.0953"></polygon>
</a>
</g>
<g id="a_edge62-label"><a xlink:title="main.main -> main.allocate (1)">
<text text-anchor="middle" x="1214.5" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N21 -->
<g id="node21" class="node">
<title>N21</title>
<g id="a_node21"><a xlink:title="main.makeByteSlice (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="1336.5,-1650 1247.5,-1650 1247.5,-1603 1336.5,-1603 1336.5,-1650"></polygon>
<text text-anchor="middle" x="1292" y="-1636.4" font-family="Times,serif" font-size="12.00" fill="#000000">main</text>
<text text-anchor="middle" x="1292" y="-1623.4" font-family="Times,serif" font-size="12.00" fill="#000000">makeByteSlice</text>
<text text-anchor="middle" x="1292" y="-1610.4" font-family="Times,serif" font-size="12.00" fill="#000000">1 (1.67%)</text>
</a>
</g>
</g>
<!-- NN21_0 -->
<g id="NN21_0" class="node">
<title>NN21_0</title>
<g id="a_NN21_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1319,-1548 1269,-1548 1265,-1544 1265,-1512 1315,-1512 1319,-1516 1319,-1548"></polygon>
<polyline fill="none" stroke="#000000" points="1315,-1544 1265,-1544 "></polyline>
<polyline fill="none" stroke="#000000" points="1315,-1544 1315,-1512 "></polyline>
<polyline fill="none" stroke="#000000" points="1315,-1544 1319,-1548 "></polyline>
<text text-anchor="middle" x="1292" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N21&#45;&gt;NN21_0 -->
<g id="edge17" class="edge">
<title>N21-&gt;NN21_0</title>
<g id="a_edge17"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1292,-1602.6461C1292,-1589.2142 1292,-1572.3595 1292,-1558.2671"></path>
<polygon fill="#000000" stroke="#000000" points="1295.5001,-1558.1314 1292,-1548.1314 1288.5001,-1558.1315 1295.5001,-1558.1314"></polygon>
</a>
</g>
<g id="a_edge17-label"><a xlink:title="1">
<text text-anchor="middle" x="1297.5" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN22_0 -->
<g id="NN22_0" class="node">
<title>NN22_0</title>
<g id="a_NN22_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="567,-1340.5 517,-1340.5 513,-1336.5 513,-1304.5 563,-1304.5 567,-1308.5 567,-1340.5"></polygon>
<polyline fill="none" stroke="#000000" points="563,-1336.5 513,-1336.5 "></polyline>
<polyline fill="none" stroke="#000000" points="563,-1336.5 563,-1304.5 "></polyline>
<polyline fill="none" stroke="#000000" points="563,-1336.5 567,-1340.5 "></polyline>
<text text-anchor="middle" x="540" y="-1320.6" font-family="Times,serif" font-size="8.00" fill="#000000">128B</text>
</a>
</g>
</g>
<!-- N22&#45;&gt;NN22_0 -->
<g id="edge18" class="edge">
<title>N22-&gt;NN22_0</title>
<g id="a_edge18"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M540,-1409.9598C540,-1392.6172 540,-1368.9589 540,-1350.6212"></path>
<polygon fill="#000000" stroke="#000000" points="543.5001,-1350.5269 540,-1340.527 536.5001,-1350.527 543.5001,-1350.5269"></polygon>
</a>
</g>
<g id="a_edge18-label"><a xlink:title="1">
<text text-anchor="middle" x="545.5" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- NN23_0 -->
<g id="NN23_0" class="node">
<title>NN23_0</title>
<g id="a_NN23_0"><a xlink:title="1">
<polygon fill="#f8f8f8" stroke="#000000" points="1368,-36 1318,-36 1314,-32 1314,0 1364,0 1368,-4 1368,-36"></polygon>
<polyline fill="none" stroke="#000000" points="1364,-32 1314,-32 "></polyline>
<polyline fill="none" stroke="#000000" points="1364,-32 1364,0 "></polyline>
<polyline fill="none" stroke="#000000" points="1364,-32 1368,-36 "></polyline>
<text text-anchor="middle" x="1341" y="-16.1" font-family="Times,serif" font-size="8.00" fill="#000000">16B</text>
</a>
</g>
</g>
<!-- N23&#45;&gt;NN23_0 -->
<g id="edge19" class="edge">
<title>N23-&gt;NN23_0</title>
<g id="a_edge19"><a xlink:title="1">
<path fill="none" stroke="#000000" d="M1341,-86.679C1341,-74.3839 1341,-59.2986 1341,-46.4008"></path>
<polygon fill="#000000" stroke="#000000" points="1344.5001,-46.1905 1341,-36.1905 1337.5001,-46.1905 1344.5001,-46.1905"></polygon>
</a>
</g>
<g id="a_edge19-label"><a xlink:title="1">
<text text-anchor="middle" x="1346.5" y="-57.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N26 -->
<g id="node26" class="node">
<title>N26</title>
<g id="a_node26"><a xlink:title="log.Printf (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1246.5,-1451.5 1183.5,-1451.5 1183.5,-1415.5 1246.5,-1415.5 1246.5,-1451.5"></polygon>
<text text-anchor="middle" x="1215" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">log</text>
<text text-anchor="middle" x="1215" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">Printf</text>
<text text-anchor="middle" x="1215" y="-1422.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 4 (6.67%)</text>
</a>
</g>
</g>
<!-- N24&#45;&gt;N26 -->
<g id="edge39" class="edge">
<title>N24-&gt;N26</title>
<g id="a_edge39"><a xlink:title="github.com/pkg/profile.Start.func2 -> log.Printf (4)">
<path fill="none" stroke="#b29877" d="M1215,-1507.6184C1215,-1493.9086 1215,-1476.2427 1215,-1461.5969"></path>
<polygon fill="#b29877" stroke="#b29877" points="1218.5001,-1461.566 1215,-1451.566 1211.5001,-1461.566 1218.5001,-1461.566"></polygon>
</a>
</g>
<g id="a_edge39-label"><a xlink:title="github.com/pkg/profile.Start.func2 -> log.Printf (4)">
<text text-anchor="middle" x="1220.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N46 -->
<g id="node46" class="node">
<title>N46</title>
<g id="a_node46"><a xlink:title="time.Time.Date (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-1005 1246.5,-1005 1246.5,-961 1309.5,-961 1309.5,-1005"></polygon>
<text text-anchor="middle" x="1278" y="-994.6" font-family="Times,serif" font-size="8.00" fill="#000000">time</text>
<text text-anchor="middle" x="1278" y="-985.6" font-family="Times,serif" font-size="8.00" fill="#000000">Time</text>
<text text-anchor="middle" x="1278" y="-976.6" font-family="Times,serif" font-size="8.00" fill="#000000">Date</text>
<text text-anchor="middle" x="1278" y="-967.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N25&#45;&gt;N46 -->
<g id="edge45" class="edge">
<title>N25-&gt;N46</title>
<g id="a_edge45"><a xlink:title="log.(*Logger).formatHeader -> time.Time.Date (3)">
<path fill="none" stroke="#b2a085" d="M1278,-1055.9663C1278,-1043.9427 1278,-1028.8283 1278,-1015.4785"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-1015.2433 1278,-1005.2433 1274.5001,-1015.2433 1281.5001,-1015.2433"></polygon>
</a>
</g>
<g id="a_edge45-label"><a xlink:title="log.(*Logger).formatHeader -> time.Time.Date (3)">
<text text-anchor="middle" x="1283.5" y="-1026.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N26&#45;&gt;N17 -->
<g id="edge40" class="edge">
<title>N26-&gt;N17</title>
<g id="a_edge40"><a xlink:title="log.Printf -> log.(*Logger).Output (4)">
<path fill="none" stroke="#b29877" d="M1215,-1415.1706C1215,-1402.7447 1215,-1385.7181 1215,-1369.6477"></path>
<polygon fill="#b29877" stroke="#b29877" points="1218.5001,-1369.2831 1215,-1359.2831 1211.5001,-1369.2832 1218.5001,-1369.2831"></polygon>
</a>
</g>
<g id="a_edge40-label"><a xlink:title="log.Printf -> log.(*Logger).Output (4)">
<text text-anchor="middle" x="1220.5" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N27&#45;&gt;N21 -->
<g id="edge61" class="edge">
<title>N27-&gt;N21</title>
<g id="a_edge61"><a xlink:title="main.allocate -> main.makeByteSlice (1)">
<path fill="none" stroke="#b2aea3" d="M1261.2765,-1716.5945C1266.5545,-1701.1172 1274.2351,-1678.5945 1280.5963,-1659.9406"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="1283.9616,-1660.9158 1283.8766,-1650.3213 1277.3362,-1658.6564 1283.9616,-1660.9158"></polygon>
</a>
</g>
<g id="a_edge61-label"><a xlink:title="main.allocate -> main.makeByteSlice (1)">
<text text-anchor="middle" x="1281.5" y="-1671.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N29 -->
<g id="node29" class="node">
<title>N29</title>
<g id="a_node29"><a xlink:title="os/signal.enableSignal (4)">
<polygon fill="#ede9e5" stroke="#b29877" points="1127.5,-1451.5 1064.5,-1451.5 1064.5,-1415.5 1127.5,-1415.5 1127.5,-1451.5"></polygon>
<text text-anchor="middle" x="1096" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">signal</text>
<text text-anchor="middle" x="1096" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">enableSignal</text>
<text text-anchor="middle" x="1096" y="-1422.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 4 (6.67%)</text>
</a>
</g>
</g>
<!-- N28&#45;&gt;N29 -->
<g id="edge42" class="edge">
<title>N28-&gt;N29</title>
<g id="a_edge42"><a xlink:title="os/signal.Notify.func1 -> os/signal.enableSignal (4)">
<path fill="none" stroke="#b29877" d="M1096,-1507.6184C1096,-1493.9086 1096,-1476.2427 1096,-1461.5969"></path>
<polygon fill="#b29877" stroke="#b29877" points="1099.5001,-1461.566 1096,-1451.566 1092.5001,-1461.566 1099.5001,-1461.566"></polygon>
</a>
</g>
<g id="a_edge42-label"><a xlink:title="os/signal.Notify.func1 -> os/signal.enableSignal (4)">
<text text-anchor="middle" x="1101.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N29&#45;&gt;N13 -->
<g id="edge43" class="edge">
<title>N29-&gt;N13</title>
<g id="a_edge43"><a xlink:title="os/signal.enableSignal -> os/signal.signal_enable (4)">
<path fill="none" stroke="#b29877" d="M1096,-1415.1706C1096,-1400.794 1096,-1380.2588 1096,-1362.1713"></path>
<polygon fill="#b29877" stroke="#b29877" points="1099.5001,-1362.1502 1096,-1352.1503 1092.5001,-1362.1503 1099.5001,-1362.1502"></polygon>
</a>
</g>
<g id="a_edge43-label"><a xlink:title="os/signal.enableSignal -> os/signal.signal_enable (4)">
<text text-anchor="middle" x="1101.5" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 4</text>
</a>
</g>
</g>
<!-- N41 -->
<g id="node41" class="node">
<title>N41</title>
<g id="a_node41"><a xlink:title="runtime.startTemplateThread (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="799.5,-1451.5 716.5,-1451.5 716.5,-1415.5 799.5,-1415.5 799.5,-1451.5"></polygon>
<text text-anchor="middle" x="758" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="758" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">startTemplateThread</text>
<text text-anchor="middle" x="758" y="-1422.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N30&#45;&gt;N41 -->
<g id="edge46" class="edge">
<title>N30-&gt;N41</title>
<g id="a_edge46"><a xlink:title="runtime.LockOSThread -> runtime.startTemplateThread (3)">
<path fill="none" stroke="#b2a085" d="M787.2287,-1511.8491C781.8987,-1497.5617 774.3816,-1477.4119 768.3055,-1461.1246"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="771.5155,-1459.7154 764.741,-1451.5695 764.9571,-1462.1622 771.5155,-1459.7154"></polygon>
</a>
</g>
<g id="a_edge46-label"><a xlink:title="runtime.LockOSThread -> runtime.startTemplateThread (3)">
<text text-anchor="middle" x="784.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N31 -->
<g id="node31" class="node">
<title>N31</title>
<g id="a_node31"><a xlink:title="runtime.chansend (1)">
<polygon fill="#edeceb" stroke="#b2aea3" points="913.5,-1451.5 850.5,-1451.5 850.5,-1415.5 913.5,-1415.5 913.5,-1451.5"></polygon>
<text text-anchor="middle" x="882" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="882" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">chansend</text>
<text text-anchor="middle" x="882" y="-1422.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1 (1.67%)</text>
</a>
</g>
</g>
<!-- N31&#45;&gt;N14 -->
<g id="edge63" class="edge">
<title>N31-&gt;N14</title>
<g id="a_edge63"><a xlink:title="runtime.chansend -> runtime.acquireSudog (1)">
<path fill="none" stroke="#b2aea3" d="M891.0821,-1415.1706C898.9482,-1399.2954 910.5351,-1375.911 920.1012,-1356.6049"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="923.306,-1358.02 924.6098,-1347.5057 917.0338,-1354.9121 923.306,-1358.02"></polygon>
</a>
</g>
<g id="a_edge63-label"><a xlink:title="runtime.chansend -> runtime.acquireSudog (1)">
<text text-anchor="middle" x="914.5" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N32&#45;&gt;N31 -->
<g id="edge64" class="edge">
<title>N32-&gt;N31</title>
<g id="a_edge64"><a xlink:title="runtime.chansend1 -> runtime.chansend (1)">
<path fill="none" stroke="#b2aea3" d="M877.9405,-1511.8491C878.6738,-1497.6965 879.7051,-1477.7915 880.5447,-1461.5866"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="884.0415,-1461.7373 881.0638,-1451.5695 877.0509,-1461.375 884.0415,-1461.7373"></polygon>
</a>
</g>
<g id="a_edge64-label"><a xlink:title="runtime.chansend1 -> runtime.chansend (1)">
<text text-anchor="middle" x="884.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N33 -->
<g id="node33" class="node">
<title>N33</title>
<g id="a_node33"><a xlink:title="runtime.handoffp (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="742.5,-1548 679.5,-1548 679.5,-1512 742.5,-1512 742.5,-1548"></polygon>
<text text-anchor="middle" x="711" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="711" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">handoffp</text>
<text text-anchor="middle" x="711" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N33&#45;&gt;N11 -->
<g id="edge48" class="edge">
<title>N33-&gt;N11</title>
<g id="a_edge48"><a xlink:title="runtime.handoffp -> runtime.startm (3)">
<path fill="none" stroke="#b2a085" d="M695.0122,-1511.8491C681.9525,-1497.0226 663.332,-1475.8828 648.7234,-1459.2978"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="651.1524,-1456.7602 641.9162,-1451.5695 645.8996,-1461.387 651.1524,-1456.7602"></polygon>
</a>
</g>
<g id="a_edge48-label"><a xlink:title="runtime.handoffp -> runtime.startm (3)">
<text text-anchor="middle" x="679.5" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N35 -->
<g id="node35" class="node">
<title>N35</title>
<g id="a_node35"><a xlink:title="runtime.mpreinit (7)">
<polygon fill="#ede5df" stroke="#b27b4a" points="673.5,-910 606.5,-910 606.5,-874 673.5,-874 673.5,-910"></polygon>
<text text-anchor="middle" x="640" y="-899.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="640" y="-890.1" font-family="Times,serif" font-size="8.00" fill="#000000">mpreinit</text>
<text text-anchor="middle" x="640" y="-881.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 7 (11.67%)</text>
</a>
</g>
</g>
<!-- N34&#45;&gt;N35 -->
<g id="edge34" class="edge">
<title>N34-&gt;N35</title>
<g id="a_edge34"><a xlink:title="runtime.mcommoninit -> runtime.mpreinit (7)">
<path fill="none" stroke="#b27b4a" d="M667.9172,-964.5848C662.951,-951.6726 656.2249,-934.1849 650.6193,-919.6101"></path>
<polygon fill="#b27b4a" stroke="#b27b4a" points="653.8094,-918.1543 646.9528,-910.0773 647.2759,-920.6672 653.8094,-918.1543"></polygon>
</a>
</g>
<g id="a_edge34-label"><a xlink:title="runtime.mcommoninit -> runtime.mpreinit (7)">
<text text-anchor="middle" x="665.5" y="-931.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- N35&#45;&gt;N1 -->
<g id="edge35" class="edge">
<title>N35-&gt;N1</title>
<g id="a_edge35"><a xlink:title="runtime.mpreinit -> runtime.malg (7)">
<path fill="none" stroke="#b27b4a" d="M628.6061,-873.5055C621.5666,-862.0792 612.152,-846.7974 602.9766,-831.904"></path>
<polygon fill="#b27b4a" stroke="#b27b4a" points="605.8468,-829.8901 597.6217,-823.212 599.887,-833.5618 605.8468,-829.8901"></polygon>
</a>
</g>
<g id="a_edge35-label"><a xlink:title="runtime.mpreinit -> runtime.malg (7)">
<text text-anchor="middle" x="623.5" y="-844.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 7</text>
</a>
</g>
</g>
<!-- N36&#45;&gt;N5 -->
<g id="edge50" class="edge">
<title>N36-&gt;N5</title>
<g id="a_edge50"><a xlink:title="runtime.mstart1 -> runtime.schedule (3)">
<path fill="none" stroke="#b2a085" d="M544.5803,-1835.9265C560.6505,-1816.0062 586.6421,-1783.7874 604.9272,-1761.1215"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="607.7277,-1763.2243 611.2825,-1753.2436 602.2795,-1758.8292 607.7277,-1763.2243"></polygon>
</a>
</g>
<g id="a_edge50-label"><a xlink:title="runtime.mstart1 -> runtime.schedule (3)">
<text text-anchor="middle" x="589.5" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N37&#45;&gt;N18 -->
<g id="edge28" class="edge">
<title>N37-&gt;N18</title>
<g id="a_edge28"><a xlink:title="runtime.newproc.func1 -> runtime.newproc1 (11)">
<path fill="none" stroke="#b2500e" d="M480,-1604.1184C480,-1590.4086 480,-1572.7427 480,-1558.0969"></path>
<polygon fill="#b2500e" stroke="#b2500e" points="483.5001,-1558.066 480,-1548.066 476.5001,-1558.066 483.5001,-1558.066"></polygon>
</a>
</g>
<g id="a_edge28-label"><a xlink:title="runtime.newproc.func1 -> runtime.newproc1 (11)">
<text text-anchor="middle" x="489" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 11</text>
</a>
</g>
</g>
<!-- N38&#45;&gt;N5 -->
<g id="edge23" class="edge">
<title>N38-&gt;N5</title>
<g id="a_edge23"><a xlink:title="runtime.park_m -> runtime.schedule (15)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M626,-1835.9265C626,-1816.7051 626,-1786.0332 626,-1763.5417"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="629.5001,-1763.2436 626,-1753.2436 622.5001,-1763.2437 629.5001,-1763.2436"></polygon>
</a>
</g>
<g id="a_edge23-label"><a xlink:title="runtime.park_m -> runtime.schedule (15)">
<text text-anchor="middle" x="635" y="-1790.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 15</text>
</a>
</g>
</g>
<!-- N43 -->
<g id="node43" class="node">
<title>N43</title>
<g id="a_node43"><a xlink:title="runtime.wakep (15)">
<polygon fill="#edddd5" stroke="#b23c00" points="661.5,-1548 590.5,-1548 590.5,-1512 661.5,-1512 661.5,-1548"></polygon>
<text text-anchor="middle" x="626" y="-1537.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="626" y="-1528.1" font-family="Times,serif" font-size="8.00" fill="#000000">wakep</text>
<text text-anchor="middle" x="626" y="-1519.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 15 (25.00%)</text>
</a>
</g>
</g>
<!-- N39&#45;&gt;N43 -->
<g id="edge24" class="edge">
<title>N39-&gt;N43</title>
<g id="a_edge24"><a xlink:title="runtime.resetspinning -> runtime.wakep (15)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M626,-1608.3491C626,-1594.1965 626,-1574.2915 626,-1558.0866"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="629.5001,-1558.0695 626,-1548.0695 622.5001,-1558.0696 629.5001,-1558.0695"></polygon>
</a>
</g>
<g id="a_edge24-label"><a xlink:title="runtime.resetspinning -> runtime.wakep (15)">
<text text-anchor="middle" x="635" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 15</text>
</a>
</g>
</g>
<!-- N40&#45;&gt;N14 -->
<g id="edge68" class="edge">
<title>N40-&gt;N14</title>
<g id="a_edge68"><a xlink:title="runtime.selectgo -> runtime.acquireSudog (1)">
<path fill="none" stroke="#b2aea3" d="M956.1731,-1511.9487C952.6995,-1477.6265 945.0869,-1402.4059 940.5687,-1357.7626"></path>
<polygon fill="#b2aea3" stroke="#b2aea3" points="944.0347,-1357.2488 939.5455,-1347.652 937.0703,-1357.9537 944.0347,-1357.2488"></polygon>
</a>
</g>
<g id="a_edge68-label"><a xlink:title="runtime.selectgo -> runtime.acquireSudog (1)">
<text text-anchor="middle" x="955.5" y="-1429.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1</text>
</a>
</g>
</g>
<!-- N41&#45;&gt;N9 -->
<g id="edge52" class="edge">
<title>N41-&gt;N9</title>
<g id="a_edge52"><a xlink:title="runtime.startTemplateThread -> runtime.newm (3)">
<path fill="none" stroke="#b2a085" d="M736.4981,-1415.4189C714.4449,-1396.8741 679.9846,-1367.8961 655.414,-1347.2345"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="657.3989,-1344.3306 647.4927,-1340.5734 652.8937,-1349.6882 657.3989,-1344.3306"></polygon>
</a>
</g>
<g id="a_edge52-label"><a xlink:title="runtime.startTemplateThread -> runtime.newm (3)">
<text text-anchor="middle" x="713.5" y="-1380.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N42&#45;&gt;N33 -->
<g id="edge53" class="edge">
<title>N42-&gt;N33</title>
<g id="a_edge53"><a xlink:title="runtime.stoplockedm -> runtime.handoffp (3)">
<path fill="none" stroke="#b2a085" d="M711,-1608.3491C711,-1594.1965 711,-1574.2915 711,-1558.0866"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="714.5001,-1558.0695 711,-1548.0695 707.5001,-1558.0696 714.5001,-1558.0695"></polygon>
</a>
</g>
<g id="a_edge53-label"><a xlink:title="runtime.stoplockedm -> runtime.handoffp (3)">
<text text-anchor="middle" x="716.5" y="-1573.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N43&#45;&gt;N11 -->
<g id="edge26" class="edge">
<title>N43-&gt;N11</title>
<g id="a_edge26"><a xlink:title="runtime.wakep -> runtime.startm (15)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M626,-1511.8491C626,-1497.6965 626,-1477.7915 626,-1461.5866"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="629.5001,-1461.5695 626,-1451.5695 622.5001,-1461.5696 629.5001,-1461.5695"></polygon>
</a>
</g>
<g id="a_edge26-label"><a xlink:title="runtime.wakep -> runtime.startm (15)">
<text text-anchor="middle" x="635" y="-1478.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 15</text>
</a>
</g>
</g>
<!-- N44 -->
<g id="node44" class="node">
<title>N44</title>
<g id="a_node44"><a xlink:title="sync.(*Once).Do (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-591 1246.5,-591 1246.5,-547 1309.5,-547 1309.5,-591"></polygon>
<text text-anchor="middle" x="1278" y="-580.6" font-family="Times,serif" font-size="8.00" fill="#000000">sync</text>
<text text-anchor="middle" x="1278" y="-571.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Once)</text>
<text text-anchor="middle" x="1278" y="-562.6" font-family="Times,serif" font-size="8.00" fill="#000000">Do</text>
<text text-anchor="middle" x="1278" y="-553.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N49 -->
<g id="node49" class="node">
<title>N49</title>
<g id="a_node49"><a xlink:title="time.initLocal (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-496 1246.5,-496 1246.5,-460 1309.5,-460 1309.5,-496"></polygon>
<text text-anchor="middle" x="1278" y="-485.1" font-family="Times,serif" font-size="8.00" fill="#000000">time</text>
<text text-anchor="middle" x="1278" y="-476.1" font-family="Times,serif" font-size="8.00" fill="#000000">initLocal</text>
<text text-anchor="middle" x="1278" y="-467.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N44&#45;&gt;N49 -->
<g id="edge54" class="edge">
<title>N44-&gt;N49</title>
<g id="a_edge54"><a xlink:title="sync.(*Once).Do -> time.initLocal (3)">
<path fill="none" stroke="#b2a085" d="M1278,-546.9714C1278,-534.8075 1278,-519.5615 1278,-506.499"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-506.1556 1278,-496.1556 1274.5001,-506.1556 1281.5001,-506.1556"></polygon>
</a>
</g>
<g id="a_edge54-label"><a xlink:title="sync.(*Once).Do -> time.initLocal (3)">
<text text-anchor="middle" x="1283.5" y="-517.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N45 -->
<g id="node45" class="node">
<title>N45</title>
<g id="a_node45"><a xlink:title="time.(*Location).get (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-686 1246.5,-686 1246.5,-642 1309.5,-642 1309.5,-686"></polygon>
<text text-anchor="middle" x="1278" y="-675.6" font-family="Times,serif" font-size="8.00" fill="#000000">time</text>
<text text-anchor="middle" x="1278" y="-666.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Location)</text>
<text text-anchor="middle" x="1278" y="-657.6" font-family="Times,serif" font-size="8.00" fill="#000000">get</text>
<text text-anchor="middle" x="1278" y="-648.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N45&#45;&gt;N44 -->
<g id="edge55" class="edge">
<title>N45-&gt;N44</title>
<g id="a_edge55"><a xlink:title="time.(*Location).get -> sync.(*Once).Do (3)">
<path fill="none" stroke="#b2a085" d="M1278,-641.9663C1278,-629.9427 1278,-614.8283 1278,-601.4785"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-601.2433 1278,-591.2433 1274.5001,-601.2433 1281.5001,-601.2433"></polygon>
</a>
</g>
<g id="a_edge55-label"><a xlink:title="time.(*Location).get -> sync.(*Once).Do (3)">
<text text-anchor="middle" x="1283.5" y="-612.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N48 -->
<g id="node48" class="node">
<title>N48</title>
<g id="a_node48"><a xlink:title="time.Time.date (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-910 1246.5,-910 1246.5,-874 1309.5,-874 1309.5,-910"></polygon>
<text text-anchor="middle" x="1278" y="-899.1" font-family="Times,serif" font-size="8.00" fill="#000000">Time</text>
<text text-anchor="middle" x="1278" y="-890.1" font-family="Times,serif" font-size="8.00" fill="#000000">date</text>
<text text-anchor="middle" x="1278" y="-881.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N46&#45;&gt;N48 -->
<g id="edge56" class="edge">
<title>N46-&gt;N48</title>
<g id="a_edge56"><a xlink:title="time.Time.Date -> time.Time.date (3)">
<path fill="none" stroke="#b2a085" d="M1278,-960.9714C1278,-948.8075 1278,-933.5615 1278,-920.499"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-920.1556 1278,-910.1556 1274.5001,-920.1556 1281.5001,-920.1556"></polygon>
</a>
</g>
<g id="a_edge56-label"><a xlink:title="time.Time.Date -> time.Time.date (3)">
<text text-anchor="middle" x="1283.5" y="-931.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N47 -->
<g id="node47" class="node">
<title>N47</title>
<g id="a_node47"><a xlink:title="time.Time.abs (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-798 1246.5,-798 1246.5,-762 1309.5,-762 1309.5,-798"></polygon>
<text text-anchor="middle" x="1278" y="-787.1" font-family="Times,serif" font-size="8.00" fill="#000000">Time</text>
<text text-anchor="middle" x="1278" y="-778.1" font-family="Times,serif" font-size="8.00" fill="#000000">abs</text>
<text text-anchor="middle" x="1278" y="-769.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N47&#45;&gt;N45 -->
<g id="edge57" class="edge">
<title>N47-&gt;N45</title>
<g id="a_edge57"><a xlink:title="time.Time.abs -> time.(*Location).get (3)">
<path fill="none" stroke="#b2a085" d="M1278,-761.875C1278,-744.4315 1278,-717.6614 1278,-696.5246"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-696.3659 1278,-686.366 1274.5001,-696.366 1281.5001,-696.3659"></polygon>
</a>
</g>
<g id="a_edge57-label"><a xlink:title="time.Time.abs -> time.(*Location).get (3)">
<text text-anchor="middle" x="1283.5" y="-707.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N48&#45;&gt;N47 -->
<g id="edge58" class="edge">
<title>N48-&gt;N47</title>
<g id="a_edge58"><a xlink:title="time.Time.date -> time.Time.abs (3)">
<path fill="none" stroke="#b2a085" d="M1278,-873.5055C1278,-855.7282 1278,-828.6184 1278,-808.1587"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-808.1503 1278,-798.1504 1274.5001,-808.1504 1281.5001,-808.1503"></polygon>
</a>
</g>
<g id="a_edge58-label"><a xlink:title="time.Time.date -> time.Time.abs (3)">
<text text-anchor="middle" x="1283.5" y="-844.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N50 -->
<g id="node50" class="node">
<title>N50</title>
<g id="a_node50"><a xlink:title="time.loadLocation (3)">
<polygon fill="#edeae7" stroke="#b2a085" points="1309.5,-409 1246.5,-409 1246.5,-373 1309.5,-373 1309.5,-409"></polygon>
<text text-anchor="middle" x="1278" y="-398.1" font-family="Times,serif" font-size="8.00" fill="#000000">time</text>
<text text-anchor="middle" x="1278" y="-389.1" font-family="Times,serif" font-size="8.00" fill="#000000">loadLocation</text>
<text text-anchor="middle" x="1278" y="-380.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3 (5.00%)</text>
</a>
</g>
</g>
<!-- N49&#45;&gt;N50 -->
<g id="edge59" class="edge">
<title>N49-&gt;N50</title>
<g id="a_edge59"><a xlink:title="time.initLocal -> time.loadLocation (3)">
<path fill="none" stroke="#b2a085" d="M1278,-459.9735C1278,-448.1918 1278,-432.5607 1278,-419.1581"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-419.0033 1278,-409.0034 1274.5001,-419.0034 1281.5001,-419.0033"></polygon>
</a>
</g>
<g id="a_edge59-label"><a xlink:title="time.initLocal -> time.loadLocation (3)">
<text text-anchor="middle" x="1283.5" y="-430.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
<!-- N50&#45;&gt;N19 -->
<g id="edge60" class="edge">
<title>N50-&gt;N19</title>
<g id="a_edge60"><a xlink:title="time.loadLocation -> time.LoadLocationFromTZData (3)">
<path fill="none" stroke="#b2a085" d="M1278,-372.9432C1278,-361.6199 1278,-346.5228 1278,-332.3139"></path>
<polygon fill="#b2a085" stroke="#b2a085" points="1281.5001,-332.1913 1278,-322.1913 1274.5001,-332.1913 1281.5001,-332.1913"></polygon>
</a>
</g>
<g id="a_edge60-label"><a xlink:title="time.loadLocation -> time.LoadLocationFromTZData (3)">
<text text-anchor="middle" x="1283.5" y="-343.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3</text>
</a>
</g>
</g>
</g>
</g></svg>
</div>


#### 3.5.6 블록 프로파일링

마지막으로 볼 프로파일 유형은 블록 프로파일 링입니다. `net/http` 패키지의 `ClientServer` 벤치 마크를 사용합니다

```sh
% go test -run=XXX -bench=ClientServer$ -blockprofile=/tmp/block.p net/http
% go tool pprof -http=:8080 /tmp/block.p
```

<div>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%px" height="100%px">
<script type="text/ecmascript">
/**
 *  SVGPan library 1.2.2
 * ======================
 *
 * Given an unique existing element with id "viewport" (or when missing, the
 * first g-element), including the library into any SVG adds the following
 * capabilities:
 *
 *  - Mouse panning
 *  - Mouse zooming (using the wheel)
 *  - Object dragging
 *
 * You can configure the behaviour of the pan/zoom/drag with the variables
 * listed in the CONFIGURATION section of this file.
 *
 * Known issues:
 *
 *  - Zooming (while panning) on Safari has still some issues
 *
 * Releases:
 *
 * 1.2.2, Tue Aug 30 17:21:56 CEST 2011, Andrea Leofreddi
 *	- Fixed viewBox on root tag (#7)
 *	- Improved zoom speed (#2)
 *
 * 1.2.1, Mon Jul  4 00:33:18 CEST 2011, Andrea Leofreddi
 *	- Fixed a regression with mouse wheel (now working on Firefox 5)
 *	- Working with viewBox attribute (#4)
 *	- Added "use strict;" and fixed resulting warnings (#5)
 *	- Added configuration variables, dragging is disabled by default (#3)
 *
 * 1.2, Sat Mar 20 08:42:50 GMT 2010, Zeng Xiaohui
 *	Fixed a bug with browser mouse handler interaction
 *
 * 1.1, Wed Feb  3 17:39:33 GMT 2010, Zeng Xiaohui
 *	Updated the zoom code to support the mouse wheel on Safari/Chrome
 *
 * 1.0, Andrea Leofreddi
 *	First release
 *
 * This code is licensed under the following BSD license:
 *
 * Copyright 2009-2017 Andrea Leofreddi &lt;a.leofreddi@vleo.net&gt;. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification, are
 * permitted provided that the following conditions are met:
 *
 *    1. Redistributions of source code must retain the above copyright
 *       notice, this list of conditions and the following disclaimer.
 *    2. Redistributions in binary form must reproduce the above copyright
 *       notice, this list of conditions and the following disclaimer in the
 *       documentation and/or other materials provided with the distribution.
 *    3. Neither the name of the copyright holder nor the names of its
 *       contributors may be used to endorse or promote products derived from
 *       this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY COPYRIGHT HOLDERS AND CONTRIBUTORS ''AS IS'' AND ANY EXPRESS
 * OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
 * AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL COPYRIGHT HOLDERS OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 * The views and conclusions contained in the software and documentation are those of the
 * authors and should not be interpreted as representing official policies, either expressed
 * or implied, of Andrea Leofreddi.
 */

"use strict";

/// CONFIGURATION
/// ====&gt;

var enablePan = 1; // 1 or 0: enable or disable panning (default enabled)
var enableZoom = 1; // 1 or 0: enable or disable zooming (default enabled)
var enableDrag = 0; // 1 or 0: enable or disable dragging (default disabled)
var zoomScale = 0.2; // Zoom sensitivity

/// &lt;====
/// END OF CONFIGURATION

var root = document.documentElement;

var state = 'none', svgRoot = null, stateTarget, stateOrigin, stateTf;

setupHandlers(root);

/**
 * Register handlers
 */
function setupHandlers(root){
	setAttributes(root, {
		"onmouseup" : "handleMouseUp(evt)",
		"onmousedown" : "handleMouseDown(evt)",
		"onmousemove" : "handleMouseMove(evt)",
		//"onmouseout" : "handleMouseUp(evt)", // Decomment this to stop the pan functionality when dragging out of the SVG element
	});

	if(navigator.userAgent.toLowerCase().indexOf('webkit') &gt;= 0)
		window.addEventListener('mousewheel', handleMouseWheel, false); // Chrome/Safari
	else
		window.addEventListener('DOMMouseScroll', handleMouseWheel, false); // Others
}

/**
 * Retrieves the root element for SVG manipulation. The element is then cached into the svgRoot global variable.
 */
function getRoot(root) {
	if(svgRoot == null) {
		var r = root.getElementById("viewport") ? root.getElementById("viewport") : root.documentElement, t = r;

		while(t != root) {
			if(t.getAttribute("viewBox")) {
				setCTM(r, t.getCTM());

				t.removeAttribute("viewBox");
			}

			t = t.parentNode;
		}

		svgRoot = r;
	}

	return svgRoot;
}

/**
 * Instance an SVGPoint object with given event coordinates.
 */
function getEventPoint(evt) {
	var p = root.createSVGPoint();

	p.x = evt.clientX;
	p.y = evt.clientY;

	return p;
}

/**
 * Sets the current transform matrix of an element.
 */
function setCTM(element, matrix) {
	var s = "matrix(" + matrix.a + "," + matrix.b + "," + matrix.c + "," + matrix.d + "," + matrix.e + "," + matrix.f + ")";

	element.setAttribute("transform", s);
}

/**
 * Dumps a matrix to a string (useful for debug).
 */
function dumpMatrix(matrix) {
	var s = "[ " + matrix.a + ", " + matrix.c + ", " + matrix.e + "\n  " + matrix.b + ", " + matrix.d + ", " + matrix.f + "\n  0, 0, 1 ]";

	return s;
}

/**
 * Sets attributes of an element.
 */
function setAttributes(element, attributes){
	for (var i in attributes)
		element.setAttributeNS(null, i, attributes[i]);
}

/**
 * Handle mouse wheel event.
 */
function handleMouseWheel(evt) {
	if(!enableZoom)
		return;

	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var delta;

	if(evt.wheelDelta)
		delta = evt.wheelDelta / 360; // Chrome/Safari
	else
		delta = evt.detail / -9; // Mozilla

	var z = Math.pow(1 + zoomScale, delta);

	var g = getRoot(svgDoc);

	var p = getEventPoint(evt);

	p = p.matrixTransform(g.getCTM().inverse());

	// Compute new scale matrix in current mouse position
	var k = root.createSVGMatrix().translate(p.x, p.y).scale(z).translate(-p.x, -p.y);

        setCTM(g, g.getCTM().multiply(k));

	if(typeof(stateTf) == "undefined")
		stateTf = g.getCTM().inverse();

	stateTf = stateTf.multiply(k.inverse());
}

/**
 * Handle mouse move event.
 */
function handleMouseMove(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(state == 'pan' &amp;&amp; enablePan) {
		// Pan mode
		var p = getEventPoint(evt).matrixTransform(stateTf);

		setCTM(g, stateTf.inverse().translate(p.x - stateOrigin.x, p.y - stateOrigin.y));
	} else if(state == 'drag' &amp;&amp; enableDrag) {
		// Drag mode
		var p = getEventPoint(evt).matrixTransform(g.getCTM().inverse());

		setCTM(stateTarget, root.createSVGMatrix().translate(p.x - stateOrigin.x, p.y - stateOrigin.y).multiply(g.getCTM().inverse()).multiply(stateTarget.getCTM()));

		stateOrigin = p;
	}
}

/**
 * Handle click event.
 */
function handleMouseDown(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	var g = getRoot(svgDoc);

	if(
		evt.target.tagName == "svg"
		|| !enableDrag // Pan anyway when drag is disabled and the user clicked on an element
	) {
		// Pan mode
		state = 'pan';

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	} else {
		// Drag mode
		state = 'drag';

		stateTarget = evt.target;

		stateTf = g.getCTM().inverse();

		stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
	}
}

/**
 * Handle mouse button release event.
 */
function handleMouseUp(evt) {
	if(evt.preventDefault)
		evt.preventDefault();

	evt.returnValue = false;

	var svgDoc = evt.target.ownerDocument;

	if(state == 'pan' || state == 'drag') {
		// Quit pan mode
		state = '';
	}
}
</script><g id="viewport" transform="scale(0.5,0.5) translate(0,0)"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 1715)">
<title>unnamed</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-1715 606,-1715 606,4 -4,4"></polygon>
<g id="clust1" class="cluster">
<title>cluster_L</title>
<polygon fill="none" stroke="#000000" points="18,-1607 18,-1703 418,-1703 418,-1607 18,-1607"></polygon>
</g>
<!-- Type: delay -->
<g id="node1" class="node">
<title>Type: delay</title>
<polygon fill="#f8f8f8" stroke="#000000" points="409.5,-1695 26.5,-1695 26.5,-1615 409.5,-1615 409.5,-1695"></polygon>
<text text-anchor="start" x="34.5" y="-1678.2" font-family="Times,serif" font-size="16.00" fill="#000000">Type: delay</text>
<text text-anchor="start" x="34.5" y="-1660.2" font-family="Times,serif" font-size="16.00" fill="#000000">Time: Mar 23, 2019 at 6:05pm (CET)</text>
<text text-anchor="start" x="34.5" y="-1642.2" font-family="Times,serif" font-size="16.00" fill="#000000">Showing nodes accounting for 7.82s, 100% of 7.82s total</text>
<text text-anchor="start" x="34.5" y="-1624.2" font-family="Times,serif" font-size="16.00" fill="#000000">Dropped 39 nodes (cum &lt;= 0.04s)</text>
</g>
<!-- N1 -->
<g id="node1" class="node">
<title>N1</title>
<g id="a_node1"><a xlink:title="runtime.selectgo (4.55s)">
<polygon fill="#edd8d5" stroke="#b21a00" points="164,-86 0,-86 0,0 164,0 164,-86"></polygon>
<text text-anchor="middle" x="82" y="-62.8" font-family="Times,serif" font-size="24.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="82" y="-36.8" font-family="Times,serif" font-size="24.00" fill="#000000">selectgo</text>
<text text-anchor="middle" x="82" y="-10.8" font-family="Times,serif" font-size="24.00" fill="#000000">4.55s (58.18%)</text>
</a>
</g>
</g>
<!-- N2 -->
<g id="node2" class="node">
<title>N2</title>
<g id="a_node2"><a xlink:title="testing.(*B).runN (5.06s)">
<polygon fill="#edd8d5" stroke="#b21500" points="508,-1176 428,-1176 428,-1132 508,-1132 508,-1176"></polygon>
<text text-anchor="middle" x="468" y="-1165.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-1156.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*B)</text>
<text text-anchor="middle" x="468" y="-1147.6" font-family="Times,serif" font-size="8.00" fill="#000000">runN</text>
<text text-anchor="middle" x="468" y="-1138.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 5.06s (64.63%)</text>
</a>
</g>
</g>
<!-- N7 -->
<g id="node7" class="node">
<title>N7</title>
<g id="a_node7"><a xlink:title="net/http_test.BenchmarkClientServer (1.94s)">
<polygon fill="#edddd5" stroke="#b23c00" points="410,-1077 316,-1077 316,-1041 410,-1041 410,-1077"></polygon>
<text text-anchor="middle" x="363" y="-1066.1" font-family="Times,serif" font-size="8.00" fill="#000000">http_test</text>
<text text-anchor="middle" x="363" y="-1057.1" font-family="Times,serif" font-size="8.00" fill="#000000">BenchmarkClientServer</text>
<text text-anchor="middle" x="363" y="-1048.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.94s (24.83%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N7 -->
<g id="edge13" class="edge">
<title>N2-&gt;N7</title>
<g id="a_edge13"><a xlink:title="testing.(*B).runN -> net/http_test.BenchmarkClientServer (1.94s)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M443.647,-1131.9663C427.8199,-1117.6466 407.1473,-1098.9428 390.6948,-1084.0572"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="392.7604,-1081.2062 382.9968,-1077.0924 388.064,-1086.3969 392.7604,-1081.2062"></polygon>
</a>
</g>
<g id="a_edge13-label"><a xlink:title="testing.(*B).runN -> net/http_test.BenchmarkClientServer (1.94s)">
<text text-anchor="middle" x="439" y="-1102.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.94s</text>
</a>
</g>
</g>
<!-- N36 -->
<g id="node36" class="node">
<title>N36</title>
<g id="a_node36"><a xlink:title="testing.runBenchmarks.func1 (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1081 428,-1081 428,-1037 508,-1037 508,-1081"></polygon>
<text text-anchor="middle" x="468" y="-1070.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-1061.6" font-family="Times,serif" font-size="8.00" fill="#000000">runBenchmarks</text>
<text text-anchor="middle" x="468" y="-1052.6" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
<text text-anchor="middle" x="468" y="-1043.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N2&#45;&gt;N36 -->
<g id="edge4" class="edge">
<title>N2-&gt;N36</title>
<g id="a_edge4"><a xlink:title="testing.(*B).runN -> testing.runBenchmarks.func1 (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1131.9663C468,-1119.9427 468,-1104.8283 468,-1091.4785"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1091.2433 468,-1081.2433 464.5001,-1091.2433 471.5001,-1091.2433"></polygon>
</a>
</g>
<g id="a_edge4-label"><a xlink:title="testing.(*B).runN -> testing.runBenchmarks.func1 (3.11s)">
<text text-anchor="middle" x="485" y="-1102.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N3 -->
<g id="node3" class="node">
<title>N3</title>
<g id="a_node3"><a xlink:title="runtime.chanrecv1 (3.23s)">
<polygon fill="#eddad5" stroke="#b22900" points="544,-407 392,-407 392,-327 544,-327 544,-407"></polygon>
<text text-anchor="middle" x="468" y="-385.4" font-family="Times,serif" font-size="22.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="468" y="-361.4" font-family="Times,serif" font-size="22.00" fill="#000000">chanrecv1</text>
<text text-anchor="middle" x="468" y="-337.4" font-family="Times,serif" font-size="22.00" fill="#000000">3.23s (41.25%)</text>
</a>
</g>
</g>
<!-- N4 -->
<g id="node4" class="node">
<title>N4</title>
<g id="a_node4"><a xlink:title="runtime.main (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1673 428,-1673 428,-1637 508,-1637 508,-1673"></polygon>
<text text-anchor="middle" x="468" y="-1662.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
<text text-anchor="middle" x="468" y="-1653.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="468" y="-1644.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N16 -->
<g id="node16" class="node">
<title>N16</title>
<g id="a_node16"><a xlink:title="main.main (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1560 428,-1560 428,-1524 508,-1524 508,-1560"></polygon>
<text text-anchor="middle" x="468" y="-1549.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="468" y="-1540.1" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
<text text-anchor="middle" x="468" y="-1531.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N4&#45;&gt;N16 -->
<g id="edge3" class="edge">
<title>N4-&gt;N16</title>
<g id="a_edge3"><a xlink:title="runtime.main -> main.main (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1636.8446C468,-1618.8877 468,-1591.1466 468,-1570.2969"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1570.1103 468,-1560.1103 464.5001,-1570.1104 471.5001,-1570.1103"></polygon>
</a>
</g>
<g id="a_edge3-label"><a xlink:title="runtime.main -> main.main (3.11s)">
<text text-anchor="middle" x="485" y="-1585.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N5 -->
<g id="node5" class="node">
<title>N5</title>
<g id="a_node5"><a xlink:title="net/http.(*persistConn).writeLoop (2.54s)">
<polygon fill="#eddcd5" stroke="#b23300" points="122,-181 42,-181 42,-137 122,-137 122,-181"></polygon>
<text text-anchor="middle" x="82" y="-170.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="82" y="-161.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*persistConn)</text>
<text text-anchor="middle" x="82" y="-152.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeLoop</text>
<text text-anchor="middle" x="82" y="-143.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 2.54s (32.44%)</text>
</a>
</g>
</g>
<!-- N5&#45;&gt;N1 -->
<g id="edge12" class="edge">
<title>N5-&gt;N1</title>
<g id="a_edge12"><a xlink:title="net/http.(*persistConn).writeLoop -> runtime.selectgo (2.54s)">
<path fill="none" stroke="#b23300" stroke-width="2" d="M82,-136.9082C82,-125.3769 82,-110.7235 82,-96.4411"></path>
<polygon fill="#b23300" stroke="#b23300" stroke-width="2" points="85.5001,-96.1567 82,-86.1568 78.5001,-96.1568 85.5001,-96.1567"></polygon>
</a>
</g>
<g id="a_edge12-label"><a xlink:title="net/http.(*persistConn).writeLoop -> runtime.selectgo (2.54s)">
<text text-anchor="middle" x="99" y="-107.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 2.54s</text>
</a>
</g>
</g>
<!-- N6 -->
<g id="node6" class="node">
<title>N6</title>
<g id="a_node6"><a xlink:title="testing.(*B).launch (1.94s)">
<polygon fill="#edddd5" stroke="#b23c00" points="410,-1271 330,-1271 330,-1227 410,-1227 410,-1271"></polygon>
<text text-anchor="middle" x="370" y="-1260.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="370" y="-1251.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*B)</text>
<text text-anchor="middle" x="370" y="-1242.6" font-family="Times,serif" font-size="8.00" fill="#000000">launch</text>
<text text-anchor="middle" x="370" y="-1233.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.94s (24.82%)</text>
</a>
</g>
</g>
<!-- N6&#45;&gt;N2 -->
<g id="edge14" class="edge">
<title>N6-&gt;N2</title>
<g id="a_edge14"><a xlink:title="testing.(*B).launch -> testing.(*B).runN (1.94s)">
<path fill="none" stroke="#b23c00" stroke-width="2" d="M392.7295,-1226.9663C406.1152,-1213.9903 423.2145,-1197.4145 437.7315,-1183.3419"></path>
<polygon fill="#b23c00" stroke="#b23c00" stroke-width="2" points="440.3103,-1185.7167 445.0543,-1176.2433 435.438,-1180.6906 440.3103,-1185.7167"></polygon>
</a>
</g>
<g id="a_edge14-label"><a xlink:title="testing.(*B).launch -> testing.(*B).runN (1.94s)">
<text text-anchor="middle" x="442" y="-1197.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.94s</text>
</a>
</g>
</g>
<!-- N14 -->
<g id="node14" class="node">
<title>N14</title>
<g id="a_node14"><a xlink:title="io/ioutil.ReadAll (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="409,-982 333,-982 333,-946 409,-946 409,-982"></polygon>
<text text-anchor="middle" x="371" y="-971.1" font-family="Times,serif" font-size="8.00" fill="#000000">ioutil</text>
<text text-anchor="middle" x="371" y="-962.1" font-family="Times,serif" font-size="8.00" fill="#000000">ReadAll</text>
<text text-anchor="middle" x="371" y="-953.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N7&#45;&gt;N14 -->
<g id="edge32" class="edge">
<title>N7-&gt;N14</title>
<g id="a_edge32"><a xlink:title="net/http_test.BenchmarkClientServer -> io/ioutil.ReadAll (0.11s)">
<path fill="none" stroke="#b2aea5" d="M364.5425,-1040.683C365.6935,-1027.0149 367.2877,-1008.0833 368.6041,-992.4508"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="372.1107,-992.5192 369.4623,-982.2607 365.1354,-991.9317 372.1107,-992.5192"></polygon>
</a>
</g>
<g id="a_edge32-label"><a xlink:title="net/http_test.BenchmarkClientServer -> io/ioutil.ReadAll (0.11s)">
<text text-anchor="middle" x="385" y="-1007.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N28 -->
<g id="node28" class="node">
<title>N28</title>
<g id="a_node28"><a xlink:title="net/http.Get (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="315,-982 235,-982 235,-946 315,-946 315,-982"></polygon>
<text text-anchor="middle" x="275" y="-971.1" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="275" y="-962.1" font-family="Times,serif" font-size="8.00" fill="#000000">Get</text>
<text text-anchor="middle" x="275" y="-953.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N7&#45;&gt;N28 -->
<g id="edge22" class="edge">
<title>N7-&gt;N28</title>
<g id="a_edge22"><a xlink:title="net/http_test.BenchmarkClientServer -> net/http.Get (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M346.0327,-1040.683C332.6412,-1026.2263 313.7953,-1005.8813 298.8802,-989.7798"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="301.2785,-987.2185 291.9152,-982.2607 296.1432,-991.9754 301.2785,-987.2185"></polygon>
</a>
</g>
<g id="a_edge22-label"><a xlink:title="net/http_test.BenchmarkClientServer -> net/http.Get (1.83s)">
<text text-anchor="middle" x="342" y="-1007.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N8 -->
<g id="node8" class="node">
<title>N8</title>
<g id="a_node8"><a xlink:title="net/http.(*persistConn).readLoop (0.18s)">
<polygon fill="#edecea" stroke="#b2ab9d" points="216,-181 140,-181 140,-137 216,-137 216,-181"></polygon>
<text text-anchor="middle" x="178" y="-170.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="178" y="-161.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*persistConn)</text>
<text text-anchor="middle" x="178" y="-152.6" font-family="Times,serif" font-size="8.00" fill="#000000">readLoop</text>
<text text-anchor="middle" x="178" y="-143.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.18s (2.36%)</text>
</a>
</g>
</g>
<!-- N8&#45;&gt;N1 -->
<g id="edge25" class="edge">
<title>N8-&gt;N1</title>
<g id="a_edge25"><a xlink:title="net/http.(*persistConn).readLoop -> runtime.selectgo (0.18s)">
<path fill="none" stroke="#b2ab9d" d="M159.7172,-136.9082C149.6181,-124.7052 136.6254,-109.0057 124.1652,-93.9496"></path>
<polygon fill="#b2ab9d" stroke="#b2ab9d" points="126.788,-91.6292 117.7159,-86.1568 121.3953,-96.0922 126.788,-91.6292"></polygon>
</a>
</g>
<g id="a_edge25-label"><a xlink:title="net/http.(*persistConn).readLoop -> runtime.selectgo (0.18s)">
<text text-anchor="middle" x="162" y="-107.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.18s</text>
</a>
</g>
</g>
<!-- N9 -->
<g id="node9" class="node">
<title>N9</title>
<g id="a_node9"><a xlink:title="sync.(*Cond).Wait (0.04s)">
<polygon fill="#ededec" stroke="#b2b1ad" points="600.5,-1374 527.5,-1374 527.5,-1322 600.5,-1322 600.5,-1374"></polygon>
<text text-anchor="middle" x="564" y="-1362" font-family="Times,serif" font-size="10.00" fill="#000000">sync</text>
<text text-anchor="middle" x="564" y="-1351" font-family="Times,serif" font-size="10.00" fill="#000000">(*Cond)</text>
<text text-anchor="middle" x="564" y="-1340" font-family="Times,serif" font-size="10.00" fill="#000000">Wait</text>
<text text-anchor="middle" x="564" y="-1329" font-family="Times,serif" font-size="10.00" fill="#000000">0.04s (0.57%)</text>
</a>
</g>
</g>
<!-- N10 -->
<g id="node10" class="node">
<title>N10</title>
<g id="a_node10"><a xlink:title="net/http.(*conn).serve (0.04s)">
<polygon fill="#ededec" stroke="#b2b1ad" points="602,-1677 526,-1677 526,-1633 602,-1633 602,-1677"></polygon>
<text text-anchor="middle" x="564" y="-1666.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="564" y="-1657.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*conn)</text>
<text text-anchor="middle" x="564" y="-1648.6" font-family="Times,serif" font-size="8.00" fill="#000000">serve</text>
<text text-anchor="middle" x="564" y="-1639.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.04s (0.57%)</text>
</a>
</g>
</g>
<!-- N27 -->
<g id="node27" class="node">
<title>N27</title>
<g id="a_node27"><a xlink:title="net/http.(*response).finishRequest (0.04s)">
<polygon fill="#ededec" stroke="#b2b1ad" points="602,-1564 526,-1564 526,-1520 602,-1520 602,-1564"></polygon>
<text text-anchor="middle" x="564" y="-1553.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="564" y="-1544.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*response)</text>
<text text-anchor="middle" x="564" y="-1535.6" font-family="Times,serif" font-size="8.00" fill="#000000">finishRequest</text>
<text text-anchor="middle" x="564" y="-1526.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.04s (0.57%)</text>
</a>
</g>
</g>
<!-- N10&#45;&gt;N27 -->
<g id="edge33" class="edge">
<title>N10-&gt;N27</title>
<g id="a_edge33"><a xlink:title="net/http.(*conn).serve -> net/http.(*response).finishRequest (0.04s)">
<path fill="none" stroke="#b2b1ad" d="M564,-1632.9442C564,-1616.2222 564,-1592.9909 564,-1574.194"></path>
<polygon fill="#b2b1ad" stroke="#b2b1ad" points="567.5001,-1574.0001 564,-1564.0002 560.5001,-1574.0002 567.5001,-1574.0001"></polygon>
</a>
</g>
<g id="a_edge33-label"><a xlink:title="net/http.(*conn).serve -> net/http.(*response).finishRequest (0.04s)">
<text text-anchor="middle" x="581" y="-1585.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.04s</text>
</a>
</g>
</g>
<!-- N11 -->
<g id="node11" class="node">
<title>N11</title>
<g id="a_node11"><a xlink:title="testing.(*B).Run (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-986 428,-986 428,-942 508,-942 508,-986"></polygon>
<text text-anchor="middle" x="468" y="-975.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-966.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*B)</text>
<text text-anchor="middle" x="468" y="-957.6" font-family="Times,serif" font-size="8.00" fill="#000000">Run</text>
<text text-anchor="middle" x="468" y="-948.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N32 -->
<g id="node32" class="node">
<title>N32</title>
<g id="a_node32"><a xlink:title="testing.(*B).run (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-891 428,-891 428,-847 508,-847 508,-891"></polygon>
<text text-anchor="middle" x="468" y="-880.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-871.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*B)</text>
<text text-anchor="middle" x="468" y="-862.6" font-family="Times,serif" font-size="8.00" fill="#000000">run</text>
<text text-anchor="middle" x="468" y="-853.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.77%)</text>
</a>
</g>
</g>
<!-- N11&#45;&gt;N32 -->
<g id="edge8" class="edge">
<title>N11-&gt;N32</title>
<g id="a_edge8"><a xlink:title="testing.(*B).Run -> testing.(*B).run (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-941.9663C468,-929.9427 468,-914.8283 468,-901.4785"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-901.2433 468,-891.2433 464.5001,-901.2433 471.5001,-901.2433"></polygon>
</a>
</g>
<g id="a_edge8-label"><a xlink:title="testing.(*B).Run -> testing.(*B).run (3.11s)">
<text text-anchor="middle" x="485" y="-912.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N12 -->
<g id="node12" class="node">
<title>N12</title>
<g id="a_node12"><a xlink:title="net/http.(*Transport).roundTrip (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-276 234,-276 234,-232 314,-232 314,-276"></polygon>
<text text-anchor="middle" x="274" y="-265.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-256.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Transport)</text>
<text text-anchor="middle" x="274" y="-247.6" font-family="Times,serif" font-size="8.00" fill="#000000">roundTrip</text>
<text text-anchor="middle" x="274" y="-238.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N26 -->
<g id="node26" class="node">
<title>N26</title>
<g id="a_node26"><a xlink:title="net/http.(*persistConn).roundTrip (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-181 234,-181 234,-137 314,-137 314,-181"></polygon>
<text text-anchor="middle" x="274" y="-170.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-161.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*persistConn)</text>
<text text-anchor="middle" x="274" y="-152.6" font-family="Times,serif" font-size="8.00" fill="#000000">roundTrip</text>
<text text-anchor="middle" x="274" y="-143.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.36%)</text>
</a>
</g>
</g>
<!-- N12&#45;&gt;N26 -->
<g id="edge23" class="edge">
<title>N12-&gt;N26</title>
<g id="a_edge23"><a xlink:title="net/http.(*Transport).roundTrip -> net/http.(*persistConn).roundTrip (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-231.9663C274,-219.9427 274,-204.8283 274,-191.4785"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-191.2433 274,-181.2433 270.5001,-191.2433 277.5001,-191.2433"></polygon>
</a>
</g>
<g id="a_edge23-label"><a xlink:title="net/http.(*Transport).roundTrip -> net/http.(*persistConn).roundTrip (1.83s)">
<text text-anchor="middle" x="291" y="-202.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N13 -->
<g id="node13" class="node">
<title>N13</title>
<g id="a_node13"><a xlink:title="bytes.(*Buffer).ReadFrom (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="409,-796 333,-796 333,-752 409,-752 409,-796"></polygon>
<text text-anchor="middle" x="371" y="-785.6" font-family="Times,serif" font-size="8.00" fill="#000000">bytes</text>
<text text-anchor="middle" x="371" y="-776.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Buffer)</text>
<text text-anchor="middle" x="371" y="-767.6" font-family="Times,serif" font-size="8.00" fill="#000000">ReadFrom</text>
<text text-anchor="middle" x="371" y="-758.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N22 -->
<g id="node22" class="node">
<title>N22</title>
<g id="a_node22"><a xlink:title="net/http.(*bodyEOFSignal).Read (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="409.5,-701 332.5,-701 332.5,-657 409.5,-657 409.5,-701"></polygon>
<text text-anchor="middle" x="371" y="-690.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="371" y="-681.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*bodyEOFSignal)</text>
<text text-anchor="middle" x="371" y="-672.6" font-family="Times,serif" font-size="8.00" fill="#000000">Read</text>
<text text-anchor="middle" x="371" y="-663.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N13&#45;&gt;N22 -->
<g id="edge26" class="edge">
<title>N13-&gt;N22</title>
<g id="a_edge26"><a xlink:title="bytes.(*Buffer).ReadFrom -> net/http.(*bodyEOFSignal).Read (0.11s)">
<path fill="none" stroke="#b2aea5" d="M371,-751.9663C371,-739.9427 371,-724.8283 371,-711.4785"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="374.5001,-711.2433 371,-701.2433 367.5001,-711.2433 374.5001,-711.2433"></polygon>
</a>
</g>
<g id="a_edge26-label"><a xlink:title="bytes.(*Buffer).ReadFrom -> net/http.(*bodyEOFSignal).Read (0.11s)">
<text text-anchor="middle" x="388" y="-722.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N15 -->
<g id="node15" class="node">
<title>N15</title>
<g id="a_node15"><a xlink:title="io/ioutil.readAll (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="409,-887 333,-887 333,-851 409,-851 409,-887"></polygon>
<text text-anchor="middle" x="371" y="-876.1" font-family="Times,serif" font-size="8.00" fill="#000000">ioutil</text>
<text text-anchor="middle" x="371" y="-867.1" font-family="Times,serif" font-size="8.00" fill="#000000">readAll</text>
<text text-anchor="middle" x="371" y="-858.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N14&#45;&gt;N15 -->
<g id="edge27" class="edge">
<title>N14-&gt;N15</title>
<g id="a_edge27"><a xlink:title="io/ioutil.ReadAll -> io/ioutil.readAll (0.11s)">
<path fill="none" stroke="#b2aea5" d="M371,-945.683C371,-932.0149 371,-913.0833 371,-897.4508"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="374.5001,-897.2607 371,-887.2607 367.5001,-897.2608 374.5001,-897.2607"></polygon>
</a>
</g>
<g id="a_edge27-label"><a xlink:title="io/ioutil.ReadAll -> io/ioutil.readAll (0.11s)">
<text text-anchor="middle" x="388" y="-912.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N15&#45;&gt;N13 -->
<g id="edge28" class="edge">
<title>N15-&gt;N13</title>
<g id="a_edge28"><a xlink:title="io/ioutil.readAll -> bytes.(*Buffer).ReadFrom (0.11s)">
<path fill="none" stroke="#b2aea5" d="M371,-850.683C371,-838.1876 371,-821.2931 371,-806.542"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="374.5001,-806.2732 371,-796.2733 367.5001,-806.2733 374.5001,-806.2732"></polygon>
</a>
</g>
<g id="a_edge28-label"><a xlink:title="io/ioutil.readAll -> bytes.(*Buffer).ReadFrom (0.11s)">
<text text-anchor="middle" x="388" y="-817.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N30 -->
<g id="node30" class="node">
<title>N30</title>
<g id="a_node30"><a xlink:title="net/http_test.TestMain (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1465 428,-1465 428,-1429 508,-1429 508,-1465"></polygon>
<text text-anchor="middle" x="468" y="-1454.1" font-family="Times,serif" font-size="8.00" fill="#000000">http_test</text>
<text text-anchor="middle" x="468" y="-1445.1" font-family="Times,serif" font-size="8.00" fill="#000000">TestMain</text>
<text text-anchor="middle" x="468" y="-1436.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N16&#45;&gt;N30 -->
<g id="edge1" class="edge">
<title>N16-&gt;N30</title>
<g id="a_edge1"><a xlink:title="main.main -> net/http_test.TestMain (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1523.683C468,-1510.0149 468,-1491.0833 468,-1475.4508"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1475.2607 468,-1465.2607 464.5001,-1475.2608 471.5001,-1475.2607"></polygon>
</a>
</g>
<g id="a_edge1-label"><a xlink:title="main.main -> net/http_test.TestMain (3.11s)">
<text text-anchor="middle" x="485" y="-1490.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N17 -->
<g id="node17" class="node">
<title>N17</title>
<g id="a_node17"><a xlink:title="net/http.(*Client).Do (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-796 234,-796 234,-752 314,-752 314,-796"></polygon>
<text text-anchor="middle" x="274" y="-785.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-776.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Client)</text>
<text text-anchor="middle" x="274" y="-767.6" font-family="Times,serif" font-size="8.00" fill="#000000">Do</text>
<text text-anchor="middle" x="274" y="-758.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N19 -->
<g id="node19" class="node">
<title>N19</title>
<g id="a_node19"><a xlink:title="net/http.(*Client).do (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-701 234,-701 234,-657 314,-657 314,-701"></polygon>
<text text-anchor="middle" x="274" y="-690.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-681.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Client)</text>
<text text-anchor="middle" x="274" y="-672.6" font-family="Times,serif" font-size="8.00" fill="#000000">do</text>
<text text-anchor="middle" x="274" y="-663.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N17&#45;&gt;N19 -->
<g id="edge15" class="edge">
<title>N17-&gt;N19</title>
<g id="a_edge15"><a xlink:title="net/http.(*Client).Do -> net/http.(*Client).do (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-751.9663C274,-739.9427 274,-724.8283 274,-711.4785"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-711.2433 274,-701.2433 270.5001,-711.2433 277.5001,-711.2433"></polygon>
</a>
</g>
<g id="a_edge15-label"><a xlink:title="net/http.(*Client).Do -> net/http.(*Client).do (1.83s)">
<text text-anchor="middle" x="291" y="-722.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N18 -->
<g id="node18" class="node">
<title>N18</title>
<g id="a_node18"><a xlink:title="net/http.(*Client).Get (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-891 234,-891 234,-847 314,-847 314,-891"></polygon>
<text text-anchor="middle" x="274" y="-880.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-871.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Client)</text>
<text text-anchor="middle" x="274" y="-862.6" font-family="Times,serif" font-size="8.00" fill="#000000">Get</text>
<text text-anchor="middle" x="274" y="-853.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N18&#45;&gt;N17 -->
<g id="edge16" class="edge">
<title>N18-&gt;N17</title>
<g id="a_edge16"><a xlink:title="net/http.(*Client).Get -> net/http.(*Client).Do (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-846.9663C274,-834.9427 274,-819.8283 274,-806.4785"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-806.2433 274,-796.2433 270.5001,-806.2433 277.5001,-806.2433"></polygon>
</a>
</g>
<g id="a_edge16-label"><a xlink:title="net/http.(*Client).Get -> net/http.(*Client).Do (1.83s)">
<text text-anchor="middle" x="291" y="-817.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N20 -->
<g id="node20" class="node">
<title>N20</title>
<g id="a_node20"><a xlink:title="net/http.(*Client).send (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-606 234,-606 234,-562 314,-562 314,-606"></polygon>
<text text-anchor="middle" x="274" y="-595.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-586.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Client)</text>
<text text-anchor="middle" x="274" y="-577.6" font-family="Times,serif" font-size="8.00" fill="#000000">send</text>
<text text-anchor="middle" x="274" y="-568.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N19&#45;&gt;N20 -->
<g id="edge17" class="edge">
<title>N19-&gt;N20</title>
<g id="a_edge17"><a xlink:title="net/http.(*Client).do -> net/http.(*Client).send (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-656.9663C274,-644.9427 274,-629.8283 274,-616.4785"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-616.2433 274,-606.2433 270.5001,-616.2433 277.5001,-616.2433"></polygon>
</a>
</g>
<g id="a_edge17-label"><a xlink:title="net/http.(*Client).do -> net/http.(*Client).send (1.83s)">
<text text-anchor="middle" x="291" y="-627.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N29 -->
<g id="node29" class="node">
<title>N29</title>
<g id="a_node29"><a xlink:title="net/http.send (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-502.5 234,-502.5 234,-466.5 314,-466.5 314,-502.5"></polygon>
<text text-anchor="middle" x="274" y="-491.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-482.6" font-family="Times,serif" font-size="8.00" fill="#000000">send</text>
<text text-anchor="middle" x="274" y="-473.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N20&#45;&gt;N29 -->
<g id="edge18" class="edge">
<title>N20-&gt;N29</title>
<g id="a_edge18"><a xlink:title="net/http.(*Client).send -> net/http.send (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-561.9177C274,-547.4041 274,-528.2841 274,-512.677"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-512.5331 274,-502.5331 270.5001,-512.5331 277.5001,-512.5331"></polygon>
</a>
</g>
<g id="a_edge18-label"><a xlink:title="net/http.(*Client).send -> net/http.send (1.83s)">
<text text-anchor="middle" x="291" y="-532.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N21 -->
<g id="node21" class="node">
<title>N21</title>
<g id="a_node21"><a xlink:title="net/http.(*Transport).RoundTrip (1.83s)">
<polygon fill="#edddd5" stroke="#b23f00" points="314,-389 234,-389 234,-345 314,-345 314,-389"></polygon>
<text text-anchor="middle" x="274" y="-378.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="274" y="-369.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Transport)</text>
<text text-anchor="middle" x="274" y="-360.6" font-family="Times,serif" font-size="8.00" fill="#000000">RoundTrip</text>
<text text-anchor="middle" x="274" y="-351.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.83s (23.37%)</text>
</a>
</g>
</g>
<!-- N21&#45;&gt;N12 -->
<g id="edge19" class="edge">
<title>N21-&gt;N12</title>
<g id="a_edge19"><a xlink:title="net/http.(*Transport).RoundTrip -> net/http.(*Transport).roundTrip (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-344.9442C274,-328.2222 274,-304.9909 274,-286.194"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-286.0001 274,-276.0002 270.5001,-286.0002 277.5001,-286.0001"></polygon>
</a>
</g>
<g id="a_edge19-label"><a xlink:title="net/http.(*Transport).RoundTrip -> net/http.(*Transport).roundTrip (1.83s)">
<text text-anchor="middle" x="291" y="-297.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N23 -->
<g id="node23" class="node">
<title>N23</title>
<g id="a_node23"><a xlink:title="net/http.(*bodyEOFSignal).condfn (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="424.5,-606 347.5,-606 347.5,-562 424.5,-562 424.5,-606"></polygon>
<text text-anchor="middle" x="386" y="-595.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="386" y="-586.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*bodyEOFSignal)</text>
<text text-anchor="middle" x="386" y="-577.6" font-family="Times,serif" font-size="8.00" fill="#000000">condfn</text>
<text text-anchor="middle" x="386" y="-568.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N22&#45;&gt;N23 -->
<g id="edge29" class="edge">
<title>N22-&gt;N23</title>
<g id="a_edge29"><a xlink:title="net/http.(*bodyEOFSignal).Read -> net/http.(*bodyEOFSignal).condfn (0.11s)">
<path fill="none" stroke="#b2aea5" d="M374.479,-656.9663C376.3775,-644.9427 378.764,-629.8283 380.8718,-616.4785"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="384.3854,-616.6668 382.4879,-606.2433 377.471,-615.575 384.3854,-616.6668"></polygon>
</a>
</g>
<g id="a_edge29-label"><a xlink:title="net/http.(*bodyEOFSignal).Read -> net/http.(*bodyEOFSignal).condfn (0.11s)">
<text text-anchor="middle" x="397" y="-627.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N25 -->
<g id="node25" class="node">
<title>N25</title>
<g id="a_node25"><a xlink:title="net/http.(*persistConn).readLoop.func4 (0.11s)">
<polygon fill="#edeceb" stroke="#b2aea5" points="440,-511 364,-511 364,-458 440,-458 440,-511"></polygon>
<text text-anchor="middle" x="402" y="-500.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="402" y="-491.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*persistConn)</text>
<text text-anchor="middle" x="402" y="-482.6" font-family="Times,serif" font-size="8.00" fill="#000000">readLoop</text>
<text text-anchor="middle" x="402" y="-473.6" font-family="Times,serif" font-size="8.00" fill="#000000">func4</text>
<text text-anchor="middle" x="402" y="-464.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.11s (1.46%)</text>
</a>
</g>
</g>
<!-- N23&#45;&gt;N25 -->
<g id="edge30" class="edge">
<title>N23-&gt;N25</title>
<g id="a_edge30"><a xlink:title="net/http.(*bodyEOFSignal).condfn -> net/http.(*persistConn).readLoop.func4 (0.11s)">
<path fill="none" stroke="#b2aea5" d="M389.5509,-561.9177C391.4731,-549.9644 393.8976,-534.8865 396.0953,-521.2199"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="399.5874,-521.5481 397.7195,-511.1192 392.6762,-520.4367 399.5874,-521.5481"></polygon>
</a>
</g>
<g id="a_edge30-label"><a xlink:title="net/http.(*bodyEOFSignal).condfn -> net/http.(*persistConn).readLoop.func4 (0.11s)">
<text text-anchor="middle" x="412" y="-532.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N24 -->
<g id="node24" class="node">
<title>N24</title>
<g id="a_node24"><a xlink:title="net/http.(*connReader).abortPendingRead (0.04s)">
<polygon fill="#ededec" stroke="#b2b1ad" points="602,-1469 526,-1469 526,-1425 602,-1425 602,-1469"></polygon>
<text text-anchor="middle" x="564" y="-1458.6" font-family="Times,serif" font-size="8.00" fill="#000000">http</text>
<text text-anchor="middle" x="564" y="-1449.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*connReader)</text>
<text text-anchor="middle" x="564" y="-1440.6" font-family="Times,serif" font-size="8.00" fill="#000000">abortPendingRead</text>
<text text-anchor="middle" x="564" y="-1431.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.04s (0.57%)</text>
</a>
</g>
</g>
<!-- N24&#45;&gt;N9 -->
<g id="edge34" class="edge">
<title>N24-&gt;N9</title>
<g id="a_edge34"><a xlink:title="net/http.(*connReader).abortPendingRead -> sync.(*Cond).Wait (0.04s)">
<path fill="none" stroke="#b2b1ad" d="M564,-1424.5354C564,-1412.7414 564,-1397.9936 564,-1384.6003"></path>
<polygon fill="#b2b1ad" stroke="#b2b1ad" points="567.5001,-1384.2317 564,-1374.2317 560.5001,-1384.2318 567.5001,-1384.2317"></polygon>
</a>
</g>
<g id="a_edge34-label"><a xlink:title="net/http.(*connReader).abortPendingRead -> sync.(*Cond).Wait (0.04s)">
<text text-anchor="middle" x="581" y="-1395.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.04s</text>
</a>
</g>
</g>
<!-- N25&#45;&gt;N3 -->
<g id="edge31" class="edge">
<title>N25-&gt;N3</title>
<g id="a_edge31"><a xlink:title="net/http.(*persistConn).readLoop.func4 -> runtime.chanrecv1 (0.11s)">
<path fill="none" stroke="#b2aea5" d="M407.6455,-457.835C410.4989,-447.2388 414.5394,-435.175 420,-425 421.7403,-421.7573 423.6784,-418.5263 425.7497,-415.3436"></path>
<polygon fill="#b2aea5" stroke="#b2aea5" points="428.6744,-417.2682 431.4997,-407.057 422.9232,-413.2776 428.6744,-417.2682"></polygon>
</a>
</g>
<g id="a_edge31-label"><a xlink:title="net/http.(*persistConn).readLoop.func4 -> runtime.chanrecv1 (0.11s)">
<text text-anchor="middle" x="437" y="-428.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
</a>
</g>
</g>
<!-- N26&#45;&gt;N1 -->
<g id="edge24" class="edge">
<title>N26-&gt;N1</title>
<g id="a_edge24"><a xlink:title="net/http.(*persistConn).roundTrip -> runtime.selectgo (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M237.2515,-136.7857C220.6768,-126.7674 200.8542,-114.7871 183,-104 176.2813,-99.9407 169.3109,-95.7299 162.3148,-91.5038"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="163.8256,-88.3274 153.4563,-86.1529 160.2063,-94.3192 163.8256,-88.3274"></polygon>
</a>
</g>
<g id="a_edge24-label"><a xlink:title="net/http.(*persistConn).roundTrip -> runtime.selectgo (1.83s)">
<text text-anchor="middle" x="223" y="-107.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N27&#45;&gt;N24 -->
<g id="edge35" class="edge">
<title>N27-&gt;N24</title>
<g id="a_edge35"><a xlink:title="net/http.(*response).finishRequest -> net/http.(*connReader).abortPendingRead (0.04s)">
<path fill="none" stroke="#b2b1ad" d="M564,-1519.9663C564,-1507.9427 564,-1492.8283 564,-1479.4785"></path>
<polygon fill="#b2b1ad" stroke="#b2b1ad" points="567.5001,-1479.2433 564,-1469.2433 560.5001,-1479.2433 567.5001,-1479.2433"></polygon>
</a>
</g>
<g id="a_edge35-label"><a xlink:title="net/http.(*response).finishRequest -> net/http.(*connReader).abortPendingRead (0.04s)">
<text text-anchor="middle" x="581" y="-1490.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.04s</text>
</a>
</g>
</g>
<!-- N28&#45;&gt;N18 -->
<g id="edge20" class="edge">
<title>N28-&gt;N18</title>
<g id="a_edge20"><a xlink:title="net/http.Get -> net/http.(*Client).Get (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274.8072,-945.683C274.6757,-933.1876 274.4978,-916.2931 274.3425,-901.542"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.8396,-901.2359 274.2345,-891.2733 270.84,-901.3096 277.8396,-901.2359"></polygon>
</a>
</g>
<g id="a_edge20-label"><a xlink:title="net/http.Get -> net/http.(*Client).Get (1.83s)">
<text text-anchor="middle" x="292" y="-912.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N29&#45;&gt;N21 -->
<g id="edge21" class="edge">
<title>N29-&gt;N21</title>
<g id="a_edge21"><a xlink:title="net/http.send -> net/http.(*Transport).RoundTrip (1.83s)">
<path fill="none" stroke="#b23f00" stroke-width="2" d="M274,-466.3981C274,-448.5845 274,-420.9917 274,-399.3937"></path>
<polygon fill="#b23f00" stroke="#b23f00" stroke-width="2" points="277.5001,-399.3417 274,-389.3417 270.5001,-399.3417 277.5001,-399.3417"></polygon>
</a>
</g>
<g id="a_edge21-label"><a xlink:title="net/http.send -> net/http.(*Transport).RoundTrip (1.83s)">
<text text-anchor="middle" x="291" y="-428.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.83s</text>
</a>
</g>
</g>
<!-- N33 -->
<g id="node33" class="node">
<title>N33</title>
<g id="a_node33"><a xlink:title="testing.(*M).Run (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1370 428,-1370 428,-1326 508,-1326 508,-1370"></polygon>
<text text-anchor="middle" x="468" y="-1359.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-1350.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*M)</text>
<text text-anchor="middle" x="468" y="-1341.6" font-family="Times,serif" font-size="8.00" fill="#000000">Run</text>
<text text-anchor="middle" x="468" y="-1332.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N30&#45;&gt;N33 -->
<g id="edge2" class="edge">
<title>N30-&gt;N33</title>
<g id="a_edge2"><a xlink:title="net/http_test.TestMain -> testing.(*M).Run (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1428.8419C468,-1415.2878 468,-1396.4077 468,-1380.3016"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1380.2021 468,-1370.2021 464.5001,-1380.2022 471.5001,-1380.2021"></polygon>
</a>
</g>
<g id="a_edge2-label"><a xlink:title="net/http_test.TestMain -> testing.(*M).Run (3.11s)">
<text text-anchor="middle" x="485" y="-1395.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N31 -->
<g id="node31" class="node">
<title>N31</title>
<g id="a_node31"><a xlink:title="testing.(*B).doBench (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-701 428,-701 428,-657 508,-657 508,-701"></polygon>
<text text-anchor="middle" x="468" y="-690.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-681.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*B)</text>
<text text-anchor="middle" x="468" y="-672.6" font-family="Times,serif" font-size="8.00" fill="#000000">doBench</text>
<text text-anchor="middle" x="468" y="-663.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.77%)</text>
</a>
</g>
</g>
<!-- N31&#45;&gt;N3 -->
<g id="edge9" class="edge">
<title>N31-&gt;N3</title>
<g id="a_edge9"><a xlink:title="testing.(*B).doBench -> runtime.chanrecv1 (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-656.5617C468,-606.79 468,-486.1141 468,-417.2858"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-417.1125 468,-407.1125 464.5001,-417.1125 471.5001,-417.1125"></polygon>
</a>
</g>
<g id="a_edge9-label"><a xlink:title="testing.(*B).doBench -> runtime.chanrecv1 (3.11s)">
<text text-anchor="middle" x="485" y="-532.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N34 -->
<g id="node34" class="node">
<title>N34</title>
<g id="a_node34"><a xlink:title="testing.(*benchContext).processBench (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-796 428,-796 428,-752 508,-752 508,-796"></polygon>
<text text-anchor="middle" x="468" y="-785.6" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-776.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*benchContext)</text>
<text text-anchor="middle" x="468" y="-767.6" font-family="Times,serif" font-size="8.00" fill="#000000">processBench</text>
<text text-anchor="middle" x="468" y="-758.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.77%)</text>
</a>
</g>
</g>
<!-- N32&#45;&gt;N34 -->
<g id="edge10" class="edge">
<title>N32-&gt;N34</title>
<g id="a_edge10"><a xlink:title="testing.(*B).run -> testing.(*benchContext).processBench (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-846.9663C468,-834.9427 468,-819.8283 468,-806.4785"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-806.2433 468,-796.2433 464.5001,-806.2433 471.5001,-806.2433"></polygon>
</a>
</g>
<g id="a_edge10-label"><a xlink:title="testing.(*B).run -> testing.(*benchContext).processBench (3.11s)">
<text text-anchor="middle" x="485" y="-817.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N35 -->
<g id="node35" class="node">
<title>N35</title>
<g id="a_node35"><a xlink:title="testing.runBenchmarks (3.11s)">
<polygon fill="#eddbd5" stroke="#b22b00" points="508,-1267 428,-1267 428,-1231 508,-1231 508,-1267"></polygon>
<text text-anchor="middle" x="468" y="-1256.1" font-family="Times,serif" font-size="8.00" fill="#000000">testing</text>
<text text-anchor="middle" x="468" y="-1247.1" font-family="Times,serif" font-size="8.00" fill="#000000">runBenchmarks</text>
<text text-anchor="middle" x="468" y="-1238.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 3.11s (39.80%)</text>
</a>
</g>
</g>
<!-- N33&#45;&gt;N35 -->
<g id="edge5" class="edge">
<title>N33-&gt;N35</title>
<g id="a_edge5"><a xlink:title="testing.(*M).Run -> testing.runBenchmarks (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1325.5354C468,-1311.3043 468,-1292.7726 468,-1277.5091"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1277.0807 468,-1267.0808 464.5001,-1277.0808 471.5001,-1277.0807"></polygon>
</a>
</g>
<g id="a_edge5-label"><a xlink:title="testing.(*M).Run -> testing.runBenchmarks (3.11s)">
<text text-anchor="middle" x="485" y="-1292.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N34&#45;&gt;N31 -->
<g id="edge11" class="edge">
<title>N34-&gt;N31</title>
<g id="a_edge11"><a xlink:title="testing.(*benchContext).processBench -> testing.(*B).doBench (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-751.9663C468,-739.9427 468,-724.8283 468,-711.4785"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-711.2433 468,-701.2433 464.5001,-711.2433 471.5001,-711.2433"></polygon>
</a>
</g>
<g id="a_edge11-label"><a xlink:title="testing.(*benchContext).processBench -> testing.(*B).doBench (3.11s)">
<text text-anchor="middle" x="485" y="-722.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N35&#45;&gt;N2 -->
<g id="edge6" class="edge">
<title>N35-&gt;N2</title>
<g id="a_edge6"><a xlink:title="testing.runBenchmarks -> testing.(*B).runN (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1230.683C468,-1218.1876 468,-1201.2931 468,-1186.542"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-1186.2732 468,-1176.2733 464.5001,-1186.2733 471.5001,-1186.2732"></polygon>
</a>
</g>
<g id="a_edge6-label"><a xlink:title="testing.runBenchmarks -> testing.(*B).runN (3.11s)">
<text text-anchor="middle" x="485" y="-1197.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
<!-- N36&#45;&gt;N11 -->
<g id="edge7" class="edge">
<title>N36-&gt;N11</title>
<g id="a_edge7"><a xlink:title="testing.runBenchmarks.func1 -> testing.(*B).Run (3.11s)">
<path fill="none" stroke="#b22b00" stroke-width="2" d="M468,-1036.9663C468,-1024.9427 468,-1009.8283 468,-996.4785"></path>
<polygon fill="#b22b00" stroke="#b22b00" stroke-width="2" points="471.5001,-996.2433 468,-986.2433 464.5001,-996.2433 471.5001,-996.2433"></polygon>
</a>
</g>
<g id="a_edge7-label"><a xlink:title="testing.runBenchmarks.func1 -> testing.(*B).Run (3.11s)">
<text text-anchor="middle" x="485" y="-1007.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 3.11s</text>
</a>
</g>
</g>
</g>
</g></svg>
</div>

#### 3.5.7 쓰레드 생성 프로파일링

Go 1.11(?)은 운영체제 쓰레드 작성 프로파일링에 대한 지원이 추가되었습니다.

> 쓰레드 작성 프로파일 링을 godoc에 추가하고 프로파일링 결과인 `godoc -http=:8080 -index`를 살펴보세요.

#### 3.5.8 프레임 포인터

Go 1.7이 출시되었으며 amd64용 새 컴파일러와 함께 컴파일러는 기본적으로 프레임 포인터를 활성화합니다.

프레임 포인터는 항상 현재 스택 프레임의 상단을 가리키는 레지스터입니다.

프레임 포인터를 사용하면 `gdb(1)` 및 `perf(1)`와 같은 도구가 Go 호출 스택을 이해할 수 있습니다.

이 워크샵에서는 이러한 도구를 다루지 않지만 Go 프로그램을 프로파일 링하는 7가지 방법에 대해 발표한 프레젠테이션을 읽고 볼 수 있습니다.

- [Go 프로그램을 프로파일링하는 7가지 방법](https://talks.godoc.org/github.com/davecheney/presentations/seven.slide) (슬라이드)
- [Go 프로그램을 프로파일링하는 7가지 방법](https://www.youtube.com/watch?v=2h_NFBFrciI) (비디오, 30분)
- [Go 프로그램을 프로파일링하는 7가지 방법](https://www.bigmarker.com/remote-meetup-go/Seven-ways-to-profile-a-Go-program) (웹캐스트, 60분)

#### 3.5.9 연습문제

잘 알고 있는 코드 조각에서 프로파일을 생성합니다. 코드 샘플이 없으면 `godoc`을 프로파일링해 봅니다.

```sh
% go get golang.org/x/tools/cmd/godoc
% cd $GOPATH/src/golang.org/x/tools/cmd/godoc
% vim main.go
```
	
한 머신에서 프로필을 생성하고 다른 머신에서 검사하려면 어떻게 해야할까요?

## 4. 컴파일러 최적화

이 섹션에서는 Go 컴파일러가 수행하는 최적화 중 일부를 다룹니다. 

예를 들면 다음과 같습니다.

- 이스케이프 분석
- 인라인
- 데드 코드 제거

코드는 여전히 AST 형식이지만, 컴파일러의 프론트 엔드에서 모두 처리됩니다. 그런 다음 코드는 SSA 컴파일러로 전달되어 더욱 최적화됩니다.

### 4.1 Go 컴파일러의 역사

Go 컴파일러는 2007년경 Plan9 컴파일러 툴 체인의 포크로 시작했습니다. 당시 컴파일러는 아호(Aho)나 울만(Ullman)의 [드래곤북(Principles of Compiler Design)](https://www.goodreads.com/book/show/112269.Principles_of_Compiler_Design)과 매우 닯아 있습니다.

2015년 당시 Go 1.5 컴파일러는 기계적으로 [C에서 Go](https://golang.org/doc/go1.5#c)로 변환되었습니다.

1 년 후, Go 1.7은 [SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form) 기술을 기반으로 한 [새로운 컴파일러 백엔드](https://blog.golang.org/go1.7)를 도입하여 이전의 Plan 9 스타일 코드 생성을 대체했습니다. 이 새로운 백엔드는 일반 및 아키텍처별 최적화에 대한 많은 기회를 제공했습니다.

### 4.2 이스케이프 분석 (Escape analysis)

우리가 논의하고자 하는 첫 번째 최적화는 이스케이프 분석입니다.

이스케이프 분석이 무엇을 하는 것인지 보여 주려면, Go 스펙에 힙이나 스택이 언급되어 있지 않음을 상기하셔야 합니다. 도입부에서 언어가 가비지 콜렉트된다는 것을 언급할 뿐, 이것이 어떻게 이루어지는지에 대한 힌트는 주지 않습니다.

Go 스펙을 준수하는 Go 구현은 모든 할당을 힙에 저장할 수 있습니다. 그렇게 되면 가비지 컬렉터에 큰 부담이 되겠지만, 잘못된 것은 아닙니다. 몇 년 동안, gccgo는 이스케이프 분석에 대한 지원이 매우 제한적이었으므로 힙 모드에서 효과적으로 작동하는 것으로 간주될 수 있습니다.

그러나 고루틴의 스택은 지역 변수를 저장하는데 값싼 장소입니다. 왜냐하면 스택에 대해서 가비지 콜렉션할 필요가 없기 때문입니다. 따라서 스택에 저장하는 것이 안전한 경우, 스택에 할당하는 것이 더 효율적입니다.

일부 언어, 예를 들면 C 및 C++에서, 스택 또는 힙 할당을 선택하는 것은 프로그래머의 수작업이었습니다. 힙 할당은 `malloc`과 `free`로 이루어지고 무료이며 스택 할당은 `alloca`를 통해 이루어집니다. 이러한 메커니즘을 사용해서 발생하는 실수가 메모리 손상 버그의 흔한 원인입니다.

Go에서는, 컴파일러가 함수 호출의 라이프 사이클보다 오래 존재하는 값은 힙에 자동적으로 이동시킵니다. 이를 값이 힙으로 빠져나갔다(이스케이프되었다)고 말합니다.

```golang
type Foo struct {
	a, b, c, d int
}

func NewFoo() *Foo {
	return &Foo{a: 3, b: 1, c: 4, d: 7}
}
```

이 예제에서 `NewFoo`에 할당된 `Foo`는 힙으로 이동하여 `NewFoo`가 반환된 후에도 내용이 유효하게 유지됩니다.

이것은 Go의 초기 시절부터 존재했습니다. 자동 수정 기능만큼의 최적화는 아닙니다. Go에서 실수로 스택에 할당된 변수의 주소를 반환할 수 없습니다.

그러나 컴파일러는 그 반대로 할 수도 있습니다. 힙에 할당된 것으로 추정되는 것을 찾아 스택으로 옮길 수 있습니다.

예제를 살펴보겠습니다.

```golang
func Sum() int {
	const count = 100
	numbers := make([]int, count)
	for i := range numbers {
		numbers[i] = i + 1
	}

	var sum int
	for _, i := range numbers {
		sum += i
	}
	return sum
}

func main() {
	answer := Sum()
	fmt.Println(answer)
}
```

Sum은 1과 100 사이에 `int`를 더하고 결과를 반환합니다.

숫자 slice는 Sum 내에서만 참조되므로, 컴파일러는 해당 슬라이스에 대한 100개의 정수를 힙이 아닌 스택에 저장합니다. 가비지 콜렉트할 `numbers`가 필요 없습니다. `Sum`이 반환될 때 자동으로 해제됩니다.

#### 4.2.1 증명하기
컴파일러 이스케이프 분석 디시즌을 출력하려면, `-m` 플래그를 사용합니다.

```sh
% go build -gcflags=-m examples/esc/sum.go
# command-line-arguments
examples/esc/sum.go:22:13: inlining call to fmt.Println
examples/esc/sum.go:8:17: Sum make([]int, count) does not escape
examples/esc/sum.go:22:13: answer escapes to heap
examples/esc/sum.go:22:13: io.Writer(os.Stdout) escapes to heap
examples/esc/sum.go:22:13: main []interface {} literal does not escape
<autogenerated>:1: os.(*File).close .this does not escape
```

8행은 컴파일러가 `make ([] int, 100)`의 결과가 힙으로 이스케이프되지 않음을 올바르게 추론했음을 보여줍니다. 

22번 째 줄은 `answer`가 힙으로 이스케이프되었다고 보고하는데 `fmt.Println`은 가변적(variadic) 함수입니다. 가변적 함수에 대한 매개 변수는 Slice로 박싱됩니다, 이 경우에는 `[]interface {}`입니다. 그래서 `answer`는 인터페이스 값으로 배치됩니다. 왜냐하면 `fmt.Println` 호출에 의해 참조되기 때문입니다. Go 1.6부터 가비지 콜렉터는 인터페이스를 통해 전달된 모든 값이 포인터가 되어야하므로 컴파일러가 보는 관점은 대략 다음과 같습니다.

```golang
var answer = Sum()
fmt.Println([]interface{&answer}...)
```

이는 다음과 같이= `-gcflags="-m -m"` 플래그를 사용해서 확인할 수 있습니다. 

```sh
% go build -gcflags='-m -m' examples/esc/sum.go 2>&1 | grep sum.go:22
examples/esc/sum.go:22:13: inlining call to fmt.Println func(...interface {}) (int, error) { return fmt.Fprintln(io.Writer(os.Stdout), fmt.a...) }
examples/esc/sum.go:22:13: answer escapes to heap
examples/esc/sum.go:22:13:      from ~arg0 (assign-pair) at examples/esc/sum.go:22:13
examples/esc/sum.go:22:13: io.Writer(os.Stdout) escapes to heap
examples/esc/sum.go:22:13:      from io.Writer(os.Stdout) (passed to call[argument escapes]) at examples/esc/sum.go:22:13
examples/esc/sum.go:22:13: main []interface {} literal does not escape
```

요약하자면, 22번 째 줄은 걱정하지 않아도 됩니다. 이번 논의에서 중요하지 않습니다.

#### 4.2.2 연습문제

- 이 최적화가 모든 `count` 값에 참일까요?
- `count`가 상수가 아닌 변수인 경우, 이 최적화가 참일까요?
- `count`가 `Sum`의 매개변수인 경우, 이 최적화가 참일까요?

#### 4.2.3 이스케이프 분석 (계속)

이 예제는 약간 인위적인 예제입니다. 실제 코드가 아니라 예제일 뿐입니다.

```golang
type Point struct{ X, Y int }

const Width = 640
const Height = 480

func Center(p *Point) {
	p.X = Width / 2
	p.Y = Height / 2
}

func NewPoint() {
	p := new(Point)
	Center(p)
	fmt.Println(p.X, p.Y)
}
```

`NewPoint`는 새 `*Point` 값 `p`를 생성합니다. `p`를 스크린 중앙의 좌표로 이동시키는 `Center` 함수로 전달합니다. 마지막으로 `p.X` 와 `p.Y`를 출력합니다.


```sh
% go build -gcflags=-m examples/esc/center.go
# command-line-arguments
examples/esc/center.go:11:6: can inline Center
examples/esc/center.go:18:8: inlining call to Center
examples/esc/center.go:19:13: inlining call to fmt.Println
examples/esc/center.go:11:13: Center p does not escape
examples/esc/center.go:19:15: p.X escapes to heap
examples/esc/center.go:19:20: p.Y escapes to heap
examples/esc/center.go:19:13: io.Writer(os.Stdout) escapes to heap
examples/esc/center.go:17:10: NewPoint new(Point) does not escape
examples/esc/center.go:19:13: NewPoint []interface {} literal does not escape
<autogenerated>:1: os.(*File).close .this does not escape
```

`p`가 `new` 함수로 할당 되었더라도, 참조가 없는 `p`가 `Center` 함수를 이스케이프하지 않기 때문에, `p`가 힙에 저장되지 않습니다.

질문: 19행은 어떤가요? `p`가 이스케이프되지 않으면, 힙으로 이스케이프되는 건 무엇일까요?

> `Sum`이 할당되지 않도록 벤치마크를 작성해보세요

### 4.3 인라이닝

Go 함수에서 호출은 고정된 오버헤드를 가지는데, 스택 및 선점(preemption)을 체크합니다.

이 중 일부는 하드웨어 분기 예측 변수에 의해 개선되지만, 함수 크기와 클럭 주기에 있어서는 여전히 비용이 듭니다.

인라이닝은 이러한 비용을 피하는 고전적인 최적화입니다.

Go 1.11 인라이닝이 오직 리프 함수에만 작용하기 전까지는, 함수는 다른 함수를 호출하지 않습니다. 이에 대한 정당성은 다음과 같습니다.

만약 함수가 많은 일을 한다면, 프리앰블 오버헤드는 무시할 수 있습니다. 이는 함수가 특정 크기를 초과하는 이유입니다. (현재 일부 명령어 카운트와 추가로 인라이닝을 방지하는 몇 가지 작업을 합쳐서입니다. (예: Go 1.7 이전 스위치))

반면에 작은 함수는 상대적으로 적은 양의 유용한 작업을 수행하기 위해 고정된 오버헤드를 지닙니다. 가장 많은 혜택을 받기 때문에 이들이 인라이닝 타겟이 되는 함수들입니다.

또 다른 이유는 인라인이 많으면, 스택 트레이싱을 따라가기가 어렵습니다.

#### 4.3.1 인라이닝 (예제)

```golang
func Max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

func F() {
	const a, b = 100, 20
	if Max(a, b) == b {
		panic(b)
	}
}
```

다시 `-gcflags = -m` 플래그를 사용하여 컴파일러 최적화 디시즌을 살펴봅시다.

```sh
% go build -gcflags=-m examples/inl/max.go
# command-line-arguments
examples/inl/max.go:4:6: can inline Max
examples/inl/max.go:11:6: can inline F
examples/inl/max.go:13:8: inlining call to Max
examples/inl/max.go:20:6: can inline main
examples/inl/max.go:21:3: inlining call to F
examples/inl/max.go:21:3: inlining call to Max
```

컴파일러는 두 줄을 출력했습니다.

- 첫 번째는 3행에서, `Max` 선언은, 인라인될 수 있음을 알려줍니다.
- 두 번째는 `Max`의 본문이 12행에서 호출자에게 인라인되었다고 보고합니다.

> `//go:noinline` 주석을 사용하지 않고, `Max`를 다시 작성하여 여전히 올바른 답을 리턴하지만, 더 이상 컴파일러가 인라인될 수 있다고 고려하지 않도록 해봅시다.

### 4.3.2 인라인은 어떻게 생겼나요?

`max.go`를 컴파일하고 `F()`의 최적화된 버전을 봅시다.

```sh
% go build -gcflags=-S examples/inl/max.go 2>&1 | grep -A5 '"".F STEXT'
"".F STEXT nosplit size=2 args=0x0 locals=0x0
        0x0000 00000 (/Users/dfc/devel/high-performance-go-workshop/examples/inl/max.go:11)     TEXT    "".F(SB), NOSPLIT|ABIInternal, $0-0
        0x0000 00000 (/Users/dfc/devel/high-performance-go-workshop/examples/inl/max.go:11)     FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0000 00000 (/Users/dfc/devel/high-performance-go-workshop/examples/inl/max.go:11)     FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0000 00000 (/Users/dfc/devel/high-performance-go-workshop/examples/inl/max.go:11)     FUNCDATA        $3, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0000 00000 (/Users/dfc/devel/high-performance-go-workshop/examples/inl/max.go:13)     PCDATA  $2, $0
```

이는 Max가 인라인되었을 때 F의 바디입니다. 이 함수에는 아무 일도 일어나지 않습니다. 쓸데없이 화면에 텍스트가 많이 나오는 건 알고 있지만, 제 말을 믿으세요. 한가지 일어나는 일은 RET뿐입니다. 실제로 F는 다음과 같이 되었습니다.

```golang
func F() {
        return
}
```

{{< notice note >}}
FUNCDATA와 PCDATA가 무엇인가요?
`-S`의 출력은 바이너리로 들어가는 최종 기계코드가 아닙니다. 링커는 최종 링크단계에서 일부를 처리합니다. `FUNCDATA` 및 `PCDATA`와 같은 줄은 연결시 다른 곳으로 이동하는 가비지 콜렉터의 메타 데이터입니다. `-S`의 출력을 읽을 때 `FUNCDATA` 및 `PCDATA` 행은 그냥 무시하세요. 그들은 최종 바이너리의 일부가 아니니깐요.
{{< /notice >}}

#### 4.3.3 논의

`F()`에서 왜 `a`와 `b`를 상수로 선언했을까요?

> `a`와 `b`가 변수로 선언되면 어떻게 될까요? a와 b가 매개변수로 F()에 전달되면 어떻게 될까요?

{{< notice note >}}
`-gcflags = -S`는 작업 디렉토리에 최종 바이너리가 빌드되는 것을 막지 않습니다. 이후의 `go build..` 실행으로 출력이 생성되지 않으면 작업 디렉토리에서 `./max` 바이너리를 삭제하세요.
{{< /notice >}}

#### 4.3.4 인라이닝 레벨 조정

인라인 레벨은 `-gcflags = -l` 플래그를 사용하여 조정됩니다. 다소 혼란스럽지만, 하나의 `-l`을 전달하면 인라인이 비활성화되고 더 공격적인 설정으로 둘 이상을 전달하면 인라인이 활성화됩니다.

- `-gcflags = -l`, 인라인 비활성화
- 아무것도 전달하지 않음, 일반적인 인라이닝
- `-gcflags = "-l -l"` 인라인 레벨 2, 더 공격적이며, 더 빨라질 수 있으며, 더 큰 바이너리를 만듭니다.
- `-gcflags = "-l -l -l"` 인라인 레벨 3, 좀 더 공격적이며, 확실히 더 큰 바이너리를 만들고, 좀 더 빠를 수 있지만 버그가 있을 수 있습니다.
- - Go 1.11의 `-gcflags = -l = 4` (4 개의 `-l`)는 실험적인 중간 스택 인라인 최적화를 가능하게 합니다.

#### 4.3.5 중간 스택 인라이닝

Go 1.12부터 중간 스택(mid stack)이라고 부르는 인라이닝이 활성화되었습니다 (이전에 Go 1.11 미리보기에서 `-gcflags='-l -l -l'` 플래그로 사용할 수 있었습니다.)

이전 예에서 중간 스택의 인라이닝 예제를 볼 수 있습니다. Go 1.11 및 이전 `F`는 리프 함수가 될 수 없었습니다. `max`를 호출했었죠. 그러나 개선된 `F` 인라이닝으로 인해 F는 이제 호출자에게 인라이닝됩니다. 이것은 두 가지 이유입니다. `max`가 `F`에 인라인딩되면, `F`는 다른 함수 호출을 포함하지 않으므로, 복잡성 예산(complexity budget)을 초과하지 않았다고 가정하며, 잠재적 리프 함수가 됩니다. 왜냐하면 F는 단순한 함수이기 때문에 `max` 호출에 관계없이 중간 스택 인라이닝에 적합합니다. 인라이닝과 데드 코드 제거로 복잡성 예산(complexity budget)의 많은 부분이 제거되었기 때문이죠.

{{< notice tip >}}
중간 스택 인라이닝을 사용하면 함수의 고속 경로를 인라인화하여, 고속 경로에서 함수 호출 오버헤드를 제거할 수 있습니다. [Go 1.13에 들어온 최근의 CL](https://go-review.googlesource.com/c/go/+/152698)은 `sync.RWMutex.Unlock()` 적용된 이 기술을 보여줍니다.
{{< /notice >}}

**더 읽을거리**

- [David Lazar의 Go 컴파일러 프레젠테이션의 중간 스택 인라인](https://docs.google.com/presentation/d/1Wcblp3jpfeKwA0Y4FOmj63PW52M_qmNqlQkNaLj0P5o/edit#slide=id.p)
- [Go 컴파일러에서 중간 스택 인라니닝](https://github.com/golang/proposal/blob/master/design/19348-midstack-inlining.md)

### 4.4 데드 코드 제거

`a`와 `b`가 상수라는 것이 왜 중요할까요?

어떤 일이 발생했는지 이해하기 위해 컴파일러에서 `Max`를 `F`로 인라인으로 설정한 후 무엇을 발생하는지 보겠습니다. 컴파일러로부터 이를 쉽게 얻을 수는 없지만, 손으로 직접하는 것이 좋습니다.

이전:

```golang
func Max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

func F() {
	const a, b = 100, 20
	if Max(a, b) == b {
		panic(b)
	}
}
```

이후:

```golang
func F() {
	const a, b = 100, 20
	var result int
	if a > b {
		result = a
	} else {
		result = b
	}
	if result == b {
		panic(b)
	}
}
```

`a`와 `b`는 상수이므로 컴파일시에 분기문이 절대 false가 아님을 입증합니다. `100`은 항상 `20`보다 크니깐요. 따라서 컴파일러는 F를 더욱 최적화할 수 있습니다.

```golang
func F() {
	const a, b = 100, 20
	var result int
	if true {
		result = a
	} else {
		result = b
	}
	if result == b {
		panic(b)
	}
}
```

이제 분기문 결과를 알게 되어 결과의 내용도 알게 됩니다. 이것을 분기문 제거(branch elimination)라고 부릅니다.

```golang
func F() {
        const a, b = 100, 20
        const result = a
        if result == b {
                panic(b)
        }
}
```

이제 분기문이 제거되어 결과가 항상 a와 같다는 것을 알 수 있으며 a가 상수이기 때문에 결과가 상수라는 것을 알 수 있습니다. 컴파일러는 이 증명을 두 번째 분기문에 적용합니다

```golang
func F() {
        const a, b = 100, 20
        const result = a
        if false {
                panic(b)
        }
}
```

그리고 다시 분기문 제거를 사용하여 최종 형태 F로 줄입니다.

```golang
func F() {
        const a, b = 100, 20
        const result = a
}
```

마지막으로 그냥 저렇게 됩니다.

```golang
func F() {
}
```

#### 4.4.1 데드 코드 제거 
분기 제거는 데드 코드 제거로 알려진 최적화 방법 중 하나입니다. 실제로 정적 증거(static proofs)를 사용하여 코드 조각에 도달할 수 없거나 죽은 것으로 알려져 있으므로 최종 바이너리에서 컴파일, 최적화 또는 생성할 필요가 없습니다.

데드 코드 제거가 인라인과 함께 작동하여 도달할 수 없는 것으로 입증된 루프와 분기문을 제거하여 생성된 코드의 양을 줄이는 방법을 보았습니다.

이를 활용하여 고가의 디버깅을 구현하고 숨길 수 있습니다

```golang
const debug = false
```

빌드 태그와 함께 사용하면 매우 유용해질 수 있습니다.

#### 4.4.2 더 읽을거리
- [Using // +build to switch between debug and release builds](http://dave.cheney.net/2014/09/28/using-build-to-switch-between-debug-and-release)
- [How to use conditional compilation with the go build tool](http://dave.cheney.net/2013/10/12/how-to-use-conditional-compilation-with-the-go-build-tool)

### 4.5 컴파일러 플래그 연습문제

컴파일러 플래그는 다음과 같이 주어집니다.

```sh
go build -gcflags=$FLAGS
```

아래 컴파일러 기능들의 동작을 살펴봅시다.

- `-S`는 컴파일할 패키지의 (Go Flavoured) 어셈블리를 출력합니다.
- `-l`은 인라이너의 동작을 제어합니다. `-l`은 인라이닝을 비활성화하고 `-l -l`은 인라이닝 코드에 대한 컴파일러의 식욕을 자극합니다. 컴파일 시간, 프로그램 크기 및 실행 시간의 차이를 실험할 수 있습니다.
- `-m`은 인라이닝, 이스케이프 분석과 같은 디시즌을 제어합니다. `-m-m`은 컴파일러가 생각하고 있는 것들을 자세히 출력합니다.
- `-l -N`은 모든 최적화를 비활성화합니다.

{{< notice note >}}
`go build...` 실행 결과로 출력이 생성되지 않으면, 작업 디렉토리에서 `./max` 바이너리를 삭제하세요.
{{< /notice >}}

#### 4.5.1 더 읽을거리
- [Codegen Inspection by Jaana Burcu Dogan](http://go-talks.appspot.com/github.com/rakyll/talks/gcinspect/talk.slide#1)

### 4.6 바운드 체크 제거

Go는 바운드 체크 언어입니다. 이는 배열 및 슬라이스 구독 연산이 해당 타입의 바운드 내에 있는지 체크한다는 것을 뜻합니다.

배열의 경우, 컴파일 타임에 수행합니다. 슬라이스의 경우, 런타임에만 수행됩니다.

```golang
var v = make([]int, 9)

var A, B, C, D, E, F, G, H, I int

func BenchmarkBoundsCheckInOrder(b *testing.B) {
	for n := 0; n < b.N; n++ {
		A = v[0]
		B = v[1]
		C = v[2]
		D = v[3]
		E = v[4]
		F = v[5]
		G = v[6]
		H = v[7]
		I = v[8]
	}
}
```

> `-gcflags = -S`를 사용해서 `BenchmarkBoundsCheckInOrder`를 디셈블보세요. 루프당 몇 개의 바운드 체크 작업이 수행되나요?

```golang
func BenchmarkBoundsCheckOutOfOrder(b *testing.B) {
	for n := 0; n < b.N; n++ {
		I = v[8]
		A = v[0]
		B = v[1]
		C = v[2]
		D = v[3]
		E = v[4]
		F = v[5]
		G = v[6]
		H = v[7]
	}
}
```

>  A부터 I까지의 순서를 재정렬하면 어셈블리에 영향을 주어야 합니다. `BenchmarkBoundsCheckOutOfOrder`를 디셈블하고 찾아보세요.

#### 4.6.1 연습문제
- 구독 연산의 순서를 재정렬하면 함수 크기에 영향을 주나요? 함수 속도에 영향을 주나요?
- `v`가 `Benchmark` 함수 내부로 이동하면 어떻게 될까요?
- `v`가 `var v[9] int?` 배열로 선언되면 어떻게 될까요? 

## 5. 엑스큐션 트레이서

엑스큐션 트레이서는 Go 1.5때 [Dmitry Vyukov](https://github.com/dvyukov)에 의해 개발되었으며, 몇 년 동안 문서화 및 활용률이 낮았습니다.

샘플 기반 프로파일링과 달리 엑스큐션 트레이서는 Go 런타임에 통합되므로 특정 시점에서 Go 프로그램이 무엇을 하고 있는지 알 수 있을 뿐 아니라 그 이유도 알 수 있습니다.

### 5.1 엑스큐션 트레이서란 무엇인가, 왜 필요한가

저는 엑스큐션 트레이서가 무엇을 하는지 그리고 왜 중요한지를 설명하는 가장 쉬운 방법은 pprof, 즉 `go tool pprof`가 잘 수행되지 않는 코드 조각을 살펴보는 것이라고 생각합니다.

`examples/mandelbrot` 디렉토리에는 간단한 만델브로트 생성기가 들어 있습니다. 이 코드는 [프란체스 캄포이 저장소](https://github.com/campoy/mandelbrot)에서 따왔습니다.

```sh
cd examples/mandelbrot
go build && ./mandelbrot
```
![](https://dave.cheney.net/high-performance-go-workshop/images/mandelbrot.png)

### 5.1.1 얼마나 오래 걸리는가

자, 1024 x 1024 픽셀의 이미지를 생성하는데 얼마나 오래 걸릴까요? 가장 간단한 방법은 `time(1)`과 같은 것들을 사용하는 것입니다. 

```sh
% time ./mandelbrot
real    0m1.654s
user    0m1.630s
sys     0m0.015s
```

{{< notice note >}}
`time go run man`을 사용하지 마세요. 그렇지 않으면 프로그램을 컴파일하고 실행하는데 걸리는 시간이 길어집니다.
{{< /notice >}}

### 5.1.2 프로그램은 무엇을 하고 있을까요

따라서, 이 예제에서 프로그램은 만델브로트를 생성하고 png파일에 쓰는 데 1.6초가 걸렸습니다.

괜찮나요? 더 빨리 만들 수 있을까요?

질문에 대한 한 가지 답변은 Go 빌트인 pprof 지원 기능을 사용하여 프로그램을 프로파일링하는 것이 될 수 있습니다.

한번 해봅시다.

## 5.2 프로파일링 생성

프로파일을 생성하려면 다음 중 하나를 선택해야 합니다.

- `runtime/pprof` 패키지를 직접 사용합니다.
- `github.com/pkg/profile`과 같은 래퍼를 사용하여 자동화합니다.

## 5.3 runtime/pprof 사용한 프로파일 생성

마술이 없다는 것을 보여주기 위해 `os.Stdout`에 CPU 프로파일링을 쓰도록 프로그램을 수정해 봅시다.

```golang
import "runtime/pprof"

func main() {
	pprof.StartCPUProfile(os.Stdout)
	defer pprof.StopCPUProfile()
```

이 코드를 main 함수의 맨 위에 추가하면 프로그램은 `os.Stdout`에 프로파일을 작성합니다.

```sh
cd examples/mandelbrot-runtime-pprof
go run mandelbrot.go > cpu.pprof
```

{{< notice note >}}
CPU 프로파일에는 컴파일이 아닌 `mandelbrot.go`의 실행만 포함되므로 이 경우 `go run`을 사용할 수 있습니다.
{{< /notice >}}

#### 5.3.1 github.com/pkg/profile 사용한 프로파일 생성

이전 슬라이드는 프로파일을 생성하는 매우 값싼 방법을 보여 주었지만, 몇 가지 문제가 있습니다.

- 출력을 파일로 리디렉션하는 것을 잊어버린 경우 해당 터미널 세션을 날리게 됩니다. 😞 (힌트 : `reset (1)`은 여러분의 친구입니다)
- `os.Stdout`에 다른 것을 쓰게 되면(예 : fmt.Println), 트레이싱에 오류가 생깁니다.

`runtime/pprof`을 사용하는 권장 방법은 [트레이싱을 파일에 쓰는](https://godoc.org/runtime/pprof#hdr-Profiling_a_Go_program) 것입니다. 하지만, 누군가가 ^C으로 프로그램을 중단시켰다면, 트레이싱이 멈춰 있는지, 그리고 프로그램이 중단되기 전에 파일이 닫혀 있는지 확인해야 합니다. 

그래서, 몇 년 전에 저는 이를 관리하기 위해 [패키지](https://godoc.org/github.gom/pkg/profile)를 작성했습니다.

```golang
import "github.com/pkg/profile"

func main() {
	defer profile.Start(profile.CPUProfile, profile.ProfilePath(".")).Stop()
```

이 버전을 실행하면, 현재 작업 디렉토리에 프로파일이 작성됩니다.

```sh
% go run mandelbrot.go
2017/09/17 12:22:06 profile: cpu profiling enabled, cpu.pprof
2017/09/17 12:22:08 profile: cpu profiling disabled, cpu.pprof
```

{{< notice note >}}
`pkg/profile`을 사용하는 것이 필수는 아니지만, 트레이싱 수집 및 기록과 관련된 많은 보일러플레이트가 필요하므로 이 워크샵의 나머지 부분에서 사용합니다.
{{< /notice >}}

### 5.3.2 프로파일 분석하기

이제 프로파일이 있으니, `go tool pprof`를 사용해서 분석할 수 있습니다.

```sh
% go tool pprof -http=:8080 cpu.pprof
```

이 실행에서 프로그램이 1.81초 동안 실행되었음을 알 수 있습니다 (프로파일링은 약간의 오버헤드를 추가시킴). pprof는 샘플 기반이므로 운영체제의 `SIGPROF` 타이머에 의존하여 `pprof`가 1.53초 동안만 데이터를 캡처했음을 알 수 있습니다.

{{< notice tip >}}
Go 1.9 이후 `pprof` 트레이싱에는 트레이싱을 분석하는데 필요한 모든 정보가 포함됩니다. 더 이상 트레이싱을 생성하는 일치하는 바이너리가 없어도 됩니다. 🎉
{{< /notice >}}

`top` pprof 명령어를 사용하여 트레이싱이 기록한 함수를 정렬할 수 있습니다.

```sh
% go tool pprof cpu.pprof
Type: cpu
Time: Mar 24, 2019 at 5:18pm (CET)
Duration: 2.16s, Total samples = 1.91s (88.51%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 1.90s, 99.48% of 1.91s total
Showing top 10 nodes out of 35
      flat  flat%   sum%        cum   cum%
     0.82s 42.93% 42.93%      1.63s 85.34%  main.fillPixel
     0.81s 42.41% 85.34%      0.81s 42.41%  main.paint
     0.11s  5.76% 91.10%      0.12s  6.28%  runtime.mallocgc
     0.04s  2.09% 93.19%      0.04s  2.09%  runtime.memmove
     0.04s  2.09% 95.29%      0.04s  2.09%  runtime.nanotime
     0.03s  1.57% 96.86%      0.03s  1.57%  runtime.pthread_cond_signal
     0.02s  1.05% 97.91%      0.04s  2.09%  compress/flate.(*compressor).deflate
     0.01s  0.52% 98.43%      0.01s  0.52%  compress/flate.(*compressor).findMatch
     0.01s  0.52% 98.95%      0.01s  0.52%  compress/flate.hash4
     0.01s  0.52% 99.48%      0.01s  0.52%  image/png.filter
```

pprof가 스택을 캡처했을 때 CPU에는 `main.fillPixel` 함수가 가장 많았습니다.

스택에서 `main.paint`를 찾는 것은 놀라운 일이 아닙니다. 픽셀을 그리는 것은 이 프로그램이 하는 일입니다. 그러나 페인트가 너무 많은 시간을 소비하는 원인은 무엇일까요? top에 대해 누적 플래그(cummulative flag)를 사용해서 확인할 수 있습니다.

```sh
(pprof) top --cum
Showing nodes accounting for 1630ms, 85.34% of 1910ms total
Showing top 10 nodes out of 35
      flat  flat%   sum%        cum   cum%
         0     0%     0%     1840ms 96.34%  main.main
         0     0%     0%     1840ms 96.34%  runtime.main
     820ms 42.93% 42.93%     1630ms 85.34%  main.fillPixel
         0     0% 42.93%     1630ms 85.34%  main.seqFillImg
     810ms 42.41% 85.34%      810ms 42.41%  main.paint
         0     0% 85.34%      210ms 10.99%  image/png.(*Encoder).Encode
         0     0% 85.34%      210ms 10.99%  image/png.Encode
         0     0% 85.34%      160ms  8.38%  main.(*img).At
         0     0% 85.34%      160ms  8.38%  runtime.convT2Inoptr
         0     0% 85.34%      150ms  7.85%  image/png.(*encoder).writeIDATs
```

정렬 결과는 `main.fillPixed`가 대부분의 작업을 처리하고 있다고 제안하고 있습니다.

{{< notice note >}}
`web` 커맨드를 사용하여 프로파일을 시각화할 수도 있습니다.

<div>
	<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%px" height="100%px">
	<script type="text/ecmascript">
	/**
	 *  SVGPan library 1.2.1
	 * ======================
	 *
	 * Given an unique existing element with id "viewport" (or when missing, the first g
	 * element), including the the library into any SVG adds the following capabilities:
	 *
	 *  - Mouse panning
	 *  - Mouse zooming (using the wheel)
	 *  - Object dragging
	 *
	 * You can configure the behaviour of the pan/zoom/drag with the variables
	 * listed in the CONFIGURATION section of this file.
	 *
	 * Known issues:
	 *
	 *  - Zooming (while panning) on Safari has still some issues
	 *
	 * Releases:
	 *
	 * 1.2.1, Mon Jul  4 00:33:18 CEST 2011, Andrea Leofreddi
	 *	- Fixed a regression with mouse wheel (now working on Firefox 5)
	 *	- Working with viewBox attribute (#4)
	 *	- Added "use strict;" and fixed resulting warnings (#5)
	 *	- Added configuration variables, dragging is disabled by default (#3)
	 *
	 * 1.2, Sat Mar 20 08:42:50 GMT 2010, Zeng Xiaohui
	 *	Fixed a bug with browser mouse handler interaction
	 *
	 * 1.1, Wed Feb  3 17:39:33 GMT 2010, Zeng Xiaohui
	 *	Updated the zoom code to support the mouse wheel on Safari/Chrome
	 *
	 * 1.0, Andrea Leofreddi
	 *	First release
	 *
	 * This code is licensed under the following BSD license:
	 *
	 * Copyright 2009-2010 Andrea Leofreddi &lt;a.leofreddi@itcharm.com&gt;. All rights reserved.
	 *
	 * Redistribution and use in source and binary forms, with or without modification, are
	 * permitted provided that the following conditions are met:
	 *
	 *    1. Redistributions of source code must retain the above copyright notice, this list of
	 *       conditions and the following disclaimer.
	 *
	 *    2. Redistributions in binary form must reproduce the above copyright notice, this list
	 *       of conditions and the following disclaimer in the documentation and/or other materials
	 *       provided with the distribution.
	 *
	 * THIS SOFTWARE IS PROVIDED BY Andrea Leofreddi ``AS IS'' AND ANY EXPRESS OR IMPLIED
	 * WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
	 * FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL Andrea Leofreddi OR
	 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
	 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
	 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
	 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
	 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
	 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
	 *
	 * The views and conclusions contained in the software and documentation are those of the
	 * authors and should not be interpreted as representing official policies, either expressed
	 * or implied, of Andrea Leofreddi.
	 */
	
	"use strict";
	
	/// CONFIGURATION
	/// ====&gt;
	
	var enablePan = 1; // 1 or 0: enable or disable panning (default enabled)
	var enableZoom = 1; // 1 or 0: enable or disable zooming (default enabled)
	var enableDrag = 0; // 1 or 0: enable or disable dragging (default disabled)
	
	/// &lt;====
	/// END OF CONFIGURATION
	
	var root = document.documentElement;
	
	var state = 'none', svgRoot, stateTarget, stateOrigin, stateTf;
	
	setupHandlers(root);
	
	/**
	 * Register handlers
	 */
	function setupHandlers(root){
		setAttributes(root, {
			"onmouseup" : "handleMouseUp(evt)",
			"onmousedown" : "handleMouseDown(evt)",
			"onmousemove" : "handleMouseMove(evt)",
			//"onmouseout" : "handleMouseUp(evt)", // Decomment this to stop the pan functionality when dragging out of the SVG element
		});
	
		if(navigator.userAgent.toLowerCase().indexOf('webkit') &gt;= 0)
			window.addEventListener('mousewheel', handleMouseWheel, false); // Chrome/Safari
		else
			window.addEventListener('DOMMouseScroll', handleMouseWheel, false); // Others
	}
	
	/**
	 * Retrieves the root element for SVG manipulation. The element is then cached into the svgRoot global variable.
	 */
	function getRoot(root) {
		if(typeof(svgRoot) == "undefined") {
			var g = null;
	
			g = root.getElementById("viewport");
	
			if(g == null)
				g = root.getElementsByTagName('g')[0];
	
			if(g == null)
				alert('Unable to obtain SVG root element');
	
			setCTM(g, g.getCTM());
	
			g.removeAttribute("viewBox");
	
			svgRoot = g;
		}
	
		return svgRoot;
	}
	
	/**
	 * Instance an SVGPoint object with given event coordinates.
	 */
	function getEventPoint(evt) {
		var p = root.createSVGPoint();
	
		p.x = evt.clientX;
		p.y = evt.clientY;
	
		return p;
	}
	
	/**
	 * Sets the current transform matrix of an element.
	 */
	function setCTM(element, matrix) {
		var s = "matrix(" + matrix.a + "," + matrix.b + "," + matrix.c + "," + matrix.d + "," + matrix.e + "," + matrix.f + ")";
	
		element.setAttribute("transform", s);
	}
	
	/**
	 * Dumps a matrix to a string (useful for debug).
	 */
	function dumpMatrix(matrix) {
		var s = "[ " + matrix.a + ", " + matrix.c + ", " + matrix.e + "\n  " + matrix.b + ", " + matrix.d + ", " + matrix.f + "\n  0, 0, 1 ]";
	
		return s;
	}
	
	/**
	 * Sets attributes of an element.
	 */
	function setAttributes(element, attributes){
		for (var i in attributes)
			element.setAttributeNS(null, i, attributes[i]);
	}
	
	/**
	 * Handle mouse wheel event.
	 */
	function handleMouseWheel(evt) {
		if(!enableZoom)
			return;
	
		if(evt.preventDefault)
			evt.preventDefault();
	
		evt.returnValue = false;
	
		var svgDoc = evt.target.ownerDocument;
	
		var delta;
	
		if(evt.wheelDelta)
			delta = evt.wheelDelta / 3600; // Chrome/Safari
		else
			delta = evt.detail / -90; // Mozilla
	
		var z = 1 + delta; // Zoom factor: 0.9/1.1
	
		var g = getRoot(svgDoc);
	
		var p = getEventPoint(evt);
	
		p = p.matrixTransform(g.getCTM().inverse());
	
		// Compute new scale matrix in current mouse position
		var k = root.createSVGMatrix().translate(p.x, p.y).scale(z).translate(-p.x, -p.y);
	
	        setCTM(g, g.getCTM().multiply(k));
	
		if(typeof(stateTf) == "undefined")
			stateTf = g.getCTM().inverse();
	
		stateTf = stateTf.multiply(k.inverse());
	}
	
	/**
	 * Handle mouse move event.
	 */
	function handleMouseMove(evt) {
		if(evt.preventDefault)
			evt.preventDefault();
	
		evt.returnValue = false;
	
		var svgDoc = evt.target.ownerDocument;
	
		var g = getRoot(svgDoc);
	
		if(state == 'pan' &amp;&amp; enablePan) {
			// Pan mode
			var p = getEventPoint(evt).matrixTransform(stateTf);
	
			setCTM(g, stateTf.inverse().translate(p.x - stateOrigin.x, p.y - stateOrigin.y));
		} else if(state == 'drag' &amp;&amp; enableDrag) {
			// Drag mode
			var p = getEventPoint(evt).matrixTransform(g.getCTM().inverse());
	
			setCTM(stateTarget, root.createSVGMatrix().translate(p.x - stateOrigin.x, p.y - stateOrigin.y).multiply(g.getCTM().inverse()).multiply(stateTarget.getCTM()));
	
			stateOrigin = p;
		}
	}
	
	/**
	 * Handle click event.
	 */
	function handleMouseDown(evt) {
		if(evt.preventDefault)
			evt.preventDefault();
	
		evt.returnValue = false;
	
		var svgDoc = evt.target.ownerDocument;
	
		var g = getRoot(svgDoc);
	
		if(
			evt.target.tagName == "svg"
			|| !enableDrag // Pan anyway when drag is disabled and the user clicked on an element
		) {
			// Pan mode
			state = 'pan';
	
			stateTf = g.getCTM().inverse();
	
			stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
		} else {
			// Drag mode
			state = 'drag';
	
			stateTarget = evt.target;
	
			stateTf = g.getCTM().inverse();
	
			stateOrigin = getEventPoint(evt).matrixTransform(stateTf);
		}
	}
	
	/**
	 * Handle mouse button release event.
	 */
	function handleMouseUp(evt) {
		if(evt.preventDefault)
			evt.preventDefault();
	
		evt.returnValue = false;
	
		var svgDoc = evt.target.ownerDocument;
	
		if(state == 'pan' || state == 'drag') {
			// Quit pan mode
			state = '';
		}
	}
	
	</script><g id="viewport" transform="scale(0.5,0.5) translate(0,0)"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 2858)">
	<title>unnamed</title>
	<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-2858 822.5,-2858 822.5,4 -4,4"></polygon>
	<g id="clust1" class="cluster">
	<title>cluster_L</title>
	<polygon fill="none" stroke="#000000" points="8,-2750 8,-2846 316,-2846 316,-2750 8,-2750"></polygon>
	</g>
	<!-- Type: cpu -->
	<g id="node1" class="node">
	<title>Type: cpu</title>
	<polygon fill="#f8f8f8" stroke="#000000" points="307.5,-2838 16.5,-2838 16.5,-2758 307.5,-2758 307.5,-2838"></polygon>
	<text text-anchor="start" x="24.5" y="-2821.2" font-family="Times,serif" font-size="16.00" fill="#000000">Type: cpu</text>
	<text text-anchor="start" x="24.5" y="-2803.2" font-family="Times,serif" font-size="16.00" fill="#000000">Time: Sep 17, 2017 at 12:22pm (AEST)</text>
	<text text-anchor="start" x="24.5" y="-2785.2" font-family="Times,serif" font-size="16.00" fill="#000000">Duration: 1.81s, Total samples = 1.53s (84.33%)</text>
	<text text-anchor="start" x="24.5" y="-2767.2" font-family="Times,serif" font-size="16.00" fill="#000000">Showing nodes accounting for 1.53s, 100% of 1.53s total</text>
	</g>
	<!-- N1 -->
	<g id="node2" class="node">
	<title>N1</title>
	<g id="a_node2"><a xlink:title="main.paint /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1s)">
	<polygon fill="#edd8d5" stroke="#b21500" points="416.5,-2348 295.5,-2348 295.5,-2236 416.5,-2236 416.5,-2348"></polygon>
	<text text-anchor="middle" x="356" y="-2324.8" font-family="Times,serif" font-size="24.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2298.8" font-family="Times,serif" font-size="24.00" fill="#000000">paint</text>
	<text text-anchor="middle" x="356" y="-2272.8" font-family="Times,serif" font-size="24.00" fill="#000000">mandelbrot.go</text>
	<text text-anchor="middle" x="356" y="-2246.8" font-family="Times,serif" font-size="24.00" fill="#000000">1s (65.36%)</text>
	</a>
	</g>
	</g>
	<!-- N2 -->
	<g id="node3" class="node">
	<title>N2</title>
	<g id="a_node3"><a xlink:title="runtime.main /Users/dfc/go/src/runtime/proc.go (1.53s)">
	<polygon fill="#edd5d5" stroke="#b20000" points="386,-2820 326,-2820 326,-2776 386,-2776 386,-2820"></polygon>
	<text text-anchor="middle" x="356" y="-2809.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="356" y="-2800.6" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2791.6" font-family="Times,serif" font-size="8.00" fill="#000000">proc.go</text>
	<text text-anchor="middle" x="356" y="-2782.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.53s (100%)</text>
	</a>
	</g>
	</g>
	<!-- N4 -->
	<g id="node5" class="node">
	<title>N4</title>
	<g id="a_node5"><a xlink:title="main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.53s)">
	<polygon fill="#edd5d5" stroke="#b20000" points="386,-2707 326,-2707 326,-2663 386,-2663 386,-2707"></polygon>
	<text text-anchor="middle" x="356" y="-2696.6" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2687.6" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2678.6" font-family="Times,serif" font-size="8.00" fill="#000000">mandelbrot.go</text>
	<text text-anchor="middle" x="356" y="-2669.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.53s (100%)</text>
	</a>
	</g>
	</g>
	<!-- N2&#45;&gt;N4 -->
	<g id="edge1" class="edge">
	<title>N2-&gt;N4</title>
	<g id="a_edge1"><a xlink:title="runtime.main /Users/dfc/go/src/runtime/proc.go -> main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.53s)">
	<path fill="none" stroke="#b20000" stroke-width="6" d="M356,-2775.9442C356,-2759.2222 356,-2735.9909 356,-2717.194"></path>
	<polygon fill="#b20000" stroke="#b20000" stroke-width="6" points="361.2501,-2717.0001 356,-2707.0002 350.7501,-2717.0002 361.2501,-2717.0001"></polygon>
	</a>
	</g>
	<g id="a_edge1-label"><a xlink:title="runtime.main /Users/dfc/go/src/runtime/proc.go -> main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.53s)">
	<text text-anchor="middle" x="369" y="-2728.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.53s</text>
	</a>
	</g>
	</g>
	<!-- N3 -->
	<g id="node4" class="node">
	<title>N3</title>
	<g id="a_node4"><a xlink:title="main.fillPixel /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<polygon fill="#edd6d5" stroke="#b20900" points="410.5,-2517 301.5,-2517 301.5,-2414 410.5,-2414 410.5,-2517"></polygon>
	<text text-anchor="middle" x="356" y="-2499.4" font-family="Times,serif" font-size="17.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2480.4" font-family="Times,serif" font-size="17.00" fill="#000000">fillPixel</text>
	<text text-anchor="middle" x="356" y="-2461.4" font-family="Times,serif" font-size="17.00" fill="#000000">mandelbrot.go</text>
	<text text-anchor="middle" x="356" y="-2442.4" font-family="Times,serif" font-size="17.00" fill="#000000">0.27s (17.65%)</text>
	<text text-anchor="middle" x="356" y="-2423.4" font-family="Times,serif" font-size="17.00" fill="#000000">of 1.27s (83.01%)</text>
	</a>
	</g>
	</g>
	<!-- N3&#45;&gt;N1 -->
	<g id="edge4" class="edge">
	<title>N3-&gt;N1</title>
	<g id="a_edge4"><a xlink:title="main.fillPixel /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.paint /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1s)">
	<path fill="none" stroke="#b21500" stroke-width="4" d="M356,-2413.914C356,-2396.4678 356,-2376.7376 356,-2358.3118"></path>
	<polygon fill="#b21500" stroke="#b21500" stroke-width="4" points="359.5001,-2358.2321 356,-2348.2322 352.5001,-2358.2322 359.5001,-2358.2321"></polygon>
	</a>
	</g>
	<g id="a_edge4-label"><a xlink:title="main.fillPixel /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.paint /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1s)">
	<text text-anchor="middle" x="373" y="-2384.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1s</text>
	<text text-anchor="middle" x="373" y="-2369.8" font-family="Times,serif" font-size="14.00" fill="#000000"> (inline)</text>
	</a>
	</g>
	</g>
	<!-- N35 -->
	<g id="node36" class="node">
	<title>N35</title>
	<g id="a_node36"><a xlink:title="image/png.Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<polygon fill="#ede1d9" stroke="#b2591a" points="484,-2612 420,-2612 420,-2568 484,-2568 484,-2612"></polygon>
	<text text-anchor="middle" x="452" y="-2601.6" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="452" y="-2592.6" font-family="Times,serif" font-size="8.00" fill="#000000">Encode</text>
	<text text-anchor="middle" x="452" y="-2583.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="452" y="-2574.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.26s (16.99%)</text>
	</a>
	</g>
	</g>
	<!-- N4&#45;&gt;N35 -->
	<g id="edge6" class="edge">
	<title>N4-&gt;N35</title>
	<g id="a_edge6"><a xlink:title="main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> image/png.Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<path fill="none" stroke="#b2591a" d="M378.2656,-2662.9663C391.3782,-2649.9903 408.1285,-2633.4145 422.3493,-2619.3419"></path>
	<polygon fill="#b2591a" stroke="#b2591a" points="424.8764,-2621.7651 429.5226,-2612.2433 419.9527,-2616.7895 424.8764,-2621.7651"></polygon>
	</a>
	</g>
	<g id="a_edge6-label"><a xlink:title="main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> image/png.Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<text text-anchor="middle" x="422" y="-2633.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.26s</text>
	</a>
	</g>
	</g>
	<!-- N38 -->
	<g id="node39" class="node">
	<title>N38</title>
	<g id="a_node39"><a xlink:title="main.seqFillImg /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<polygon fill="#edd6d5" stroke="#b20900" points="388,-2612 324,-2612 324,-2568 388,-2568 388,-2612"></polygon>
	<text text-anchor="middle" x="356" y="-2601.6" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
	<text text-anchor="middle" x="356" y="-2592.6" font-family="Times,serif" font-size="8.00" fill="#000000">seqFillImg</text>
	<text text-anchor="middle" x="356" y="-2583.6" font-family="Times,serif" font-size="8.00" fill="#000000">mandelbrot.go</text>
	<text text-anchor="middle" x="356" y="-2574.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 1.27s (83.01%)</text>
	</a>
	</g>
	</g>
	<!-- N4&#45;&gt;N38 -->
	<g id="edge2" class="edge">
	<title>N4-&gt;N38</title>
	<g id="a_edge2"><a xlink:title="main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.seqFillImg /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<path fill="none" stroke="#b20900" stroke-width="5" d="M356,-2662.9663C356,-2650.9427 356,-2635.8283 356,-2622.4785"></path>
	<polygon fill="#b20900" stroke="#b20900" stroke-width="5" points="360.3751,-2622.2432 356,-2612.2433 351.6251,-2622.2433 360.3751,-2622.2432"></polygon>
	</a>
	</g>
	<g id="a_edge2-label"><a xlink:title="main.main /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.seqFillImg /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<text text-anchor="middle" x="369" y="-2633.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.27s</text>
	</a>
	</g>
	</g>
	<!-- N5 -->
	<g id="node6" class="node">
	<title>N5</title>
	<g id="a_node6"><a xlink:title="runtime.mallocgc /Users/dfc/go/src/runtime/malloc.go (0.16s)">
	<polygon fill="#ede6e0" stroke="#b28355" points="594,-1873 502,-1873 502,-1790 594,-1790 594,-1873"></polygon>
	<text text-anchor="middle" x="548" y="-1857.8" font-family="Times,serif" font-size="14.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="548" y="-1842.8" font-family="Times,serif" font-size="14.00" fill="#000000">mallocgc</text>
	<text text-anchor="middle" x="548" y="-1827.8" font-family="Times,serif" font-size="14.00" fill="#000000">malloc.go</text>
	<text text-anchor="middle" x="548" y="-1812.8" font-family="Times,serif" font-size="14.00" fill="#000000">0.13s (8.50%)</text>
	<text text-anchor="middle" x="548" y="-1797.8" font-family="Times,serif" font-size="14.00" fill="#000000">of 0.16s (10.46%)</text>
	</a>
	</g>
	</g>
	<!-- N41 -->
	<g id="node42" class="node">
	<title>N41</title>
	<g id="a_node42"><a xlink:title="runtime.(*mcache).nextFree /Users/dfc/go/src/runtime/malloc.go (0.03s)">
	<polygon fill="#edecea" stroke="#b2ada1" points="600.5,-1728.5 539.5,-1728.5 539.5,-1675.5 600.5,-1675.5 600.5,-1728.5"></polygon>
	<text text-anchor="middle" x="570" y="-1718.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="570" y="-1709.1" font-family="Times,serif" font-size="8.00" fill="#000000">(*mcache)</text>
	<text text-anchor="middle" x="570" y="-1700.1" font-family="Times,serif" font-size="8.00" fill="#000000">nextFree</text>
	<text text-anchor="middle" x="570" y="-1691.1" font-family="Times,serif" font-size="8.00" fill="#000000">malloc.go</text>
	<text text-anchor="middle" x="570" y="-1682.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.03s (1.96%)</text>
	</a>
	</g>
	</g>
	<!-- N5&#45;&gt;N41 -->
	<g id="edge35" class="edge">
	<title>N5-&gt;N41</title>
	<g id="a_edge35"><a xlink:title="runtime.mallocgc /Users/dfc/go/src/runtime/malloc.go -> runtime.(*mcache).nextFree /Users/dfc/go/src/runtime/malloc.go (0.03s)">
	<path fill="none" stroke="#b2ada1" d="M555.082,-1789.8126C557.8847,-1773.3148 561.0778,-1754.5194 563.7887,-1738.5621"></path>
	<polygon fill="#b2ada1" stroke="#b2ada1" points="567.2672,-1738.9833 565.4916,-1728.5383 560.3661,-1737.8108 567.2672,-1738.9833"></polygon>
	</a>
	</g>
	<g id="a_edge35-label"><a xlink:title="runtime.mallocgc /Users/dfc/go/src/runtime/malloc.go -> runtime.(*mcache).nextFree /Users/dfc/go/src/runtime/malloc.go (0.03s)">
	<text text-anchor="middle" x="573" y="-1760.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.03s</text>
	</a>
	</g>
	</g>
	<!-- N6 -->
	<g id="node7" class="node">
	<title>N6</title>
	<g id="a_node7"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<polygon fill="#ede5de" stroke="#b27743" points="499,-2185 435,-2185 435,-2132 499,-2132 499,-2185"></polygon>
	<text text-anchor="middle" x="467" y="-2174.6" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="467" y="-2165.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*encoder)</text>
	<text text-anchor="middle" x="467" y="-2156.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeImage</text>
	<text text-anchor="middle" x="467" y="-2147.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="467" y="-2138.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.19s (12.42%)</text>
	</a>
	</g>
	</g>
	<!-- N8 -->
	<g id="node9" class="node">
	<title>N8</title>
	<g id="a_node9"><a xlink:title="main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (0.18s)">
	<polygon fill="#ede5df" stroke="#b27b49" points="580,-2081 516,-2081 516,-2028 580,-2028 580,-2081"></polygon>
	<text text-anchor="middle" x="548" y="-2070.6" font-family="Times,serif" font-size="8.00" fill="#000000">main</text>
	<text text-anchor="middle" x="548" y="-2061.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*img)</text>
	<text text-anchor="middle" x="548" y="-2052.6" font-family="Times,serif" font-size="8.00" fill="#000000">At</text>
	<text text-anchor="middle" x="548" y="-2043.6" font-family="Times,serif" font-size="8.00" fill="#000000">mandelbrot.go</text>
	<text text-anchor="middle" x="548" y="-2034.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.18s (11.76%)</text>
	</a>
	</g>
	</g>
	<!-- N6&#45;&gt;N8 -->
	<g id="edge11" class="edge">
	<title>N6-&gt;N8</title>
	<g id="a_edge11"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (0.11s)">
	<path fill="none" stroke="#b29572" d="M484.252,-2131.9097C491.321,-2121.437 499.7656,-2109.4412 508,-2099 510.6052,-2095.6965 513.3873,-2092.3245 516.225,-2088.9884"></path>
	<polygon fill="#b29572" stroke="#b29572" points="518.9729,-2091.1622 522.8904,-2081.3182 513.6891,-2086.5706 518.9729,-2091.1622"></polygon>
	</a>
	</g>
	<g id="a_edge11-label"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (0.11s)">
	<text text-anchor="middle" x="521" y="-2102.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.11s</text>
	</a>
	</g>
	</g>
	<!-- N18 -->
	<g id="node19" class="node">
	<title>N18</title>
	<g id="a_node19"><a xlink:title="image/png.filter /Users/dfc/go/src/image/png/writer.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="409.5,-2080.5 350.5,-2080.5 350.5,-2028.5 409.5,-2028.5 409.5,-2080.5"></polygon>
	<text text-anchor="middle" x="380" y="-2068.5" font-family="Times,serif" font-size="10.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="380" y="-2057.5" font-family="Times,serif" font-size="10.00" fill="#000000">filter</text>
	<text text-anchor="middle" x="380" y="-2046.5" font-family="Times,serif" font-size="10.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="380" y="-2035.5" font-family="Times,serif" font-size="10.00" fill="#000000">0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N6&#45;&gt;N18 -->
	<g id="edge45" class="edge">
	<title>N6-&gt;N18</title>
	<g id="a_edge45"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> image/png.filter /Users/dfc/go/src/image/png/writer.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M442.607,-2131.9174C437.3869,-2126.0755 431.9544,-2119.8776 427,-2114 420.1205,-2105.8386 412.8838,-2096.878 406.2063,-2088.4432"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="408.9391,-2086.2563 400.0054,-2080.5606 403.4374,-2090.5842 408.9391,-2086.2563"></polygon>
	</a>
	</g>
	<g id="a_edge45-label"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> image/png.filter /Users/dfc/go/src/image/png/writer.go (0.01s)">
	<text text-anchor="middle" x="440" y="-2102.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N31 -->
	<g id="node32" class="node">
	<title>N31</title>
	<g id="a_node32"><a xlink:title="compress/zlib.(*Writer).Write /Users/dfc/go/src/compress/zlib/writer.go (0.07s)">
	<polygon fill="#edeae7" stroke="#b2a289" points="488.5,-2081 427.5,-2081 427.5,-2028 488.5,-2028 488.5,-2081"></polygon>
	<text text-anchor="middle" x="458" y="-2070.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/zlib</text>
	<text text-anchor="middle" x="458" y="-2061.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Writer)</text>
	<text text-anchor="middle" x="458" y="-2052.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="458" y="-2043.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="458" y="-2034.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.07s (4.58%)</text>
	</a>
	</g>
	</g>
	<!-- N6&#45;&gt;N31 -->
	<g id="edge16" class="edge">
	<title>N6-&gt;N31</title>
	<g id="a_edge16"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> compress/zlib.(*Writer).Write /Users/dfc/go/src/compress/zlib/writer.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M461.4749,-2131.7712C460.4711,-2125.9384 459.5645,-2119.7812 459,-2114 458.2821,-2106.6481 457.8872,-2098.7615 457.6909,-2091.2124"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="461.1905,-2091.1533 457.5346,-2081.2093 454.1913,-2091.2628 461.1905,-2091.1533"></polygon>
	</a>
	</g>
	<g id="a_edge16-label"><a xlink:title="image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go -> compress/zlib.(*Writer).Write /Users/dfc/go/src/compress/zlib/writer.go (0.07s)">
	<text text-anchor="middle" x="472" y="-2102.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N7 -->
	<g id="node8" class="node">
	<title>N7</title>
	<g id="a_node8"><a xlink:title="image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<polygon fill="#ede1d9" stroke="#b2591a" points="499,-2492 435,-2492 435,-2439 499,-2439 499,-2492"></polygon>
	<text text-anchor="middle" x="467" y="-2481.6" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="467" y="-2472.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Encoder)</text>
	<text text-anchor="middle" x="467" y="-2463.6" font-family="Times,serif" font-size="8.00" fill="#000000">Encode</text>
	<text text-anchor="middle" x="467" y="-2454.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="467" y="-2445.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.26s (16.99%)</text>
	</a>
	</g>
	</g>
	<!-- N34 -->
	<g id="node35" class="node">
	<title>N34</title>
	<g id="a_node35"><a xlink:title="image/png.(*encoder).writeIDATs /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<polygon fill="#ede5de" stroke="#b27743" points="499,-2318.5 435,-2318.5 435,-2265.5 499,-2265.5 499,-2318.5"></polygon>
	<text text-anchor="middle" x="467" y="-2308.1" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="467" y="-2299.1" font-family="Times,serif" font-size="8.00" fill="#000000">(*encoder)</text>
	<text text-anchor="middle" x="467" y="-2290.1" font-family="Times,serif" font-size="8.00" fill="#000000">writeIDATs</text>
	<text text-anchor="middle" x="467" y="-2281.1" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="467" y="-2272.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.19s (12.42%)</text>
	</a>
	</g>
	</g>
	<!-- N7&#45;&gt;N34 -->
	<g id="edge7" class="edge">
	<title>N7-&gt;N34</title>
	<g id="a_edge7"><a xlink:title="image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeIDATs /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<path fill="none" stroke="#b27743" d="M467,-2438.7709C467,-2409.351 467,-2361.8667 467,-2328.9085"></path>
	<polygon fill="#b27743" stroke="#b27743" points="470.5001,-2328.7595 467,-2318.7595 463.5001,-2328.7596 470.5001,-2328.7595"></polygon>
	</a>
	</g>
	<g id="a_edge7-label"><a xlink:title="image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeIDATs /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<text text-anchor="middle" x="480" y="-2377.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.19s</text>
	</a>
	</g>
	</g>
	<!-- N36 -->
	<g id="node37" class="node">
	<title>N36</title>
	<g id="a_node37"><a xlink:title="image/png.opaque /Users/dfc/go/src/image/png/writer.go (0.07s)">
	<polygon fill="#edeae7" stroke="#b2a289" points="578.5,-2314 517.5,-2314 517.5,-2270 578.5,-2270 578.5,-2314"></polygon>
	<text text-anchor="middle" x="548" y="-2303.6" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="548" y="-2294.6" font-family="Times,serif" font-size="8.00" fill="#000000">opaque</text>
	<text text-anchor="middle" x="548" y="-2285.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="548" y="-2276.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.07s (4.58%)</text>
	</a>
	</g>
	</g>
	<!-- N7&#45;&gt;N36 -->
	<g id="edge15" class="edge">
	<title>N7-&gt;N36</title>
	<g id="a_edge15"><a xlink:title="image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go -> image/png.opaque /Users/dfc/go/src/image/png/writer.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M479.4787,-2438.7709C494.0192,-2407.6255 518.0115,-2356.2347 533.4032,-2323.266"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="536.6335,-2324.6203 537.6925,-2314.0785 530.2907,-2321.659 536.6335,-2324.6203"></polygon>
	</a>
	</g>
	<g id="a_edge15-label"><a xlink:title="image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go -> image/png.opaque /Users/dfc/go/src/image/png/writer.go (0.07s)">
	<text text-anchor="middle" x="525" y="-2377.3" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N10 -->
	<g id="node11" class="node">
	<title>N10</title>
	<g id="a_node11"><a xlink:title="runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go (0.18s)">
	<polygon fill="#ede5df" stroke="#b27b49" points="580,-1972.5 516,-1972.5 516,-1928.5 580,-1928.5 580,-1972.5"></polygon>
	<text text-anchor="middle" x="548" y="-1962.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="548" y="-1953.1" font-family="Times,serif" font-size="8.00" fill="#000000">convT2Inoptr</text>
	<text text-anchor="middle" x="548" y="-1944.1" font-family="Times,serif" font-size="8.00" fill="#000000">iface.go</text>
	<text text-anchor="middle" x="548" y="-1935.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.18s (11.76%)</text>
	</a>
	</g>
	</g>
	<!-- N8&#45;&gt;N10 -->
	<g id="edge9" class="edge">
	<title>N8-&gt;N10</title>
	<g id="a_edge9"><a xlink:title="main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go (0.18s)">
	<path fill="none" stroke="#b27b49" d="M548,-2027.9873C548,-2014.2341 548,-1997.3207 548,-1982.8083"></path>
	<polygon fill="#b27b49" stroke="#b27b49" points="551.5001,-1982.7448 548,-1972.7448 544.5001,-1982.7448 551.5001,-1982.7448"></polygon>
	</a>
	</g>
	<g id="a_edge9-label"><a xlink:title="main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go (0.18s)">
	<text text-anchor="middle" x="561" y="-1998.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.18s</text>
	</a>
	</g>
	</g>
	<!-- N9 -->
	<g id="node10" class="node">
	<title>N9</title>
	<g id="a_node10"><a xlink:title="syscall.Syscall /Users/dfc/go/src/syscall/asm_darwin_amd64.s (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="500.5,-60 405.5,-60 405.5,0 500.5,0 500.5,-60"></polygon>
	<text text-anchor="middle" x="453" y="-46.4" font-family="Times,serif" font-size="12.00" fill="#000000">syscall</text>
	<text text-anchor="middle" x="453" y="-33.4" font-family="Times,serif" font-size="12.00" fill="#000000">Syscall</text>
	<text text-anchor="middle" x="453" y="-20.4" font-family="Times,serif" font-size="12.00" fill="#000000">asm_darwin_amd64.s</text>
	<text text-anchor="middle" x="453" y="-7.4" font-family="Times,serif" font-size="12.00" fill="#000000">0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N10&#45;&gt;N5 -->
	<g id="edge10" class="edge">
	<title>N10-&gt;N5</title>
	<g id="a_edge10"><a xlink:title="runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go -> runtime.mallocgc /Users/dfc/go/src/runtime/malloc.go (0.16s)">
	<path fill="none" stroke="#b28355" d="M548,-1928.3959C548,-1915.7144 548,-1899.2214 548,-1883.4481"></path>
	<polygon fill="#b28355" stroke="#b28355" points="551.5001,-1883.234 548,-1873.234 544.5001,-1883.2341 551.5001,-1883.234"></polygon>
	</a>
	</g>
	<g id="a_edge10-label"><a xlink:title="runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go -> runtime.mallocgc /Users/dfc/go/src/runtime/malloc.go (0.16s)">
	<text text-anchor="middle" x="561" y="-1894.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.16s</text>
	</a>
	</g>
	</g>
	<!-- N13 -->
	<g id="node14" class="node">
	<title>N13</title>
	<g id="a_node14"><a xlink:title="runtime.memmove /Users/dfc/go/src/runtime/memmove_amd64.s (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="695.5,-1859.5 612.5,-1859.5 612.5,-1803.5 695.5,-1803.5 695.5,-1859.5"></polygon>
	<text text-anchor="middle" x="654" y="-1846.7" font-family="Times,serif" font-size="11.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="654" y="-1834.7" font-family="Times,serif" font-size="11.00" fill="#000000">memmove</text>
	<text text-anchor="middle" x="654" y="-1822.7" font-family="Times,serif" font-size="11.00" fill="#000000">memmove_amd64.s</text>
	<text text-anchor="middle" x="654" y="-1810.7" font-family="Times,serif" font-size="11.00" fill="#000000">0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N10&#45;&gt;N13 -->
	<g id="edge39" class="edge">
	<title>N10-&gt;N13</title>
	<g id="a_edge39"><a xlink:title="runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go -> runtime.memmove /Users/dfc/go/src/runtime/memmove_amd64.s (0.02s)">
	<path fill="none" stroke="#b2afa6" d="M567.6894,-1928.3959C582.9118,-1911.3066 604.2995,-1887.2958 621.9756,-1867.4519"></path>
	<polygon fill="#b2afa6" stroke="#b2afa6" points="624.7575,-1869.5908 628.7955,-1859.7956 619.5305,-1864.9348 624.7575,-1869.5908"></polygon>
	</a>
	</g>
	<g id="a_edge39-label"><a xlink:title="runtime.convT2Inoptr /Users/dfc/go/src/runtime/iface.go -> runtime.memmove /Users/dfc/go/src/runtime/memmove_amd64.s (0.02s)">
	<text text-anchor="middle" x="614" y="-1894.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.02s</text>
	</a>
	</g>
	</g>
	<!-- N11 -->
	<g id="node12" class="node">
	<title>N11</title>
	<g id="a_node12"><a xlink:title="compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<polygon fill="#edeae7" stroke="#b2a289" points="486.5,-1739 419.5,-1739 419.5,-1665 486.5,-1665 486.5,-1739"></polygon>
	<text text-anchor="middle" x="453" y="-1727" font-family="Times,serif" font-size="10.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1716" font-family="Times,serif" font-size="10.00" fill="#000000">(*compressor)</text>
	<text text-anchor="middle" x="453" y="-1705" font-family="Times,serif" font-size="10.00" fill="#000000">deflate</text>
	<text text-anchor="middle" x="453" y="-1694" font-family="Times,serif" font-size="10.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="453" y="-1683" font-family="Times,serif" font-size="10.00" fill="#000000">0.01s (0.65%)</text>
	<text text-anchor="middle" x="453" y="-1672" font-family="Times,serif" font-size="10.00" fill="#000000">of 0.07s (4.58%)</text>
	</a>
	</g>
	</g>
	<!-- N24 -->
	<g id="node25" class="node">
	<title>N24</title>
	<g id="a_node25"><a xlink:title="compress/flate.(*compressor).findMatch /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="565.5,-1614 504.5,-1614 504.5,-1561 565.5,-1561 565.5,-1614"></polygon>
	<text text-anchor="middle" x="535" y="-1603.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="535" y="-1594.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*compressor)</text>
	<text text-anchor="middle" x="535" y="-1585.6" font-family="Times,serif" font-size="8.00" fill="#000000">findMatch</text>
	<text text-anchor="middle" x="535" y="-1576.6" font-family="Times,serif" font-size="8.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="535" y="-1567.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N11&#45;&gt;N24 -->
	<g id="edge41" class="edge">
	<title>N11-&gt;N24</title>
	<g id="a_edge41"><a xlink:title="compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).findMatch /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M479.6231,-1664.8251C489.2902,-1651.3265 500.1782,-1636.1231 509.7255,-1622.7918"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="512.8026,-1624.5064 515.7795,-1614.3384 507.1115,-1620.4307 512.8026,-1624.5064"></polygon>
	</a>
	</g>
	<g id="a_edge41-label"><a xlink:title="compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).findMatch /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<text text-anchor="middle" x="514" y="-1635.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N26 -->
	<g id="node27" class="node">
	<title>N26</title>
	<g id="a_node27"><a xlink:title="compress/flate.(*compressor).writeBlock /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-1614 422.5,-1614 422.5,-1561 483.5,-1561 483.5,-1614"></polygon>
	<text text-anchor="middle" x="453" y="-1603.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1594.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*compressor)</text>
	<text text-anchor="middle" x="453" y="-1585.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeBlock</text>
	<text text-anchor="middle" x="453" y="-1576.6" font-family="Times,serif" font-size="8.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="453" y="-1567.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N11&#45;&gt;N26 -->
	<g id="edge20" class="edge">
	<title>N11-&gt;N26</title>
	<g id="a_edge20"><a xlink:title="compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).writeBlock /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-1664.8251C453,-1651.9693 453,-1637.5671 453,-1624.7086"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-1624.3383 453,-1614.3384 449.5001,-1624.3384 456.5001,-1624.3383"></polygon>
	</a>
	</g>
	<g id="a_edge20-label"><a xlink:title="compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).writeBlock /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<text text-anchor="middle" x="466" y="-1635.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N12 -->
	<g id="node13" class="node">
	<title>N12</title>
	<g id="a_node13"><a xlink:title="runtime.mmap /Users/dfc/go/src/runtime/sys_darwin_amd64.s (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="683,-669 597,-669 597,-613 683,-613 683,-669"></polygon>
	<text text-anchor="middle" x="640" y="-656.2" font-family="Times,serif" font-size="11.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="640" y="-644.2" font-family="Times,serif" font-size="11.00" fill="#000000">mmap</text>
	<text text-anchor="middle" x="640" y="-632.2" font-family="Times,serif" font-size="11.00" fill="#000000">sys_darwin_amd64.s</text>
	<text text-anchor="middle" x="640" y="-620.2" font-family="Times,serif" font-size="11.00" fill="#000000">0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N14 -->
	<g id="node15" class="node">
	<title>N14</title>
	<g id="a_node15"><a xlink:title="compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="488.5,-1189 417.5,-1189 417.5,-1136 488.5,-1136 488.5,-1189"></polygon>
	<text text-anchor="middle" x="453" y="-1178.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1169.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*huffmanBitWriter)</text>
	<text text-anchor="middle" x="453" y="-1160.6" font-family="Times,serif" font-size="8.00" fill="#000000">write</text>
	<text text-anchor="middle" x="453" y="-1151.6" font-family="Times,serif" font-size="8.00" fill="#000000">huffman_bit_writer.go</text>
	<text text-anchor="middle" x="453" y="-1142.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N27 -->
	<g id="node28" class="node">
	<title>N27</title>
	<g id="a_node28"><a xlink:title="compress/flate.(*dictWriter).Write /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-1085 422.5,-1085 422.5,-1032 483.5,-1032 483.5,-1085"></polygon>
	<text text-anchor="middle" x="453" y="-1074.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1065.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*dictWriter)</text>
	<text text-anchor="middle" x="453" y="-1056.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-1047.6" font-family="Times,serif" font-size="8.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="453" y="-1038.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N14&#45;&gt;N27 -->
	<g id="edge23" class="edge">
	<title>N14-&gt;N27</title>
	<g id="a_edge23"><a xlink:title="compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*dictWriter).Write /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-1135.9873C453,-1123.7358 453,-1108.9766 453,-1095.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-1095.3259 453,-1085.3259 449.5001,-1095.3259 456.5001,-1095.3259"></polygon>
	</a>
	</g>
	<g id="a_edge23-label"><a xlink:title="compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*dictWriter).Write /Users/dfc/go/src/compress/flate/deflate.go (0.05s)">
	<text text-anchor="middle" x="466" y="-1106.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N15 -->
	<g id="node16" class="node">
	<title>N15</title>
	<g id="a_node16"><a xlink:title="compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="488.5,-1397 417.5,-1397 417.5,-1344 488.5,-1344 488.5,-1397"></polygon>
	<text text-anchor="middle" x="453" y="-1386.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1377.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*huffmanBitWriter)</text>
	<text text-anchor="middle" x="453" y="-1368.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeTokens</text>
	<text text-anchor="middle" x="453" y="-1359.6" font-family="Times,serif" font-size="8.00" fill="#000000">huffman_bit_writer.go</text>
	<text text-anchor="middle" x="453" y="-1350.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N28 -->
	<g id="node29" class="node">
	<title>N28</title>
	<g id="a_node29"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBits /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="399.5,-1293 328.5,-1293 328.5,-1240 399.5,-1240 399.5,-1293"></polygon>
	<text text-anchor="middle" x="364" y="-1282.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="364" y="-1273.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*huffmanBitWriter)</text>
	<text text-anchor="middle" x="364" y="-1264.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeBits</text>
	<text text-anchor="middle" x="364" y="-1255.6" font-family="Times,serif" font-size="8.00" fill="#000000">huffman_bit_writer.go</text>
	<text text-anchor="middle" x="364" y="-1246.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N15&#45;&gt;N28 -->
	<g id="edge44" class="edge">
	<title>N15-&gt;N28</title>
	<g id="a_edge44"><a xlink:title="compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeBits /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M430.3113,-1343.9873C419.204,-1331.008 405.6881,-1315.2141 393.7636,-1301.2799"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="396.1179,-1298.648 386.9568,-1293.3259 390.7995,-1303.1993 396.1179,-1298.648"></polygon>
	</a>
	</g>
	<g id="a_edge44-label"><a xlink:title="compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeBits /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.01s)">
	<text text-anchor="middle" x="426" y="-1314.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N30 -->
	<g id="node31" class="node">
	<title>N30</title>
	<g id="a_node31"><a xlink:title="compress/flate.(*huffmanBitWriter).writeCode /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.04s)">
	<polygon fill="#edecea" stroke="#b2aa9b" points="488.5,-1293 417.5,-1293 417.5,-1240 488.5,-1240 488.5,-1293"></polygon>
	<text text-anchor="middle" x="453" y="-1282.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1273.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*huffmanBitWriter)</text>
	<text text-anchor="middle" x="453" y="-1264.6" font-family="Times,serif" font-size="8.00" fill="#000000">writeCode</text>
	<text text-anchor="middle" x="453" y="-1255.6" font-family="Times,serif" font-size="8.00" fill="#000000">huffman_bit_writer.go</text>
	<text text-anchor="middle" x="453" y="-1246.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.04s (2.61%)</text>
	</a>
	</g>
	</g>
	<!-- N15&#45;&gt;N30 -->
	<g id="edge33" class="edge">
	<title>N15-&gt;N30</title>
	<g id="a_edge33"><a xlink:title="compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeCode /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.04s)">
	<path fill="none" stroke="#b2aa9b" d="M453,-1343.9873C453,-1331.7358 453,-1316.9766 453,-1303.6407"></path>
	<polygon fill="#b2aa9b" stroke="#b2aa9b" points="456.5001,-1303.3259 453,-1293.3259 449.5001,-1303.3259 456.5001,-1303.3259"></polygon>
	</a>
	</g>
	<g id="a_edge33-label"><a xlink:title="compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeCode /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.04s)">
	<text text-anchor="middle" x="466" y="-1314.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.04s</text>
	</a>
	</g>
	</g>
	<!-- N16 -->
	<g id="node17" class="node">
	<title>N16</title>
	<g id="a_node17"><a xlink:title="runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s (0.03s)">
	<polygon fill="#edecea" stroke="#b2ada1" points="646.5,-1609.5 585.5,-1609.5 585.5,-1565.5 646.5,-1565.5 646.5,-1609.5"></polygon>
	<text text-anchor="middle" x="616" y="-1599.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="616" y="-1590.1" font-family="Times,serif" font-size="8.00" fill="#000000">systemstack</text>
	<text text-anchor="middle" x="616" y="-1581.1" font-family="Times,serif" font-size="8.00" fill="#000000">asm_amd64.s</text>
	<text text-anchor="middle" x="616" y="-1572.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.03s (1.96%)</text>
	</a>
	</g>
	</g>
	<!-- N42 -->
	<g id="node43" class="node">
	<title>N42</title>
	<g id="a_node43"><a xlink:title="runtime.(*mcache).nextFree.func1 /Users/dfc/go/src/runtime/malloc.go (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="725.5,-1510 664.5,-1510 664.5,-1448 725.5,-1448 725.5,-1510"></polygon>
	<text text-anchor="middle" x="695" y="-1499.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-1490.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mcache)</text>
	<text text-anchor="middle" x="695" y="-1481.6" font-family="Times,serif" font-size="8.00" fill="#000000">nextFree</text>
	<text text-anchor="middle" x="695" y="-1472.6" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
	<text text-anchor="middle" x="695" y="-1463.6" font-family="Times,serif" font-size="8.00" fill="#000000">malloc.go</text>
	<text text-anchor="middle" x="695" y="-1454.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N16&#45;&gt;N42 -->
	<g id="edge40" class="edge">
	<title>N16-&gt;N42</title>
	<g id="a_edge40"><a xlink:title="runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s -> runtime.(*mcache).nextFree.func1 /Users/dfc/go/src/runtime/malloc.go (0.02s)">
	<path fill="none" stroke="#b2afa6" d="M632.1775,-1565.2816C641.928,-1551.89 654.6326,-1534.4413 666.0634,-1518.742"></path>
	<polygon fill="#b2afa6" stroke="#b2afa6" points="669.1992,-1520.3815 672.2559,-1510.2371 663.5403,-1516.2611 669.1992,-1520.3815"></polygon>
	</a>
	</g>
	<g id="a_edge40-label"><a xlink:title="runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s -> runtime.(*mcache).nextFree.func1 /Users/dfc/go/src/runtime/malloc.go (0.02s)">
	<text text-anchor="middle" x="670" y="-1531.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.02s</text>
	</a>
	</g>
	</g>
	<!-- N46 -->
	<g id="node47" class="node">
	<title>N46</title>
	<g id="a_node47"><a xlink:title="runtime.(*mheap).alloc.func1 /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="646.5,-1510 585.5,-1510 585.5,-1448 646.5,-1448 646.5,-1510"></polygon>
	<text text-anchor="middle" x="616" y="-1499.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="616" y="-1490.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="616" y="-1481.6" font-family="Times,serif" font-size="8.00" fill="#000000">alloc</text>
	<text text-anchor="middle" x="616" y="-1472.6" font-family="Times,serif" font-size="8.00" fill="#000000">func1</text>
	<text text-anchor="middle" x="616" y="-1463.6" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="616" y="-1454.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N16&#45;&gt;N46 -->
	<g id="edge59" class="edge">
	<title>N16-&gt;N46</title>
	<g id="a_edge59"><a xlink:title="runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s -> runtime.(*mheap).alloc.func1 /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M616,-1565.2816C616,-1552.386 616,-1535.7282 616,-1520.4933"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="619.5001,-1520.2371 616,-1510.2371 612.5001,-1520.2372 619.5001,-1520.2371"></polygon>
	</a>
	</g>
	<g id="a_edge59-label"><a xlink:title="runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s -> runtime.(*mheap).alloc.func1 /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="629" y="-1531.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N17 -->
	<g id="node18" class="node">
	<title>N17</title>
	<g id="a_node18"><a xlink:title="compress/flate.matchLen /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="567,-1505 507,-1505 507,-1453 567,-1453 567,-1505"></polygon>
	<text text-anchor="middle" x="537" y="-1493" font-family="Times,serif" font-size="10.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="537" y="-1482" font-family="Times,serif" font-size="10.00" fill="#000000">matchLen</text>
	<text text-anchor="middle" x="537" y="-1471" font-family="Times,serif" font-size="10.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="537" y="-1460" font-family="Times,serif" font-size="10.00" fill="#000000">0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N19 -->
	<g id="node20" class="node">
	<title>N19</title>
	<g id="a_node20"><a xlink:title="runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="725.5,-1189 664.5,-1189 664.5,-1136 725.5,-1136 725.5,-1189"></polygon>
	<text text-anchor="middle" x="695" y="-1178.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-1169.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mcentral)</text>
	<text text-anchor="middle" x="695" y="-1160.6" font-family="Times,serif" font-size="8.00" fill="#000000">grow</text>
	<text text-anchor="middle" x="695" y="-1151.6" font-family="Times,serif" font-size="8.00" fill="#000000">mcentral.go</text>
	<text text-anchor="middle" x="695" y="-1142.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N45 -->
	<g id="node46" class="node">
	<title>N45</title>
	<g id="a_node46"><a xlink:title="runtime.(*mheap).alloc /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="725.5,-1085 664.5,-1085 664.5,-1032 725.5,-1032 725.5,-1085"></polygon>
	<text text-anchor="middle" x="695" y="-1074.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-1065.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="695" y="-1056.6" font-family="Times,serif" font-size="8.00" fill="#000000">alloc</text>
	<text text-anchor="middle" x="695" y="-1047.6" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="695" y="-1038.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N19&#45;&gt;N45 -->
	<g id="edge46" class="edge">
	<title>N19-&gt;N45</title>
	<g id="a_edge46"><a xlink:title="runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go -> runtime.(*mheap).alloc /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M695,-1135.9873C695,-1123.7358 695,-1108.9766 695,-1095.6407"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="698.5001,-1095.3259 695,-1085.3259 691.5001,-1095.3259 698.5001,-1095.3259"></polygon>
	</a>
	</g>
	<g id="a_edge46-label"><a xlink:title="runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go -> runtime.(*mheap).alloc /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="708" y="-1106.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N51 -->
	<g id="node52" class="node">
	<title>N51</title>
	<g id="a_node52"><a xlink:title="runtime.heapBits.initSpan /Users/dfc/go/src/runtime/mbitmap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="811.5,-1085 750.5,-1085 750.5,-1032 811.5,-1032 811.5,-1085"></polygon>
	<text text-anchor="middle" x="781" y="-1074.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="781" y="-1065.6" font-family="Times,serif" font-size="8.00" fill="#000000">heapBits</text>
	<text text-anchor="middle" x="781" y="-1056.6" font-family="Times,serif" font-size="8.00" fill="#000000">initSpan</text>
	<text text-anchor="middle" x="781" y="-1047.6" font-family="Times,serif" font-size="8.00" fill="#000000">mbitmap.go</text>
	<text text-anchor="middle" x="781" y="-1038.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N19&#45;&gt;N51 -->
	<g id="edge47" class="edge">
	<title>N19-&gt;N51</title>
	<g id="a_edge47"><a xlink:title="runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go -> runtime.heapBits.initSpan /Users/dfc/go/src/runtime/mbitmap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M716.9239,-1135.9873C727.6568,-1123.008 740.7172,-1107.2141 752.2397,-1093.2799"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="755.1416,-1095.2628 758.8171,-1085.3259 749.7471,-1090.8019 755.1416,-1095.2628"></polygon>
	</a>
	</g>
	<g id="a_edge47-label"><a xlink:title="runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go -> runtime.heapBits.initSpan /Users/dfc/go/src/runtime/mbitmap.go (0.01s)">
	<text text-anchor="middle" x="756" y="-1106.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N20 -->
	<g id="node21" class="node">
	<title>N20</title>
	<g id="a_node21"><a xlink:title="runtime.memclrNoHeapPointers /Users/dfc/go/src/runtime/memclr_amd64.s (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="739.5,-980.5 650.5,-980.5 650.5,-928.5 739.5,-928.5 739.5,-980.5"></polygon>
	<text text-anchor="middle" x="695" y="-968.5" font-family="Times,serif" font-size="10.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-957.5" font-family="Times,serif" font-size="10.00" fill="#000000">memclrNoHeapPointers</text>
	<text text-anchor="middle" x="695" y="-946.5" font-family="Times,serif" font-size="10.00" fill="#000000">memclr_amd64.s</text>
	<text text-anchor="middle" x="695" y="-935.5" font-family="Times,serif" font-size="10.00" fill="#000000">0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N21 -->
	<g id="node22" class="node">
	<title>N21</title>
	<g id="a_node22"><a xlink:title="bufio.(*Writer).Flush /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-877 422.5,-877 422.5,-824 483.5,-824 483.5,-877"></polygon>
	<text text-anchor="middle" x="453" y="-866.6" font-family="Times,serif" font-size="8.00" fill="#000000">bufio</text>
	<text text-anchor="middle" x="453" y="-857.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Writer)</text>
	<text text-anchor="middle" x="453" y="-848.6" font-family="Times,serif" font-size="8.00" fill="#000000">Flush</text>
	<text text-anchor="middle" x="453" y="-839.6" font-family="Times,serif" font-size="8.00" fill="#000000">bufio.go</text>
	<text text-anchor="middle" x="453" y="-830.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N32 -->
	<g id="node33" class="node">
	<title>N32</title>
	<g id="a_node33"><a xlink:title="image/png.(*encoder).Write /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-773 422.5,-773 422.5,-720 483.5,-720 483.5,-773"></polygon>
	<text text-anchor="middle" x="453" y="-762.6" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="453" y="-753.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*encoder)</text>
	<text text-anchor="middle" x="453" y="-744.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-735.6" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="453" y="-726.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N21&#45;&gt;N32 -->
	<g id="edge18" class="edge">
	<title>N21-&gt;N32</title>
	<g id="a_edge18"><a xlink:title="bufio.(*Writer).Flush /Users/dfc/go/src/bufio/bufio.go -> image/png.(*encoder).Write /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-823.9873C453,-811.7358 453,-796.9766 453,-783.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-783.3259 453,-773.3259 449.5001,-783.3259 456.5001,-783.3259"></polygon>
	</a>
	</g>
	<g id="a_edge18-label"><a xlink:title="bufio.(*Writer).Flush /Users/dfc/go/src/bufio/bufio.go -> image/png.(*encoder).Write /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<text text-anchor="middle" x="466" y="-794.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N22 -->
	<g id="node23" class="node">
	<title>N22</title>
	<g id="a_node23"><a xlink:title="bufio.(*Writer).Write /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-981 422.5,-981 422.5,-928 483.5,-928 483.5,-981"></polygon>
	<text text-anchor="middle" x="453" y="-970.6" font-family="Times,serif" font-size="8.00" fill="#000000">bufio</text>
	<text text-anchor="middle" x="453" y="-961.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Writer)</text>
	<text text-anchor="middle" x="453" y="-952.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-943.6" font-family="Times,serif" font-size="8.00" fill="#000000">bufio.go</text>
	<text text-anchor="middle" x="453" y="-934.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N22&#45;&gt;N21 -->
	<g id="edge19" class="edge">
	<title>N22-&gt;N21</title>
	<g id="a_edge19"><a xlink:title="bufio.(*Writer).Write /Users/dfc/go/src/bufio/bufio.go -> bufio.(*Writer).Flush /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-927.9873C453,-915.7358 453,-900.9766 453,-887.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-887.3259 453,-877.3259 449.5001,-887.3259 456.5001,-887.3259"></polygon>
	</a>
	</g>
	<g id="a_edge19-label"><a xlink:title="bufio.(*Writer).Write /Users/dfc/go/src/bufio/bufio.go -> bufio.(*Writer).Flush /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<text text-anchor="middle" x="466" y="-898.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N23 -->
	<g id="node24" class="node">
	<title>N23</title>
	<g id="a_node24"><a xlink:title="compress/flate.(*Writer).Write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<polygon fill="#edeae7" stroke="#b2a289" points="486.5,-1977 425.5,-1977 425.5,-1924 486.5,-1924 486.5,-1977"></polygon>
	<text text-anchor="middle" x="456" y="-1966.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="456" y="-1957.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*Writer)</text>
	<text text-anchor="middle" x="456" y="-1948.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="456" y="-1939.6" font-family="Times,serif" font-size="8.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="456" y="-1930.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.07s (4.58%)</text>
	</a>
	</g>
	</g>
	<!-- N25 -->
	<g id="node26" class="node">
	<title>N25</title>
	<g id="a_node26"><a xlink:title="compress/flate.(*compressor).write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<polygon fill="#edeae7" stroke="#b2a289" points="483.5,-1858 422.5,-1858 422.5,-1805 483.5,-1805 483.5,-1858"></polygon>
	<text text-anchor="middle" x="453" y="-1847.6" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1838.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*compressor)</text>
	<text text-anchor="middle" x="453" y="-1829.6" font-family="Times,serif" font-size="8.00" fill="#000000">write</text>
	<text text-anchor="middle" x="453" y="-1820.6" font-family="Times,serif" font-size="8.00" fill="#000000">deflate.go</text>
	<text text-anchor="middle" x="453" y="-1811.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.07s (4.58%)</text>
	</a>
	</g>
	</g>
	<!-- N23&#45;&gt;N25 -->
	<g id="edge12" class="edge">
	<title>N23-&gt;N25</title>
	<g id="a_edge12"><a xlink:title="compress/flate.(*Writer).Write /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M455.3267,-1923.7941C454.9137,-1907.4111 454.3791,-1886.2042 453.9265,-1868.2506"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="457.422,-1868.0254 453.671,-1858.1168 450.4242,-1868.2019 457.422,-1868.0254"></polygon>
	</a>
	</g>
	<g id="a_edge12-label"><a xlink:title="compress/flate.(*Writer).Write /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<text text-anchor="middle" x="467" y="-1894.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N24&#45;&gt;N17 -->
	<g id="edge42" class="edge">
	<title>N24-&gt;N17</title>
	<g id="a_edge42"><a xlink:title="compress/flate.(*compressor).findMatch /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.matchLen /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M535.4893,-1560.9579C535.7405,-1547.3274 536.0506,-1530.5068 536.3243,-1515.658"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="539.8308,-1515.3309 536.5158,-1505.2681 532.832,-1515.2018 539.8308,-1515.3309"></polygon>
	</a>
	</g>
	<g id="a_edge42-label"><a xlink:title="compress/flate.(*compressor).findMatch /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.matchLen /Users/dfc/go/src/compress/flate/deflate.go (0.01s)">
	<text text-anchor="middle" x="549" y="-1531.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N25&#45;&gt;N11 -->
	<g id="edge13" class="edge">
	<title>N25-&gt;N11</title>
	<g id="a_edge13"><a xlink:title="compress/flate.(*compressor).write /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M453,-1804.9812C453,-1788.9977 453,-1768.1718 453,-1749.4341"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="456.5001,-1749.283 453,-1739.283 449.5001,-1749.2831 456.5001,-1749.283"></polygon>
	</a>
	</g>
	<g id="a_edge13-label"><a xlink:title="compress/flate.(*compressor).write /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*compressor).deflate /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<text text-anchor="middle" x="466" y="-1760.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N29 -->
	<g id="node30" class="node">
	<title>N29</title>
	<g id="a_node30"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBlock /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="488.5,-1505.5 417.5,-1505.5 417.5,-1452.5 488.5,-1452.5 488.5,-1505.5"></polygon>
	<text text-anchor="middle" x="453" y="-1495.1" font-family="Times,serif" font-size="8.00" fill="#000000">compress/flate</text>
	<text text-anchor="middle" x="453" y="-1486.1" font-family="Times,serif" font-size="8.00" fill="#000000">(*huffmanBitWriter)</text>
	<text text-anchor="middle" x="453" y="-1477.1" font-family="Times,serif" font-size="8.00" fill="#000000">writeBlock</text>
	<text text-anchor="middle" x="453" y="-1468.1" font-family="Times,serif" font-size="8.00" fill="#000000">huffman_bit_writer.go</text>
	<text text-anchor="middle" x="453" y="-1459.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N26&#45;&gt;N29 -->
	<g id="edge21" class="edge">
	<title>N26-&gt;N29</title>
	<g id="a_edge21"><a xlink:title="compress/flate.(*compressor).writeBlock /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*huffmanBitWriter).writeBlock /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-1560.9579C453,-1547.4283 453,-1530.7558 453,-1515.9883"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-1515.6441 453,-1505.6441 449.5001,-1515.6441 456.5001,-1515.6441"></polygon>
	</a>
	</g>
	<g id="a_edge21-label"><a xlink:title="compress/flate.(*compressor).writeBlock /Users/dfc/go/src/compress/flate/deflate.go -> compress/flate.(*huffmanBitWriter).writeBlock /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<text text-anchor="middle" x="466" y="-1531.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N27&#45;&gt;N22 -->
	<g id="edge22" class="edge">
	<title>N27-&gt;N22</title>
	<g id="a_edge22"><a xlink:title="compress/flate.(*dictWriter).Write /Users/dfc/go/src/compress/flate/deflate.go -> bufio.(*Writer).Write /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-1031.9873C453,-1019.7358 453,-1004.9766 453,-991.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-991.3259 453,-981.3259 449.5001,-991.3259 456.5001,-991.3259"></polygon>
	</a>
	</g>
	<g id="a_edge22-label"><a xlink:title="compress/flate.(*dictWriter).Write /Users/dfc/go/src/compress/flate/deflate.go -> bufio.(*Writer).Write /Users/dfc/go/src/bufio/bufio.go (0.05s)">
	<text text-anchor="middle" x="466" y="-1002.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N28&#45;&gt;N14 -->
	<g id="edge43" class="edge">
	<title>N28-&gt;N14</title>
	<g id="a_edge43"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBits /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M386.6887,-1239.9873C397.796,-1227.008 411.3119,-1211.2141 423.2364,-1197.2799"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="426.2005,-1199.1993 430.0432,-1189.3259 420.8821,-1194.648 426.2005,-1199.1993"></polygon>
	</a>
	</g>
	<g id="a_edge43-label"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBits /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.01s)">
	<text text-anchor="middle" x="426" y="-1210.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N29&#45;&gt;N15 -->
	<g id="edge24" class="edge">
	<title>N29-&gt;N15</title>
	<g id="a_edge24"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBlock /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-1452.4579C453,-1438.9283 453,-1422.2558 453,-1407.4883"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-1407.1441 453,-1397.1441 449.5001,-1407.1441 456.5001,-1407.1441"></polygon>
	</a>
	</g>
	<g id="a_edge24-label"><a xlink:title="compress/flate.(*huffmanBitWriter).writeBlock /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).writeTokens /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.05s)">
	<text text-anchor="middle" x="466" y="-1418.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N30&#45;&gt;N14 -->
	<g id="edge32" class="edge">
	<title>N30-&gt;N14</title>
	<g id="a_edge32"><a xlink:title="compress/flate.(*huffmanBitWriter).writeCode /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.04s)">
	<path fill="none" stroke="#b2aa9b" d="M453,-1239.9873C453,-1227.7358 453,-1212.9766 453,-1199.6407"></path>
	<polygon fill="#b2aa9b" stroke="#b2aa9b" points="456.5001,-1199.3259 453,-1189.3259 449.5001,-1199.3259 456.5001,-1199.3259"></polygon>
	</a>
	</g>
	<g id="a_edge32-label"><a xlink:title="compress/flate.(*huffmanBitWriter).writeCode /Users/dfc/go/src/compress/flate/huffman_bit_writer.go -> compress/flate.(*huffmanBitWriter).write /Users/dfc/go/src/compress/flate/huffman_bit_writer.go (0.04s)">
	<text text-anchor="middle" x="466" y="-1210.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.04s</text>
	</a>
	</g>
	</g>
	<!-- N31&#45;&gt;N23 -->
	<g id="edge14" class="edge">
	<title>N31-&gt;N23</title>
	<g id="a_edge14"><a xlink:title="compress/zlib.(*Writer).Write /Users/dfc/go/src/compress/zlib/writer.go -> compress/flate.(*Writer).Write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M457.4901,-2027.9873C457.2545,-2015.7358 456.9707,-2000.9766 456.7142,-1987.6407"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="460.2076,-1987.2567 456.5159,-1977.3259 453.2089,-1987.3914 460.2076,-1987.2567"></polygon>
	</a>
	</g>
	<g id="a_edge14-label"><a xlink:title="compress/zlib.(*Writer).Write /Users/dfc/go/src/compress/zlib/writer.go -> compress/flate.(*Writer).Write /Users/dfc/go/src/compress/flate/deflate.go (0.07s)">
	<text text-anchor="middle" x="470" y="-1998.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N33 -->
	<g id="node34" class="node">
	<title>N33</title>
	<g id="a_node34"><a xlink:title="image/png.(*encoder).writeChunk /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-667.5 422.5,-667.5 422.5,-614.5 483.5,-614.5 483.5,-667.5"></polygon>
	<text text-anchor="middle" x="453" y="-657.1" font-family="Times,serif" font-size="8.00" fill="#000000">image/png</text>
	<text text-anchor="middle" x="453" y="-648.1" font-family="Times,serif" font-size="8.00" fill="#000000">(*encoder)</text>
	<text text-anchor="middle" x="453" y="-639.1" font-family="Times,serif" font-size="8.00" fill="#000000">writeChunk</text>
	<text text-anchor="middle" x="453" y="-630.1" font-family="Times,serif" font-size="8.00" fill="#000000">writer.go</text>
	<text text-anchor="middle" x="453" y="-621.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N32&#45;&gt;N33 -->
	<g id="edge25" class="edge">
	<title>N32-&gt;N33</title>
	<g id="a_edge25"><a xlink:title="image/png.(*encoder).Write /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeChunk /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-719.8779C453,-707.1335 453,-691.6669 453,-677.8173"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-677.6136 453,-667.6136 449.5001,-677.6136 456.5001,-677.6136"></polygon>
	</a>
	</g>
	<g id="a_edge25-label"><a xlink:title="image/png.(*encoder).Write /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeChunk /Users/dfc/go/src/image/png/writer.go (0.05s)">
	<text text-anchor="middle" x="466" y="-690.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N39 -->
	<g id="node40" class="node">
	<title>N39</title>
	<g id="a_node40"><a xlink:title="os.(*File).Write /Users/dfc/go/src/os/file.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-562 422.5,-562 422.5,-509 483.5,-509 483.5,-562"></polygon>
	<text text-anchor="middle" x="453" y="-551.6" font-family="Times,serif" font-size="8.00" fill="#000000">os</text>
	<text text-anchor="middle" x="453" y="-542.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*File)</text>
	<text text-anchor="middle" x="453" y="-533.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-524.6" font-family="Times,serif" font-size="8.00" fill="#000000">file.go</text>
	<text text-anchor="middle" x="453" y="-515.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N33&#45;&gt;N39 -->
	<g id="edge26" class="edge">
	<title>N33-&gt;N39</title>
	<g id="a_edge26"><a xlink:title="image/png.(*encoder).writeChunk /Users/dfc/go/src/image/png/writer.go -> os.(*File).Write /Users/dfc/go/src/os/file.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-614.3779C453,-601.6335 453,-586.1669 453,-572.3173"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-572.1136 453,-562.1136 449.5001,-572.1136 456.5001,-572.1136"></polygon>
	</a>
	</g>
	<g id="a_edge26-label"><a xlink:title="image/png.(*encoder).writeChunk /Users/dfc/go/src/image/png/writer.go -> os.(*File).Write /Users/dfc/go/src/os/file.go (0.05s)">
	<text text-anchor="middle" x="466" y="-583.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N34&#45;&gt;N6 -->
	<g id="edge8" class="edge">
	<title>N34-&gt;N6</title>
	<g id="a_edge8"><a xlink:title="image/png.(*encoder).writeIDATs /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<path fill="none" stroke="#b27743" d="M467,-2265.3051C467,-2245.2046 467,-2217.4141 467,-2195.2196"></path>
	<polygon fill="#b27743" stroke="#b27743" points="470.5001,-2195.1486 467,-2185.1486 463.5001,-2195.1486 470.5001,-2195.1486"></polygon>
	</a>
	</g>
	<g id="a_edge8-label"><a xlink:title="image/png.(*encoder).writeIDATs /Users/dfc/go/src/image/png/writer.go -> image/png.(*encoder).writeImage /Users/dfc/go/src/image/png/writer.go (0.19s)">
	<text text-anchor="middle" x="480" y="-2206.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.19s</text>
	</a>
	</g>
	</g>
	<!-- N35&#45;&gt;N7 -->
	<g id="edge5" class="edge">
	<title>N35-&gt;N7</title>
	<g id="a_edge5"><a xlink:title="image/png.Encode /Users/dfc/go/src/image/png/writer.go -> image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<path fill="none" stroke="#b2591a" d="M454.6817,-2567.7419C456.8687,-2549.5896 460.0037,-2523.5695 462.5636,-2502.3221"></path>
	<polygon fill="#b2591a" stroke="#b2591a" points="466.0461,-2502.6772 463.7674,-2492.3303 459.0963,-2501.8398 466.0461,-2502.6772"></polygon>
	</a>
	</g>
	<g id="a_edge5-label"><a xlink:title="image/png.Encode /Users/dfc/go/src/image/png/writer.go -> image/png.(*Encoder).Encode /Users/dfc/go/src/image/png/writer.go (0.26s)">
	<text text-anchor="middle" x="471" y="-2538.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.26s</text>
	</a>
	</g>
	</g>
	<!-- N36&#45;&gt;N8 -->
	<g id="edge17" class="edge">
	<title>N36-&gt;N8</title>
	<g id="a_edge17"><a xlink:title="image/png.opaque /Users/dfc/go/src/image/png/writer.go -> main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (0.07s)">
	<path fill="none" stroke="#b2a289" d="M548,-2269.6566C548,-2228.6906 548,-2141.196 548,-2091.3206"></path>
	<polygon fill="#b2a289" stroke="#b2a289" points="551.5001,-2091.279 548,-2081.279 544.5001,-2091.279 551.5001,-2091.279"></polygon>
	</a>
	</g>
	<g id="a_edge17-label"><a xlink:title="image/png.opaque /Users/dfc/go/src/image/png/writer.go -> main.(*img).At /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (0.07s)">
	<text text-anchor="middle" x="561" y="-2154.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.07s</text>
	</a>
	</g>
	</g>
	<!-- N37 -->
	<g id="node38" class="node">
	<title>N37</title>
	<g id="a_node38"><a xlink:title="internal/poll.(*FD).Write /Users/dfc/go/src/internal/poll/fd_unix.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-354 422.5,-354 422.5,-301 483.5,-301 483.5,-354"></polygon>
	<text text-anchor="middle" x="453" y="-343.6" font-family="Times,serif" font-size="8.00" fill="#000000">internal/poll</text>
	<text text-anchor="middle" x="453" y="-334.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*FD)</text>
	<text text-anchor="middle" x="453" y="-325.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-316.6" font-family="Times,serif" font-size="8.00" fill="#000000">fd_unix.go</text>
	<text text-anchor="middle" x="453" y="-307.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N56 -->
	<g id="node57" class="node">
	<title>N56</title>
	<g id="a_node57"><a xlink:title="syscall.Write /Users/dfc/go/src/syscall/syscall_unix.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-250 422.5,-250 422.5,-206 483.5,-206 483.5,-250"></polygon>
	<text text-anchor="middle" x="453" y="-239.6" font-family="Times,serif" font-size="8.00" fill="#000000">syscall</text>
	<text text-anchor="middle" x="453" y="-230.6" font-family="Times,serif" font-size="8.00" fill="#000000">Write</text>
	<text text-anchor="middle" x="453" y="-221.6" font-family="Times,serif" font-size="8.00" fill="#000000">syscall_unix.go</text>
	<text text-anchor="middle" x="453" y="-212.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N37&#45;&gt;N56 -->
	<g id="edge27" class="edge">
	<title>N37-&gt;N56</title>
	<g id="a_edge27"><a xlink:title="internal/poll.(*FD).Write /Users/dfc/go/src/internal/poll/fd_unix.go -> syscall.Write /Users/dfc/go/src/syscall/syscall_unix.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-300.8358C453,-288.2993 453,-273.2498 453,-260.0933"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-260.0321 453,-250.0321 449.5001,-260.0322 456.5001,-260.0321"></polygon>
	</a>
	</g>
	<g id="a_edge27-label"><a xlink:title="internal/poll.(*FD).Write /Users/dfc/go/src/internal/poll/fd_unix.go -> syscall.Write /Users/dfc/go/src/syscall/syscall_unix.go (0.05s)">
	<text text-anchor="middle" x="466" y="-271.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N38&#45;&gt;N3 -->
	<g id="edge3" class="edge">
	<title>N38-&gt;N3</title>
	<g id="a_edge3"><a xlink:title="main.seqFillImg /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.fillPixel /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<path fill="none" stroke="#b20900" stroke-width="5" d="M356,-2567.7419C356,-2556.2626 356,-2541.6368 356,-2527.0613"></path>
	<polygon fill="#b20900" stroke="#b20900" stroke-width="5" points="360.3751,-2527.0179 356,-2517.0179 351.6251,-2527.018 360.3751,-2527.0179"></polygon>
	</a>
	</g>
	<g id="a_edge3-label"><a xlink:title="main.seqFillImg /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go -> main.fillPixel /Users/dfc/devel/understanding-the-execution-tracer/examples/mandelbrot-pkg-profile/mandelbrot.go (1.27s)">
	<text text-anchor="middle" x="369" y="-2538.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 1.27s</text>
	</a>
	</g>
	</g>
	<!-- N40 -->
	<g id="node41" class="node">
	<title>N40</title>
	<g id="a_node41"><a xlink:title="os.(*File).write /Users/dfc/go/src/os/file_unix.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="483.5,-458 422.5,-458 422.5,-405 483.5,-405 483.5,-458"></polygon>
	<text text-anchor="middle" x="453" y="-447.6" font-family="Times,serif" font-size="8.00" fill="#000000">os</text>
	<text text-anchor="middle" x="453" y="-438.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*File)</text>
	<text text-anchor="middle" x="453" y="-429.6" font-family="Times,serif" font-size="8.00" fill="#000000">write</text>
	<text text-anchor="middle" x="453" y="-420.6" font-family="Times,serif" font-size="8.00" fill="#000000">file_unix.go</text>
	<text text-anchor="middle" x="453" y="-411.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N39&#45;&gt;N40 -->
	<g id="edge28" class="edge">
	<title>N39-&gt;N40</title>
	<g id="a_edge28"><a xlink:title="os.(*File).Write /Users/dfc/go/src/os/file.go -> os.(*File).write /Users/dfc/go/src/os/file_unix.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-508.9873C453,-496.7358 453,-481.9766 453,-468.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-468.3259 453,-458.3259 449.5001,-468.3259 456.5001,-468.3259"></polygon>
	</a>
	</g>
	<g id="a_edge28-label"><a xlink:title="os.(*File).Write /Users/dfc/go/src/os/file.go -> os.(*File).write /Users/dfc/go/src/os/file_unix.go (0.05s)">
	<text text-anchor="middle" x="466" y="-479.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N40&#45;&gt;N37 -->
	<g id="edge29" class="edge">
	<title>N40-&gt;N37</title>
	<g id="a_edge29"><a xlink:title="os.(*File).write /Users/dfc/go/src/os/file_unix.go -> internal/poll.(*FD).Write /Users/dfc/go/src/internal/poll/fd_unix.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-404.9873C453,-392.7358 453,-377.9766 453,-364.6407"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-364.3259 453,-354.3259 449.5001,-364.3259 456.5001,-364.3259"></polygon>
	</a>
	</g>
	<g id="a_edge29-label"><a xlink:title="os.(*File).write /Users/dfc/go/src/os/file_unix.go -> internal/poll.(*FD).Write /Users/dfc/go/src/internal/poll/fd_unix.go (0.05s)">
	<text text-anchor="middle" x="466" y="-375.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N41&#45;&gt;N16 -->
	<g id="edge34" class="edge">
	<title>N41-&gt;N16</title>
	<g id="a_edge34"><a xlink:title="runtime.(*mcache).nextFree /Users/dfc/go/src/runtime/malloc.go -> runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s (0.03s)">
	<path fill="none" stroke="#b2ada1" d="M580.6689,-1675.4436C587.4319,-1658.6098 596.2142,-1636.7493 603.3309,-1619.035"></path>
	<polygon fill="#b2ada1" stroke="#b2ada1" points="606.6096,-1620.2625 607.0898,-1609.6785 600.1142,-1617.6529 606.6096,-1620.2625"></polygon>
	</a>
	</g>
	<g id="a_edge34-label"><a xlink:title="runtime.(*mcache).nextFree /Users/dfc/go/src/runtime/malloc.go -> runtime.systemstack /Users/dfc/go/src/runtime/asm_amd64.s (0.03s)">
	<text text-anchor="middle" x="610" y="-1635.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.03s</text>
	</a>
	</g>
	</g>
	<!-- N43 -->
	<g id="node44" class="node">
	<title>N43</title>
	<g id="a_node44"><a xlink:title="runtime.(*mcache).refill /Users/dfc/go/src/runtime/mcache.go (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="725.5,-1397 664.5,-1397 664.5,-1344 725.5,-1344 725.5,-1397"></polygon>
	<text text-anchor="middle" x="695" y="-1386.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-1377.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mcache)</text>
	<text text-anchor="middle" x="695" y="-1368.6" font-family="Times,serif" font-size="8.00" fill="#000000">refill</text>
	<text text-anchor="middle" x="695" y="-1359.6" font-family="Times,serif" font-size="8.00" fill="#000000">mcache.go</text>
	<text text-anchor="middle" x="695" y="-1350.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N42&#45;&gt;N43 -->
	<g id="edge36" class="edge">
	<title>N42-&gt;N43</title>
	<g id="a_edge36"><a xlink:title="runtime.(*mcache).nextFree.func1 /Users/dfc/go/src/runtime/malloc.go -> runtime.(*mcache).refill /Users/dfc/go/src/runtime/mcache.go (0.02s)">
	<path fill="none" stroke="#b2afa6" d="M695,-1447.9083C695,-1435.259 695,-1420.5418 695,-1407.3573"></path>
	<polygon fill="#b2afa6" stroke="#b2afa6" points="698.5001,-1407.181 695,-1397.181 691.5001,-1407.1811 698.5001,-1407.181"></polygon>
	</a>
	</g>
	<g id="a_edge36-label"><a xlink:title="runtime.(*mcache).nextFree.func1 /Users/dfc/go/src/runtime/malloc.go -> runtime.(*mcache).refill /Users/dfc/go/src/runtime/mcache.go (0.02s)">
	<text text-anchor="middle" x="708" y="-1418.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.02s</text>
	</a>
	</g>
	</g>
	<!-- N44 -->
	<g id="node45" class="node">
	<title>N44</title>
	<g id="a_node45"><a xlink:title="runtime.(*mcentral).cacheSpan /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<polygon fill="#edeceb" stroke="#b2afa6" points="725.5,-1293 664.5,-1293 664.5,-1240 725.5,-1240 725.5,-1293"></polygon>
	<text text-anchor="middle" x="695" y="-1282.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="695" y="-1273.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mcentral)</text>
	<text text-anchor="middle" x="695" y="-1264.6" font-family="Times,serif" font-size="8.00" fill="#000000">cacheSpan</text>
	<text text-anchor="middle" x="695" y="-1255.6" font-family="Times,serif" font-size="8.00" fill="#000000">mcentral.go</text>
	<text text-anchor="middle" x="695" y="-1246.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.02s (1.31%)</text>
	</a>
	</g>
	</g>
	<!-- N43&#45;&gt;N44 -->
	<g id="edge37" class="edge">
	<title>N43-&gt;N44</title>
	<g id="a_edge37"><a xlink:title="runtime.(*mcache).refill /Users/dfc/go/src/runtime/mcache.go -> runtime.(*mcentral).cacheSpan /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<path fill="none" stroke="#b2afa6" d="M695,-1343.9873C695,-1331.7358 695,-1316.9766 695,-1303.6407"></path>
	<polygon fill="#b2afa6" stroke="#b2afa6" points="698.5001,-1303.3259 695,-1293.3259 691.5001,-1303.3259 698.5001,-1303.3259"></polygon>
	</a>
	</g>
	<g id="a_edge37-label"><a xlink:title="runtime.(*mcache).refill /Users/dfc/go/src/runtime/mcache.go -> runtime.(*mcentral).cacheSpan /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<text text-anchor="middle" x="708" y="-1314.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.02s</text>
	</a>
	</g>
	</g>
	<!-- N44&#45;&gt;N19 -->
	<g id="edge38" class="edge">
	<title>N44-&gt;N19</title>
	<g id="a_edge38"><a xlink:title="runtime.(*mcentral).cacheSpan /Users/dfc/go/src/runtime/mcentral.go -> runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<path fill="none" stroke="#b2afa6" d="M695,-1239.9873C695,-1227.7358 695,-1212.9766 695,-1199.6407"></path>
	<polygon fill="#b2afa6" stroke="#b2afa6" points="698.5001,-1199.3259 695,-1189.3259 691.5001,-1199.3259 698.5001,-1199.3259"></polygon>
	</a>
	</g>
	<g id="a_edge38-label"><a xlink:title="runtime.(*mcentral).cacheSpan /Users/dfc/go/src/runtime/mcentral.go -> runtime.(*mcentral).grow /Users/dfc/go/src/runtime/mcentral.go (0.02s)">
	<text text-anchor="middle" x="708" y="-1210.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.02s</text>
	</a>
	</g>
	</g>
	<!-- N45&#45;&gt;N20 -->
	<g id="edge48" class="edge">
	<title>N45-&gt;N20</title>
	<g id="a_edge48"><a xlink:title="runtime.(*mheap).alloc /Users/dfc/go/src/runtime/mheap.go -> runtime.memclrNoHeapPointers /Users/dfc/go/src/runtime/memclr_amd64.s (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M695,-1031.9873C695,-1019.4227 695,-1004.2205 695,-990.621"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="698.5001,-990.6041 695,-980.6041 691.5001,-990.6041 698.5001,-990.6041"></polygon>
	</a>
	</g>
	<g id="a_edge48-label"><a xlink:title="runtime.(*mheap).alloc /Users/dfc/go/src/runtime/mheap.go -> runtime.memclrNoHeapPointers /Users/dfc/go/src/runtime/memclr_amd64.s (0.01s)">
	<text text-anchor="middle" x="708" y="-1002.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N48 -->
	<g id="node49" class="node">
	<title>N48</title>
	<g id="a_node49"><a xlink:title="runtime.(*mheap).alloc_m /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="645.5,-1397 584.5,-1397 584.5,-1344 645.5,-1344 645.5,-1397"></polygon>
	<text text-anchor="middle" x="615" y="-1386.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="615" y="-1377.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="615" y="-1368.6" font-family="Times,serif" font-size="8.00" fill="#000000">alloc_m</text>
	<text text-anchor="middle" x="615" y="-1359.6" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="615" y="-1350.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N46&#45;&gt;N48 -->
	<g id="edge49" class="edge">
	<title>N46-&gt;N48</title>
	<g id="a_edge49"><a xlink:title="runtime.(*mheap).alloc.func1 /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).alloc_m /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M615.7134,-1447.9083C615.5969,-1435.259 615.4612,-1420.5418 615.3397,-1407.3573"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="618.838,-1407.1483 615.2459,-1397.181 611.8383,-1407.2129 618.838,-1407.1483"></polygon>
	</a>
	</g>
	<g id="a_edge49-label"><a xlink:title="runtime.(*mheap).alloc.func1 /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).alloc_m /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="628" y="-1418.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N47 -->
	<g id="node48" class="node">
	<title>N47</title>
	<g id="a_node48"><a xlink:title="runtime.(*mheap).allocSpanLocked /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="643.5,-1293 582.5,-1293 582.5,-1240 643.5,-1240 643.5,-1293"></polygon>
	<text text-anchor="middle" x="613" y="-1282.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="613" y="-1273.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="613" y="-1264.6" font-family="Times,serif" font-size="8.00" fill="#000000">allocSpanLocked</text>
	<text text-anchor="middle" x="613" y="-1255.6" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="613" y="-1246.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N49 -->
	<g id="node50" class="node">
	<title>N49</title>
	<g id="a_node50"><a xlink:title="runtime.(*mheap).grow /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="634.5,-1189 573.5,-1189 573.5,-1136 634.5,-1136 634.5,-1189"></polygon>
	<text text-anchor="middle" x="604" y="-1178.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="604" y="-1169.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="604" y="-1160.6" font-family="Times,serif" font-size="8.00" fill="#000000">grow</text>
	<text text-anchor="middle" x="604" y="-1151.6" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="604" y="-1142.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N47&#45;&gt;N49 -->
	<g id="edge50" class="edge">
	<title>N47-&gt;N49</title>
	<g id="a_edge50"><a xlink:title="runtime.(*mheap).allocSpanLocked /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).grow /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M610.7056,-1239.9873C609.6454,-1227.7358 608.3682,-1212.9766 607.2141,-1199.6407"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="610.6707,-1198.9869 606.3215,-1189.3259 603.6968,-1199.5904 610.6707,-1198.9869"></polygon>
	</a>
	</g>
	<g id="a_edge50-label"><a xlink:title="runtime.(*mheap).allocSpanLocked /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).grow /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="622" y="-1210.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N48&#45;&gt;N47 -->
	<g id="edge51" class="edge">
	<title>N48-&gt;N47</title>
	<g id="a_edge51"><a xlink:title="runtime.(*mheap).alloc_m /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).allocSpanLocked /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M614.4901,-1343.9873C614.2545,-1331.7358 613.9707,-1316.9766 613.7142,-1303.6407"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="617.2076,-1303.2567 613.5159,-1293.3259 610.2089,-1303.3914 617.2076,-1303.2567"></polygon>
	</a>
	</g>
	<g id="a_edge51-label"><a xlink:title="runtime.(*mheap).alloc_m /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).allocSpanLocked /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="627" y="-1314.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N50 -->
	<g id="node51" class="node">
	<title>N50</title>
	<g id="a_node51"><a xlink:title="runtime.(*mheap).sysAlloc /Users/dfc/go/src/runtime/malloc.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="633.5,-1085 572.5,-1085 572.5,-1032 633.5,-1032 633.5,-1085"></polygon>
	<text text-anchor="middle" x="603" y="-1074.6" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="603" y="-1065.6" font-family="Times,serif" font-size="8.00" fill="#000000">(*mheap)</text>
	<text text-anchor="middle" x="603" y="-1056.6" font-family="Times,serif" font-size="8.00" fill="#000000">sysAlloc</text>
	<text text-anchor="middle" x="603" y="-1047.6" font-family="Times,serif" font-size="8.00" fill="#000000">malloc.go</text>
	<text text-anchor="middle" x="603" y="-1038.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N49&#45;&gt;N50 -->
	<g id="edge52" class="edge">
	<title>N49-&gt;N50</title>
	<g id="a_edge52"><a xlink:title="runtime.(*mheap).grow /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).sysAlloc /Users/dfc/go/src/runtime/malloc.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M603.7451,-1135.9873C603.6273,-1123.7358 603.4854,-1108.9766 603.3571,-1095.6407"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="606.854,-1095.2917 603.2579,-1085.3259 599.8543,-1095.3591 606.854,-1095.2917"></polygon>
	</a>
	</g>
	<g id="a_edge52-label"><a xlink:title="runtime.(*mheap).grow /Users/dfc/go/src/runtime/mheap.go -> runtime.(*mheap).sysAlloc /Users/dfc/go/src/runtime/malloc.go (0.01s)">
	<text text-anchor="middle" x="616" y="-1106.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N55 -->
	<g id="node56" class="node">
	<title>N55</title>
	<g id="a_node56"><a xlink:title="runtime.sysMap /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="632.5,-976.5 571.5,-976.5 571.5,-932.5 632.5,-932.5 632.5,-976.5"></polygon>
	<text text-anchor="middle" x="602" y="-966.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="602" y="-957.1" font-family="Times,serif" font-size="8.00" fill="#000000">sysMap</text>
	<text text-anchor="middle" x="602" y="-948.1" font-family="Times,serif" font-size="8.00" fill="#000000">mem_darwin.go</text>
	<text text-anchor="middle" x="602" y="-939.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N50&#45;&gt;N55 -->
	<g id="edge53" class="edge">
	<title>N50-&gt;N55</title>
	<g id="a_edge53"><a xlink:title="runtime.(*mheap).sysAlloc /Users/dfc/go/src/runtime/malloc.go -> runtime.sysMap /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M602.7451,-1031.9873C602.6128,-1018.2341 602.4502,-1001.3207 602.3107,-986.8083"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="605.81,-986.7107 602.2139,-976.7448 598.8103,-986.778 605.81,-986.7107"></polygon>
	</a>
	</g>
	<g id="a_edge53-label"><a xlink:title="runtime.(*mheap).sysAlloc /Users/dfc/go/src/runtime/malloc.go -> runtime.sysMap /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<text text-anchor="middle" x="615" y="-1002.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N53 -->
	<g id="node54" class="node">
	<title>N53</title>
	<g id="a_node54"><a xlink:title="runtime.newMarkBits /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="818.5,-976.5 757.5,-976.5 757.5,-932.5 818.5,-932.5 818.5,-976.5"></polygon>
	<text text-anchor="middle" x="788" y="-966.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="788" y="-957.1" font-family="Times,serif" font-size="8.00" fill="#000000">newMarkBits</text>
	<text text-anchor="middle" x="788" y="-948.1" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="788" y="-939.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N51&#45;&gt;N53 -->
	<g id="edge54" class="edge">
	<title>N51-&gt;N53</title>
	<g id="a_edge54"><a xlink:title="runtime.heapBits.initSpan /Users/dfc/go/src/runtime/mbitmap.go -> runtime.newMarkBits /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M782.7845,-1031.9873C783.7102,-1018.2341 784.8486,-1001.3207 785.8254,-986.8083"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="789.3232,-986.9573 786.5028,-976.7448 782.339,-986.4872 789.3232,-986.9573"></polygon>
	</a>
	</g>
	<g id="a_edge54-label"><a xlink:title="runtime.heapBits.initSpan /Users/dfc/go/src/runtime/mbitmap.go -> runtime.newMarkBits /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="797" y="-1002.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N52 -->
	<g id="node53" class="node">
	<title>N52</title>
	<g id="a_node53"><a xlink:title="runtime.newArenaMayUnlock /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="776,-872.5 706,-872.5 706,-828.5 776,-828.5 776,-872.5"></polygon>
	<text text-anchor="middle" x="741" y="-862.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="741" y="-853.1" font-family="Times,serif" font-size="8.00" fill="#000000">newArenaMayUnlock</text>
	<text text-anchor="middle" x="741" y="-844.1" font-family="Times,serif" font-size="8.00" fill="#000000">mheap.go</text>
	<text text-anchor="middle" x="741" y="-835.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N54 -->
	<g id="node55" class="node">
	<title>N54</title>
	<g id="a_node55"><a xlink:title="runtime.sysAlloc /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<polygon fill="#edecec" stroke="#b2b1ac" points="708.5,-768.5 647.5,-768.5 647.5,-724.5 708.5,-724.5 708.5,-768.5"></polygon>
	<text text-anchor="middle" x="678" y="-758.1" font-family="Times,serif" font-size="8.00" fill="#000000">runtime</text>
	<text text-anchor="middle" x="678" y="-749.1" font-family="Times,serif" font-size="8.00" fill="#000000">sysAlloc</text>
	<text text-anchor="middle" x="678" y="-740.1" font-family="Times,serif" font-size="8.00" fill="#000000">mem_darwin.go</text>
	<text text-anchor="middle" x="678" y="-731.1" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.01s (0.65%)</text>
	</a>
	</g>
	</g>
	<!-- N52&#45;&gt;N54 -->
	<g id="edge55" class="edge">
	<title>N52-&gt;N54</title>
	<g id="a_edge55"><a xlink:title="runtime.newArenaMayUnlock /Users/dfc/go/src/runtime/mheap.go -> runtime.sysAlloc /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M727.6393,-828.4443C718.6185,-813.5528 706.587,-793.6913 696.6284,-777.2517"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="699.5162,-775.2636 691.3414,-768.5239 693.5291,-778.8905 699.5162,-775.2636"></polygon>
	</a>
	</g>
	<g id="a_edge55-label"><a xlink:title="runtime.newArenaMayUnlock /Users/dfc/go/src/runtime/mheap.go -> runtime.sysAlloc /Users/dfc/go/src/runtime/mem_darwin.go (0.01s)">
	<text text-anchor="middle" x="726" y="-794.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N53&#45;&gt;N52 -->
	<g id="edge56" class="edge">
	<title>N53-&gt;N52</title>
	<g id="a_edge56"><a xlink:title="runtime.newMarkBits /Users/dfc/go/src/runtime/mheap.go -> runtime.newArenaMayUnlock /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M778.0325,-932.4443C771.365,-917.6907 762.4928,-898.0586 755.1042,-881.7092"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="758.2608,-880.1952 750.9531,-872.5239 751.882,-883.078 758.2608,-880.1952"></polygon>
	</a>
	</g>
	<g id="a_edge56-label"><a xlink:title="runtime.newMarkBits /Users/dfc/go/src/runtime/mheap.go -> runtime.newArenaMayUnlock /Users/dfc/go/src/runtime/mheap.go (0.01s)">
	<text text-anchor="middle" x="780" y="-898.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N54&#45;&gt;N12 -->
	<g id="edge57" class="edge">
	<title>N54-&gt;N12</title>
	<g id="a_edge57"><a xlink:title="runtime.sysAlloc /Users/dfc/go/src/runtime/mem_darwin.go -> runtime.mmap /Users/dfc/go/src/runtime/sys_darwin_amd64.s (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M670.034,-724.3837C665.2851,-711.1995 659.129,-694.1081 653.6276,-678.8345"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="656.8101,-677.3416 650.1283,-669.1194 650.2242,-679.7138 656.8101,-677.3416"></polygon>
	</a>
	</g>
	<g id="a_edge57-label"><a xlink:title="runtime.sysAlloc /Users/dfc/go/src/runtime/mem_darwin.go -> runtime.mmap /Users/dfc/go/src/runtime/sys_darwin_amd64.s (0.01s)">
	<text text-anchor="middle" x="674" y="-690.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N55&#45;&gt;N12 -->
	<g id="edge58" class="edge">
	<title>N55-&gt;N12</title>
	<g id="a_edge58"><a xlink:title="runtime.sysMap /Users/dfc/go/src/runtime/mem_darwin.go -> runtime.mmap /Users/dfc/go/src/runtime/sys_darwin_amd64.s (0.01s)">
	<path fill="none" stroke="#b2b1ac" d="M604.6721,-932.4553C611.0704,-879.6692 627.3338,-745.4958 635.3664,-679.2272"></path>
	<polygon fill="#b2b1ac" stroke="#b2b1ac" points="638.8533,-679.546 636.5821,-669.1974 631.9042,-678.7036 638.8533,-679.546"></polygon>
	</a>
	</g>
	<g id="a_edge58-label"><a xlink:title="runtime.sysMap /Users/dfc/go/src/runtime/mem_darwin.go -> runtime.mmap /Users/dfc/go/src/runtime/sys_darwin_amd64.s (0.01s)">
	<text text-anchor="middle" x="634" y="-794.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.01s</text>
	</a>
	</g>
	</g>
	<!-- N57 -->
	<g id="node58" class="node">
	<title>N57</title>
	<g id="a_node58"><a xlink:title="syscall.write /Users/dfc/go/src/syscall/zsyscall_darwin_amd64.go (0.05s)">
	<polygon fill="#edebe9" stroke="#b2a895" points="494,-155 412,-155 412,-111 494,-111 494,-155"></polygon>
	<text text-anchor="middle" x="453" y="-144.6" font-family="Times,serif" font-size="8.00" fill="#000000">syscall</text>
	<text text-anchor="middle" x="453" y="-135.6" font-family="Times,serif" font-size="8.00" fill="#000000">write</text>
	<text text-anchor="middle" x="453" y="-126.6" font-family="Times,serif" font-size="8.00" fill="#000000">zsyscall_darwin_amd64.go</text>
	<text text-anchor="middle" x="453" y="-117.6" font-family="Times,serif" font-size="8.00" fill="#000000">0 of 0.05s (3.27%)</text>
	</a>
	</g>
	</g>
	<!-- N56&#45;&gt;N57 -->
	<g id="edge30" class="edge">
	<title>N56-&gt;N57</title>
	<g id="a_edge30"><a xlink:title="syscall.Write /Users/dfc/go/src/syscall/syscall_unix.go -> syscall.write /Users/dfc/go/src/syscall/zsyscall_darwin_amd64.go (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-205.9663C453,-193.9427 453,-178.8283 453,-165.4785"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-165.2433 453,-155.2433 449.5001,-165.2433 456.5001,-165.2433"></polygon>
	</a>
	</g>
	<g id="a_edge30-label"><a xlink:title="syscall.Write /Users/dfc/go/src/syscall/syscall_unix.go -> syscall.write /Users/dfc/go/src/syscall/zsyscall_darwin_amd64.go (0.05s)">
	<text text-anchor="middle" x="466" y="-176.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	<!-- N57&#45;&gt;N9 -->
	<g id="edge31" class="edge">
	<title>N57-&gt;N9</title>
	<g id="a_edge31"><a xlink:title="syscall.write /Users/dfc/go/src/syscall/zsyscall_darwin_amd64.go -> syscall.Syscall /Users/dfc/go/src/syscall/asm_darwin_amd64.s (0.05s)">
	<path fill="none" stroke="#b2a895" d="M453,-110.9039C453,-99.1279 453,-84.2673 453,-70.5053"></path>
	<polygon fill="#b2a895" stroke="#b2a895" points="456.5001,-70.2649 453,-60.2649 449.5001,-70.265 456.5001,-70.2649"></polygon>
	</a>
	</g>
	<g id="a_edge31-label"><a xlink:title="syscall.write /Users/dfc/go/src/syscall/zsyscall_darwin_amd64.go -> syscall.Syscall /Users/dfc/go/src/syscall/asm_darwin_amd64.s (0.05s)">
	<text text-anchor="middle" x="466" y="-81.8" font-family="Times,serif" font-size="14.00" fill="#000000"> 0.05s</text>
	</a>
	</g>
	</g>
	</g>
	</g></svg>
	</div>
{{< /notice >}}

## 5.4 트레이싱과 프로파일링

이 예제가 프로파일링의 한계를 보여주길 바랍니다. 프로파일링은 프로파일러가 본 내용을 알려줍니다. `fillPixel`은 모든 작업을 수행했습니다. 프로파일링이 그에 대해 할 수 있는 일이 많지 않은 듯 보입니다.

이제 동일한 프로그램에 대한 다른 관점을 제공하는 엑스큐션 트레이서 프로그램을 소개하는 것이 좋을 듯합니다.

### 5.4.1 엑스큐션 트레이서 사용하기

트레이서를 사용하는 것은 <profile.TraceProfile>을 호출하는 것이 전부입니다. 바뀌는 게 없습니다.

```golang

import "github.com/pkg/profile"

func main() {
	defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
```

프로그램을 실행하면 현재 작업 디렉토리에 `trace.out` 파일이 생성됩니다.

```sh
% go build mandelbrot.go
% % time ./mandelbrot
2017/09/17 13:19:10 profile: trace enabled, trace.out
2017/09/17 13:19:12 profile: trace disabled, trace.out

real    0m1.740s
user    0m1.707s
sys     0m0.020s
```

pprof와 마찬가지로 go 명령에는 트레이싱을 분석하는 도구가 있습니다.

```sh
% go tool trace trace.out
2017/09/17 12:41:39 Parsing trace...
2017/09/17 12:41:40 Serializing trace...
2017/09/17 12:41:40 Splitting trace...
2017/09/17 12:41:40 Opening browser. Trace viewer s listening on http://127.0.0.1:57842
```

이 도구는 `go tool pprof`와 약간 다릅니다. 엑스큐션 트레이서 프로그램은 크롬에 내장된 많은 프로파일 시각화 인프라를 재사용하므로 `go tool trace`은 서버로 작동하여 원시 엑스큐션 트레이싱을 크롬이 기본적으로 표시 할 수 있는 데이터로 변환합니다.

### 5.4.2 트레이싱 분석하기

트레이싱에서 프로그램이 하나의 cpu만 사용하고 있다는 것을 알 수 있습니다.

```golang
func seqFillImg(m *img) {
	for i, row := range m.m {
		for j := range row {
			fillPixel(m, i, j)
		}
	}
}
```

기본적으로 `mandelbrot.go`는 각 행의 각 픽셀에 대해 `fillPixel`을 순서대로 호출합니다.

이미지가 페인트되면 엑스큐션 스위치가 `.png` 파일 쓰기로 전환되는 것을 참고하세요. 이는 힙에 가비지를 생성하여 해당 시점에서 트레이싱가 변경되므로 가비지 콜렉터 힙의 고전적인 톱니 패턴을 볼 수 있습니다.

트레이싱 프로파일은 마이크로초 수준까지 타이밍 레졸루션을 제공합니다. 이는 외부 프로파일링으로는 얻을 수 없는 것입니다.

{{< notice note >}}
> **go tool trace**
 
계속하기 전에 추적 도구의 사용법에 대해 이야기해야 할 것이 있습니다.

- 이 도구는 크롬에 내장된 자바 스크립트 디버깅 지원을 사용합니다. 트레이싱 프로필은 크롬에서만 볼 수 있으며 Firefox, Safari, IE Edge에서는 작동하지 않습니다. 쏘리.
- 이 제품은 Google 제품이므로 키보드 단축키를 지원합니다. `WASD`를 사용하여 탐색하고, `?` 사용해서 목록을 얻을 수 있습니다.
- 트레이싱를 보려면 많은 메모리가 필요할 수 있습니다. 실제로 4Gb는 부족합니다. 8Gb가 아마 최소 수준일 것입니다.
- Fedora와 같은 OS 배포판에서 Go를 설치한 경우 트레이싱 뷰어의 지원 파일이 기본 `golang` deb/rpm의 일부가 아닌 어떤 `-extra` 패키지에 있을 수 있습니다.
{{< /notice >}}

## 5.5 하나 이상의 CPU 사용하기

이전 트레이싱에서 프로그램이 순차적으로 실행되고 있으며, 이 시스템의 다른 CPU를 활용하지 않는 것으로 확인했습니다.

만델브로트 생성은 `embarassingly_parallel`로 알려져 있습니다. 각 픽셀은 다른 픽셀과 독립적이며 모두 병렬로 계산할 수 있습니다. 자, 그럼 한번 해보죠.

```sh
% go build mandelbrot.go
% time ./mandelbrot -mode px
2017/09/17 13:19:48 profile: trace enabled, trace.out
2017/09/17 13:19:50 profile: trace disabled, trace.out

real    0m1.764s
user    0m4.031s
sys     0m0.865s
```

실행 시간은 기본적으로 동일했습니다. 더 많은 `user` 시간이 있었는데, 말이 됩니다. 모든 CPU를 사용하고 있었는데도, real (wall clock, 벽시계) 시간은 거의 똑같습니다.

트레이싱를 한번 볼까요.

보시다시피, 이 추적은 훨씬 더 많은 데이터를 생성했습니다.

- 많은 작업이 완료된 것 같지만 확대하면 약간의 차이가 있습니다. 이것은 스케줄러로 여겨집니다.
- 4개의 코어를 모두 사용하는 동안 각 `fillPixel`은 상대적으로 적은 양의 작업이므로 오버헤드를 예약하는 데 많은 시간을 소비하고 있습니다.

## 5.6 작업 배치하기

픽셀 당 하나의 고루틴을 사용하는 것은 매우 세분화된 작업입니다. 고루틴 비용을 정당화할 수 있는 작업이 충분하지 않았습니다.

대신에, 고루틴 한 개당 한 행씩 처리해 봅시다.

```sh
% go build mandelbrot.go
% time ./mandelbrot -mode row
2017/09/17 13:41:55 profile: trace enabled, trace.out
2017/09/17 13:41:55 profile: trace disabled, trace.out

real    0m0.764s
user    0m1.907s
sys     0m0.025s
```

좋게 개선되었습니다. 프로그램의 실행 시간을 거의 절반으로 줄였습니다. 트레이싱를 봅시다.

보시다시피 이제 트레이싱는 더 작고 작업하기가 더 쉽습니다. 우리는 전체 트레이싱를 볼 수 있는데, 이는 멋진 보너스입니다.

- 프로그램이 시작될 때 고루틴의 수가 최대 약 1,000 개로 증가하는 것을 볼 수 있습니다. 이것은 이전 트레이싱에서 보았던 1 << 20에 비해 개선된 것입니다.
- 확대하면 `onePerRowFillImg`이 더 오래 실행되는 것을 볼 수 있으며 고루틴 생성 작업이 조기에 완료되므로 스케줄러는 나머지 실행 가능한 고루틴을 효율적으로 처리합니다.

## 5.7 워커 사용하기

`mandelbrot.go`는 다른 모드를 지원하는데요, 해봅시다.

```sh
% go build mandelbrot.go
% time ./mandelbrot -mode workers
2017/09/17 13:49:46 profile: trace enabled, trace.out
2017/09/17 13:49:50 profile: trace disabled, trace.out

real    0m4.207s
user    0m4.459s
sys     0m1.284s
```

자, 실행시간은 이전보다 훨씬 나쁩니다. 트레이싱를 보고 무슨 일이 있었는지 봅시다.

트레이싱를 살펴보면 하나뿐인 워커 프로세스가, 프로듀서 하나와 컨슈머 하나가 있으므로, 프로듀서와 컨슈머를 교대하는 경향이 있다는 것을 알 수 있습니다. 워커 수를 늘려 봅시다.

```sh
% go build mandelbrot.go
% time ./mandelbrot -mode workers -workers 4
2017/09/17 13:52:51 profile: trace enabled, trace.out
2017/09/17 13:52:57 profile: trace disabled, trace.out

real    0m5.528s
user    0m7.307s
sys     0m4.311s
```

자, 더 나빠졌습니다! real 시간이 늘어났고, CPU 시간도 늘어났습니다. 무슨 일이 있었는지 트레이싱를 살펴 봅시다.

트레이싱가 엉망입니다. 더 많은 워커들이 있지만, 그 일을 하려고 싸우는 데 모든 시간을 보내는 것처럼 보입니다.

이는 채널이 언버퍼드(unbuffered)이기 때문입니다. 버퍼링되지 않은 채널은 수신할 준비가 된 사람이 있을 때까지 보낼 수 없습니다.

- 프로듀셔는 수신할 준비가 된 워커가 있을 때까지 작업을 보낼 수 없습니다.
- 워커들은 누군가 보낼 준비가 될 때까지 작업을 받을 수 없기 때문에 기다릴 때 서로 경쟁합니다.
- 송신자는 권한이 없으므로 이미 실행 중인 워커보다 우선 순위를 가질 수 없습니다.

여기서 볼 수 있는 것은 버퍼되지 않은 채널에 의해 도입되는 많은 지연 시간입니다. 스케줄러 내부에는 많은 중지 및 시작이 있으며 작업을 기다리는 동안 잠재적으로 잠금 및 뮤텍스가 발생할 수 있으며, 이는 `sys` 시간이 더 높은 이유입니다.

## 5.8 버퍼드 채널 사용하기

```golang
import "github.com/pkg/profile"

func main() {
	defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
```

```sh
% go build mandelbrot.go
% time ./mandelbrot -mode workers -workers 4
2017/09/17 14:23:56 profile: trace enabled, trace.out
2017/09/17 14:23:57 profile: trace disabled, trace.out

real    0m0.905s
user    0m2.150s
sys     0m0.121s
```

위의 row당 모드와 매우 유사합니다.

버퍼드(buffered) 채널을 사용한 트레이싱 결과는 다음과 같습니다.

- 프로듀서는 워커가 도착할 때까지 기다릴 필요가 없고, 빠르게 채널을 채울 수 있습니다.
- 프로듀서는 작업을 기다리지 않고 채널에서 다음 항목을 빠르게 가져올 수 있습니다.

이 방법을 사용하면 이전에 row마다 고루틴을 스케쥴링했던 것보다 픽셀마다 작업을 전달하기 위해 채널을 사용하는 속도와 거의 같은 속도를 얻게 됩니다.

> 행마다 동작하도록 `nWorkersFillImg`를 수정하세요. 결과 시간을 측정하고 트레이싱을 분석해보세요.

## 5.9 만델브로트 마이크로 서비스

2019년입니다. 만델브로트를 서버리스 마이크로서비스로 인터넷에 배포하지 않고 생성하는 것은 큰 의미가 없죠. 따라서, 만델웹을 선물로 드릴게요.

```sh
% go run examples/mandelweb/mandelweb.go
2017/09/17 15:29:21 listening on http://127.0.0.1:8080/
```

[http://127.0.0.1:8080/mandelbrot](http://127.0.0.1:8080/mandelbrot)

### 5.9.1 실행 중인 어플리케이션 트레이싱

이전 예제에서는 전체 프로그램에 대해 트레이싱을 실행했습니다.

보시다시피, 소량이라도 트레이싱이 매우 클 수 있으므로 트레이싱 데이터를 지속적으로 수집하면 너무 많은 데이터가 생성됩니다. 또한 트레이싱은 프로그램 속도에 영향을 줄 수 있습니다. 특히 활동이 많은 경우에 그렇습니다.

우리가 원하는 것은 실행중인 프로그램에서 짧은 트레이싱을 수집하는 방법입니다.

다행히도 `net/http/pprof` 패키지에는 이러한 기능이 있습니다.

### 5.9.2 http를 통한 트레이싱 수집

모두가 `net/http/pprof` 패키지에 대해 알고 있을 거라고 생각합니다.

```golang
import _ "net/http/pprof"
```

{{< notice warning >}}
`net/http/pprof`를 임포트하면 `http.DefaultServeMux`에 트레이싱 및 프로파일링 경로를 등록합니다. Go 1.5부터 트레이싱 프로파일러가 포함됩니다.

`curl` (또는 `wget`)을 사용해서 mandelweb에서 5초 트레이싱을 확보할 수 있습니다.

```sh
% curl -o trace.out http://127.0.0.1:8080/debug/pprof/trace?seconds=5
```
{{< /notice >}}

### 5.9.3 부하 생성하기

이전 예제는 흥미롭지만 유휴 상태인 웹 서버에는, 정의상, 성능 문제가 없습니다. 약간의 부하를 발생시켜야합니다. 이를 위해 [JBD](https://github.com/rakyll/hey)가 개발하신 `hey`를 사용할게요.

```sh
% go get -u github.com/rakyll/hey
```

초당 1개의 요청으로 시작합니다.

```sh
% hey -c 1 -n 1000 -q 1 http://127.0.0.1:8080/mandelbrot
```

실행시키며, 다른 창에서 트레이싱을 수집합니다.

```sh
% curl -o trace.out http://127.0.0.1:8080/debug/pprof/trace?seconds=5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 66169    0 66169    0     0  13233      0 --:--:--  0:00:05 --:--:-- 17390
% go tool trace trace.out
2017/09/17 16:09:30 Parsing trace...
2017/09/17 16:09:30 Serializing trace...
2017/09/17 16:09:30 Splitting trace...
2017/09/17 16:09:30 Opening browser.
Trace viewer is listening on http://127.0.0.1:60301
```

### 5.9.4 과부하 생성하기

초당 5개의 요청으로 속도를 높이겠습니다.

```sh
% hey -c 5 -n 1000 -q 5 http://127.0.0.1:8080/mandelbrot
```

실행시키며, 다른 창에서 트레이싱을 수집합니다.

```sh
% curl -o trace.out http://127.0.0.1:8080/debug/pprof/trace?seconds=5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                Dload  Upload   Total   Spent    Left  Speed
100 66169    0 66169    0     0  13233      0 --:--:--  0:00:05 --:--:-- 17390
% go tool trace trace.out
2017/09/17 16:09:30 Parsing trace...
2017/09/17 16:09:30 Serializing trace...
2017/09/17 16:09:30 Splitting trace...
2017/09/17 16:09:30 Opening browser. Trace viewer is listening on http://127.0.0.1:60301
```

### 5.9.5 추가 작품, 에라토스테네스의 체

[동시 프라임 체 (concurrent prime sieve)](https://github.com/golang/go/blob/master/doc/play/sieve.go)는 처음 작성된 Go 프로그램 중 하나입니다.

이반 다닐루크 (Ivan Daniluk)는 이를 시각화하는 멋진 포스트를 작성했습니다.

엑스큐션 트레이서를 사용한 동작 상황을 살펴보겠습니다.

### 5.9.6 더 많은 자료들

- Rhys Hiltner, [Go’s execution tracer](https://www.youtube.com/watch?v=mmqDlbWk_XA) (dotGo 2016)
- Rhys Hiltner, [An Introduction to "go tool trace"](https://www.youtube.com/watch?v=V74JnrGTwKA) (GopherCon 2017)
- Dave Cheney, [Seven ways to profile Go programs](https://www.youtube.com/watch?v=2h_NFBFrciI) (GolangUK 2016)
- Dave Cheney, [High performance Go workshop](https://dave.cheney.net/training#high-performance-go)
- Ivan Daniluk, [Visualizing Concurrency in Go](https://www.youtube.com/watch?v=KyuFeiG3Y60) (GopherCon 2016)
- Kavya Joshi, [Understanding Channels](https://www.youtube.com/watch?v=KBZlN0izeiY) (GopherCon 2017)
- Francesc Campoy, [Using the Go execution tracer](https://www.youtube.com/watch?v=ySy3sR1LFCQ)

# 6.메모리 및 가비지 콜렉터

Go는 가비지 콜렉트되는 언어입니다. 이는 디자인 원칙이며 변치 않을 겁니다.

가비지 콜렉트 언어인 Go 프로그램의 성능은 가비지 콜렉터와의 상호 작용에 의해 결정됩니다.

선택한 알고리즘 다음으로는, 메모리 소비가 애플리케이션의 성능과 확장성을 결정하는 가장 중요한 요소입니다.

이 섹션에서는 가비지 콜렉터의 작동, 프로그램의 메모리 사용량을 측정하는 방법 및 가비지 콜렉터 성능이 병목 지점일 경우 메모리 사용량을 낮추기 위한 전략에 대해 설명합니다.

## 6.1 가비지 콜렉터 월드뷰 

가비지 콜렉터의 목적은 프로그램에서 사용할 수 있는 무한한 양의 메모리가 있다는 환상을 보여주는 것입니다.

이 설명에 동의하지 않을 수도 있지만, 가비지 콜렉터 디자이너의 작동 방식에 대한 기본 가정이 이렇습니다.

스탑-더-월드 (STW, stop the world)인 마크 스윕 GC는 전체 실행 시간 측면에서 가장 효율적이며 일괄 처리, 시뮬레이션 등에 적합합니다. 그러나 시간이 지남에 따라 Go GC는 순수 스탑-더-월드 콜렉터에서 동시성 비압축 컬렉터(concurrent, non compacting, collector)로 이동했습니다. 이는 Go GC가 레이턴시가 짧은 서버 및 대화형 애플리케이션을 위해 설계되었기 때문입니다.

Go GC의 설계는 최대 처리량보다 짧은 레이턴시를 선호합니다. 즉, 이후의 정리 비용을 줄이기 위해 할당 비용의 일부를 mutator로 옮깁니다.

## 6.2 가비지 콜렉터 디자인

Go GC의 설계는 수년에 걸쳐 변경되었습니다.

- Go 1.0, tcmalloc에 크게 의존하는 스탑-더-월드 마크 스윕 콜렉터
- Go 1.3, 힙의 큰 숫자를 포인터로 오인하지 않아, 메모리 누수가 발생하는, 완전 정밀 콜렉터
- Go 1.5, 처리량보다 레이턴시에 중점을 둔 새로운 GC 설계
- Go 1.6, 레이턴시가 짧은 더 큰 힙을 처리하는 GC 개선
- Go 1.7, 리팩토링 위주의 작은 GC 개선
- Go 1.8, STW 시간을 줄이기 위한 추가 작업, 이제 100 마이크로초 범위로 내려감.
- Go 1.10+, 전체 GC 사이클을 트리거할 때 레이턴시를 낮추기 위한 순수 협업 (pure cooperative) 고루틴에서 벗어남.

## 6.3 가비지 콜렉터 모니터링

가비지 콜렉터가 얼마나 열심히 작동하는지 간단하게 알 수 있는 방법은 GC 로깅 출력을 활성화하는 것입니다.

이 통계는 항상 수집되지만 일반적으로 억제되므로 GODEBUG 환경 변수를 설정하여 해당 통계를 표시할 수 있습니다.

```sh
% env GODEBUG=gctrace=1 godoc -http=:8080
gc 1 @0.012s 2%: 0.026+0.39+0.10 ms clock, 0.21+0.88/0.52/0+0.84 ms cpu, 4->4->0 MB, 5 MB goal, 8 P
gc 2 @0.016s 3%: 0.038+0.41+0.042 ms clock, 0.30+1.2/0.59/0+0.33 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 3 @0.020s 4%: 0.054+0.56+0.054 ms clock, 0.43+1.0/0.59/0+0.43 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 4 @0.025s 4%: 0.043+0.52+0.058 ms clock, 0.34+1.3/0.64/0+0.46 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 5 @0.029s 5%: 0.058+0.64+0.053 ms clock, 0.46+1.3/0.89/0+0.42 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 6 @0.034s 5%: 0.062+0.42+0.050 ms clock, 0.50+1.2/0.63/0+0.40 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 7 @0.038s 6%: 0.057+0.47+0.046 ms clock, 0.46+1.2/0.67/0+0.37 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 8 @0.041s 6%: 0.049+0.42+0.057 ms clock, 0.39+1.1/0.57/0+0.46 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
gc 9 @0.045s 6%: 0.047+0.38+0.042 ms clock, 0.37+0.94/0.61/0+0.33 ms cpu, 4->4->1 MB, 5 MB goal, 8 P
```

트레이스 출력은 GC 활동의 일반적인 측정값을 제공합니다. `gctrace = 1`의 출력 형식은 런타임 패키지 설명서에 설명되어 있습니다.

DEMO: `GODEBUG=gctrace=1`을 활성화하여 `godoc`을 표시합니다.

{{< notice tip >}}
프러덕션 환경에서 이 환경 변수를 사용하세요. 성능에 영향을 미치지 않습니다.
{{< /notice >}}
	
`GODEBUG=gctrace=1`은 문제가 있는 것을 알고 있을 때 사용하면 좋습니다. 그러나 Go 애플리케이션의 일반적인 원격 측정의 경우는 `net/http/pprof` 인터페이스를 사용하는 것을 추천합니다.

```golang
import _ "net/http/pprof"
```

`net/http/pprof` 패키지를 임포트하면 `/debug/pprof`에 핸들러를 다음과 같은 다양한 런타임 메트릭으로 등록합니다.

- 실행 중인 모든 고루틴의 목록, `/debug/pprof/heap?debug=1`.
- 메모리 할당 통계 리포트, `/debug/pprof/heap?debug=1`.

{{< notice warning >}}
`net/http/pprof`는 기본 `http.ServeMux`에 등록됩니다.
`http.ListenAndServe(address, nil)`을 사용하면 이것이 표시되므로 주의하세요.
{{< /notice >}}
	
DEMO: `godoc -http=:8080`, `/debug/pprof`를 보여줍니다.


### 6.3.1 가비지 콜렉터 튜닝

Go 런타임은 GC, `GOGC`를 조정하기 위한 환경 변수를 제공합니다.

GOGC의 공식은 다음과 같습니다.

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mstyle displaystyle="true">
    <mi>g</mi>
    <mi>o</mi>
    <mi>a</mi>
    <mi>l</mi>
    <mo>=</mo>
    <mi>r</mi>
    <mi>e</mi>
    <mi>a</mi>
    <mi>c</mi>
    <mi>h</mi>
    <mi>a</mi>
    <mi>b</mi>
    <mi>l</mi>
    <mi>e</mi>
    <mo>&#x22C5;</mo>
    <mrow>
      <mo>(</mo>
      <mn>1</mn>
      <mo>+</mo>
      <mfrac>
        <mrow>
          <mi>G</mi>
          <mi>O</mi>
          <mi>G</mi>
          <mi>C</mi>
        </mrow>
        <mn>100</mn>
      </mfrac>
      <mo>)</mo>
    </mrow>
  </mstyle>
</math>


예를 들어, 현재 256MB 힙이 있고 `GOGC = 100 (기본값)` 인 경우 힙이 가득 차면 다음과 같이 증가합니다.

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mstyle displaystyle="true">
    <mn>512</mn>
    <mi>M</mi>
    <mi>B</mi>
    <mo>=</mo>
    <mn>256</mn>
    <mi>M</mi>
    <mi>B</mi>
    <mo>&#x22C5;</mo>
    <mrow>
      <mo>(</mo>
      <mn>1</mn>
      <mo>+</mo>
      <mfrac>
        <mn>100</mn>
        <mn>100</mn>
      </mfrac>
      <mo>)</mo>
    </mrow>
  </mstyle>
</math>

- `GOGC` 값이 100보다 크면 힙이 더 빨리 커져서 GC에 대한 압력이 감소합니다.

- `GOGC` 값이 100보다 작으면 힙이 느리게 커져 GC에 대한 압력이 증가합니다.

기본값 100은 `just_a_guide`입니다. 프로덕션 부하로 애플리케이션을 프로파일링한 후 고유한 값을 선택해야 합니다.

## 6.5 string과 []bytes

Go에서, `string` 값은 변경할 수 없으며 `[]byte`는 변경할 수 있습니다.

대부분의 프로그램은 `string` 작업을 선호하지만 대부분의 IO는 `[]byte`로 수행됩니다.

가능하다면 `[]byte`에서 `string` 변환을 피하세요. 이는 일반적으로 값으로 `string` 또는 `[]byte` 중 한가지 표현을 선택함을 의미합니다. 네트워크나 디스크에서 데이터를 읽는 경우 종종 `[]byte`가 됩니다.

`bytes` 패키지에는 `strings` 패키지와 동일한 작업 (`Split`, `Compare`, `HasPrefix`, `Trim` 등)이 많이 포함되어 있습니다.

내부적으로, `strings`은 `bytes` 패키지와 동일한 어셈블리 프리미티브를 사용합니다.

## 6.6 맵 키로 []byte 사용

`string`을 맵 키로 사용하는 것이 매우 일반적이지만, 종종 `[]byte`를 사용합니다.

컴파일러는 이 경우에 특별한 최적화를 구현합니다.

```golang
var m map[string]string
v, ok := m[string(bytes)]
```

이렇게 하면 맵 룩업 시에 바이트 슬라이스를 문자열로 변환하지 않아도 됩니다. 다음과 같은 코딩할 경우 작동하지 않는데, 아주 특이하죠.

```golang
key := string(bytes)
val, ok := m[key]
```

> 이게 아직도 사실인지 확인해 봅시다. []byte를 string 맵 키로 사용하는 이 두 가지 방법을 비교하는 벤치마크를 작성해보세요.


## 6.7 문자열 연결 피하기

Go 문자열은 불변입니다. 두 개의 문자열을 결합하면 세 번째 문자열이 생성됩니다. 다음 중 가장 빠른 것은 무엇일까요?

```golang
		s := request.ID
		s += " " + client.Addr().String()
		s += " " + time.Now().String()
		r = s
```

```golang
		var b bytes.Buffer
		fmt.Fprintf(&b, "%s %v %v", request.ID, client.Addr(), time.Now())
		r = b.String()
```

```golang
		r = fmt.Sprintf("%s %v %v", request.ID, client.Addr(), time.Now())
```

```golang
		b := make([]byte, 0, 40)
		b = append(b, request.ID...)
		b = append(b, ' ')
		b = append(b, client.Addr().String()...)
		b = append(b, ' ')
		b = time.Now().AppendFormat(b, "2006-01-02 15:04:05.999999999 -0700 MST")
		r = string(b)
```

```golang
		var b strings.Builder
		b.WriteString(request.ID)
		b.WriteString(" ")
		b.WriteString(client.Addr().String())
		b.WriteString(" ")
		b.WriteString(time.Now().String())
		r = b.String()
```

DEMO: `go test -bench=. ./examples/concat`

## 6.8 길이가 알려진 경우, 슬라이스를 미리 할당

`Append`는 편리하지만, 낭비입니다.

슬라이스는 최대 1024개의 요소를 두 배로 늘린 후 그 후 약 25% 증가합니다. 요소를 하나 더 추가한 후에 `b`의 용량은 얼마일까요?

```golang
func main() {
	b := make([]int, 1024)
	b = append(b, 99)
	fmt.Println("len:", len(b), "cap:", cap(b))
}
```

Append 패턴을 사용하면 많은 데이터를 복사하며 많은 가비지를 만들 수 있습니다.

슬라이스의 길이를 미리 알고 있으면, 복사를 피하고, 대상이 정확히 맞는 크기인지 확인하기 위해 대상을 미리 할당합니다. 

**이전**

```golang
var s []string
for _, v := range fn() {
        s = append(s, v)
}
return s
```

**이후**

```golang
vals := fn()
s := make([]string, len(vals))
for i, v := range vals {
        s[i] = v
}
return s
```

## 6.9 sync.Pool 사용

동기화 패키지는 공통 객체를 재사용하는데 사용되는 `sync.Pool` 타입과 함께 제공됩니다.

{{< notice warning >}}
`sync.Pool`에는 고정 크기 또는 최대 용량이 없습니다. GC가 발생할 때까지 추가하고 가져와 무조건 비웁니다. 이는 [설계를 따른 것](https://groups.google.com/forum/#!searchin/golang-dev/gc-aware/golang-dev/kJ_R6vYVYHU/LjoGriFTYxMJ)입니다.
{{< /notice >}}

{{< notice tip >}}
가비지 콜렉션이 너무 빠르거나 가비지 콜렉션이 너무 늦으면 Pool을 비우기에 적절한 시간은 가비지 콜렉션 중이어야 합니다. 즉, Pool 타입의 세만틱스는 각 가비지 콜렉션에서 비어져야 한다는 것입니다. — 러스 콕스

```golang
var pool = sync.Pool{New: func() interface{} { return make([]byte, 4096) }}

func fn() {
	buf := pool.Get().([]byte) // takes from pool or calls New
	// do work
	pool.Put(buf) // returns buf to the pool
}
```
{{< /notice >}}

## 6.10 연습문제

- `godoc`(또는 다른 프로그램)을 사용해서 `GODEBUG=gctrace=1`을 사용한 `GOGC` 변경 내용을 관찰하세요.

- byte의 string(byte) 맵 키를 벤치마크해보세요.

- 서로 다른 연결(concat) 방법의 메모리 할당을 벤치마크해보세요.

# 7. 팁과 트립

팁과 제안을 무작위로 살펴보겠습니다.

이 마지막 섹션에는 Go 코드를 마이크로 최적화하기 위한 여러가지 팁이 포함되어 있습니다.

## 7.1 고루틴

현대 하드웨어에 잘 어울리는 Go의 주요 기능은 고루틴입니다.

고루틴은 사용하기 쉽고 만들기에도 저렴합니다. 고루틴을 거의 공짜라고 생각할 수 있습니다.

Go 런타임은 일반적으로 수만 개의 고루틴이 있는 프로그램을 위해 작성되었으나 수십만 고루틴을 예상치 못한 것은 아닙니다.

그러나 각 고루틴은 현재 최소 2k인 고루틴 스택에 대해 최소한의 메모리를 소비합니다.

2048 * 백만 고루틴 == 2GB의 메모리이고, 아직 시작도 안했죠.

어쩌면 이것이 많을 수도 있고, 어쩌면 애플리케이션의 다른 용도로 사용되지 않을 수도 있습니다.

### 7.1.1 고루팁을 언제 멈추어야 하는지 알기 

고루틴은 시작 비용이 저렴하고 실행 비용이 저렴합니다. 하지만 메모리 사용 공간 측면에서 비용이 한정되어 있습니다. 무한히 많은 고루틴을 만들 수는 없습니다.

여러분은 고루틴을 런칭시키기 위해 프로그램에서 Go 키워드를 사용할 때마다 고루틴이 어떻게 언제 종료되는지 알아야 합니다.

여러분의 설계에서, 몇몇 고루틴은 프로그램이 끝날 때까지 실행될 수 있습니다. 이러한 고루틴은 예외적인 규칙이 아닐 정도로 희귀합니다.

답을 모르신다면, 고루틴이 스택에서 접근할 수 있는 모든 힙 할당 변수뿐만 아니라 스택의 메모리를 힙에 고정하기 때문에 메모리 누수가 발생할 수 있습니다.

{{< notice tip >}}
고루틴을 어떻게 멈출 지 모른채로 시작하지 마세요.
{{< /notice >}}

### 7.1.2 더 읽을거리

- [Concurrency Made Easy](https://www.youtube.com/watch?v=yKQOunhhf4A&index=16&list=PLq2Nv-Sh8EbZEjZdPLaQt1qh_ohZFMDj8) (video)

- [Concurrency Made Easy](https://dave.cheney.net/paste/concurrency-made-easy.pdf) (slides)

- [Never start a goroutine without knowning when it will stop](https://dave.cheney.net/practical-go/presentations/qcon-china.html#_never_start_a_goroutine_without_knowning_when_it_will_stop) (Practical Go, QCon Shanghai 2018)

## 7.2 Go는 일부 요청에 대해 효율적인 네트워크 폴링을 사용합니다.

Go 런타임은 효율적인 운영 체제 폴링 메커니즘 (kqueue, epoll, windows IOCP 등)을 사용하여 네트워크 IO를 처리합니다. 대기 중인 많은 고루틴은 단일 운영 체제 쓰레드로 서비스됩니다.

그러나 로컬 파일 IO의 경우 Go는 IO 폴링을 구현하지 않습니다. `* os.File`의 각 작업은 진행 중인 동안 운영 체제 쓰레드를 하나 사용합니다.

로컬 파일 IO를 많이 사용하면 프로그램이 수백 또는 수천 개의 쓰레드를 생성할 수 있습니다. 운영 체제가 허용하는 것보다 더 많을 수도 있습니다.

디스크 서브 시스템은 수백 또는 수천 개의 동시 IO 요청을 처리할 수 없을 것으로 예상됩니다.

{{< notice tip >}}
동시 블럭킹 IO의 양을 제한하려면 작업자 고루틴 풀 또는 버퍼링된 채널을 세마포어로 사용하세요. 

```golang
	var semaphore = make(chan struct{}, 10)
	
	func processRequest(work *Work) {
		semaphore <- struct{}{} // acquire semaphore
		// process request
		<-semaphore // release semaphore
	}
```
{{< /notice >}}


## 7.3 애플리케이션에서 IO 멀티 플라이어를 조심하세요

서버 프로세스를 작성하는 경우 해당 서버의 주 작업은 네트워크를 통해 연결된 클라이언트와 애플리케이션에 저장된 데이터를 멀리플렉싱하는 것입니다.

대부분의 서버 프로그램은 요청을 받고, 처리를 한 다음에, 결과를 반환합니다. 간단하게 들리지만 결과에 따라 클라이언트가 서버에서 많은 양의 리소소를 사용할 수 있습니다. 다음은 주의해야 할 몇 가지 사항입니다.

- 들어오는 요청당 IO 요청량, 즉 단일 클라이언트 요청이 얼마나 많은 IO 이벤트를 생성할까요? 캐시에서 많은 요청이 서비스되는 경우 평균적으로 1 또는 1 미만일 수 있습니다.

- 쿼리를 처리하는 데 필요한 읽기량, 이는 고정적이며, N + 1 또는 선형입니다 (전체 테이블 읽어 마지막 결과 페이지를 생성).

메모리가 느리면, 상대적으로, IO가 너무 느리기 때문에 어떠한 비용도 들이지 않도록 해야합니다. 요청과 관련하여 IO를 수행하지 않는 것이 가장 중요합니다. 즉, 사용자가 디스크 서브 시스템이 디스크에 쓰기나 읽기 조차도 기다리게 하지 않도록 해야 합니다.

## 7.4 스트리밍 IO 인터페이스를 사용하세요

가능하다면 `[]byte`로 데이터를 읽고 전달하는 것을 피하세요.

요청에 따라 메가 바이트 (또는 그 이상)의 데이터를 메모리로 읽을 수 있습니다. 이로 인해 GC에 큰 압박이 가해져 애플리케이션의 평균 레이턴시가 증가합니다.

대신 `io.Reader` 및 `io.Writer`를 사용해서 프로세싱 파이프라인을 구성하고 요청당 사용 중인 메모리 양을 제한하세요.

효율성을 높이기 위해, `io.Copy`를 많이 사용하는 경우 `io.ReaderFrom`/`io.WriterTo`를 구현하는 것이 좋습니다. 이러한 인터페이스는 보다 효율적이며 메모리를 임시 버퍼에 복사하지 않습니다.

## 7.5 타임아웃, 타임아웃, 타임아웃

최대 소요 시간을 알지 못한 채로 IO를 시작해서는 안 됩니다.

`SetDeadline`, `SetReadDeadline`, `SetWriteDeadline`을 사용하여 수행하는 모든 네트워크 요청에 대해 타임아웃을 설정해야 합니다.

## 7.6 Defer는 비쌉니다.

`defer`는 비용이 많이 듭니다. 왜냐하면 defer 인자에 대한 클로저(closure)를 기록해야 하기 때문입니다.

```golang
defer mu.Unlock()
```

는 아래와 동일합니다.

```golang
defer func() {
        mu.Unlock()
}()
```

수행되는 작업이 작으면 `defer`는 비싸지만, 전형적인 예는 구조체 변수 또는 맵 조회 주위에 뮤텍스 잠금 해제를 defering하는 것입니다. 그러한 상황에서 `defer`를 피하도록 선택할 수 있습니다.

성능 향상을 위해 가독성과 유지 관리가 희생되는 경우입니다.

이러한 결정을 항상 다시 방문하십시오.

## 7.7 Finaler를 피하세요

파이널리제이션(Finalization)는 가지비 콜렉션을 앞둔 객체를 동작(behaviour)를 부착하는 기술입니다.

따라서, 파이널리제이션은 결정적이지 않습니다.

파이널라이저를 실행하려면 어떤 것도 객체에 접근할 수 없어야 합니다. 맵에 있는 객체에 대한 참조를 실수로 보관하면 파이널리제이션이 수행되지 않습니다.

파이널라이저는 gc 사이클의 일부로 실행되며, 이는 언제 실행할지 예측할 수 없으며 gc 작동을 줄인다는 목표에 상충합니다.

힙이 크고, 가비지를 최소화하도록 애플리케이션을 조정한 경우,  파이널라이저가 오랫동안 실행되지 않을 수 있습니다.

## 7.8 cgo를 최소화하세요

cgo를 사용하면 Go 프로그램이 C 라이브러리를 호출할 수 있습니다.

C 코드와 Go 코드는 서로 다른 두 우주에 살고 있으며, cgo는 두 우주 사이의 경계를 통과합니다.

이 전환은 무료가 아니며 코드의 위치에 따라 비용이 많이 들 수 있습니다.

cgo 호출은 IO 차단과 유사하며 작동 중에 쓰레드를 소비합니다.

빡빡한 루프 중간에 C 코드를 호출하면 안됩니다.

### 7.8.1 사실, cgo는 피하세요

cgo는 오버헤드가 높습니다.

최상의 성능을 위해서는 애플리케이션에서 cgo를 피하는 것이 좋습니다.

C코드에 시간이 오래 걸리면, cgo 오버헤드가 중요하지 않습니다.

cgo를 사용해서 오버헤드가 가장 눈에 띄는 매우 짧은 C함수를 호출하는 경우, Go로 해당 코드를 다시 작성하세요. 원래 짧으니깐요.

타이트한 루프(tight loop)내에 호출되는 비싼 C코드를 많이 사용하는 경우, 왜 Go를 사용하시나요?

비싼 C코드를 자주 호출하기 위해, cgo를 사용하는 분이 있으신가요?

**더 읽을 거리**

- [cgo is not Go](http://dave.cheney.net/2016/01/18/cgo-is-not-go)

## 7.9 항상 최신 릴리즈된 버전의 Go를 사용하세요

이전 버전의 Go는 결코 나아지지 않을 것입니다. 버그 수정이나 최적화가 절대 이뤄지지 않을 겁니다.

- Go 1.4는 사용해서는 안 됩니다.
- Go 1.5와 1.6은 컴파일러 속도가 느렸지만, 코드도 더 빠르고, GC는 더 빠릅니다.
- Go 1.7은 1.6보다 컴파일 속도가 약 30% 향상 되었으며, 링크 속도가 2배 향상되었습니다 (이전 모든 버전의 Go보다 좋음)
- Go 1.8은 이 시점에서 컴파일 속도가 약간 향상되지만, 인텔 이외의 아키텍처에서는 코드 품질이 크게 향상됩니다.
- Go 1.9-1.12는 지속적으로 생성된 코드의 성능을 개선하고, 버그를 수정하며, 인라인을 개선하고 디버깅을 개선합니다.

{{< notice tip >}}
이전 버전의 Go에는 업데이트가 없습니다. 사용해서는 안됩니다. 최신버전을 사용하면 최상의 성능을 얻을 수 있습니다.
{{< /notice >}}

### 7.9.1 더 읽을거리

- [Go 1.7 toolchain improvements](http://dave.cheney.net/2016/04/02/go-1-7-toolchain-improvements)
- [Go 1.8 performance improvements](http://dave.cheney.net/2016/09/18/go-1-8-performance-improvements-one-month-in)

### 7.9.2 핫 필드를 구조체의 상단으로 이동

(내용없음)

## 7.10

질문 있나요?

# 마지막 질문과 결론

> 읽기 쉽다는 것은 신뢰할 수 있다는 뜻입니다. — Rob Pike

가장 간단한 코드부터 시작하세요.

측정하세요. 병목 현상을 확인하기 위해 코드를 프로파일링하세요.

성능이 좋으면 멈추세요. 모든 것을 최적화할 필요는 없습니다. 코드에서 가장 중요한 부분만 다루세요.

애플리케이션이 성장하거나 트래픽 패턴이 진화함에 따라 성능 핫 스팟이 변경될 것입니다.

성능에 중요하지 않은 복잡한 코드를 그대로 두지 말고, 병목이 다른 곳으로 이동하면 더 간단한 동작으로 다시 작성하세요.

항상 가장 간단한 코드를 작성하세요. 컴파일러는 일반코드에 맞게 최적화되어 있습니다.

더 짧은 코드는 더 빠른 코드입니다. Go는 C++이 아닙니다. 컴파일러에서 복잡한 추상화를 풀 것으로 기대하지 마세요.

더 짧은 코드는 더 작은 코드입니다. CPU 캐시에 중요합니다. 

할당에 매우 주의하세요. 가능한 경우 불필요한 할당을 피하세요.

> 정확하지 않아도 된다면 전 아주 빠르게 만들 수 있습니다. — Russ Cox

성능과 신뢰성은 똑같이 중요합니다.

저는 패닉, 교착 상태 또는 OOM의 기반 위에서 매우 빠른 서버를 만드는 것은 별로 가치가 없다고 봅니다.

성능을 신뢰성과 바꾸지 마세요.