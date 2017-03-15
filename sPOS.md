# sPOS - привязка к координатам

    Purpose:             0-19 bytes (character string)
    Null separator:      1 byte
    pictX:           1 or more bytes (ASCII floating-point)
    Null separator:  1 byte
    pictY:           1 or more bytes (ASCII floating-point)
    Null separator:  1 byte
    Compression flag:   1 byte
    Compression method: 1 byte
    CRS_Text:           1 or more bytes(UTF-8 or compressed UTF8)
    Number of coords:   1 byte(unsigned int)
    1st coord:      1 or more bytes (ASCII floating-point)
    Null separator: 1 byte
    ...
    N'th coord:     1 or more bytes (ASCII floating-point)
    Null separator: 1 byte 
    X axis link:       1 byte(signed int)
    Y axis link:       1 byte(signed int)
    value axis link:   1 byte(signed int)
    1st rotate coord(pitch):  1 or more bytes (ASCII floating-point)
    Null separator:           1 byte
    2nd rotate coord(yaw):    1 or more bytes (ASCII floating-point)
    Null separator:           1 byte
    3rd rotate coord(roll):   1 or more bytes (ASCII floating-point)
    Null separator:           1 byte
    Local pixel width:  1 or more bytes (ASCII floating-point)
    Null separator:     1 byte
    Local pixel height: 1 or more bytes (ASCII floating-point)
   
   
## Purpose - характеристика привязки (точки).
Возможные значения:
* center - центр изображения
* position - положение камеры
* subject, subject_N - положение объекта, расположенного на снимке
* Также допустимо пустое значение.
   
## pictX; pictY - координаты точки на изображении.
Могут описывать координаты с субпиксельной точностью. Целому значению соответствует центр соответствующего пикселя, нумерация идёт с 0. Записываются в соответствии с п. 1.2 (Floating point) ftp://ftp.simplesystems.org/pub/png/documents/pngextensions.html  (текстовая запись десятичного числа с плавающей точкой), отделяясь друг от друга и от последующих даных нулевым байтом - разделителем.

Если значение поля purpose ненулевое - могут быть представлены пустыми строками. Для purpose "center" это обозначает, что pictX должен быть принят как ширина изображения, делённая на 2, а pictY, соответственно, как высота делённая на 2; для остальных значений - что сопоставления между координатами не производится.

## Описание внешней системы координат и положения в ней
Внешняя система координат описывается с использованием стандарта ISO 19162:2015(E), сохраня текстовое представление в формате, аналогичном записи чанка iTXt:
+ Compression flag - признак сжатой строки
+ Compression method - формат сжатия данных в соотвтетствии https://www.w3.org/TR/PNG/#10Compression
+ CRS_Text - Описание системы координат с использованием Well known text representation of coordinate reference systems ( http://docs.opengeospatial.org/is/12-063r5/12-063r5.html )

Дополнительно к WKT text указывается число координат. Оно может использоваться для проверки правильности парсинга WKT, а также для считывания последующих координат.

Координаты во внешней системе координат записываются аналогично координатам точки на изображении (ASCII floating-point with null separator) с единственной разницей, что их не фиксированно 2, а Number of coords.

## Локальная система координат
Точка привязки может задавать не только соответсвие точки на изображении точке во внешней системе координат, но и увязывать систему координат изображения с внешней системой координат. Такая связь может распространяться на всё изображение(если она единственная) или только на окрестности точки привязки.

Для связи требуется задать значения "axis link". Если соответствующая константа отрицательная - это обозначает, что направление оси при сопоставлении инвертируется.
* Если константа равна 0 - ось изображения не сопоставлена оси во внешней системе координат.
* Ось X изображения - номер пикселя в строке, нумеруется слева направо.
* Ось Y изображения - номер строки в изображении, нумеруется сверху вниз.
* Ось значений изображения определима только для изображений с заданным чанком pCAL (соответвие данных физическому параметру). Если чанк pCAL отсутствует - сопоставление не производится, значение "value axis link" должно приниматься за 0 вне зависимости от настоящего значения. Направление оси значений условно принимается "от наблюдателя", то есть соответствует правилу правой руки относительно осей X и Y.
   
Положение системы координат изображения (с учётом инверсии) относительно внешней системы координат описывается углами Тэйта-Брайана в порядке Z-Y'-X'', при этом оси смещаются от начала координат к точке привязки. Оси системы координат изображения (с учётом инверсии) считаются полученными из соответствующих осей внешней системы координат следующим образом:
* Первый поворот (рыскание) выполняется относительно оси Z(оси, соответствующей привязке значений или нормали к двум другим осям по правилу правой руки, если привязка остутствует).
* Второй поворот (тангаж) выполняется относительно оси Y после поворота (оси Y').
* Третий поворот (крен) выполняется относительно оси X после двух предыдущих поворотов (оси X'')

Если понятие угла не может быть определено - сохраняется пустая строка(нулевой символ, обозначающий конец строки, без самой строки). При выполнении преобразования неопределённые углы могут приниматься равными 0, но неопределённое состояние указывает что фактическое измерение не производилось(в отличие от случая, если угол действительно равен 0 с доступной для измерения точностью). Аналогично могут быть не определены углы в некоторых вариациях смешанной системы координат.

## Локальный масштаб
В каждой конкретной точке может быть определён масштаб для осей X и Y (масштаб для оси значений описывается в pCAL).
Значение описывает ширину и высоту пикселя в величинах сопоставленной оси координат.
Описание координат поворота и связи осей без задания масштаба и координат точки(для назначения "position") может использоваться для описания ориентации камеры в пространстве.
Значения, также как и поворот, могут быть описаны пустыми строками, что обозначает либо неприменимость масштаба(локальная система координат не задана), либо соответствие локального масштаба глобальному, заданному при помощи sCAL.

## Правила пересчета
Пересчёт координат пикселя в значения внешней системы координат производится следующим образом.
### Перевод пикселей в значения

<img src="https://latex.codecogs.com/gif.latex?\begin{bmatrix}&space;A_{0}&space;\\&space;B_{0}&space;\\&space;C_{0}&space;\end{bmatrix}=\begin{bmatrix}&space;X&space;*&space;pixelWidth&space;\\&space;Y&space;*&space;pixelHeight&space;\\&space;pCAL(P)&space;\end{bmatrix}" title="\begin{bmatrix} A_{0} \\ B_{0} \\ C_{0} \end{bmatrix}=\begin{bmatrix} X * pixelWidth \\ Y * pixelHeight \\ pCAL(P) \end{bmatrix}" />

### Поворот

<img src="https://latex.codecogs.com/gif.latex?\begin{bmatrix}&space;A_{1}&space;&&space;B_{1}&space;&&space;C_{1}&space;\end{bmatrix}&space;=\\&space;{\begin{bmatrix}&space;c_{1}c_{2}&c_{2}s_{1}&-s_{2}\\&space;c_{1}s_{2}s_{3}-c_{3}s_{1}&c_{1}c_{3}&plus;s_{1}s_{2}s_{3}&c_{2}s_{3}\\&space;s_{1}s_{3}&plus;c_{1}c_{3}s_{2}&c_{3}s_{1}s_{2}-c_{1}s_{3}&c_{2}c_{3}&space;\end{bmatrix}}&space;\bullet&space;\begin{bmatrix}&space;A_{0}&space;\\&space;B_{0}&space;\\&space;C_{0}&space;\end{bmatrix}&space;=\\&space;{\begin{bmatrix}&space;c_{1}c_{2}&c_{2}s_{1}&-s_{2}\\&space;c_{1}s_{2}s_{3}-c_{3}s_{1}&c_{1}c_{3}&plus;s_{1}s_{2}s_{3}&c_{2}s_{3}\\&space;s_{1}s_{3}&plus;c_{1}c_{3}s_{2}&c_{3}s_{1}s_{2}-c_{1}s_{3}&c_{2}c_{3}&space;\end{bmatrix}}&space;\bullet&space;\begin{bmatrix}&space;X&space;*&space;pixelWidth&space;\\&space;Y&space;*&space;pixelHeight&space;\\&space;pCAL(P)&space;\end{bmatrix}" title="\begin{bmatrix} A_{1} & B_{1} & C_{1} \end{bmatrix} =\\ {\begin{bmatrix} c_{1}c_{2}&c_{2}s_{1}&-s_{2}\\ c_{1}s_{2}s_{3}-c_{3}s_{1}&c_{1}c_{3}+s_{1}s_{2}s_{3}&c_{2}s_{3}\\ s_{1}s_{3}+c_{1}c_{3}s_{2}&c_{3}s_{1}s_{2}-c_{1}s_{3}&c_{2}c_{3} \end{bmatrix}} \bullet \begin{bmatrix} A_{0} \\ B_{0} \\ C_{0} \end{bmatrix} =\\ {\begin{bmatrix} c_{1}c_{2}&c_{2}s_{1}&-s_{2}\\ c_{1}s_{2}s_{3}-c_{3}s_{1}&c_{1}c_{3}+s_{1}s_{2}s_{3}&c_{2}s_{3}\\ s_{1}s_{3}+c_{1}c_{3}s_{2}&c_{3}s_{1}s_{2}-c_{1}s_{3}&c_{2}c_{3} \end{bmatrix}} \bullet \begin{bmatrix} X * pixelWidth \\ Y * pixelHeight \\ pCAL(P) \end{bmatrix}" />

### Инверсия осей (если надо)
<img src="https://latex.codecogs.com/gif.latex?\begin{bmatrix}&space;A_{2}&space;&&space;B_{2}&space;&&space;C_{2}&space;\end{bmatrix}&space;=\\&space;\begin{bmatrix}&space;sgn(X)&space;&&space;0&space;&&space;0&space;\\&space;0&space;&&space;sgn(Y)&space;&&space;0\\&space;0&space;&&space;0&space;&&space;sgn(Z)&space;\end{bmatrix}&space;\bullet&space;\begin{bmatrix}&space;A_{1}&space;\\&space;B_{1}&space;\\&space;C_{1}&space;\end{bmatrix}&space;=\\&space;{\begin{bmatrix}&space;sgn(X)c_{1}c_{2}&sgn(X)c_{2}s_{1}&-sgn(X)s_{2}\\&space;sgn(Y)c_{1}s_{2}s_{3}-c_{3}s_{1}&sgn(Y)c_{1}c_{3}&plus;s_{1}s_{2}s_{3}&sgn(Y)c_{2}s_{3}\\&space;sgn(Z)s_{1}s_{3}&plus;c_{1}c_{3}s_{2}&sgn(Z)c_{3}s_{1}s_{2}-c_{1}s_{3}&sgn(Z)c_{2}c_{3}&space;\end{bmatrix}}&space;\bullet&space;\begin{bmatrix}&space;X&space;*&space;pixelWidth&space;\\&space;Y&space;*&space;pixelHeight&space;\\&space;pCAL(P)&space;\end{bmatrix}" title="\begin{bmatrix} A_{2} & B_{2} & C_{2} \end{bmatrix} =\\ \begin{bmatrix} sgn(X) & 0 & 0 \\ 0 & sgn(Y) & 0\\ 0 & 0 & sgn(Z) \end{bmatrix} \bullet \begin{bmatrix} A_{1} \\ B_{1} \\ C_{1} \end{bmatrix} =\\ {\begin{bmatrix} sgn(X)c_{1}c_{2}&sgn(X)c_{2}s_{1}&-sgn(X)s_{2}\\ sgn(Y)c_{1}s_{2}s_{3}-c_{3}s_{1}&sgn(Y)c_{1}c_{3}+s_{1}s_{2}s_{3}&sgn(Y)c_{2}s_{3}\\ sgn(Z)s_{1}s_{3}+c_{1}c_{3}s_{2}&sgn(Z)c_{3}s_{1}s_{2}-c_{1}s_{3}&sgn(Z)c_{2}c_{3} \end{bmatrix}} \bullet \begin{bmatrix} X * pixelWidth \\ Y * pixelHeight \\ pCAL(P) \end{bmatrix}" />

### Учет положения точки
<img src="https://latex.codecogs.com/gif.latex?\begin{bmatrix}&space;A&space;&&space;B&space;&&space;C&space;\end{bmatrix}&space;=\\&space;{\begin{bmatrix}&space;sgn(X)c_{1}c_{2}&sgn(X)c_{2}s_{1}&-sgn(X)s_{2}\\&space;sgn(Y)c_{1}s_{2}s_{3}-c_{3}s_{1}&sgn(Y)c_{1}c_{3}&plus;s_{1}s_{2}s_{3}&sgn(Y)c_{2}s_{3}\\&space;sgn(Z)s_{1}s_{3}&plus;c_{1}c_{3}s_{2}&sgn(Z)c_{3}s_{1}s_{2}-c_{1}s_{3}&sgn(Z)c_{2}c_{3}&space;\end{bmatrix}}&space;\bullet&space;\begin{bmatrix}&space;(Col&space;-&space;pictX)&space;*&space;pixelWidth&space;\\&space;(Row&space;-&space;pictY)&space;*&space;pixelHeight&space;\\&space;pCAL(P)&space;\end{bmatrix}&space;&plus;&space;\begin{bmatrix}&space;posX&space;&&space;posY&space;&&space;posZ&space;\end{bmatrix}" title="\begin{bmatrix} A & B & C \end{bmatrix} =\\ {\begin{bmatrix} sgn(X)c_{1}c_{2}&sgn(X)c_{2}s_{1}&-sgn(X)s_{2}\\ sgn(Y)c_{1}s_{2}s_{3}-c_{3}s_{1}&sgn(Y)c_{1}c_{3}+s_{1}s_{2}s_{3}&sgn(Y)c_{2}s_{3}\\ sgn(Z)s_{1}s_{3}+c_{1}c_{3}s_{2}&sgn(Z)c_{3}s_{1}s_{2}-c_{1}s_{3}&sgn(Z)c_{2}c_{3} \end{bmatrix}} \bullet \begin{bmatrix} (Col - pictX) * pixelWidth \\ (Row - pictY) * pixelHeight \\ pCAL(P) \end{bmatrix} + \begin{bmatrix} posX & posY & posZ \end{bmatrix}" />
