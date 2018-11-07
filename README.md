

# Теория машинного обучения #
## Метрические алгоритмы классификации ##
Метрические алгоритмы классификации - алгоритмы, основанные на вычислении оценок сходства между объектами на основе весовых функций.
### Метод *k* ближайших соседей (*kNN*) и частный случай при *k = 1* (*1NN*) ###
Один из наиболее простых метрических алгоритмов классификации.
Работает следующим образом: дан классифицируемый объект *z* и обучающая выборка ![](http://latex.codecogs.com/gif.latex?%24X%5El%24). Требуется определить класс объекта *z* на основе данных из обучающей выборки. Для этого:
1. Вся выборка ![](http://latex.codecogs.com/gif.latex?%24X%5El%24) сортируется по возрастанию расстояния от объекта *z* до каждого объекта выборки.
2. Проверяются классы *k* ближайших соседей объекта *z*. Класс, встречаемый наиболее часто среди *k* соседей, присваивается объекту *z*.  

Исходными данными для алгоритма являются: классифицируемый объект, обучающая выборка и параметр *k* - число рассматриваемых ближайших соседей.
Результатом работы метода является класс классифицируемого объекта.

При *k = 1* алгоритм превращается в частный случай *1NN*. В таком слачае, рассматриваемый объект *z* присваивается к классу его первого ближайшего соседа. В свою очередь, остальные объекты не рассматриваются.

Пример работы *1NN* при использовании в качестве обучающей выборки Ирисы Фишера:

![1NN.png](https://github.com/VladislavDuma/SMPR/blob/master/img/1nn_allelem_2.png)

Для проверки оптимальности *k* используется Критерий Скользящего Контроля *LOO* (Leave One Out).
Данный критерий проверяет оптимальность значения *k* следующим образом:
1. Из обучающей выборки удаляется *i*-й объект ![](http://latex.codecogs.com/gif.latex?%24x%5Ei%24).
2. Запоминаем "старый" класс *i*-го объекта.
3. Запускаем алгоритм для оставшейся выборки. В результате работы *i*-му элементу присваивается "новый" класс на основе имеющейся выборки. Если значения "нового" и "старого" класса совпали, то *i*-ый элемент классифицировало верно. При их же несовпадении сумма ошибки увеличивается на 1.
4. Шаги 1-3 повторяются для каждого объекта выборки при фиксированном *k*. По окончании работы алгоритма полученная сумма ошибки *sum* делится на размер выборки *l*: ![sum=sum/l](http://latex.codecogs.com/gif.latex?sum%3D%20%5Cfrac%7Bsum%7D%7Bl%7D) .  Потом значение *k* меняется, и алгоритм повторяется для нового значения. *k* с наименьшим значением суммы ошибки будет оптимальным.
#### Реализация kNN
При реализации алгоритма, в качестве обучающей выборки использовалась выборка ирисов Фишера. В качестве признаков объектов использовались значения длины и ширины лепестка. Значение *k* подбиралось по *LOO*.

Алгоритм:

    kNN <- function(xl, k, z) {
	  orderedXL <- sortObjectByDist(xl, z)
	  n <- dim(orderedXL)[2]
	  classes <- orderedXL[1:k, n] 
	  counts <- table(classes) # Таблица встречаемости каждого класса среди k ближайших соседей объекта
	  class <- names(which.max(counts)) # Наиболее часто встречаемый класс
	  return (class)
	}
где *xl* - обучающая выборка.

Пример работы метода kNN при k = 10 даёт следующий результат.

![kNN.png](https://github.com/VladislavDuma/SMPR/blob/master/img/kNN_10elem_2.png)

Применив критерий *LOO* для получения оптимального *k* мы получаем следующий ответ:

![LOO_for_kNN.png](https://github.com/VladislavDuma/SMPR/blob/master/img/LOO_for_kNN_3.png)

Следовательно оптимальным *k* на выборке Ирисы Фишера явлется значение *k = 6*. Построим графики *kNN*:

![kNN_k6.png](https://github.com/VladislavDuma/SMPR/blob/master/img/kNN_k6_v1.png)

Достоинства алгоритма:
1. Простота реализации (относительно)
2. Хорошее качество, при правильно подобранной метрике и параметрах *k* и *q*

Недостатки алгоритма:
1. Необходимость хранить выборку целиком, как следствие - неэффективное использование памяти
2. Малый набор параметров
3. Качество классификации сильно зависит от выбранной метрики
4. "Выбросы" могут значительно ухудшить точность

### Метод *k* ближайших соседей (*kwNN*) ###
Метод *kwNN* отличается от *kNN* тем, что вес ближайших соседей зависит не от ранга соседа, а от расстояния до объекта z. В методе *kNN* считается, что вес каждого *k*-соседа равен 1. По сути, мы считали частоту появления классов среди ближайших *k* соседей.
Применяя же метод *kwNN* в качестве весовой функции мы используем *w* = *q^i*, что соответствует методу *k* экспоненциально взвешенных ближайших соседей. Предполагается, что *q* принадлежит [0.1; 1.0] и рассматриваем с шагом 0.1.

#### Реализация kwNN

Алгоритм:

	kwNN <- function(xl, z, k, q)
	{
	  orderedXl <- sortObjectByDist(xl, z)	# сортируем выборку
	  n <- dim(xl)[2]			# размерность выборки по столбцам

	  classes <- orderedXl[1:k, n] 		# берём k ближайших соседей

	  classes <- table(classes)    		# создаём таблицу для них
	  classes[1:length(classes)] <- 0	# обнуляем значения
	  
	  for (i in names(classes)) {		# для всех классов
	    for (j in 1:k) {			# для всех значений k
	      if (orderedXl[j, n] == i)		
		classes[i] = classes[i] + q^j	# суммируем веса для всех объектов одного класса
	    }
	  }
	  class <- names(which.max(classes))	# возвращаем класс соответствующий максимальному весу
	  return (class)
	}
	
Для выборки Ирисы Фишера получаем следующий результат:

![kwNN.png](https://github.com/VladislavDuma/SMPR/blob/master/img/kwNN_v1.png)

В результате работы мы получили, что наиболее оптимальными явлеются значения *k = 6*.

Достоинства алгоритма:
1. Простота реализации
2. Хорошее качество, при правильно подобранной метрике и параметре *k*

Недостатки алгоритма:
1. Необходимость хранить выборку целиком, как следствие - неэффективное использование памяти
2. Чрезмерное усложнение решающего правила
3. Качество классификации сильно зависит от выбранной метрики
4. Поиск ближайшего соседа предполагает сравнение классифицируемого объекта со всеми объектами выборки, что требует линейного по длине выборки числа операций.

### Метод парзеновского окна ###
Основное отличие метода от kNN и kwNN заключается в том, что весовую функцию мы рассматриваем не как ранговую, а как функцию расстояния.

Функция веса в таком случае выглядит следующим образом: ![](http://latex.codecogs.com/gif.latex?w%28i%2C%20z%29%20%3D%20K%28%5Cfrac%7B%5Crho%28z%2C%20x_%7Bi%7D%29%7D%7Bh%7D%29)

Где i - номер объекта выборки, z - классифицируемый объект, xi - i-й объект выборки, h - ширина окна, K - функция ядра.

В результате на выборке Ириса Фишера мы получили следующие результаты:

*Ядро Епанечникова*
![parz_ep.png](https://github.com/VladislavDuma/SMPR/blob/master/img/parzen_Ep.png)

*Квартическое ядро*
![parz_qt.png](https://github.com/VladislavDuma/SMPR/blob/master/img/parzen_Qart.png)

*Треугольное ядро*
![parz_tr.png](https://github.com/VladislavDuma/SMPR/blob/master/img/parzen_trian.png)

*Гауссовское ядро*
![parz_gs.png](https://github.com/VladislavDuma/SMPR/blob/master/img/Parzen_Gauss.png)

*Прямоугольное ядро*
![parz_gs.png](https://github.com/VladislavDuma/SMPR/blob/master/img/parzen_Pr.png)
