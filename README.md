# Лаб3
## Задание: 
Необходимо реализовать простейшую систему классификации изображений 
на основе сверточных нейронных сетей.
1. Реализовать систему классификации согласно описанию, используя не 
менее трех различных архитектур нейронной сети.
2. Сравнить качество работы, скорость и количество потребляемой памяти 
для каждой архитектуры.
Система должна загружать изображение с диска, преобразовывать в нужный 
для обработки моделью нейронной сети формат (тензор), выполнять 
предобработку, если требуется (например, изменение размера и 
нормирование), затем обрабатывать с помощью нейронной сети и выводить 
результат (номер класса, название, его вероятность для первых 5 наиболее 
вероятных классов).
Каждую модель необходимо протестировать на собственном наборе 
из 50 изображений, которые не использовались для обучения, в качестве метрик 
использовать top-1 accuracy и top-5 accuracy.

Для данной работы были взяты следующие модели: ResNet50, VGG19, AlexNet.

## Теория: 
Все из представленных сетей являются сверточными. Главное отличие в архитектуре и количестве параметров нейросетей.
AlexNet и VGG имеют похожую архитектуру и по факту отличаются только количеством слоев свертки и пулинга.

В AlexNet 11 слоев и 62 млн параметров, в VGG19 - 19 слоев и 143 млн параметров.

<img src="pic1_alex_vgg19.png" alt="drawing" width="400"/>

Рисунок 1 – Архитектуры AlexNet, VGG16, VGG19


ResNet50 уже имеет небольшие отличия от первых двух архитектур: появились блоки Residual blocks, которые используют промежуточные соединения, и часть информации проходит через блоки, минуя слои свертки, где затем складывается с информацией, прошедшей через них. Это помогает при обучении, чтобы градиенты не затухали, проходя большое число слоев.

<img src="pic1_resnet50.png" alt="drawing" width="400"/>

Рисунок 2 – Архитектура ResNet50

В ResNet50 50 слоев и 25млн параметров.

## Реализация: 
Программа считывает изображения, в названии которых хранится правильная метка класса (одного из классов датасета IMAGENET1K_V1). Далее происходит предобработка изображения: изменение размера до (256,256), затем обрезка по центру до (224,224), далее нормализация для каждого канала со средними mean=[0.485, 0.456, 0.406] и дисперсией std=[0.229, 0.224, 0.225]. После этого происходит подача на вход cnn на выходе которой после применения softmax получаем вероятности принадлежноости к классам, считаем top 1 и top 5 accuracy. Также произведено измерение количества потребляемой памяти gpu и скорость работы на всем датасете из 50 картинок.

## Результаты работы программы:
![картинка 3](pic1_l3.png "Рисунок 3 – Все картинки, поданные на вход, 8 классов, 50 изображений")

Рисунок 3 – Все картинки, поданные на вход, 8 классов, 50 изображений

Сравнение по точности, времени выполнения, памяти:

|         |Top 5 accuracy | Top 1 accuracy | time | GPU usage |
:---------|---------------|----------------|------|-----------|
|ResNet50 |   0.837       |0.51           | 2.66 s|    0.2 GB|
|VGG19     |0.857 | 0.571 | 2.92 s | 0.6 GB |
|AlexNet   | 0.755| 0.49 | 2.22 s |  0.2 GB | 



## Выводы:
В результате работы было реализована система классификации изображений тремя вариантами сверточных нейросетей: ResNet50, VGG19, AlexNet. Измерено время выполнения и количество затраченной памяти gpu.
Как можно видеть наиболее требовательной к ресурсам является VGG19, это связанго с тем, что в ней больше всего слоев и параметров. При этом она показывает наибольшую точность как по Top 5 accuracy, так и по Top 1 accuracy. ResNet50 не сильно отстает по точности, но при этом занимает намного меньше памяти gpu. AlexNet достаточно сильно отстает по точности, но при этом занимает памяти столько же, сколько ResNet50; при этом время выполнения относительно других сетей меньше на небольшую величину. Поэтому, если важна наибольшая точность и есть возможность использовать больше памяти gpu, то лучше использовать VGG19; если же есть ограничение по ресурсам, то стоит взять ResNet50.      





# Лаб2
## Задание: 
Реализовать программу согласно описанию. Можно использовать языки 
C++ или Python и любые библиотеки, при этом необходимо чтобы вся
задача не решалась только с помощью одной встроенной функции 
(например, lib.detect_template(image, template).
Сравнить качество работы двух вариантов реализации по точности 
детектирования.

Необходимо реализовать два примитивных детектора объектов на 
изображении, работающих с помощью поиска эталона на входном 
изображении. 
1. Прямой поиск одного изображения на другом (template matching)
2. Поиск ключевых точек эталона на входном изображении (например, с 
помощью SIFT, ORB..)


## Теория: 
Функция match_template проверяет для каждого пикселя исходного изображения насколько окрестности этого пикселя совпадают с шаблоном.

ORB представляет собой сочетание детектора ключевых точек FAST и дескриптора BRIEF со многими модификациями для повышения производительности. Сначала он использует FAST, чтобы найти ключевые точки, а затем применяет угловую меру Харриса, чтобы найти топ N точек среди них. 

Как работает FAST: 
1. Выберает пиксель p на изображении, который должен быть идентифицирован как точка интереса. Пусть его интенсивность будет Ip.
2. Выберает соответствующее пороговое значение t.
3. Рассматривает круг из 16 пикселей вокруг тестируемого пикселя. Важно, что круг состоит не из 16 пикселей, а его "длина" равна 16.
4. Теперь пиксель p является углом, если существует множество n смежных пикселей в круге (из 16 пикселей), которые ярче, чем Ip+t, или все темнее, чем Ip-t. Обычно n=12.

Как работает BRIEF:
Он берет патч сглаженного изображения и выбирает набор nd (x,y) пар местоположений уникальным образом. Затем для этих пар местоположений выполняются некоторые сравнения интенсивности пикселей. Например, пусть первые пары местоположений будут p и q. Если I(р)<I(q), то его результат равен 1, иначе он равен 0. Это применяется ко всем nd парам местоположений, чтобы получить nd-размерную битовую строку.


Для дескрипторов ORB использует дескрипторы BRIEF. Но BRIEF плохо работает с вращением. Итак, что делает ORB, так это «направляет» BRIEF в соответствии с ориентацией ключевых точек. Для любого набора n бинарных тестов на месте (xi,уi) определяется матрица C(2×n), которая содержит координаты этих пикселей. Затем, используя ориентацию патча,θ, находится его матрица вращения и поворачивает С на θ, получая повернутую Сθ.
ORB дискретизирует угол с шагом 2π/30 (12 градусов) и строит справочную таблицу предварительно вычисленных шаблонов BRIEF. Пока ориентация ключевой точки θ согласуется между представлениями, правильный набор точек Сθ будет использоваться для вычисления его дескриптора.


## Реализация: 
Программа считывает оригинальное изображение и изображения "template", которые мы будем искать на исходном изображении. 
Реализация с функцией match_template принимает на вход два изображения и отрисовывает прямоугольник на исходном изображении в том месте, где произошло наибольшее совпадение с изображением template.

Реализация с ORB, после запуска методов ORB возвращает массив дескрипторов ключевых точек для изображения. Это делается как для исходного, так и для template изображения. Затем с помощью функции BF.Matcher.match, которая принимает на вход массивы дескрипторов обоих изображений, получаем список из структур, содержащих разности между дескрипторами, и берем n (задаем сами) дескрипторов, находящихся ближе всего. Это и есть точки на исходном и template изображении, которые "похожи" друг надруга сильнее всего. 

## Результаты работы программы:
Возьмем три набора изображений: 
- Бурдж-Халифа, одно из 5 template изоражений является вырезанным с оригинальной картинки, остальные - фотографии под другим углом объектов, которые есть на фото
- Стол с ноутбуком и прочими вещами, 2 изображения вырезаны, остальные под другим углом
- Тетрадки, мышка и наклейка, 1 изображение вырезано, остальные под другим углом
### С помощью функции match_template
![картинка 1](pic1_l2.png "Рисунок 1 – Бурдж-Халифа match_template")

Рисунок 1 – Бурдж-Халифа match_template

Как можно видеть для двух изображений границы определились неправильно. Это изображения башни под другим углом. Остальные изображения снятые под другим углом нашлись верно.
                            
                            
![картинка 2](pic2_l2.png "Рисунок 2 – Стол match_template")

Рисунок 2 – Стол match_template

Для одного из изображений границы определились неправильно. Для одного не определились вообще. Оба из них снятые под другим углом объекты.
                            
![картинка 3](pic3_l2.png "Рисунок 3 – Тетрадки match_template")

Рисунок 3 – Тетрадки match_template                            

Только для одного изображения определились границы правильно, это изображение вырезано из оригинала. Для всех снятынх под другим углом объектов границы не определились вообще.

### С помощью функции ORB
![картинка 4](pic4_l2.png "Рисунок 4 – Бурдж-Халифа ORB")

Рисунок 4 – Бурдж-Халифа ORB

Как можно видеть в целом все границы определились праавильно, но есть небольише неточности для объектов снятых под другим углом.
                            
                            
![картинка 5](pic5_l2.png "Рисунок 5 – Стол ORB")

Рисунок 5 – Стол ORB

На обоих изображениях с кружкой границы нашлись неправильно, для остальных - достаточно точно.
                            
![картинка 6](pic6_l2.png "Рисунок 6 – Тетрадки ORB")

Рисунок 6 – Тетрадки ORB                           

Для двух изображений границы определились верно, еще для двух - шире, чем нужно. Для одного определены неправильно.

## Выводы: 
В результате работы были реализованы два варианта нахождения части изображения на исходном: с помощью функции match_template и с помощью ORB. Как показали результаты работы программы, видно, что вариант с ORB работает достаточно лучше, он реже грубо ошибается, поэтому намного лучше подходит для варинтов, когда изображение, которое необходимо найти на исходном не выреазно оттуда, а сфотографировано под другим углом.

|Количество верно найденных изображений|match_template|ORB|
:------------|-----------------------|-----------------|
|Вырезаны из исходного| 4/4 | 3/4 
|Сфотографированы с другого ракурса| 3/11 | 6-7/11



# Лаб1
## Задание: 
1. Реализовать программу согласно варианту задания. Базовый алгоритм, используемый в программе, необходимо реализовать в 3 вариантах: с использованием встроенных функций какой-либо библиотеки (OpenCV, PIL и др.) и нативно на Python + |с использованием Numba или C++|. 
2. Сравнить быстродействие реализованных вариантов.
Примечание. Программа работает с видео. На вход должен поступать видеопоток с устройства (камеры) или видео должно читаться из файла. Каждый алгоритм нужно реализовать в 3 вариантах: с использованием сторонних библиотек на Python, с помощью примитивных операций и циклов на Python (можно использовать NumPy массивы) и с помощью компилируемого кода (на C++ либо с использованием, например, Numba на Python). Если указано, что выходное изображение переключается между черно-белым и после обработки, это значит, что на вход обработки подается черно-белое изображение.
## Вариант задания:
Бинаризация с адаптивным порогом. На вход поступает изображение, программа отрисовывает окно, в которое выводится либо исходное изображение после преобразования в черно-белое, либо после бинаризации (переключение по нажатию клавиши).
## Теория: 
Суть бинаризации заключается в определении порога яркости оттенков серого для картинки. Если значение яркости пикселя выше порогового, то пиксель заменяется на черный (255), ниже – на белый (0). 
Существует бинаризация с простым порогом и адаптивным. В данной лабораторной необходимо реализовать бинаризацию с адаптивным порогом методом среднего. Для этого все изображение разбивается на блоки размером blocksize* blocksize, для которого считается среднее значение яркости пикселя, затем из него вычитается константа, единая для каждого из блоков. Также можно домножать получившееся среднее для каждого блока на общую вторую константу const2.

## Реализация: 
Программа считывает видео из файла с помощью функции OpenCV. Затем идет цикл по кадрам, в котором для каждого кадра происходит переход в оттенки серого, а затем происходит бинаризация изображения. Далее этот кадр выводится на экран, а также сохраняется с помощью функции VideoWriter, которая принимает на вход кадры и сохраняет их в видео.

## Результаты работы программы:
![картинка 1](pic1.png "Рисунок 1 – Исходное изображение")

Рисунок 1 – Исходное изображение

![картинка 2](pic2.png "Рисунок 2 – Изображение обработанное с OpenCV")

Рисунок 2 –Изображение обработанное с OpenCV

![картинка 3](pic3.png "Рисунок 3 – Изображение обработанное с помощью своей функции")

Рисунок 3 –Изображение обработанное с помощью своей функции

![картинка 4](pic4.png "Рисунок 4 – Исходное изображение 2")

Рисунок 4 – Исходное изображение 2

![картинка 5](pic5.png "Рисунок 5 – Изображение 2 обработанное с OpenCV")

Рисунок 5 – Изображение 2 обработанное с OpenCV

![картинка 6](pic6.png "Рисунок 6 – Изображение 2 обработанное с помощью своей функции")

Рисунок 6 –Изображение 2 обработанное с помощью своей функции

Теперь попробуем вычесть обработанные изображения разными методами друг из друга. 
Изображения при обработке с циклами и с циклами, но скомпилированном кодом совпадают полностью.
Разница между методами с циклами с функцией OpenCV - не все пиксели совпали, а только 75%.

## Выводы:
В результате работы была реализована бинаризация видео 3 вариантами: с использованием сторонних библиотек на Python, с помощью примитивных операций и циклов на Python и с помощью компилируемого кода с использованием Numba.
Время выполнения бинаризации одного и того же видео:

|            |С использованием OpenCV|Нативно на python|Компилируемый код|
:------------|-----------------------|-----------------|-----------------|
|Время работы| 7.7 сек | 1440 сек | 9.65 сек 


Скомпилированный код работает немного медленнее, чем функция OpenCV, это связанно с тем, что в функции грамотно распараллелены вычисления.
По картинкам видно, что они немного отличаются друг от друга.
При этом обработанные картинки обоими вариантами выглядят «адекватно», но при использовании готового решения не так заметны квадратные области, для каждой из которых рассчитывается свой порог. Это, скорее всего, связанно с тем, что в функции OpenCV происходит какая-то последующая обработка изображения.
