# Система типов [00:35]
## Общее [00:10]
* Что такое "переменная"? Моё определение: кусок памяти, которому дали имя.
  Это не совсем точно, потому что переменная в процессе компиляции и оптимизации может полностью исчезнуть, ну да ладно.
* Что такое "тип переменной"? Моё определение: это пара: (какие значения могут храниться в переменной, какие операции можно с ней выполнять).
* Базовое разделение бывает просто по значениям переменной: числа целые, вещественные, строчки, массивы.
* Зачем это надо: чтобы компилятор знал, что конкретно на уровне байт делать, когда пишем
  `a = b * 2`, для целых и вещественных это разное.
* Компилятор может проверять типы на этапе компиляции и запрещать некорректные операции,
  даже если им можно придумать смысл (или несколько):
  ```
  int a = 10;
  char *x = a;
  ```
  Или как-то хитро генерировать код:
  ```
  double d = 1.2;
  int x = d;  // Скомпилируется, округление к нулю автоматически.
  ```
* Так что можно в типах записывать не только "сколько байт и как кодируется", но и добавлять
  дополнительные ограничения.
  Пример из физики: на C++ можно сделать типы "метр", "секунда", "метр в секунду" и компилятор будет проверять размерность:
  ```
  meter distance = 10m;
  seconds time = 10s;
  mps speed1 = 2 * distance / time;
  mps speed2 = time / distance;  // Compilation error.
  mps speed3 = 10.0;  // Compilation error.
  ```

## const у переменной [00:10]
* Зачем: говорим, что значение установлено один раз и менять нельзя.
  Компилятор проверит и убедится, что мы нигде случайно не ошиблись.
  * Математическая константа: `const double PI = 3.1415;`
  * Константы для задачи:
    ```
    const int dx[] = { 1, 0, -1, 0 };
    const int dy[] = { 0, 1, 0, -1 };
    ```
* Есть и в Си, и в С++ (ошибка с прошлой лекции).
  Отличие в том, что в C++ константы можно использовать для размера массива и ещё некоторых случаев, а в Си — нет:
  ```
  const int N = 10;
  int arr[N];
  ```
  https://stackoverflow.com/a/5249001/767632
* Альтернативный синтаксис: `double const PI = 3.1415;`, тогда "`const` защищает то, что слева".
* Не любой `const` компилятор может вычислить на этапе работы программы и встроить.
  Можно делать глобальные, а можно делать локальные, и даже параметры:
  ```
  void solve(const int w, const int h) {
      const int area = w * h;
      // ...
      printf("%d\n", answer);
  }
  int main() {
      int w, h;
      while (scanf("%d%d", &w, &h) == 2) {
          solve(w, h); // Константность требуется только у копии внутри solve()
      }
  }
  ```
  Тут мы не можем забыть "пересчитать" `area`, потому что она зависит только
  от `w` и `h`, которые нельзя поменять.
  А если уж очень надо — то, наверное, это вообще начался новый тест, так что
  надо пересчитать вообще всё. И поэтому лучше делать это в функции, а не в
  глобальных переменных, чтобы компилятор мог проверить.

## const у указателя [00:15]
* Указатель на константную память.
  Пример — строковой литерал. Указывает на кусок памяти, в котором хранится код программы, менять его нельзя.
  ```
  const char *s1 = "hello";
  s1[0] = 'H';  // Compilation error
  char *s2 = "world";  // Compilation error в C++, в Си это окей, потому что const не сразу был.
  s2[0] = 'w';  // UB в Си (runtime)
  ```
* Альтернативный синтаксис: `char const *s1`, тогда "`const` защищает то, что слева".
  Или: разыменовали `s1`, получили `char const`.
* При этом сам указатель можно менять:
  ```
  const char *s1 = "hello";
  *s1; // 'h'
  s1 = s1 + 2;
  *s1; // 'e'
  ```
* Можно сделать константный указатель, который менять нельзя:
  ```
  char arr[10] = "hello";  // Специальный синтаксис "скопируй строчку в локальный массив".
  char *const s1 = arr;
  *s1 = 'H';  // OK
  s1 = s1 + 2;  // Compilation error.
  ```
  Тут `const` защищает "то, что слева", то есть `*`, указатель.
  * Можно читать так: `s1` можно разыменовать, но он сам по себе `const`.
  * cdecl.org
* Путаница с терминологие: иногда говорят "константный указатель", подразумевая указатель на константную память.
* А можно совместить:
  ```
  char const *const s1 = "hello";
  *s1 = 'H';  // Compilation error.
  s1 = s1 + 2;  // Compilation error.
  ```
* Есть стиль, в котором все переменные делаются константными.
  Но тогда надо много-много функций, которые бегают по массивам, считают частичные суммы
  и прочее, это называется "функциональный стиль", в нём довольно приятно писать,
  если есть хорошая библиотека.

# Универсальная сортировка [00:15]
## Пример вызова [00:05]
Это будет в домашке.
Напомню сигнатуру:

```
void sort(void *data, int elements, size_t element_size, int (*compare)(void*, void*));
```
Что означает каждый параметр?
Давайте напишем пример вызова:

```
int arr[10] = { 4, 3, 2, 5, 6, 0, 0, 9, 9, 10 };
sort(arr, 10, sizeof(int));
sort(arr, 10, sizeof arr[0]);
sort(arr, sizeof arr / sizeof arr[0], sizeof arr[0]); // Осторожно!
int *parr = arr;
sort(arr, sizeof parr / sizeof parr[0], sizeof parr[0]); // Осторожно!
```

## Добавляем const [00:05]
```
void sort(const void *data, int elements, size_t element_size, int (*compare)(const void*, const void*));
```

При этом добавлять `const` к самим параметрам не надо.
Это часть реализации функции, а не сигнатуры.
Скомпилируется, всё окей, но не надо.

И к возвращаемому значению не надо, потому что оно всё равно скопируется в отдельную переменную.

```
void sort(const void *const data, const int elements, const size_t element_size, int (*const compare)(const void*, const void*)) {
}
```

## Компаратор [00:05]
Как сделать `compare` для `int*` (важн)?
```
int int_compare(const void *pa, const void *pb) {
    int *pia = (int*)pa;
    int *pib = (int*)pb;
    return *pia - *pib;
}
```

# Стандартная библиотека [00:30]
## Зачем стандартная библиотека-1 [00:03]
* Пусть у нас есть язык программирования: массивы, переменные, указатели.
  В некоторых программах возникают одни и те же операции:
  `strlen` (узнать длину строки), `strcmp` (сравнить строки).
  Их можно реализовать как функции, но они нужны просто всем.
* Первый шаг: давайте их вынесем в отдельный файл, который будем
  использовать во всех проектах (заголовок и реализацию).
* Второй шаг: будем рядом с компилятором давать сразу заголовок
  и скомпилированный файл, чтобы не надо было компилировать.
  Получаем "библиотеку".
* Третий шаг: давайте потребуем, чтобы все компиляторы такой
  файл сами включали автоматически.
  Получаем "стандартную библиотеку".

## Зачем стандартная библиотека-2 [00:03]
* ОС не даёт прямой доступ к оборудованию.
  Мы можем общаться только с "ядром" ОС, а оно уже
  что-то делает.
* Программа должна взаимодействовать с ОС.
  Например, под Linux для записи в стандартный вывод
  надо сделать системный вызов `write`.
  Это делается на уровне ассемблера: записываем константы
  в регистры, вызываем инструкцию ассемблера.
  Под Windows аналогично, но магия другая.
* Так взаимодействовать нужно всем подряд.
* Вывод: давайте сделаем функцию `putchar`, которая выводит
  один символ в стандартный вывод независимо от ОС.
  И потребуем, чтобы все компиляторы её давали в стандартной
  библиотеке.
* Теперь можно писать одинаковый код под разные ОС, если
  пользоваться только стандартной библиотекой.

## libc [00:04]
* В языке Си стандартная библиотека на жаргоне называется `libc`.
* Есть реализации вроде `glibс`, которые могут дополнительных функций добавлять.
* Вообще очень много программ её используют, в том числе наверняка интерпретатор Python/компилятор Java.
  Чтобы не с нуля взаимодействовать с ОС всё-таки.
* Почти всегда лучше взять функцию из стандартной библиотеки и аккуратно изучить документацию
  (а то вдруг UB при каких-то аргументах).
* Все объектные файлы автоматически подключаются.
  А вот заголовки надо подключать явно:
  `<stdio.h>` — ввод/вывод (файлы)
  `<string.h>` — работа со строчками
  `<math.h>` — математика
  `<time.h>` — базовые операции со временем (но плохо работают с часовыми поясами, переводами стрелок...)
  `<stdlib.h>` — работа с памятью, случайные числа, "хламник"
* Документация:
  См. cppreference.com, cplusplus.com, MSDN.
  Все должны более-менее совпадать.
* Ещё бывают расширения конкретного компилятора (и к языку, и к библиотеке).

## stdio: FILE* [00:05]
* `stdio.h` объявляет тип `FILE`.
* На самом деле это структура, внутри которой лежат всякие объекты для связи с ОС,
  буфер для записи, ещё что-то, но нас это не интересует.
* Более того, никто не знает, как `FILE` устроена, все используют только
  `FILE*`, все функции работают только с ним.
  Сами как-то выделяют и освобождают.
* `FILE *f1 = fopen("a.txt", "r")`
  Здесь есть имя файла и "режим": `r` — чтение __текстового__ файла.
  Может вернуть `NULL`, если файл не открылся.
  Ещё можно `w` для перезаписи, `a` для дописывания в конец.
* `fclose(f1)` — если не закрыть, буфер не сбросится, Винда файл не отдаст.
* Можно `fflush(f1)` для сброса буфера.
  Но только до уровня ОС, дальше у неё ещё свои кэши есть.
* Ещё бывает `freopen` — переиспользует `FILE*`, полезно для `stdin`/`stdout`.

## Текстовые и бинарные файлы [00:05]
* Любой файл — это последовательность байт на диске.
* Если нам важны в точности значения этих байт, то мы говорим, что работаем с "бинарным" файлом.
  Например, можно записать `char` как один байт или `int` как четыре.
  Это занимает мало места, но не прочитать без программы.
* Ещё в файл можно писать "текст", тогда это "текстовый" файл (строгого определения нет).
  Числа пишем в десятичной системе, используем пробелы, табуляции, переводы строки.
  Просто открыть в блокноте.
* Тонкость: под разным ОС разные конвеции "текстовых" файлов.
  Например, в Linux для перевода строки используется один символ `\n` (10),
  а в Windows — два символа `\r\n` (13 10 CR LF).
  Если открыть текстовый файл в программе, которая ожидает другие переводы строк,
  будет странно.
* Стандартная библиотека Си (и любых других языков) при чтении текстовых файлов
  заменяет системный перевод строки на простой `\n` и наоборот.