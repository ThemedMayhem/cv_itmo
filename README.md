# Задание: 
1. Реализовать программу согласно варианту задания. Базовый алгоритм, используемый в программе, необходимо реализовать в 3 вариантах: с использованием встроенных функций какой-либо библиотеки (OpenCV, PIL и др.) и нативно на Python + |с использованием Numba или C++|. 
2. Сравнить быстродействие реализованных вариантов.
Примечание. Программа работает с видео. На вход должен поступать видеопоток с устройства (камеры) или видео должно читаться из файла. Каждый алгоритм нужно реализовать в 3 вариантах: с использованием сторонних библиотек на Python, с помощью примитивных операций и циклов на Python (можно использовать NumPy массивы) и с помощью компилируемого кода (на C++ либо с использованием, например, Numba на Python). Если указано, что выходное изображение переключается между черно-белым и после обработки, это значит, что на вход обработки подается черно-белое изображение.
# Вариант задания:
Бинаризация с адаптивным порогом. На вход поступает изображение, программа отрисовывает окно, в которое выводится либо исходное изображение после преобразования в черно-белое, либо после бинаризации (переключение по нажатию клавиши).
# Теория: 
Суть бинаризации заключается в определении порога яркости оттенков серого для картинки. Если значение яркости пикселя выше порогового, то пиксель заменяется на черный (255), ниже – на белый (0). 
Существует бинаризация с простым порогом и адаптивным. В данной лабораторной необходимо реализовать бинаризацию с адаптивным порогом методом среднего. Для этого все изображение разбивается на блоки размером blocksize* blocksize, для которого считается среднее значение яркости пикселя, затем из него вычитается константа, единая для каждого из блоков. Также можно домножать получившееся среднее для каждого блока на общую вторую константу const2.

# Реализация: 
Программа считывает видео из файла с помощью функции OpenCV. Затем идет цикл по кадрам, в котором для каждого кадра происходит переход в оттенки серого, а затем происходит бинаризация изображения. Далее этот кадр выводится на экран, а также сохраняется с помощью функции VideoWriter, которая принимает на вход кадры и сохраняет их в видео.

# Результаты работы программы:
![картинка 1](pic1.png "Рисунок 1 – Исходное изображение")

Рисунок 1 – Исходное изображение

![картинка 2](pic2.png "Рисунок 2 –Изображение обработанное с OpenCV")

Рисунок 2 –Изображение обработанное с OpenCV

![картинка 3](pic3.png "Рисунок 3 –Изображение обработанное с помощью своей функции")

Рисунок 3 –Изображение обработанное с помощью своей функции

![картинка 4](pic4.png "Рисунок 4 – Исходное изображение 2")

Рисунок 4 – Исходное изображение 2

![картинка 5](pic5.png "Рисунок 5 – Изображение 2 обработанное с OpenCV")

Рисунок 5 – Изображение 2 обработанное с OpenCV

![картинка 6](pic6.png "Рисунок 6 –Изображение 2 обработанное с помощью своей функции")

Рисунок 6 –Изображение 2 обработанное с помощью своей функции

Теперь попробуем вычесть обработанные изображения разными методами друг из друга. 
Изображения при обработке с циклами и с циклами, но скомпилированном кодом совпадают полностью.
Разница между методами с циклами с функцией OpenCV - не все пиксели совпали, а только 75%.

# Выводы:
В результате работы была реализована бинаризация видео 3 вариантами: с использованием сторонних библиотек на Python, с помощью примитивных операций и циклов на Python и с помощью компилируемого кода с использованием Numba.
Время выполнения бинаризации одного и того же видео:

|            |С использованием OpenCV|Нативно на python|Компилируемый код|
:------------|-----------------------|-----------------|-----------------|
|Время работы| 7.7 сек | 1440 сек | 9.65 сек 


Скомпилированный код работает немного медленнее, чем функция OpenCV, это связанно с тем, что в функции грамотно распараллелены вычисления.
По картинкам видно, что они немного отличаются друг от друга.
При этом обработанные картинки обоими вариантами выглядят «адекватно», но при использовании готового решения не так заметны квадратные области, для каждой из которых рассчитывается свой порог. Это, скорее всего, связанно с тем, что в функции OpenCV происходит какая-то последующая обработка изображения.
