# Немного о центроидной декомпозиции

Практически очевидное утверждение: в любом дереве размера N найдётся центроид - вершина,  такая что при её удалении дерево распадётся на компоненты  связности,  размер каждой из которых меньше, чем N/2 (для перфекционистов - деление здесь не целочисленное)

Доказательство - назначим одну из вершин корнем и подвесим дерево за него, для каждой вершины подсчитаем размер её поддерева.  Изначально находимся в корне, его размер N >= N/2. На шаге поиска находим потомка вершины максимального размера,  если размер его поддерева >= N/2, переходим в него, в противном случае легко видеть, что элемент, в котором мы находимся, является центроидом. (Можно дополнительно доказать, что найденный центроид не будет зависеть от выбора корня, но это другой разговор)

Далее - есть большой класс задач, которые решаются с использованием этого факта (обычно - рекурсивным сведением задачи к той же задаче на нескольких деревьях меньшего размера)

Для затравки - простой пример:  проверить, существуют ли в заданном дереве 2 листа, 
длина пути между которыми в точности k.

Считаем, что дерево уже подвешено за вершину с номером 0 и хранится в виде массива parent, (parent[0] == 0)

```
class SearchKWay{
    final int[] parent;
    final int size;
    final int k;

    SearchKWay(int[] parent, int k){
         this.parent = parent;
         this.k = k;
         this.size = parent.length;
    }
```

Проинициализируем для каждой вершины множество соседних вершин и размер поддерева.
```
int [] subTreeSize;
List<Set<Integer>> neibours;

void init(){
    subTreeSize = new int[size];
    neibours = new ArrayList<>();
    for(int i = 0; i<size; ++i) neibours.add(new HashSet<>());
    for(int i = 1; i<size; ++i) {
        neibours.get(i).add(parent[i]);
        neibours.get(parent[i]).add(i);
    }
    initSizes(0, -1);
}

void initSizes(int vert, int prev){
    subTreeSize[vert] = 1;
    for(int i : neibours.get(i))
       if(i!=prev){
           initSizes(i, vert);
           subTreeSize[vert] += subTreeSize(i);
       }
}
```

Инициализируем список листьев:
```
  Set<Integer> leafs = new HashSet<>();
  void initLeafs(){
      for(int i = 0; i<size; ++i) 
         if(neibours.get(i).size() == 1)
              leafs.add(i);
  }
```

Зафиксируем некий малый размер дерева,  при котором поиск можно вести полным перебором, а не пытаться декомпозировать дерево

```
static final int STOP = 10;
```
(Заметим, что при таком ограничении на размер дерева центроид никогда не будет листом)

Теперь можно реализовать рекурсивную логику
```
    boolean findInTree(int root){
        int N = subTreeSize(root);
        if (N<=STOP) return findInSmallTree(root);
        
        //Ищем центроид
        int center = root;
        while(true){
            int ch = fattestChild(center);
            if(2*subTreeSize[ch]>= N) center = ch;
            else break;
        }
        //Ищем пары листьев,  таких что длина пути между ними - k, 
        //при этом центроид лежит на пути
        Set<Integer> dists = new HashSet<>();
        for(int i: neibours.get(center)){
            Set<Integer> subset = new HashSet<>();
            initSubset(i, center, 1, subset);
            for(int j: subset) 
               if(dists.contains(k-j)) return true;
            dists.addAll(subset);   
        }
        //Если не нашли - удаляем центроид и запускаемся 
        //от деревьев меньшего размера
        // лучше рассмотреть два случая - центроид совпадает с корнем и 
        
        if(root == center){
            for(int i: neibours.get(root)) {
                neibours.get(i).remove(root);
                if(findInTree(i)) return true;
            }
            return false;
        } else {
             for(int i: neibours.get(center)) {
                neibours.get(i).remove(center);
                if(findInTree(i)) return true;
            }
            int up = parent[center];
            neibours.get(up).remove(center);
            int d = subTreeSize[center];
            while(true){
                subTreeSize[up] -=d;
                if(up == root) break;
                up = parent[up];
            }
            return findInTree(root);
        }
    }
    
    void initSubset(int vert, int prev, int len, Set<Integer> collector){
        if (leafs.contains(vert)) collector.add(len);
        for(int i: neibours.get(vert))
            if (i!= prev)
               initSubset(i, vert, len+1, collector);
    }
    
    int fattestChild(final int c){
        return neibours.get(c)
             .stream()
             .filter(i->i!=parent[c])
             .reduce((i, j)-> subTreeSize[i]<subTreeSize[j]?j:i)
             .get();
    }
```

Вот и всё, только findInSmallTree писать лень

```
 boolean isPathExists(){
     init();
     initLeafs();
     return findInTree(0);
 }
```

Для перфекционистов - закрою скобку от класса

```
}
```


Пример менее очевидного применения в следующем тексте.
Задача: 
Дерево с длинами у рёбер,  реализовать структуру данных,  которая позволяет быстро выполнять две операции: находить длину пути между двумя вершинами и изменять длину ребра.
(продолжение следует)
