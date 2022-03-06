# Micr

**WARNING! Язык программирования в разработке!**

Это мой миниатюрный stack-based язык программирования.

В планах реализовать
- [x] Компиляция
- [x] Работа на основе стека
- [ ] Тьюринг полнота
- [ ] Статическа типизация
- [ ] Переписать язык на самого себя

## Примеры

Простая программа, которая выводит числа от 10 до 0:

```micr
10 while dup 0 > do
  dup dump
  1 -
end
```

## Быстрый старт

### Симуляция

Интепретация программы.

```console
$ cat program.micr
34 35 + dump
$ ./micr.py sim program.micr
69
```

### Компиляция

Компиляция генерирует асм код, собирается это все с помощью [nasm](https://www.nasm.us/), линкуется с помощью [GNU ld](https://www.gnu.org/software/binutils/). Данные программы должны быть доступны в `$PATH`.

```console
$ cat program.micr
34 35 + dump
$ ./micr.py com program.micr
[INFO] Generating ./program.asm
[CMD] nasm -felf64 ./program.asm
[CMD] ld -o ./program ./program.o
$ ./program
69
```

## Возможности языка

Это то, что язык поддерживает до сих пор. **Поскольку язык находится в стадии разработки, точный набор операций может измениться.**

### Манипуляции со стеком

- `<integer>` - поместить целое число в стек. Прямо сейчас целое число — это все, что можно передать в [int](https://docs.python.org/3/library/functions.html#int) функцию.
```
push(<integer>)
```
- `dup` - дублировать элемент поверх стека.
```
a = pop()
push(a)
push(a)
```
- `dump` - вывести элемент поверх стека на стандартный вывод и удалить его из стека.
```
a = pop()
print(a)
```

### Сравнение

- `=` - проверяет, равны ли два элемента на вершине стека. Удаляет элементы из стека и помещает «1», если они равны, и «0», если они не равны.
```
a = pop()
b = pop()
push(int(a == b))
```
- `>` - проверяет, больше ли нижний элемент в стеке, чем верхний.
```
b = pop()
a = pop()
push(int(a > b))
```

### Арифметика

- `+` - суммирует два элемента на вершине стека.
```
a = pop()
b = pop()
push(a + b)
```
- `-` - вычитает вершину стека из нижнего элемента.
```
a = pop()
b = pop()
push(b - a)
```

### Поток управления

- `if <then-branch> else <else-branch> end` - помещает элемент на вершину стека, и если элемент не равен `0`, выполняет `<then-branch>`, иначе `<else-branch>`.
- `while <condition> do <body> end` - продолжает выполнять `<condition>` и `<body>` до тех пор, пока `<condition>` производит `0` в верхней части стека. Проверка результата `<codition>` удаляет его из стека.

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
- `->` - загрузить байт с заданного адреса.
```
addr = pop()
byte = load(addr)
push(byte)
```

### Система

- `syscall1` - выполнить системный вызов с 1 аргументом.
```
syscall_number = pop()
<move syscall_number to the corresponding register>
for i in range(1):
    arg = pop()
    <move arg to i-th register according to the call convention>
<perform the syscall>
```
- `syscall3` - выполнить системный вызов с 3 аргументами.
```
syscall_number = pop()
<move syscall_number to the corresponding register>
for i in range(3):
    arg = pop()
    <move arg to i-th register according to the call convention>
<perform the syscall>
```

