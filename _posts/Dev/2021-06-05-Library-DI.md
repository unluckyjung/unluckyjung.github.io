---
title: 외부 라이브러리를 열어보고 추상화한뒤 dependency injection 시키기
date: 2021-06-05-23:30
categories:
- Dev

tags:
- Java
- Dev

---

# 외부 라이브러리를 열어보고 DI 시키자
> 라이브러리를 단순히 가져다 쓰지만 말자, 어떻게 구현되어 있는지 파악하자

---

## Goal
- 외부 라이브러리 기능을 추상화 시킨다.
- 기능자체에만 집중하여 기능에 종속적였던 코드를, 비 종속적인 코드로 변경한다.

---

## 외부 라이브러리 내부를 까보고 인터페이스를 이용해 DI 시키자.
이전까지 라이브러리를 docs를 보고 사용법만 익히고 사용했다.  
예를들어 최단 경로찾기 기능을 수행하는 메소드는, [JpraphT 라이브러리](https://github.com/jgrapht/jgrapht)를 이용해 당장 사용하는 Dijkstra 알고리즘에 종속적인 코드가 되어버렸었다.  

물론 알고리즘 별로 각각의 클래스나 메소드를 만들어주면 되지만, 
추상화하여 **한개의 메소드**로  `xx알고리즘으로, xx그래프`를 이용해서 **최단 경로를 찾아줘** 라는 메시지를 보내고 싶다면?  
근데 이게 외부 라이브러리였고, docs에 이런 자세한 설명이 없었다면?  
단순히 xx 알고리즘을 어떻게 사용하는지에 대한 **사용법** 에 대해서만 docs에 적혀있었다면?  
**어떻게 할까?**  


---

## 라이브러리 코드 분석
```java
public List<Station> getShortestPath(final Station sourceStation, final Station targetStation) {
    DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(pathGraph.getGraph());
    return dijkstraShortestPath.getPath(sourceStation, targetStation).getVertexList();
}
```

기존의 **getShortestPath** 메소드는 `DijkstraShortestPath`에 종속적인 메소드다.  
`DijkstraShortestPath`는 내가 만든 도메인이 아닌 라이브러리에 구현되어있는 클래스이다.  

나는 단순히 라이브러리를 docs에서 알려주는것만 보고 파악해서 사용하는것에서 멈추고 싶지 않다.  
`DijkstraShortestPath` 내부를 들여다보자  

![image](https://user-images.githubusercontent.com/43930419/120884022-9881eb80-c61b-11eb-8009-abdaff33a7ee.png)


```java
public final class DijkstraShortestPath<V, E>
    extends BaseShortestPathAlgorithm<V, E>
```

`DijkstraShortestPath`는 `BaseShortestPathAlgorithm` 를 상속하고 있다.

`BaseShortestPathAlgorithm`도 들여다보자  

![image](https://user-images.githubusercontent.com/43930419/120884049-b51e2380-c61b-11eb-8916-e3d6d1be2bab.png)


```java
abstract class BaseShortestPathAlgorithm<V, E>
    implements ShortestPathAlgorithm<V, E>
```

`BaseShortestPathAlgorithm`는 `ShortestPathAlgorithm` 인터페이스를 implements하고 있다.  

![image](https://user-images.githubusercontent.com/43930419/120884061-c49d6c80-c61b-11eb-885a-5f6ec59ae310.png)


```java
public interface ShortestPathAlgorithm<V, E>
```

`ShortestPathAlgorithm` 는 최상위 인터페이스로 존재하고 있는것을 확인했다.  

나는 이제 도메인을 파악했으므로 [공식문서](https://jgrapht.org/guide/UserOverview#graph-algorithms)에 있는걸 보고  
기능만 파악해 있는그대로 사용하는것이 아니라, 인터페이스를 이용해 DI를 시켜 확장성 있는 코드로 바꿀수 있는 방법을 찾았다.  

즉 `getShortestPath()` 메소드를 다익스트라에 비종속적인 메소드로 바꿀수 있게 되었다.

---

## 전략패턴을 이용하여 외부 주입시킨다.

```java
public enum ShortestPathStrategy {

    DIJKSTRA {
        @Override
        public ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph) {
            return new DijkstraShortestPath<>(pathGraph.getGraph());
        }
    },
    FLOYD_WARSHALL {
        @Override
        public ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph) {
            return new FloydWarshallShortestPaths<>(pathGraph.getGraph());
        }
    };

    public abstract ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph);
}


// 길을 찾아주는 객체
public StationPathFinder(final PathGraph pathGraph, final ShortestPathStrategy shortestPathStrategy) {
    this.pathGraph = pathGraph;
    this.shortestPathStrategy = shortestPathStrategy;

    public List<Station> getShortestPath(final Station sourceStation, final Station targetStation) {
        return shortestPathStrategy.match(pathGraph).getPath(sourceStation, targetStation).getVertexList();
    }
}
```
- `ShortestPathAlgorithm` 인터페이스를 이용해 최단경로를 찾는 전략 enum을 만든다.  
- 최단 경로를 찾아주는 `getShortestPath()` 메소드는 이제 다익스트라에 비 종속적인 메소드가 되었다.  

이제 내가 나만의 길찾기 알고리즘을 만들고 적용하고 싶다면(그럴리는 없겟지만), 단순히 Enum에 전략을 더 추가해주면 된다.  

라이브러리를 docs만 보고 그대로 사용하는것이 아니라, 내부를 까보고  
내가 내 도메인에 맞춰서, 좀 더 유연성있게, 응집성 있도록 응용해서 쓸 수 있다는 인사이트를 얻은 좋은 경험이었다.


---

## 코드 정리

### 수정전

```java
public class PathFinder {
    private final PathGraph pathGraph;

    public PathFinder(final PathGraph pathGraph) {
        this.pathGraph = pathGraph;
    }

    public List<Station> getShortestPath(final Station sourceStation, final Station targetStation) {
         //다익스트라에 종속적
        DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(pathGraph.getGraph());
        return dijkstraShortestPath.getPath(sourceStation, targetStation).getVertexList();
    }

    // public List<Station> getShortestPathUseDijstra()
    // public List<Station> getShortestPathUseFloyd()
    // 위와 같은 방식이면 매번 이런식으로 알고리즘별 메소드를 만들어 주어야함
    ...
}

```

```java
public PathResponse findShortestPath(final Long source, final Long target) {
    PathFinder pathfinder = new PathFinder(new PathGraph(lineDao.findAll()));

    Station sourceStation = stationDao.findById(source);
    Station targetStation = stationDao.findById(target);

    List<Station> shortestPath = pathfinder.getShortestPath(sourceStation, targetStation);
}
```

### 수정 후


```java
public enum ShortestPathStrategy {

    DIJKSTRA {
        @Override
        public ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph) {
            return new DijkstraShortestPath<>(pathGraph.getGraph());
        }
    },
    FLOYD_WARSHALL {
        @Override
        public ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph) {
            return new FloydWarshallShortestPaths<>(pathGraph.getGraph());
        }
    };

    public abstract ShortestPathAlgorithm<Station, DefaultWeightedEdge> match(final PathGraph pathGraph);
}
```

```java
public class StationPathFinder {
    private final PathGraph pathGraph;
    private final ShortestPathStrategy shortestPathStrategy;

    public StationPathFinder(final PathGraph pathGraph, final ShortestPathStrategy shortestPathStrategy) {
        this.pathGraph = pathGraph;
        this.shortestPathStrategy = shortestPathStrategy;
    }

    public List<Station> getShortestPath(final Station sourceStation, final Station targetStation) {
        // 더이상 다익스트라에 종속적이지 않게 됨 
        return shortestPathStrategy.match(pathGraph).getPath(sourceStation, targetStation).getVertexList();
    }
}
```

```java
public PathResponse findShortestPath(final Long source, final Long target) {
    StationPathFinder pathfinder = new StationPathFinder(
            new PathGraph(lineDao.findAll(), new WeightedMultigraph<>(DefaultWeightedEdge.class)),
            // 외부에서 원하는 알고리즘을 주입시킬 수 있게 변경
            ShortestPathStrategy.DIJKSTRA
    );
    Station sourceStation = stationDao.findById(source);
    Station targetStation = stationDao.findById(target);

    List<Station> shortestPath = pathfinder.getShortestPath(sourceStation, targetStation);
}
```

---

## Reference

- https://github.com/woowacourse/atdd-subway-path/pull/114#discussion_r634515420
- https://www.baeldung.com/jgrapht
- https://jgrapht.org/guide/UserOverview#graph-algorithms