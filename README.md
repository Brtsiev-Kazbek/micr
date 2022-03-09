# Micr

**WARNING! Язык в разработке!**

Micr будет:
- [x] Компилируемым
- [x] Нативным
- [x] Стековым
- [x] [Тьюринг полным]
- [ ] Написанным на самом себе
- [ ] Статически типизируемым

## Примеры

Hello, World:

```pascal
"Hello, World\n" 1 1 syscall3
```

Выводим числа от 99 до 0:

```pascal
100 0 while 2dup > do
    dup print 1 +
end
```

## Быстрый старт

### Симуляция

Симуляция интепретирует программу.

```console
$ cat program.micr
34 35 + print
$ ./micr.py sim program.micr
69
```

### Компиляция

Компиляция генерирует ассемблер, который собирается через [nasm](https://www.nasm.us/), и затем линкуется с помощью [GNU ld](https://www.gnu.org/software/binutils/). Они должны быть доступны в `$PATH`.

```console
$ cat program.micr
34 35 + print
$ ./micr.py com program.micr
[INFO] Generating ./program.asm
[CMD] nasm -felf64 ./program.asm
[CMD] ld -o ./program ./program.o
$ ./program
69
```

## Квази-документация


### Манипуляции со стеком

- `<integer>` - поместить целое число в стек. Прямо сейчас целое число — это все, что можно разобрать с помощью [int](https://docs.python.org/3/library/functions.html#int) функции.
```
push(<integer>)
```
- `<string>` - поместите размер и адрес строкового литерала в стек. Строковый литерал — это последовательность символов, заключенная в `"`.
```
size = len(<string>)
push(n)
ptr = static_memory_alloc(n)
copy(ptr, <string>)
push(ptr)
```
- `dup` - дублировать верхний элемент в стеке.
```
a = pop()
push(a)
push(a)
```
- `2dup` - дублировать два элемента.
```
b = pop()
a = pop()
push(a)
push(b)
push(a)
push(b)
```
- `swap` - поменять местами два элемента в стеке.
```
a = pop()
b = pop()
push(a)
push(b)
```
- `drop` - удалить верхний элемент в стеке.
```
pop()
```
- `print` - напечатать верхний элемент в стеке и удалить его оттуда.
```
a = pop()
print(a)
```
- `over`
```
a = pop()
b = pop()
push(b)
push(a)
push(b)
```

### Comparison

- `=` - проверяет, равны ли два элемента на вершине стека. Удаляет элементы из стека и помещает `1` 
если они равны и `0` если нет.
```
a = pop()
b = pop()
push(int(a == b))
```
- `!=` - проверяет, не равны ли два элемента на вершине стека.
```
a = pop()
b = pop()
push(int(a != b))
```
- `>` - проверяет, больше ли элемент ниже верхнего, чем верхний.
```
b = pop()
a = pop()
push(int(a > b))
```
- `<` - проверяет, если элемент ниже вершины меньше вершины.
```
b = pop()
a = pop()
push(int(a < b))
```
- `>=`
```
b = pop()
a = pop()
push(int(a >= b))
```
- `<=`
```
b = pop()
a = pop()
push(int(a >= b))
```

### Арифметика

- `+` - суммирует два элемента в стеке.
```
a = pop()
b = pop()
push(a + b)
```
- `-` - вычитает два элемента в стеке.
```
a = pop()
b = pop()
push(b - a)
```
- `mod`
```
a = pop()
b = pop()
push(b % a)
```

### Битовые операции

- `>>`
```
a = pop()
b = pop()
push(b >> a)
```
- `<<`
```
a = pop()
b = pop()
push(b << a)
```
- `|`
```
a = pop()
b = pop()
push(b | a)
```
- `&`
```
a = pop()
b = pop()
push(b & a)
```

### Управление потоком исполнения

- `if <then-branch> else <else-branch> end` -помещает элемент на вершину стека, и если элемент не `0` выполняет `<then-branch>`, иначе `<else-branch>`.
- `while <condition> do <body> end` - продолжит исполнять `<condition>` и `<body>` до тех пор, пока `<condition>` возвращает `0` на вершину стека. Сверяет результат с `<condition>` удаляет его из стека.

### Память

- `mem` - помещает в стек адрес начала памяти, где вы можете читать и писать.
```
push(mem_addr)
```
- `<-` - сохранить данный байт по заданному адресу.
```
byte = pop()
addr = pop()
store(addr, byte)
```
- `->` - загрузить данный байт по заданному адресу.
```
addr = pop()
byte = load(addr)
push(byte)
```

### Система

- `syscall<n>` - выполнить системный вызов с n аргументами, где n находится в диапазоне `[0..6]`. (`syscall1`, `syscall2`, и т.д.)
```
syscall_number = pop()
<move syscall_number to the corresponding register>
for i in range(n):
    arg = pop()
    <move arg to i-th register according to the call convention>
<perform the syscall>
```


## Макросы

Определите слово, которое расширяется в инструкцию во время компиляции

```
macro write
    1 1 syscall3
end
```

### Импорт

Импортровать инструкции из файла `std.micr`

```
include "std.micr"
```