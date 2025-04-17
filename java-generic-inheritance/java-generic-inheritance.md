Мне нужен некоторый контейнер, который может хранить элементы. И у него может быть множество реализаций. Причём разные реализации может быть как у контейнера, так и его элементов.

Я сначала сделал так:

```java
interface Container {  
    List<? extends Element> getElements();  
}  
  
interface Element {  
}  
```

И реализовал:

```java
class RealElement implements Element {  
    public RealElement(Datum d) {  
    }  
}  

class Datum {}  
  
  
class RealContainer implements Container {  
    @Override  
    public List <RealElement> getElements() {  
        return this.data.stream().map(RealElement::new).toList();  
    }  
  
    private List<Datum> data;  
}
```

Работает, но линтер ругается:

![](./20250417141116.png)

Не вдаваясь в подробности — смысл в том, что теория дженериков в Java гласит: дженерики c вайлдкардами (типа `<? extends SomeClass>` можно использовать только во входных параметрах. Ну, оно логично в рамках того, как обобщённые типы сделаны в Java.

Просто изменить, как просят, `getElements` в `Container`

```java
interface Container {  
    List<Element> getElements();  
}
```

не достаточно — сломается RealContainer:
![](./20250417141625.png)

Если (что логично) поменять выходной тип на `List<Element>`, то появляется другая ошибка:

![](./20250417141740.png)
 
Тут мне пришлось потратить время на поиск решения, которое оказалось простым — использовать `.collect`:

```java
        return this.data.stream().map(RealElement::new).collect(Collectors.toList());  
```

"Волшебство" заключается в том, что `.collect(Collectors.toList())` в отличие от `.toList()` параметризован более универсально:

```java
 <R, A> R collect(Collector<? super T, A, R> collector);
```

против

```java
default List<T> toList()
```


На будущее, если захочется ещё больше гибкости, то исходный `Container` можно тоже (правильно) параметризовать:

```java
interface Container<T extends Element>
```

И в этом случае, кстати, уже вполне подходит `.toList()`.

```java
    interface Container<T extends Element> {
        List<T> getElements();
    }

    interface Element {
    }


    class RealElement implements Element {
        public RealElement(Datum d) {
        }
    }

    interface Datum {}

    class RealContainer implements Container<RealElement> {
        @Override
        public List<RealElement> getElements() {
            //return this.data.stream().map(RealElement::new).toList();
            return this.data.stream().map(RealElement::new).toList();
        }

        private List<Datum> data;
    }
```


