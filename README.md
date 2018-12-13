

# Теория машинного обучения #

## Содержание
* [Метрические алгоритмы классификации](#метрические-алгоритмы-классификации) 
	* [Метод *k* ближайших соседей *(kNN)*](#метод-k-ближайших-соседей-knn)  
	* [Метод *k* взвешенных ближайших соседей (*kwNN*)](#метод-k-взвешенных-ближайших-соседей-kwnn)  
	* [Метод парзеновского окна](#метод-парзеновского-окна)
	* [Метод потенциальных функций](#метод-потенциальных-функций)
* [Байесовские алгоритмы классификации](#байесовские-алгоритмы-классификации)
	* [Нормальный дискриминантный анализ](#нормальный-дискриминантный-анализ)

#### Сводная таблица для метрических методов классификации ####

| *Метод*    							|*Оптимальный параметр*	| *LOO*, Скользящий контроль 	|*LOO, %*| 
| --------------------------------------------------------------|--------------------: 	| --------------------------:	| -----: | 
| [1NN](#метод-k-ближайших-соседей-knn)   			|  k = 1   		|   -  				|    -   |
| [kNN](#метод-k-ближайших-соседей-knn)   			|  k = 6   		|   5  				|  3.33  |
| [kwNN](#метод-k-взвешенных-ближайших-соседей-kwnn) 		|  k = 6, q = 0.1	|   5  				|  3.33  | 
| [Парзеновского окна, ядро Епанечникова](#метод-парзеновского-окна) | h = 0.4  	|   6   			|    4   |
| [Парзеновского окна, ядро Квартическое](#метод-парзеновского-окна) | h = 0.4  	|   6   			|    4   |
| [Парзеновского окна, ядро Треугольное](#метод-парзеновского-окна)  | h = 0.4  	|   6   			|    4   |
| [Парзеновского окна, ядро Прямоугольное](#метод-парзеновского-окна)| h = 0.4  	|   6   			|    4   |
| [Парзеновского окна, ядро Гауссовское](#метод-парзеновского-окна)  | h = 0.1  	|   6   			|    4   |

## Метрические алгоритмы классификации ##
Метрические алгоритмы классификации - алгоритмы, основанные на вычислении оценок сходства между объектами на основе весовых функций.
### Метод *k* ближайших соседей (*kNN*) ###
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
2. Хорошее качество, при правильно подобранной метрике и параметре *k*

Недостатки алгоритма:
1. Необходимость хранить выборку целиком, как следствие - неэффективное использование памяти
2. Малый набор параметров
3. Качество классификации сильно зависит от выбранной метрики
4. "Выбросы" могут значительно ухудшить точность

[К началу алгоритма (*kNN*)](#метод-k-ближайших-соседей-knn)

[Вернуться к содержанию](#содержание)

### Метод *k* взвешенных ближайших соседей (*kwNN*) ###
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
2. Хорошее качество, при правильно подобранной метрике и параметрах *k* и *q*

Недостатки алгоритма:
1. Необходимость хранить выборку целиком, как следствие - неэффективное использование памяти
2. Чрезмерное усложнение решающего правила
3. Качество классификации сильно зависит от выбранной метрики
4. Поиск ближайшего соседа предполагает сравнение классифицируемого объекта со всеми объектами выборки, что требует линейного по длине выборки числа операций.

[К началу алгоритма (*kwNN*)](#метод-k-взвешенных-ближайших-соседей-kwnn)

[Вернуться к содержанию](#содержание)

### Метод парзеновского окна ###
Основное отличие метода парзеновского окна от метода ближайших соседей заключается в том, что весовую функцию мы рассматриваем не как ранговую, а как функцию расстояния.

Функция веса в таком случае выглядит следующим образом:

![](http://latex.codecogs.com/gif.latex?w%28i%2C%20z%29%20%3D%20K%28%5Cfrac%7B%5Crho%28z%2C%20x_%7Bi%7D%29%7D%7Bh%7D%29)

Где i - номер объекта выборки, z - классифицируемый объект, xi - i-й объект выборки, h - ширина окна, K - функция ядра.

Функция ядра - произвольная чётная функция, невозрастающая на *[0, +inf)*. На практике применяются следующие функции ядра:
1.  ![](http://latex.codecogs.com/gif.latex?K%28r%29%20%3D%20E%28r%29%20%3D%20%5Cfrac%7B3%7D%7B4%7D%281%20-%20r%5E%7B2%7D%29%5B%7Cr%7C%3C%3D1%5D) - Ядро Епанечникова
2.  ![](http://latex.codecogs.com/gif.latex?K%28r%29%20%3D%20Q%28r%29%20%3D%20%5Cfrac%7B15%7D%7B16%7D%281%20-%20r%5E%7B2%7D%29%5E%7B2%7D%5B%7Cr%7C%3C%3D1%5D) - Квартическое ядро
3.  ![](http://latex.codecogs.com/gif.latex?K%28r%29%20%3D%20T%28r%29%20%3D%20%281%20-%20%7Cr%7C%29%5B%7Cr%7C%3C%3D1%5D) - Треугольное ядро
4.  ![](http://latex.codecogs.com/gif.latex?K%28r%29%20%3D%20P%28r%29%20%3D%20%5Cfrac%7B1%7D%7B2%7D%5B%7Cr%7C%20%3C%3D%201%5D) - Прямоугольное ядро
5.  ![](http://latex.codecogs.com/gif.latex?K%28r%29%20%3D%20G%28r%29%20%3D%20%282%5Cpi%29%5E%7B-%5Cfrac%7B1%7D%7B2%7D%7De%5E%7B%28-%5Cfrac%7B1%7D%7B2%7Dr%5E%7B2%7D%29%7D) - Гауссовское ядро

Где: ![](http://latex.codecogs.com/gif.latex?r%3D%5Cfrac%7B%5Crho%28z%2C%20x_%7Bi%7D%29%7D%7Bh%7D)

#### Реализация ####

Алгоритм:

	parzen <- function(xl, h, distances, type_core) {
  	  # Оценка весовой функции по расстоянию, а не по рангу 
	  # h - ширина окна
	  # distances - расстояния от точки z до каждого объекта из выборки xl 
	  # type_core - функция ядра
	  l <- nrow(xl) # строки
	  n <- ncol(xl) # столбцы (размерность)

	  classes <- xl[1:l, n] # Классы объектов выборки
	  weights <- table(classes) # Таблица весов классов
	  weights[1:length(weights)] <- 0

	  for (i in 1:l) { # Для всех объектов выборки
	    class <- xl[i, n] # Берём его класс
	    r <- distances[i] / h
	    weights[class] <- weights[class] + type_core(r) # И прибавляем его вес к общему весу его класса
	  }

	  if (max(weights) != 0) # Если веса точки по классам не равны 0 (точка попала в окно)
	    return (names(which.max(weights))) # Вернуть класс с максимальным весом
	  else
	    return (0) # Иначе - вернуть 0
	}

В результате работы, при *h* [0.1; 2.0] с шагом 0.1, на выборке ирисов Фишера мы получили следующие результаты:

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

#### Сводная таблица для алгоритма парзеновского окна ####

| *Тип ядра*    | *h*  | *LOO* | *LOO, %* |
| ------------- |----- | -----:| -------: |
| Епанечникова  | 0.4  |   6   |    4  	  |
| Квартическое  | 0.4  |   6   |    4     |
| Треугольное   | 0.4  |   6   |    4     |
| Прямоугольное | 0.4  |   6   |    4     |
| Гауссовское   | 0.1  |   6   |    4     |

Достоинства:
1. Простота реализации
2. Хорошее качество классификации при правильно подобранном *h*
3. Не требуется сортировка выборки, в отличии от метода ближайших соседей, что существенно ускоряет время классификации
4. Выбор ядра слабо влияет на качество классификации.

Недостатки:
1. Необходимо хранить выборку целиком
2. Слишком узкие окна *h* приводят к неустойчивой классификации; а слишком широкие - к вырождению алгоритма в константу
3. Если ни один объект выборки не попал в радиус окна *h* вокруг объекта *z*, то алгоритм не способен его проклассифицировать. Для обхода этого недостатка используется окно *h* переменной ширины

[К началу алгоритма (парзеновского окна)](#метод-парзеновского-окна)

[Вернуться к содержанию](#содержание)

### Метод потенциальных функций ###

*Метод потенциальных функций* - метрический классификатор, частный случай метода ближайших соседей. Позволяет с помощью простого алгоритма оценивать вес («важность») объектов обучающей выборки при решении задачи классификации.

#### Описание алгоритма ####

Дана обучающая выборка ![](http://latex.codecogs.com/gif.latex?%24X%5El%24) и объект *z*, который требуется классифицировать

1. Для каждого элемента из выборки задаётся ширина окна *h*
2. Для каждого элемента выборки задаётся сила потенциала *Y*
3. Каждому объекту выборки задаётся вес функции *W*
4. Суммируем веса объектов одинаковых классов. Класс с наибольшим весом присваиваетс объекту *z*.

Метод подбора *Y*

1. Задаём изначально все потенциалы равными 0 и максимальное количество ошибок.
2. Выборки последовательно или случайным образом выьирается объект из выборки *Xi*.
3. Для *Xi* запускается алгоритм классификации. Если полученный в результате класс не соответствует исходному, то сила поенциала данного объекта увеличивается на 1. В ином случае повторяем предыдущие шаги.
4. Алгоритм классификации с полученными значениями потенциалов запускается для каждого объекта выборки. Подсчитываем количество ошибок.
5. Если число ошибок меньше заданного (*eps*), то алгоритм завершает работу. В ином случае повторяем шаги 2-6.

Алгоритм метода потенциальных функций:

	potentialFunction <- function(distances, potentials, h, xl, type_core) {
	  l <- nrow(xl)
	  n <- ncol(xl)
	  classes <- xl[, n]
	  weights <- table(classes) # Таблица для весов классов
	  weights[1:length(weights)] <- 0 # По умолчанию все веса равны нулю
	  for (i in 1:l) { # Для каждого объекта выборки
	    class <- xl[i, n] # Берется его класс
	    r <- distances[i] / h[i]
	    weights[class] <- weights[class] + potentials[i] * type_core(r) # Считается его вес, и прибавляется к общему ввесу его класса
	  }
	  if (max(weights) != 0) return (names(which.max(weights))) # Если есть вес больше нуля, то вернуть класс с наибольшим весом
	  return (0) # Если точка не проклассифицировалась вернуть 0
	}

Алгоритм поиска потенциала:

	getPotentials <- function(xl, h, eps, type_core)
	{
	  l <- nrow(xl) # строки
	  n <- ncol(xl) # столбцы (размерность)
	  potentials <- rep(0, l)
	  distances_to_points <- matrix(0, l, l)
	  err <- eps + 1 # будем считать количество ошибок на выборке 
	  # err = (eps + 1) для предотвращения раннего выхода из цикла
	  # матрица дистанций до других точек выборки
	  for (i in 1:l)
	    distances_to_points[i,] <- getDistances(xl, c(xl[i, 1], xl[i, 2])) 
	  # xl - выборка, xl[i] - текущая точка и её координата
	  while(err > eps){
	    while (TRUE) {
	      # Продолжаем, пока не встретим ошибочное определение к классу
	      cur <- sample(1:l, 1) # выбираем случайную точку из выборки
	      class <- potentialFunction(distances_to_points[cur, ], potentials, h, xl, type_core)

	      if (class != xl[cur, n]) { # если не соответсвует классу
		potentials[cur] = potentials[cur] + 1 # увеличиваем потенциал
		break
	      } #if
	    } #while(true)

	    # считаем количество ошибок
	    err <- 0
	    for (i in 1:l) {
	      class <- potentialFunction(distances_to_points[i, ], potentials, h, xl, type_core)
	      err <- err + (class != xl[i, n])
	    }
	  }
	  return (potentials)
	}

Примеры работы метода потенциальных функций на разных типах ядра:

*Ядро Епанечникова*
![potential_epanech_core.png](https://github.com/VladislavDuma/SMPR/blob/master/img/potential_epanech_core_v2.png)

*Квартическое ядро*
![potential_quarter_core.png](https://github.com/VladislavDuma/SMPR/blob/master/img/potential_quarter_core.png)

Достоинства:
1. Простота реализации
2. Хранит лишь часть выборки, следовательно, экономит память

Недостатки:
1. Порождаемый алгоритм медленно сходится
2. Параметры настраиваются слишком грубо
3. При случайном подборе *Y* время работы может сильно отличаться

[К началу алгоритма (потенциальных функций)](#метод-потенциальных-функций)

[Вернуться к содержанию](#содержание)

## Байесовские алгоритмы классификации ##

### Нормальный дискриминантный анализ ###

Нормальный дискриминантный анализ - это специальный случай байесовского классификации, когда предполагается, что плотности всех классов  являются многомерными нормальными.
Функция плотности многомерного нормального распределения выглядит следующим образом:

![](http://latex.codecogs.com/gif.latex?N%28x%2C%20%5Cmu%2C%20%5CSigma%29%3D%5Cfrac%7B1%7D%7B%5Csqrt%28%282%5Cpi%29%5E%7Bn%7D%7C%5CSigma%7C%29%7De%5E%7B-%5Cfrac%7B1%7D%7B2%7D%28%28x-%5Cmu%29%5CSigma%5E%7B-1%7D%28x-%5Cmu%29%5E%7BT%7D%29%7D), где 

![](http://latex.codecogs.com/gif.latex?x%20%5Cin%20R%5E%7Bn%7D) - объект, состоящий из *n* признаков  
![](http://latex.codecogs.com/gif.latex?%5Cmu%20%5Cin%20R%5E%7Bn%7D) - мат. ожидание каждого признака  
![](http://latex.codecogs.com/gif.latex?%5CSigma%20%5Cin%20R%5E%7Bn%5Ctimes%20n%7D) - матрица ковариации признаков. Симметричная, невырожденная, положительно определённая.



*Примеры работы программы*
![level_lines_1.PNG](https://github.com/VladislavDuma/SMPR/blob/master/img/level_lines/level_lines_1.PNG)

![level_lines_2.PNG](https://github.com/VladislavDuma/SMPR/blob/master/img/level_lines/level_lines_2.PNG)

![level_lines_3.png](https://github.com/VladislavDuma/SMPR/blob/master/img/level_lines/level_lines_3.PNG)

![level_lines_4.png](https://github.com/VladislavDuma/SMPR/blob/master/img/level_lines/level_lines_4.PNG)

Работу программы с реализацией на Shiny (линии уровня) можно посмотреть здесь [здесь](https://vladislav-duma.shinyapps.io/level_lines/)

[Вернуться к содержанию](#содержание)
