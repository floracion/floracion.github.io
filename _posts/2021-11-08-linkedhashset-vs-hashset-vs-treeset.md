---
layout: post
title: LinkedHashSet VS HashSet VS TreeSet
subtitle: Sangmin Lee
categories: java, data-structure
tags: [java, data-structure]
---

이번 글에서는 `LinkedHashSet` 에 대해 간단히 설명하며, `LinkedHashSet` 과 `HashSet` , `TreeSet` 에 대한 성능 평가를 진행한다. 

여러 상황에 따라 다른 성능을 보이는 `Set` 컬렉션에 대해 분석한 결과를 기록한다.

## 개념

`LinkedHashSet` 은 해시테이블과 이중 연결 리스트를 활용하여 구현한 클래스이다.

`HashSet` 은 각 원소끼리 연결되어 있지 않지만, `LinkedHashSet` 은 각 원소끼리 연결되어 있으므로 삽입 순서를 유지할 수 있다.

`LinkedHashSet` 은 일반 `HashSet` 과 동일하게 고유한 요소만 포함된다.


## 성능 비교

- Profiler 클래스
  
    ```java
    public class Profiler extends ApplicationFrame {
    
      private static final long serialVersionUID = 1L;
    
      public interface Timeable {
        public void setup(int n);
    
        public void timeMe(int n);
      }
    
      private Timeable timeable;
    
      public Profiler(String title, Timeable timeable) {
        super(title);
        this.timeable = timeable;
      }
    
      public XYSeries timingLoop(int startN, int endMillis) {
        final XYSeries series = new XYSeries("Time (ms)");
    
        int n = startN;
        for (int i = 0; i < 20; i++) {
          // run it once to warm up
          timeIt(n);
    
          // then start timing
          long total = 0;
    
          // run 10 times and add up total runtime
          for (int j = 0; j < 10; j++) {
            total += timeIt(n);
          }
          System.out.println(n + ", " + total);
    
          // don't store data until we get to 4ms
          if (total > 4) {
            series.add(n, total);
          }
    
          // stop when the runtime exceeds the end threshold
          if (total > endMillis) {
            break;
          }
          // otherwise double the size and continue
          n *= 2;
        }
        return series;
      }
    
      public long timeIt(int n) {
        timeable.setup(n);
        final long startTime = System.currentTimeMillis();
        timeable.timeMe(n);
        final long endTime = System.currentTimeMillis();
        return endTime - startTime;
      }
    
      public void plotResults(XYSeries series) {
        double slope = estimateSlope(series);
        System.out.println("Estimated slope= " + slope);
    
        final XYSeriesCollection dataset = new XYSeriesCollection();
        dataset.addSeries(series);
    
        final JFreeChart chart = ChartFactory.createXYLineChart(
            "",          // chart title
            "",               // domain axis label
            "",                  // range axis label
            dataset,                  // data
            PlotOrientation.VERTICAL,
            false,                     // include legend
            true,
            false
        );
    
        final XYPlot plot = chart.getXYPlot();
        final NumberAxis domainAxis = new LogarithmicAxis("Problem size (n)");
        final NumberAxis rangeAxis = new LogarithmicAxis("Runtime (ms)");
        plot.setDomainAxis(domainAxis);
        plot.setRangeAxis(rangeAxis);
        chart.setBackgroundPaint(Color.white);
        plot.setOutlinePaint(Color.black);
        final ChartPanel chartPanel = new ChartPanel(chart);
        chartPanel.setPreferredSize(new java.awt.Dimension(1000, 600));
        setContentPane(chartPanel);
        pack();
        RefineryUtilities.centerFrameOnScreen(this);
        setVisible(true);
      }
    
      public double estimateSlope(XYSeries series) {
        SimpleRegression regression = new SimpleRegression();
    
        for (Object item : series.getItems()) {
          XYDataItem xy = (XYDataItem) item;
          regression.addData(Math.log(xy.getXValue()), Math.log(xy.getYValue()));
        }
        return regression.getSlope();
      }
    }
    ```
    



## 랜덤 숫자를 컬렉션에 추가하는 연산 성능 비교

성능 측정 결과는 `(원소개수, 소요된 밀리초)` 를 의미한다.



### LinkedHashSet

- 시간 측정 코드
  
    ```java
    public class LinkedHashSetAddTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable linkedHashSetProfile = new Timeable() {
    
          private int[] array;
          private LinkedHashSet<Integer> linkedHashSet;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            linkedHashSet = new LinkedHashSet<>();
          }
    
          @Override
          public void timeMe(final int n) {
            Arrays.stream(array).forEach(linkedHashSet::add);
          }
        };
    
        runProfiler("Add to LinkedHashSet profile", linkedHashSetProfile, 10_000, 1_000 * 10);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 15
  20000, 27
  40000, 29
  80000, 95
  160000, 392
  320000, 538
  640000, 1375
  1280000, 3236
  2560000, 7715
  5120000, 17649
  Estimated slope= 1.1814940757524521
  ```



### HashSet

- 시간 측정 코드
  
    ```java
    public class HashSetAddTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable hashSetProfile = new Timeable() {
    
          private int[] array;
          private HashSet<Integer> hashSet;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            hashSet = new HashSet<>();
          }
    
          @Override
          public void timeMe(final int n) {
            Arrays.stream(array).forEach(hashSet::add);
          }
        };
    
        runProfiler("Add to HashSet profile", hashSetProfile, 10_000, 1_000 * 10);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 11
  20000, 21
  40000, 32
  80000, 62
  160000, 270
  320000, 457
  640000, 961
  1280000, 2352
  2560000, 5569
  5120000, 12066
  Estimated slope= 1.156784458972608
  ```



### TreeSet

- 시간 측정 코드
  
    ```java
    public class TreeSetAddTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable treeSetProfile = new Timeable() {
    
          private int[] array;
          private TreeSet<Integer> treeSet;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            treeSet = new TreeSet<>();
          }
    
          @Override
          public void timeMe(final int n) {
            Arrays.stream(array).forEach(treeSet::add);
          }
        };
    
        runProfiler("Add to TreeSet profile", treeSetProfile, 10_000, 1_000 * 10);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 50
  20000, 70
  40000, 97
  80000, 239
  160000, 668
  320000, 1690
  640000, 4080
  1280000, 9680
  2560000, 26415
  Estimated slope= 1.185436687614054
  ```



### 측정 결과

속도 : **HashSet > LinkedHashSet > TreeSet**

삽입 연산은 `HashSet` 이 가장 빠른 것을 확인할 수 있다.



## 원소를 삽입한 순서대로 출력

[ 문자열을 추가하고 순서대로 출력하는 코드 ]

```java
public class SetTraverseTest {

  public static void main(String[] args) {
    final LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();
    final HashSet<String> hashSet = new HashSet<>();
    final TreeSet<String> treeSet = new TreeSet<>();

    addElementsToSet(linkedHashSet);
    addElementsToSet(hashSet);
    addElementsToSet(treeSet);

    System.out.println("LinkedHashSet");
    linkedHashSet.forEach(System.out::println);
    System.out.println();
    System.out.println("HashSet");
    hashSet.forEach(System.out::println);
    System.out.println();
    System.out.println("TreeSet");
    treeSet.forEach(System.out::println);
  }

  private static void addElementsToSet(final Set<String> set) {
    set.add("a");
    set.add("c");
    set.add("b");
    set.add("e");
    set.add("d");
    set.add("z");
    set.add("f");
    set.add("나");
    set.add("가");
  }

}
```

[ 실행결과 ]

```java
LinkedHashSet
a
c
b
e
d
z
f
나
가

HashSet
가
a
b
c
d
e
f
나
z

TreeSet
a
b
c
d
e
f
z
가
나
```

- `LinkedHashSet`
    - 삽입한 순서대로 원소들이 출력되는 것을 확인할 수 있다.
- `HashSet`
    - 원소의 순서가 없는 것을 확인할 수 있다.
- `TreeSet`
    - 원소들이 정렬되어 있는 것을 확인할 수 있다.



## 원소 검색 성능 비교

성능 측정 결과는 `(원소개수, 소요된 나노초)` 를 의미한다.

### LinkedHashSet

- 시간 측정 코드
  
    ```java
    public class LinkedHashSetContainsTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable linkedHashSetProfile = new Timeable() {
    
          private int[] array;
          private LinkedHashSet<Integer> linkedHashSet;
          private int valueToFind;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            linkedHashSet = new LinkedHashSet<>();
            for (int i = 0; i < array.length; i++) {
              if (array.length / 2 == i) {
                valueToFind = array[i];
              }
              linkedHashSet.add(array[i]);
            }
          }
    
          @Override
          public void timeMe(final int n) {
            linkedHashSet.contains(valueToFind);
          }
        };
    
        runProfiler("Contains to LinkedHashSet profile", linkedHashSetProfile, 10_000, 100_000);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 38626
  20000, 81781
  40000, 77435
  80000, 55726
  160000, 57648
  320000, 54719
  640000, 69082
  1280000, 93231
  2560000, 94099
  5120000, 91719
  10240000, 907834
  Estimated slope= 0.23659345176160207
  ```


### HashSet

- 시간 측정 코드
  
    ```java
    public class HashSetContainsTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable hashSetProfile = new Timeable() {
    
          private int[] array;
          private HashSet<Integer> hashSet;
          private int valueToFind;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            hashSet = new HashSet<>();
            for (int i = 0; i < array.length; i++) {
              if (array.length / 2 == i) {
                valueToFind = array[i];
              }
              hashSet.add(array[i]);
            }
          }
    
          @Override
          public void timeMe(final int n) {
            hashSet.contains(valueToFind);
          }
        };
    
        runProfiler("Contains to HashSet profile", hashSetProfile, 10_000, 100_000);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 48783
  20000, 41983
  40000, 43307
  80000, 49577
  160000, 53564
  320000, 77174
  640000, 81365
  1280000, 70816
  2560000, 98010
  5120000, 100458
  Estimated slope= 0.14642171739088547
  ```

### TreeSet

- 시간 측정 코드
  
    ```java
    public class TreeSetContainsTest {
    
      public static void runProfiler(String title, Timeable timeable, int startN, int endMillis) {
        Profiler profiler = new Profiler(title, timeable);
        XYSeries series = profiler.timingLoop(startN, endMillis);
        profiler.plotResults(series);
      }
    
      public static void main(String[] args) {
        Timeable treeSetProfile = new Timeable() {
    
          private int[] array;
          private TreeSet<Integer> treeSet;
          private int valueToFind;
    
          @Override
          public void setup(final int n) {
            array = new Random()
                .ints(n, 0, Integer.MAX_VALUE)
                .toArray();
            treeSet = new TreeSet<>();
            for (int i = 0; i < array.length; i++) {
              if (array.length / 2 == i) {
                valueToFind = array[i];
              }
              treeSet.add(array[i]);
            }
          }
    
          @Override
          public void timeMe(final int n) {
            treeSet.contains(valueToFind);
          }
        };
    
        runProfiler("Contains to TreeSet profile", treeSetProfile, 10_000, 100_000);
      }
    
    }
    ```
    
- 성능 측정 결과

  ```java
  10000, 62815
  20000, 105543
  Estimated slope= 0.7486498811689078
  ```



### 측정 결과

LinkedHashSet과 HashSet은 성능이 유사하며, TreeSet은 HashSet에 비해 속도가 떨어지는 것을 확인할 수 있다.

## Reference

[[Kotlin Collection] HashSet과 LinkedHashSet](https://kotlinworld.com/6)

[LinkedHashSet in Java - javatpoint](https://www.javatpoint.com/java-linkedhashset)

---