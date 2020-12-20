---
title: 우아한 테크 프리코스 3주차, 최종 코딩테스트 회고록
date: 2020-12-21-04:00
categories:
- Edu

tags:
- 우아한 테크 코스
- 교육

photos: 
- /post_images/woowa-logo2.jpg

---

## 우아한 테크 코스 3주차, 최종 코딩테스트
> 나의 부족함을 절실하게 알 수 있었던 마지막 미션과 시험

---

## 3주차 미션은 어떗는가
> 나의 큰 문제점을 찾아낼 수 있던 기회

---

## 3주차 미션때 찾아낸 나의 큰 문제점.
> 가지고 있는 지식만으로 해결하려는 시간이 지나치게 길다.

* 3주차 미션은 1,2 주차 미션의 난이도의 몇배 이상 이었다.
    * 난이도를 아직 부족한 내가 정의하긴 힘들지만, 소요한 시간만 비교했을떄는 이전에 비해서 최소 4배 이상의 난이도 였다고 생각한다.
    * 즉 기존의 지식만으로 해결하기에는, 부족한 부분이 굉장히 많았다.

* 하지만 나는 완전히 **본인만의 힘으로** 해결하려는 고집을 부렸다.
    * 미션 분석단계에서 구현단계로 넘어가는 시간이 굉장히 오래걸렸다.
    * 기능들은 정상적으로 동작하나, 개인적으로는 상당히 불만족 스러운 결과물을 만들게 되었다.

* 하지만, 이제 나의 문제점을 알았으니, 고쳐나가면 된다고 생각한다!

---


## 3주차는 피드백이 없었다.
> 이부분은 확실히 아쉽다.

* 사실 1,2 주차 회고록을 적을떄처럼, 피드백을 보면서 나의 문제점을 교정하고 그부분에 대해 회고하는 회고록을 적으려고 했다.
* 하지만 3 주차 미션은 결과물은 코드량이 상당히 방대하고, 이로인해서 즉각적인 피드백이 어렵다고 생각하고, 어찌보면 당연한 수순이라고 생각한다.
* 그래서 최종미션과 묶어서 같이 회고하게 됐다.

---

<br>

## 최종 미션은 어땠는가?
> 알고 있다고 생각하는것과, 정말로 아는것은 다르다.

---

## 최종 미션때 스스로에게 아쉬웠던점.
> 근본적으로 1,2,3 주차 미션을 진행하면서 배운것들을 제대로 보여주지 못했다.

* 먼저, 전날 잠을 정말 잘 못잤다 ㅠ
    * 이것도 내 잘못이다. 더 일찍부터 자려고 노력했어야 했는데...

* 문서를 꼼꼼하게 읽지 않았다.
    * 처음에 output 예시가 잘못 적혀있었는데, 이것은 처음부터 꼼꼼히 읽었으면 바로 확인하고 문의 했을 수 있었던 부분이다.
    * 꼼꼼히 읽지 않고 있다가, 나중에서나 발견하고 순간 패닉온것은 내 잘못이다.
    * 혹은, 채팅방의 공지라도 수시로 확인했다면 좋았을텐데, 그렇게 하지 못한것도 너무나 아쉽다.

* 자바의 MAP 컬렉션 응용을 잘못했다.
    * 이로 인해서 기능하나는 구현하지도 못했다.
    * 3주간 자바 사용에 대한 숙련도가 많이 늘었다고 생각했는데, 아직도 한참 부족한것 같다.

* 시간에 쫒겨서, 할 수 있는것도 못했다.
    * 위에서 이야기한 기능구현을 하지 못한 부분에 잡혀서, 리팩토링을 할 수 있는 부분들도 놓쳤다.
    * 저런 상황에서도 **우선 순위**를 재배치하는 능력을 좀 더 키워야 겠다.

---

## 스스로 리뷰해보자.
> 이제 다시 기술적인 이야기를 해보고자 한다. 3주차 미션과, 최종 코딩 테스트를 스스로 리뷰해보자.

* 리뷰가 없다면 혼자서 하면 된다!

---


### MVC 패턴을 적용해 보았다.

<br>

![img](/post_images/woowa_week3_mvc.jpg)

```java
package subway.controller;

import subway.domain.SectionManageMenu;
import subway.domain.repositories.LineRepository;
import subway.utils.Validator;
import subway.view.InputView;
import subway.view.SectionView;

public class SectionController {

    public static void sectionAdd() {
        try {
            SectionView.printLineAddReqMsg();
            String lineName = lineNameInput();
            SectionView.printStationAddReqMsg();
            String stationName = stationNameInputForAdd(lineName);
            SectionView.printLocationAddReqMsg();
            int location = locationInput(lineName);

            LineRepository.addStationToLine(lineName, stationName, location);
            SectionView.printAddSuccessMsg();
            SectionManageMenu.sectionManageMenuStop();  // 이곳에서 멈춰야 성공적으로 완료 후 메인으로 간다.
        } catch (IllegalArgumentException e) {
            System.out.println("\n[ERROR] " + e.getMessage() + "\n");
        }
    }

    public static void sectionDelete() {
        try {
            SectionView.printLineDelReqMsg();
            String lineName = lineNameInput();
            SectionView.printStationDelReqMsg();
            String stationName = stationNameInputForDelete(lineName);
            if (!LineRepository.deleteStationInLine(lineName, stationName)) {
                throw new IllegalArgumentException("노선에 포함된 역이 2개 이하 입니다");
            }

            SectionView.printDelSuccessMsg();
            SectionManageMenu.sectionManageMenuStop();
        } catch (IllegalArgumentException e) {
            System.out.println("\n[ERROR] " + e.getMessage() + "\n");
        }
    }

    private static String lineNameInput() throws IllegalArgumentException {
        String lineName = InputView.getInput();
        lineName = lineName.replace(" ", "");
        if (!Validator.isExistLineName(lineName)) {
            throw new IllegalArgumentException("DB에 존재하지 않는 노선 이름 입니다");
        }
        return lineName;
    }

    private static String stationNameInputForAdd(String lineName) throws IllegalArgumentException {
        String stationName = InputView.getInput();
        stationName = stationName.replace(" ", "");
        if (!Validator.isExistStationName(stationName)) {
            throw new IllegalArgumentException("DB에 존재 하지 않는 역 입니다");
        }
        if (Validator.isStationAlreadyInLine(stationName, lineName)) {
            throw new IllegalArgumentException("노선에서 갈래길은 생길 수 없습니다.");
        }
        return stationName;
    }

    private static String stationNameInputForDelete(String lineName) throws IllegalArgumentException {
        String stationName = InputView.getInput();
        stationName = stationName.replace(" ", "");
        if (!Validator.isExistStationName(stationName)) {
            throw new IllegalArgumentException("DB에 존재 하지 않는 역 입니다");
        }
        if (!Validator.isStationAlreadyInLine(stationName, lineName)) {
            throw new IllegalArgumentException("노선에 해당 역이 존재하지 않습니다.");
        }
        return stationName;
    }

    private static int locationInput(String lineName) throws IllegalArgumentException {
        try {
            int location = Integer.parseInt(InputView.getInput());
            if (!Validator.isValidSectionRange(lineName, location)) {
                throw new IllegalArgumentException("잘못된 범위를 입력 하셨습니다.");
            }
            return location;
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException("잘못된 형식을 입력하셨습니다");
        }
    }
}
```

* MVC 패턴을 사용하기전에, 왜 사용할지에 대해서 고민을 해보았다.
* 먼저 Line과 Station을 관리하는 프로그램을 짜는데, 이것을 어떤식으로 클래스를 나누어야 할지에 대한 고민을 해보았다.
    * Line과 Station의 정보를 조작하는 클래스를 따로 두면 되지 않을까?
        * MVC 패턴으로 `Controller`에서 Line 과 Station을 조작하면 구조가 설계가 쉬워질것 같다.
    * 나중에 여기서 더 많은 기능을 추가 하게 된다면?
        * Controller 에서 메소드만 추가해주면 되니 확장성이 좋지 않을까?

* 사실 저렇게 나눈것이 좋게 나눈것인지는 잘 모르겠다.
    * 구현을 해보니 생각보다 `Controller` 에서 담당하는 역할이 엄청나게 많아졌고, `SRP(단일 책임 원칙)` 에 위배된 것이 아닌가? 라는 생각이 들었다.
    * 이부분에 대해서는 MVC 패턴을 적용한 작은 예제를 넘어서, 다른 프로젝트 소스를 보면서 잘못된점을 짚어보아야 겠다.

---

### 람다와 스트림을 사용하지 않았다.

* 대략적인 문법은 공부했으나, 아직 익숙치 않다.
    * 그래서, 시간에 쫒기면서 코딩한 3주차 미션, 최종 코딩테스트에서 사용하지 못했다.
    * 이것은 어떤 이유를 탓할것이 아닌, 충분한 연습을 하지 않은 나의 잘못이다.
    * 현업자 친구에게 물어보니 너가 아직 익숙치 않아서 그런것이지, 나중가면 정말 너무 편해진다고 하더라.
        * 람다와 스트림을 의식적으로 사용하며 익숙해지는 기간을 좀 가지려고 한다.

---

### 컬렉션 동작을 잘못 이해하고 있었다.

* 이것도 사실 많이 사용해보지 않아서 빚어진 문제점이었다.
* 그것도 `최종 코딩테스트`에서 내 발목을 잡은 부분이었다.
    * 심지어 지푸라기라도 잡는 심정으로 docs에다가 작동안된 이유를 뇌피셜로 적어놨는데 완전히 틀렸다 ㅋㅋ
        * 적은 내용 : `map<String, map<String, Integer>> 로 구현했으나, List에서 가져온것은 문자열값은 문자열 형태라도 다른 객체라 Hash 값이 다르게 나오는듯...`
        * **틀린말이다.** 그냥 내가 같은 key로 새로운 map을 쑤셔 넣으니 `null` 이 나온것 이었다.
        * 코딩테스트가 끝나고 바로 원인을 계속 찾아보았고, 원인을 찾고 스스로 정리하여 블로그에 [포스팅](https://unluckyjung.github.io/java/2020/12/19/Java_Overlay_Map/) 해두었다.


```java
    private static Map<String, Map<String, Integer>> stationDistMap = new HashMap<>();
    private static Map<String, Map<String, Integer>> stationTimeMap = new HashMap<>();

    public static void addVertexByStationName(String stationName) {
        shortTimeGraph.addVertex(stationName);
        shortDistanceGraph.addVertex(stationName);
    }

    private static void secondLineSet() {
        ...
        stationDistMap.put("교대역", setMap("강남역", 2));
        ...
    }

    private static void thirdLineSet() {
        ...
        stationDistMap.put("교대역", setMap("남부터미널역", 3));
        ...
    }
```

* 의도한것은 `[교대역][강남역]` 하면 `2` 가 나오고,  `[교대역][남부터미널역]` 하면 3이 나오는것이었다.
* 하지만 저렇게 하면 당연히 put 할때마다, 교대역을 key 로 map이 덮어 씌워지며, 의도하지 않은 방향으로 작동하게 된다.

---

### Java의 Enum를 응용해서 사용해 보았다.

* 미션 요구사항중에서 `else` 를 사용하지 말라는 요구사항이 있었다.
* 메뉴에서 선택해야 하는데 어떻게 `else` 를 쓰지 말라는거지?, 그냥 `if` 를 여러번 쓰라는건가? 하는 고민을 하다가, 혹시 방법이 있지 않을까 하고 구글신님께 여쭈어 봤다.
    * 그러니까 나오는 [또민의 또족 기술 블로그](https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)... 참 재밌다. 해결책을 찾기 위해 구글링을 하다가 매번 자연스럽게 흘러 들어가는걸 보면, 배민 기술 블로그가 한편으로 참 대단하다 라는 생각이 든다.


```java
    private enum SectionManageMenuEnums {
        SECTION_ADD() {
            @Override
            public void SectionManageMenuSelect() {
                // 구간 등록 호출
                SectionController.sectionAdd();
            }
        },
        SECTION_DELETE() {
            @Override
            public void SectionManageMenuSelect() {
                // 구간 삭제 호출
                SectionController.sectionDelete();
            }
        };

        public abstract void SectionManageMenuSelect();
    }
```

* 나의 경우 추상 메소드를 만들어놓고 오버라이딩을 통해 몸체를 구현했다.
    * 그런데 다른분들 보면 람다식으로 뚝딱 해치우셨더라...
    * 결국 알고 모름의 차이가 또한번 생산성의 차이를 낳게 되는것 같다는 생각이 들었다.

<br>

```java
    private static void pathMenuController(String inputMsg, Scanner scanner) {
        if (inputMsg.equals("B")) {
            pathMenuStop();
            return;
        }
        // 시간 여유가 있으면 enums로 리팩토링
        // 1이면 최단 거리
        if (inputMsg.equals("1")) {
            PathFinder.findShortestDistPath(scanner);
        }
        // 2이면 최소 거리
        if (inputMsg.equals("2")) {
            PathFinder.findShortestTimePath(scanner);
            pathMenuStop();
        }
    }
```

* 하지만 최종 미션에서는 또 사용하지 못했다...
    * 역시 숙련도와 익숙함의 문제다...
    * 일단 구현을 먼저 다 해놓고 남는 시간에 리팩토링을 하려고 했으나.. 남는시간은 존재하지 않았다...

---

### 초기설정이 하드코딩으로 범벅 되어있다.

```java
    private static void thirdLineSet() {
        shortDistanceGraph.setEdgeWeight(shortDistanceGraph.addEdge("교대역", "남부터미널역"), 3);
        stationDistMap.put("교대역", setMap("남부터미널역", 3));
        shortTimeGraph.setEdgeWeight(shortTimeGraph.addEdge("교대역", "남부터미널역"), 2);
        stationTimeMap.put("교대역", setMap("남부터미널역", 2));
        shortDistanceGraph.setEdgeWeight(shortDistanceGraph.addEdge("남부터미널역", "양재역"), 6);
        stationDistMap.put("남부터미널역", setMap("양재역", 6));
        shortTimeGraph.setEdgeWeight(shortTimeGraph.addEdge("남부터미널역", "양재역"), 5);
        stationTimeMap.put("남부터미널역", setMap("양재역", 5));
        shortDistanceGraph.setEdgeWeight(shortDistanceGraph.addEdge("양재역", "매봉역"), 1);
        stationDistMap.put("양재역", setMap("매봉역", 1));
        shortTimeGraph.setEdgeWeight(shortTimeGraph.addEdge("양재역", "매봉역"), 1);
        stationTimeMap.put("양재역", setMap("매봉역", 1));
    }
```

* 다른 공간에 담아 놓고, 메소드를 통해서 정갈하게 넣을 수 있지 않았을까?
* 구현에 급급하다 보니 나오게된 끔찍한 코드라고 생각한다.
    * 심지어 map을 이용한 부분은 작동하지도 않아서, 수많은 put을 시도하는 라인들은 정말 의미없는 라인들이 되어버렸다...

---

### 미션에서 주어진 라이브러리를 의도적으로 사용해서 문제를 해결했다.

```java
    public static int getShortestDist(String name1, String name2) throws IllegalArgumentException {
        try {
            DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(shortDistanceGraph);
            double weight = 0.0;
            weight = dijkstraShortestPath.getPathWeight(name1, name2);
            return (int) weight;
        } catch (Exception e) {
            throw new IllegalArgumentException("갈수 없는 경로 입니다.");
        }
    }
```

* `jgrapht 라이브러리` 를 이용하여 다익스트라 알고리즘을 통해 최단거리, 최소 시간을 구해야 했다.
* 라이브러리를 사용하는 미션은 처음이라 처음에 좀 당황했고, 사용법을 몰라서 2차로 당황했다.
* 그냥 힙을 이용하여 다익스트라 알고리즘을 직접 구현할까 하다가.. 
    * 우형측에서 라이브러리를 사용하는것을 유도한것이라고 생각하고, 사용법을 익힌다음 문제풀이에 사용했다.

---

### Repository 에서 너무 많은 작업을 한다.

```java
    public static int getShortestDist(String name1, String name2) throws IllegalArgumentException {
        try {
            DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(shortDistanceGraph);
            double weight = 0.0;
            weight = dijkstraShortestPath.getPathWeight(name1, name2);
            return (int) weight;
        } catch (Exception e) {
            throw new IllegalArgumentException("갈수 없는 경로 입니다.");
        }
    }

    public static List<String> getShortestDistPath(String name1, String name2) {
        DijkstraShortestPath dijkstraShortestTimePath = new DijkstraShortestPath(shortDistanceGraph);
        List<String> shortestDistPath = dijkstraShortestTimePath.getPath(name1, name2).getVertexList();
        return shortestDistPath;
    }

    public static int getShortestTime(String name1, String name2) throws IllegalArgumentException {
        try {
            DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(shortTimeGraph);
            double weight = 0.0;
            weight = dijkstraShortestPath.getPathWeight(name1, name2);
            return (int) weight;
        } catch (Exception e) {
            throw new IllegalArgumentException("갈수 없는 경로 입니다.");
        }
    }

    public static List<String> getShortestTimePath(String name1, String name2) {
        DijkstraShortestPath dijkstraShortestTimePath = new DijkstraShortestPath(shortTimeGraph);
        List<String> shortestTimePath = dijkstraShortestTimePath.getPath(name1, name2).getVertexList();
        return shortestTimePath;
    }
```

* `DijkstraGraphRepository` 라고 이름을 붙인 클래스의 메소드중 일부분이다.
* 이곳에서 최소거리, 최단거리를 찾아 리턴하는 작업을 수행하고 있다.
    * 이것은 잘못된 것이라고 본다.
    * 차라리 최단 경로를 찾는 역할을 수행하는 클래스를 따로 분리시켜 처리했어야 된다고 생각한다.
    * 그리고, 코드도 보면 중복된 부분이 굉장히 많다..

---

### 중복되는 부분에 상수를 사용하지 않았다.

```java
public class Constants {
    public final static String ERROR_MSG_HEADER = "ERROR";
}
```

* 진짜 너무나도 후회되는 부분이다.
* 2주차, 3주차때 완벽하진 않아도 항상 의식하고 사용하려고 했던 부분인데, 제일 중요한 최종 코딩테스트에서 미뤄두고 있다가 사용하지 않았다.
    * 심지어 상수보관 클래스도 따로 미리 만들어두고..
* 나중에 리팩토링 해야겠다고 미뤄둔 부분들이, 시간 부족으로 한번에 곪아 터진것 같다.
    * **나중에** 리팩토링 한다는것은 아예 안한다는 말과 같다고 생각하련다.
    * 앞으로 한번에 짤때 제대로 짤 수 있도록 노력하려고 한다 ㅠ.

---

## 글을 마치며

* 솔직히 말해서 내가 우형측이라면, 기능도 전부 구현이 안되었고 이렇게 난장판인 코드를 짜는 사람을 최종합격 시키진 않을것 같다... 그리고 설계도 제대로 된것도 아니고..
    * 사실 내가 더 노력해서 프리코스를 진행했으면, 더 완성도 높은 코드를 제출했을것이다.
    * 이것은 전적인 나의 부족함, 잘못이다.

* 하지만 우테코 프리코스 과정을 통해서 
    * 나의 문제점이 무엇인지, 그리고 무엇을 고쳐야하는지
    * 앞으로 어떤것을 좀 더 공부하면 좋을지에 대한 이정표를 찾을 수 있게 된것 같다.
        * 단순히 이전처럼 취업을 위해 전산학, 알고리즘만 공부하지 않고, 좋은 코드를 짤 수 있는 연습.
        * 그리고 조그만 프로젝트를 조금씩 진행하면서 배운것을 적용해보는 연습을 해보려고 한다.
* 그리고 미션을 하는 동안, 시험을 치는 동안은 다른걸 잊고 코딩에 집중 할 수 있어서 너무 좋았던것 같다.
* 확률은 희박하지만, 이후 우테코 최종 합격 수기를 적을 수 있는 기회가 있었으면 좋겠다. 
    * ~~이후 관련 글이 없으면 ㅎㅎ..~~

* 30일 발표전까지는 우테코를 진행하면서 `TIL` 에 공부하고 정리한 문서들을 재 가공하며, 블로그에 포스팅 하려고한다. 