Мне нужен некоторый контейнер, который может хранить элементы. И у него может быть множество реализаций. Причём разные реализации может быть как у контейнера, так и его элементов.

Я сначала сделал так:

```
interface Container {  
    List<? extends Element> getElements();  
}  
  
interface Element {  
}  
```

И реализовал:

```
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

![[Pasted image 20250417141116.png]]

Смысл в том, что теория дженериков в Java гласит, что дженерики c вайлдкардами (типа `<? extends SomeClass>`) можно использовать только во входных параметрах. Ну, оно логично в рамках того, как обобщённые типы сделаны в Java.

И я долго провозился с тем, чтобы эту проблемку решить. 

Просто изменить, как просят, getElements в Container — не достаточно:

```
interface Container {  
    List<Element> getElements();  
}
```

Тогда сломается RealContainer:
![[Pasted image 20250417141625.png]]
Если (что логично) поменять выходной тип на `List<Element>`, то появляется другая ошибка:
 ![[Pasted image 20250417141740.png]]
Довольно долго искал, но, как оказалось, можно вот так сделать:
```
class RealContainer implements Container {  
    @Override  
    public List <Element> getElements() {  
        return this.data.stream().map(RealElement::new).class RealContainer implements Container {  
    @Override  
    public List <Element> getElements() {  
        return this.data.stream().map(RealElement::new).collect(Collectors.toList());  
    }  
  
    private List<Datum> data;  
};  
    }  
  
    private List<Datum> data;  
}
```

"Волшебство" заключается в том, что `.collect(Collectors.toList())`в отличие от .toList() параметризован более универсально:
```
 <R, A> R collect(Collector<? super T, A, R> collector);
```
против
```
default List<T> toList()
```
