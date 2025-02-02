# Learn OpenGL. Урок 4.8 — Продвинутый GLSL

## Продвинутый GLSL

Этот урок не продемонстрирует вам новые продвинутые средства, значительно улучшающие визуальное качество сцены. В этом уроке мы пройдем по более или менее интересным аспектам GLSL и затронем несколько неплохих приёмов, которые могут помочь вам в ваших стремлениях. В основном, *знания* и *средства*, *делающие вашу жизнь проще* при создании OpenGL приложений в сочетании с GLSL.

Мы обсудим некоторые интересные **встроенные переменные**, новые подходы в организации ввода-вывода шейдеров и очень полезный инструмент — **объект юниформ-буфера**.

## Встроенные переменные GLSL

Шейдеры самодостаточны, если нам нужны данные из какого-либо другого источника, нам придется передать их в шейдер. Мы изучили, как сделать это с помощью вершинных атрибутов, юниформ и сэмплеров. Однако существуют еще несколько переменных определенных в GLSL с префиксом *gl_*, что дает нам дополнительные возможности читать и/или писать данные. Мы уже видели двух представителей, результирующие вектора: *gl_Position* вершинного шейдера и *gl_FragCoord* фрагментного шейдера.

Мы обсудим несколько интересных встроенных в GLSL переменных как ввода, так и вывода и объясним какая от них польза. Замечу, что мы не будем обсуждать все встроенные переменные в GLSL, поэтому если вы хотите посмотреть все встроенные переменные — вы можете сделать это на соответствующей [странице](https://www.khronos.org/opengl/wiki/Built-in_Variable_(GLSL)) OpenGL.

## Переменные вершинного шейдера

Мы уже работали с переменной *gl_Position*, являющейся выходным вектором вершинного шейдера, задающим вектор положения в пространстве отсечения. Установка значения *gl_Position* является необходимым условием для вывода чего-либо на экран. Ничего нового для нас.

### gl_PointSize

Один из обрабатываемых примитивов, который мы можем выбрать — *GL_POINTS*. В этом случае каждая вершина является примитивом и обрабатывается как точка. Так же можно установить размер обрабатываемых точек с помощью функции *glPointSize*. Но мы так же можем менять это значение через шейдер.

В выходной *float* переменной *gl_PointSize*, объявленной в GLSL, можно установить высоту и ширину точек в пикселях. Описывая размер точки в вершинном шейдере, вы можете влиять на это значение для каждой вершины.

По умолчанию изменение размеров точки в вершинном шейдере отключено, но если хотите, вы можете взвести OpenGL флаг *GL_PROGRAM_POINT_SIZE*:

```cpp
glEnable(GL_PROGRAM_POINT_SIZE);
```

Простой пример изменения размеров точки – установка размера точки равным значению компоненты z пространства отсечения, которое, в свою очередь, равно расстоянию от вершины до наблюдателя. Размер точки будет увеличиваться, чем дальше мы находимся от вершины.

```glsl
void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);    
    gl_PointSize = gl_Position.z;    
}  
```

В результате, чем мы дальше от точек, тем крупнее они отображаются.

![](1.png)

Изменение размеров точки для каждой вершины будет полезно для таких приёмов как генерация частиц.

### gl_VertexID

Переменные *gl_Position* и *gl_PointSize* являются *выходными* переменными, поскольку их значения считываются как выход стадии фрагментного шейдера, а путем записи мы можем влиять на них. Вершинный шейдер также предоставляет *входную* переменную *gl_VertexID*, из которой возможно только чтение.

Целочисленная переменная *gl_VertexID* содержит идентификатор отрисовываемой вершины. Во время выполнения индексного рендера \(с помощью *glDrawElements*\) эта переменная содержит текущий индекс отрисовываемой вершины. Во время отрисовки без индексов \(через *glDrawArrays*\) переменная содержит число обработанных вершин на данный момент с момента вызова рендера.

Хоть это и не очень нужно сейчас, знать о наличии такой переменной полезно.

## Переменные фрагментного шейдера

Внутри фрагментного шейдера мы также имеем доступ к некоторым любопытным переменным. GLSL предоставляет нам две входные переменные *gl_FragCoord* и *gl_FrontFacing*.

### gl_FragCoord

Мы уже видели *gl_FragCoord* пару раз, пока обсуждали проверку глубины, т.к. z компонента вектора *gl_FragCoord* равна значению глубины конкретного фрагмента. Однако мы можем также использовать x и y компоненты для некоторых эффектов.

Компоненты x и y переменной *gl_FragCoord* являются координатами фрагмента в системе координат окна, берущими начало из левого нижнего угла окна. Мы указали размеры окна 800х600 с помощью *glViewport*, поэтому координаты фрагмента в системе координат окна будут между 0-800 по x и диапазоне 0-600 по y.

Используя фрагментный шейдер, мы можем вычислять различные цветовые значения, основанные на экранных координатах фрагмента. Обычное использование переменной *gl_FragCoord* – сравнивать видимый результат вычислений различных фрагментов, как обычно делают в технических демо-версиях. Например, мы можем разделить экран на две части, рендеря часть на левой половине экрана, а другую часть — на правой половине экрана. Ниже приведен пример фрагментного шейдера, который на основе экранных координат выводит различные цвета.

```glsl
void main()
{             
    if(gl_FragCoord.x < 400)
        FragColor = vec4(1.0, 0.0, 0.0, 1.0);
    else
        FragColor = vec4(0.0, 1.0, 0.0, 1.0);        
}  
```

Т.к. ширина окна равна 800, в случае установки координат пикселя меньше чем 400, он должен быть на левой части экрана, и это дает нам объект другого цвета.

![](2.png)

Теперь мы можем вычислить два совершенно разных результата во фрагментом шейдере и отобразить каждый на своей части экрана. Это отлично подходит для тестирования различных световых механик.

### gl_FrontFacing

Еще одна любопытная переменная во фрагментном шейдере – *gl_FrontFacing*. В уроке [отсечение граней](../../part%204/chapter%204/text.md) — мы упоминали, что OpenGL может определить, является ли грань лицевой по порядку обхода вершин. Если же мы не используем отсечение граней \(активируя флаг *GL_FACE_CULL*\), тогда переменная *gl_FrontFacing* говорит нам, является ли текущий фрагмент лицевой или нелицевой частью. Мы можем рассчитывать различные цвета для лицевой части например.

Булева переменная *gl_FrontFacing* устанавливается в значение *true*, если фрагмент на лицевой грани, иначе – *false*. Для примера мы можем создать куб с разными текстурами внутри и снаружи.

```glsl
#version 330 core
out vec4 FragColor;
  
in vec2 TexCoords;

uniform sampler2D frontTexture;
uniform sampler2D backTexture;

void main()
{             
    if(gl_FrontFacing)
        FragColor = texture(frontTexture, TexCoords);
    else
        FragColor = texture(backTexture, TexCoords);
} 
```

Если посмотрим внутрь контейнера – увидим, что там используется другая текстура.

![](3.png)

Замечу, что если вы активируете отсечение граней, то вы не увидите никаких граней внутри контейнера, поэтому использование *gl_FronFacing* будет бесполезно.

### gl_FragDepth

*gl_FragCoord* является входной переменной, которая позволяет нам читать координаты в системе координат окна и получать значение глубины текущего фрагмента, но эта переменная **только для чтения**. Мы не можем изменять координаты фрагмента в системе координат окна, но мы можем устанавливать значение глубины фрагмента. GLSL предоставляет нам выходную переменную – *gl_FragDepth*, используя которую, мы можем устанавливать значение глубины фрагмента внутри шейдера.

Установить значение глубины просто – нужно лишь записать float значение от 0.0 до 1.0 в переменную *gl_FragDepth*.

```glsl
gl_FragDepth = 0.0; // теперь значение глубины этого фрагмента равно нулю
```

Если шейдер не пишет значение в *gl_FragDepth*, то значение для этой переменной будет автоматически взято из *gl_FragCoord.z*.

Однако, самостоятельная установка значения глубины имеет значительный недостаток, т.к. OpenGL отключает все **ранние проверки глубины** \(как обсуждалось в [тест глубины](../../part%204/chapter%201/text.md)\) как только во фрагментном шейдере появится запись в gl_FragDepth. Это сделано в силу того, что OpenGL не может знать какое значение глубины будет у фрагмента **до** запуска фрагментного шейдера, т.к. фрагментный шейдер мог полностью изменить это значение.

Записывая в *gl_FragDepth*, стоит подумать о возможном падении производительности. Однако, начиная с версии OpenGL 4.2 мы можем найти компромисс, переобъявляя переменную *gl_FragDepth* в начале фрагментного шейдера с условием глубины.

```glsl
layout (depth_<condition>) out float gl_FragDepth;
```

Параметр *condition* может принимать следующие значения:

| Условие | Описание |
| --- | --- |
| *any* | Значение по умолчанию. *Ранняя проверка глубины* отключена – вы потеряете в производительности |
| *greater* | Вы можете установить значение глубины только больше чем *gl_FragCoord.z* |
| *less* | Вы можете установить значение глубины только меньше чем *gl_FragCoord.z* |
| *unchanged* | В gl_FragDepth вы записываете значение равное gl_FragCoord.z |

Указывая *greater* или *less* в качестве условия глубины, OpenGL может сделать предположение что вы будете записывать лишь значения больше или меньше чем значения глубины фрагмента. При таком раскладе OpenGL все еще может производить ранний тест значения глубины, в случае если значение меньше значения глубины фрагмента.

В примере ниже мы увеличиваем значение глубины в фрагментном шейдере, но так же хотим сохранить раннюю проверку глубины в фрагментом шейдере.

```glsl
#version 420 core // обратите внимание на версию OpenGL
out vec4 FragColor;
layout (depth_greater) out float gl_FragDepth;

void main()
{             
    FragColor = vec4(1.0);
    gl_FragDepth = gl_FragCoord.z + 0.1;
}  
```

Это свойство доступно лишь в OpenGL 4.2 и выше.

## Интерфейсные блоки

До сих пор, всякий раз когда мы хотели передать данные из вершинного шейдера во фрагментный, мы объявляли несколько соответствующих входных/выходных переменных. Этот способ является самым простым передать данные из одного шейдера в другой. С ростом сложности приложений, возможно, вы захотите передавать больше чем несколько переменных, которые могут включать массивы и/или структуры.

Чтобы помочь нам организовать переменные GLSL предоставляет такую вещь как **интерфейсные блоки**, которая позволяет группировать переменные вместе. Объявление таких интерфейсных блоков во многом похожа на объявление *структуры*, за исключением использования ключевых слов *in* и *out*, основанных на использовании блока \(входной или выходной\).

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoords;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out VS_OUT
{
    vec2 TexCoords;
} vs_out;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);    
    vs_out.TexCoords = aTexCoords;
}  
```

Здесь мы объявили интерфейсный блок *vs_out*, группирующий все выходные переменные вместе, которые будут отправлены в следующий шейдер. Это банальный пример, но только представьте как это может помочь вам организовать ваш ввод-вывод в шейдерах. Это так же будет полезным в следующем уроке по геометрическим шейдерам, когда вам нужно будет объединять ввод-вывод шейдеров в массивы.

Затем нам нужно объявить входной интерфейсный блок в следующем шейдере — фрагментном. **Имя блока** \(VS_OUT\) должно быть таким же, но **имя экземпляра** \(*vs_out*, используемое в вершинном шейдере\) может быть любое, главное избегать путаницы в именах \(например, называя экземпляр, содержащий входные данные *vs_out*\).

```glsl
#version 330 core
out vec4 FragColor;

in VS_OUT
{
    vec2 TexCoords;
} fs_in;

uniform sampler2D texture;

void main()
{             
    FragColor = texture(texture, fs_in.TexCoords);   
} 
```

Поскольку имена интерфейсных блоков одинаковые, их соответствующий ввод-вывод совмещен воедино. Это еще одно полезное свойство, помогающее организовывать ваш код и доказывающее полезность при переходе между конкретными этапами как геометрический шейдер.

## Юниформ-буфер

Мы используем OpenGL уже довольно долго и изучили несколько хороших приёмов, но также получили несколько неудобств. Например, когда мы постоянно используем больше одного шейдера, нам приходится, устанавливать юниформ-переменные, в то время как большинство из них одинаковые в каждом шейдере – почему же приходится делать это повторно?

OpenGL предоставляет нам инструмент под названием **юниформ-буфер**, позволяющий нам объявлять набор глобальных юниформ-переменных, которые остаются такими же в каждом шейдере. При использовании юниформ-буфера, нам необходимо установить нужные юниформ-переменные лишь **единожды**. Но нам все еще придется позаботиться об уникальных переменных для конкретного шейдера. Однако, для настройки объекта юниформ-буфера нам придется немного попотеть.

Поскольку юниформ-буфер является буфером, как и любой другой буфер, мы можем создать его через функцию *glGenBuffers*, привязать его привязать его к цели *GL_UNIFORMS_BUFFER*, и поместить все необходимые данные юниформ-переменных туда. Существуют определенные правила помещения данных в юниформ-буфер — мы вернемся к этому позже. Сначала мы поместим наши матрицы *проекции* и *вида* в так называемый **юниформ-блок** в вершинном шейдере.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices
{
    mat4 projection;
    mat4 view;
};

uniform mat4 model;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

В большинстве наших примеров мы задавали матрицы проекции и вида в каждой итерации рендеринга для каждого используемого шейдера. Это идеальный пример для демонстрации полезности юниформ-буфера, т.к. теперь нам нужно задать их лишь один раз. Мы объявили и назвали юниформ-блок – *Matrices*, хранящий две 4х4 матрицы. Переменные в блоке могут быть доступны напрямую без указания префикса блока. Затем мы помещаем значения этих матриц в буфер где-нибудь в коде и каждый шейдер, объявляющий этот юниформ-блок, имеет доступ к матрицам.

Вы сейчас наверное думаете, что значит выражение *std140*. Оно говорит что юниформ-блок использует особый метод размещения в памяти своего содержимого; это выражение задает **разметку \(схему размещения\) юниформ-блока**.

## Разметка юниформ-блока

Содержимое юниформ-блока хранится в объекте буффера, который по сути своей не более чем зарезервированный участок памяти. Так как этот участок памяти не содержит информацию какой тип данных он хранит, нам нужно сказать OpenGL какой участок памяти соответствует каждой из юниформ-переменных в шейдере.

Представим следующий юниформ-блок в шейдере:

```glsl
layout (std140) uniform ExampleBlock
{
    float value;
    vec3  vector;
    mat4  matrix;
    float values[3];
    bool  boolean;
    int   integer;
};  
```

Мы хотим знать размер \(в байтах\) и смещение \(от начала блока\) для каждого из этих переменных, чтобы мы могли разместить их в буфере в соответствующем порядке. Размер каждого элемента явно определен в OpenGL и напрямую соотносится к C++ типам; вектора и матрицы – большие массивы чисел с плавающей точкой. Что OpenGL явно не определяет – так это **пространство** между переменными. Это позволяет аппаратному обеспечению размещать переменные так как посчитает нужным. Например, некоторые экземпляры размещают *vec3* рядом с *float*. Не все могут обработать это и поэтому выравнивают vec3 до массива из четырех *float* до того как добавят *float*. Замечательное свойство, но для нас неудобно.

GLSL по умолчанию использует так **называемую разделяемую** разметку \(схему размещения\) для памяти юниформ-буфера. Разделяемой разметка называется, потому что смещения, определенные аппаратным обеспечением, совместно используются несколькими программами. С помощью разделяемой разметки GLSL имеет право перемещать юниформ-переменные для оптимизации, с условием того, что порядок переменных не изменяется. Т.к. мы не знаем на каком смещении каждая юниформ-переменная, мы и не узнаем как точно заполнить наш юниформ-буфер. Мы можем запросить эту информацию функциями типа *glGetUniformIndices*, но это выходит за рамки нашего урока.

Пока разделяемая разметка предоставляет нам некоторую оптимизацию затрачиваемой памяти, нам нужно запросить смещение у каждой юниформ-переменной, что превращается в большую работу. Однако распространенная практика не использовать разделяемую разметку, а использовать **std140** разметку. Std140 **явно** задает схему размещения в памяти для каждого типа переменной, устанавливая соответствующее смещение по специальным правилам. Т.к. смешения явно указаны, мы можем в ручную выяснить смещения для каждой переменной.

Каждой переменной соответствует **базовое выравнивание**, равное объему занимаемой переменной памяти \(включая байты выравнивания \(padding\)\) внутри юниформ-блока — значение этого базового выравнивания вычисляется по правилам разметки *std140*. Затем для каждой переменной, мы вычисляем **выровненное смещение** в байтах от начала блока. Выровненное байтовое смещение переменной **должно** быть кратно базовому выравниванию.

Точные правила разметки вы можете найти в спецификации по юниформ-буферу OpenGL — [здесь](http://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt). Но мы перечислим общие правила ниже. Каждый тип переменной в GLSL, такой как *int*, *float* и *bool* — определен как четырёхбайтовый, каждый четырехбайтовый объект обозначается как N.

| Тип  | Правило схемы размещения(разметки) |
| --- | --- |
| Скалярный (*int*, *bool*) | Каждый скалярный тип имеет базовое выравнивание N |
| Вектор | 2N или 4N. Это значит что *vec3* имеет базовое выравнивание 4N |
| Массив векторов или скаляров | Каждый элемент имеет базовое выравнивание равное выравниванию *vec4* |
| Матрицы | Хранятся как большие массивы колонок векторов, где каждый вектор имеет базовое выравнивание *vec4* |
| Структура | Равняется вычисленному размеру всех элементов, в соответствии с предыдущим правилом, но дополненная до кратности размера vec4 |

Как и большинство OpenGL спецификаций – проще понять на примере. Мы рассмотрим представленный ранее юниформ-блок – *ExampleBlock* и вычислим выровненное смещение каждого члена, используя разметку *std140*.

```glsl
layout (std140) uniform ExampleBlock
{
                     // базовое выравнивание  // выровненное смещение
    float value;     // 4               // 0 
    vec3 vector;     // 16              // 16  (должно быть кратно 16, потому заменяем 4 на 16)
    mat4 matrix;     // 16              // 32  (column 0)
                     // 16              // 48  (column 1)
                     // 16              // 64  (column 2)
                     // 16              // 80  (column 3)
    float values[3]; // 16              // 96  (values[0])
                     // 16              // 112 (values[1])
                     // 16              // 128 (values[2])
    bool boolean;    // 4               // 144
    int integer;     // 4               // 148
}; 
```

В качестве упражнения, попытайтесь вычислить значение смещения самостоятельно и сравнить с этой таблицей. Вычисленными значениями смещения, на основе правил разметки std140, мы можем заполнить буфер данными на каждом смещении используя функции такие как *glBufferSubData*. Std140 не самый эффективный, но гарантирует нам что память разметки останется такой же для каждой программы, объявляющей этот юниформ-блок.

Добавляя выражение layout \(std140\) до определения юниформ-блока, мы говорим OpenGL что этот блок использует разметку std140. Существуют еще две схемы размещения, которыми мы можем воспользоваться, требующие от нас запрашивать каждое смещение перед заполнением буфера. Мы уже видели в деле **разделяемую разметку**, и оставшаяся разметка – **уплотненная**. Во время использования уплотненной разметки, не существует гарантий что разметка останется такой же между программами \(не разделяемыми\), поскольку это позволяет компилятору оптимизировать юниформ-переменные, выбрасывая отдельные юниформ-переменные, что может привести к отличиям в разных шейдерах.

## Использование юниформ-буферов

Мы обсудили определение юниформ-блоков в шейдерах и указание схемы размещения памяти, но мы еще не обсудили как же их использовать.

Первое что нам понадобится – юниформ-буфер, что уже сделано с помощью *glGenBuffers*. Создав объект буфера мы привязываем его к *GL_UNIFORM_BUFFER* и выделяем требуемое количество памяти с помощью вызова *glBufferData*.

```cpp
unsigned int uboExampleBlock;
glGenBuffers(1, &uboExampleBlock);
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW); // выделяем 150 байт памяти
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

В случае когда мы захотим обновить или вставить данные в буфер, мы привязываемся к *uboExampleBlock* и используем *glBufferSubData* для обновления памяти. Нам достаточно обновить этот буфер один раз, и все шейдеры, использующие этот буфер, будут использовать обновленные данные. Но как OpenGL узнает какие юниформ-буферы соответствуют каким юниформ- блокам?

В контексте OpenGL существует некоторое количество **точек привязки**, определяющих куда мы можем привязать юниформ-буфер. Создав юниформ-буфер мы соединяем его с одной из точек привязки, а так же соединяем юниформ-блок с этой же точкой, по сути связывая их вместе. Диаграмма ниже наглядно показывает это:

![](4.png)

Как видите, мы можем привязать несколько юниформ-буферов к разным точками привязки. Поскольку шейдер A и шейдер B имеет юниформ-блок, соединенный с одной и той же точкой привязки 0, информация *uboMatrices* в юниформ-блоках становится общей для них; требуется чтобы эти шейдеры определяли один и тот же юниформ-блок *Matrices*.

Чтобы привязать юниформ-блок к точке привязки, нам нужно вызвать *glUnifomBlockBinding*, принимающую первым аргументом идентификатор объекта шейдерной программы, вторым- индекс юниформ-блока и третьим — точку привязки \(куда привязываемся\). **Индекс юниформ-блока** – индекс расположения определенного блока в шейдере. Эту информацию можно получить с помощью вызова *glGetUnifromBlocIndex*, принимающей, в качестве аргументов идентификатор объекта шейдерной программы и имя юниформ-блока. Мы можем привязать юниформ-блок *Lights* как на рис.3 к точке привязки 2 следующим образом.

```cpp
unsigned int lights_index = glGetUniformBlockIndex(shaderA.ID, "Lights");   
glUniformBlockBinding(shaderA.ID, lights_index, 2);
```

Замечу, что нам придется это повторить для каждого шейдера.
> Начиная с **OpenGL 4.2** стало возможным хранить точки привязки юниформ-блока в шейдере **явно**, добавляя дополнительный спецификатор схемы размещения, что избавляет нас от необходимости вызывать *glGetUniformBlockIndex* и *glUniformBlockBinding*. В примере ниже мы соединяем точку привязки и юниформ-блок Lights явно.
> 
> ```glsl
>     layout(std140, binding = 2) uniform Lights { ... };
> ```

Затем мы должны привязать юниформ-буфер к этой же точке привязки с помощью *glBindBufferBase* или *glBindBufferRange*.

```cpp
glBindBufferBase(GL_UNIFORM_BUFFER, 2, uboExampleBlock); 
// или
glBindBufferRange(GL_UNIFORM_BUFFER, 2, uboExampleBlock, 0, 152);
```

Функция *glBindBufferBase* в качестве параметров ожидает идентификатор цели привязки буфера – индекс точки привязки и юниформ-буфер. Эта функция соединяет *uboExampleBlock* и точку привязки 2 и с этого момента точка 2 связывает оба объекта. Вы так же можете использовать *glBindBufferRange*, принимающую дополнительные параметры смещения и размера – при таком подходе вы можете привязывать только указанный диапазон юниформ-буфера к точке привязки. Используя *glBindBufferRange*, вы можете привязать несколько юниформ-блоков к одному юниформ-буферу.

Теперь всё настроено, мы можем начать добавлять данные в юниформ-буфер. Мы могли бы добавить все данные в качестве единого массива или обновить части буфера когда нам это нужно, используя *glBufferSubData*. Для обновления юниформ-переменой boolean мы могли бы обновить юниформ-буфер вот так:

```cpp
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
int b = true; // булевы переменные в GLSL представлены как четырёхбайтовые, поэтому храним их в качестве целочисленной переменной
glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b); 
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Такая же операция проделывается со всеми юниформ-переменным внутри юниформ-блока, но с разными аргументами.

## Простой пример

Давайте продемонстрируем истинную пользу юниформ-буфера. Если посмотреть на предыдущие участки кода – мы всё время использовали 3 матрицы: проекции, вида и модельную. Из всех этих матриц часто меняется только модельная матрица. Если у нас есть несколько шейдеров, использующих набор одних и тех же матриц, нам вероятно выгоднее будет использовать объект юниформ-буфера.

Мы будем хранить матрицы проекции и вида в юниформ-блоке – *Matrices*. Мы не будет хранить здесь модельную матрицу, т.к. она меняется довольно часто в шейдерах, от таких действий мы бы не получили особой пользы.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices
{
    mat4 projection;
    mat4 view;
};
uniform mat4 model;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

Здесь не происходит ничего особенного, кроме использования схемы размещения std 140. В нашем примере мы нарисуем 4 куба, используя для каждого куба персональный шейдер. Все четыре будут использовать один и тот же вершинный шейдер, но разные фрагментные шейдеры, выводящие свой собственный цвет.

Для начала поместим юниформ-блок вершинного шейдера в точку привязки 0. Заметим, что мы должны это делать для каждого шейдера.

```cpp
unsigned int uniformBlockIndexRed    = glGetUniformBlockIndex(shaderRed.ID, "Matrices");
unsigned int uniformBlockIndexGreen  = glGetUniformBlockIndex(shaderGreen.ID, "Matrices");
unsigned int uniformBlockIndexBlue   = glGetUniformBlockIndex(shaderBlue.ID, "Matrices");
unsigned int uniformBlockIndexYellow = glGetUniformBlockIndex(shaderYellow.ID, "Matrices");  
  
glUniformBlockBinding(shaderRed.ID,    uniformBlockIndexRed, 0);
glUniformBlockBinding(shaderGreen.ID,  uniformBlockIndexGreen, 0);
glUniformBlockBinding(shaderBlue.ID,   uniformBlockIndexBlue, 0);
glUniformBlockBinding(shaderYellow.ID, uniformBlockIndexYellow, 0);
```

Далее мы создаем юниформ-буфер и так же привязываем буфер к точке 0.

```cpp
unsigned int uboMatrices
glGenBuffers(1, &uboMatrices);
  
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferData(GL_UNIFORM_BUFFER, 2 * sizeof(glm::mat4), NULL, GL_STATIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
  
glBindBufferRange(GL_UNIFORM_BUFFER, 0, uboMatrices, 0, 2 * sizeof(glm::mat4));
```

Сначала выделяем достаточно памяти под наш буфер, что равно двойному размеру *glm::mat4*. Размер матрицы GLM в точности соответствует размеру матрицы *mat4* GLSL. Затем мы соединяем определенный диапазон буфера, в нашем случае это целый буфер, с точкой привязки 0.

Теперь все что осталось – заполнить буфер. Если параметр *угла обзора* матрицы проекции сделать неизменным \(попрощаемся с возможностью зума\), то матрицу можно определить всего один раз — значит и в буфер её скопировать достаточно один раз. Поскольку мы уже выделили достаточно памяти для объекта буфера, мы можем использовать *glBufferSubData* для хранения матриц проекции до того как войдем в игровой цикл:

```cpp
glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), glm::value_ptr(projection));
glBindBuffer(GL_UNIFORM_BUFFER, 0);  
```

Здесь мы помещаем первую часть юниформ-буфера – матрицу проекций. Перед отрисовкой объектов, на каждой итерация рендеринга мы обновляем второю часть буфера – матрицу вида.

```cpp
glm::mat4 view = camera.GetViewMatrix();	       
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4), sizeof(glm::mat4), glm::value_ptr(view));
glBindBuffer(GL_UNIFORM_BUFFER, 0);  
```

На этом все для юниформ-буфера. Каждый вершинный шейдер, содержащий *Matrices* юниформ-блок, теперь будет содержать данные, хранящиеся в *uboMatrices*. Теперь если бы мы рисовали 4 куба, используя 4 разных шейдера, их матрицы проекции и вида остались бы такими же.

```cpp
glBindVertexArray(cubeVAO);
shaderRed.use();
glm::mat4 model;
model = glm::translate(model, glm::vec3(-0.75f, 0.75f, 0.0f));	// переместим в левую верхнюю часть
shaderRed.setMat4("model", model);
glDrawArrays(GL_TRIANGLES, 0, 36);        
// ... рисуем зелёный куб
// ... рисуем синий куб
// ... рисуем желтый куб	 
```

Единственная юниформ-переменная, которую мы должны установить – model . Использование юниформ-буфера в такой конфигурации спасает нас от вызовов установки значения юниформ-переменных для каждого шейдера. Результат выглядит как то так:

![](5.png)

Благодаря разным шейдерам и изменению модельной матрицы, 4 куба переместились в свои части экрана и имеют разный цвет. Это относительно простой сценарий, где мы можем использовать юниформ-буферы, но любой другой большой проект с рендерингом мог бы иметь свыше сотни активных шейдеров; это как раз тот случай, когда юниформ-буферы показывают себя во всей красе.

Вы можете найти исходный код примера юниформ приложения [здесь](src1.cpp).

Юниформ-буферы имеют несколько преимуществ по сравнению с установкой отдельных юниформ-переменных. Во-первых, установка множества юниформ-переменных за раз – быстрее чем установка нескольких юниформ-переменых несколько раз. Во-вторых, если вы хотите изменить одну и ту же юниформ-переменную в нескольких шейдерах, гораздо легче изменить юниформ-переменную один раз в юниформ-буфере. И последнее преимущество, не столь очевидное, но вы сможете использовать гораздо больше юниформ-переменных в шейдерах, используя юниформ-буфер. OpenGL имеет ограничение на количество обрабатываемых юниформ данных. Вы можете получить эту информацию с помощью *GL_MAX_VETEX_UNIFORM_COMPONENTS*. Во время использования юниформ-буфера это ограничение существенно выше. Когда достигните предела использования юниформ-переменных(например когда занимаетесь скелетной анимацией) вы всегда можете использовать юниформ-буферы.
