# SOLID принципы

- [Принцип единственной ответственности](#srp)
- [Принцип открытости/закрытости](#ocp)
- [Принцип подстановки Барбары Лисков](#lsp)
- [Принцип разделения интерфейса](#icp)
- [Принцип инверсии зависимостей](#dip)


## srp

**Принцип единственной ответственности**

Для каждого класса должно быть определено единственное назначение. 
Все ресурсы, необходимые для его осуществления, 
должны быть инкапсулированы в этот класс и подчинены только этой задаче.

Пример нарушения принципа на typescript:
```typescript
class UserManager
{
    public writeToFile() {}
    public calculatePaymentAmount() {}
    public authenticateUser() {}
}
```
Принцип единственной ответственности возник 
как попытка объединения двух важных понятий структурного 
анализа и проектирования: coupling 
(сопряжение, зацепление, связанность) 
и cohesion (сплочённость, связность, прочность модуля).

### Связность (от худшего к лучшему)

#### Случайная
Подмодули модуля в этом случае никак не взаимодействуют друг с другом и выполняют функционально не связанные задачи. Примером такой связности может быть весь тот код, который часто приводят для демонстрации нарушения принципа единственной ответственности.
Пример выше описывает именно такую связанность.

#### Логическая
Также как и в случае случайной связности подмодули модуля никак не взаимодействуют друг с другом (либо взаимодействуют слабо), однако наблюдается их логическое сходство по какому-либо признаку (например, по сходству решаемых подмодулями задач).
#### Временная
Тип связности при котором подмодули объединены в модуль по причине их совместного использования в некоторый момент времени выполнения программы, а порядок обращения к ним не важен. При этом подмодули никак функционально не связаны между собой.
#### Процедурная
Тип связности при котором подмодули объединены в модуль по причине их совместного использования в некоторый момент времени выполнения программы. Обращение к модулям происходит в определённом порядке. Подмодули могут быть функционально связаны между собой.
#### Коммуникационная/информационная
Подмодули модуля функционально связаны между собой и обрабатывают одни и те же данные. Порядок обращения к подмодулям не имеет значения.
#### Последовательностная
Подмодули модуля функционально связаны между собой. При этом выходные данные одного подмодуля становятся входными данными другого подмодуля, т.е. важен порядок обращения к подмодулям. Как правило такой модуль имеет одну точку входа.
```typescript
class ExpressionExecutor
{
    public execute(str: string) {
        const ast = this.parse(
            this.lexer(
                this.characterIterator($expression)
            )
        );
        return ast.evaluate();
    }

    protected parse(lexer: string) {}
    protected lexer(Iterator: string): Lexer {}
    protected characterIterator(expression): Iterator {} 
}
```
#### Функциональная
Все подмодули модуля функционально связаны между собой и выполняют одну хорошо определённую задачу. Этот тип связности прямая противоположность случайной связности.
```typescript
class Lexer
{
    public  construct(charIterator: CharacterIterator) {}
    public  getTokens(): Generator {}
}
```

### Сопряжение (от худшего к лучшему)

#### Патологическое (pathological)
Программный модуль оказывает влияние или зависит от внутренней реализации другого модуля. Как правило, этот тип зацепления связан с нарушением принципа сокрытия информации

#### По содержимому
Часть или все содержимое одного программного модуля включены в содержимое другого модуля. Примером такого зацепления могут служить вложенные или анонимные классы.

#### По общей области данных (common, common-environment)
Два или более модулей совместно используют общую область данных (глобальную по отношению к модулям).

#### Смешанное (hybrid)
Различные подмножества значений некоторого элемента данных используются в нескольких программных модулях для разных и несвязанных целей. Тут можно придумать такой искусственный пример: есть некоторый DTO содержащий данные потребляемыми двумя совершенно разными модулями.

#### По управлению (control coupling)
Один модуль взаимодействует с другим модулем с целью повлиять на его поведение путём передачи ему управляющей информации.

#### По данным (data)
Данные одного программного модуля поступают на вход другого модуля.


## ocp

**Принцип открытости/закрытости**

Принцип открытости/закрытости означает, что программные сущности должны быть:

- открыты для расширения: означает, что поведение сущности может быть расширено путём создания новых типов сущностей.
- закрыты для изменения: в результате расширения поведения сущности, не должны вноситься изменения в код, который эту сущность использует.

В хорошо спроектированных программах новая функциональность вводится путем добавления нового кода, а не изменением старого, уже работающего.

Пример верного использования принципа открытости/закрытости:

```typescript

interface IShape{
    calculateArea(): number
}

class Shape implements IShape{
    public calculateArea(): number {
        // some logic
    }
}

class Rectangle extends Shape {
    public width: number;
    public height: number;

    public calculateArea(): number {
        return this.width * this.height;
    }
}

class Circle extends Shape {
    public radius: number;

    public calculateArea(): number {
        return Math.PI * this.radius * this.radius;
    }
}

class Triangle extends Shape {
    public base: number;
    public height: number;

    public calculateArea(): number {
        return 0.5 * this.base * this.height;
    }
}

class AreaCalculator {
    public calculateTotalArea(shapes: Array<IShape>): number {
        let totalArea = 0;
        for (const shape of shapes) {
            totalArea += shape.calculateArea();
        }
        return totalArea;
    }
}

```


## lsp

**Принцип подстановки Барбары Лисков**

Наследующий класс должен дополнять, 
а не замещать поведение базового класса.

Пример нарушения принципа подстановки Барбары Лисков:
```typescript
class Vehicle {
  public startEngine(): void {
    console.log("Engine started");
  }
}

class Car extends Vehicle {
  public startEngine(): void {
    throw new Error("Cars cannot start their engines");
  }
}

function testDrive(vehicle: Vehicle): void {
  vehicle.startEngine();
}

const car = new Car();
testDrive(car); // will throw an error

```

В этом примере Carкласс расширяет Vehicleкласс и 
переопределяет startEngine() метод для создания исключения. 
Однако это означает, что Carкласс не является допустимой заменой
Vehicleкласса в контексте testDrive функции,
которая ожидает, что любой переданный ей объект
будет иметь работающий startEngine() метод.


## irp

**Принцип разделения интерфейса**

Принцип разделения интерфейсов говорит о том, 
что слишком «толстые» интерфейсы необходимо разделять на 
более маленькие и специфические, чтобы программные
сущности маленьких интерфейсов знали только о методах,
которые необходимы им в работе. В итоге, при изменении метода 
интерфейса не должны меняться программные сущности, 
которые этот метод не используют.


Пример нарушения принципа разделения интерфейса:

```typescript
interface Shape {
  calculateArea(): number;
  calculatePerimeter(): number;
  setWidth(width: number): void;
  setHeight(height: number): void;
}

class Circle implements Shape {
  public radius: number;

  public calculateArea(): number {
    return Math.PI * this.radius * this.radius;
  }

  public calculatePerimeter(): number {
    return 2 * Math.PI * this.radius;
  }

  public setWidth(width: number): void {
    this.radius = width / 2;
  }

  public setHeight(height: number): void {
    this.radius = height / 2;
  }
}
```
В этом примере Shapeинтерфейс определяет несколько методов, в том числе setWidth()и setHeight(), которые не применимы к Circleклассу, так как у него нет ни ширины, ни высоты. Однако Circleкласс вынужден реализовывать эти методы, потому что он реализует Shapeинтерфейс. Это нарушает принцип разделения интерфейсов, который гласит, что интерфейсы должны быть небольшими и специфичными, и что клиенты не должны зависеть от методов, которые они не используют.

Более правильным вариантом будет:

```typescript
interface Shape {
  calculateArea(): number;
  calculatePerimeter(): number;
}

interface ResizableShape extends Shape {
  setWidth(width: number): void;
  setHeight(height: number): void;
}

class Circle implements Shape {
  public radius: number;

  public calculateArea(): number {
    return Math.PI * this.radius * this.radius;
  }

  public calculatePerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}
```

## dip

**Принцип инверсии зависимостей**

Каждый раз, когда внутри функции создается объект, появляется зависимость функции от класса этого объекта. Другими словами функция жестко завязана на работу в паре с конкретным классом. Есть формальный способ, позволяющий легко проверить насколько ваш код завязан в узел. Возьмите любую функцию и мысленно представьте, что вы переносите ее в другой проект. Сколько за собой она потянет зависимостей (а те в свою очередь свои зависимости)? Если перенос функции потребует переноса большого количества кода, то говорят, что в коде высокая связанность.

Для развязки кода придуман даже специальный термин: Принцип Инверсии Зависимостей. Еще он известен как DIP из SOLID. Вот его формулировка:

- Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
- Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.



Пример нарушения принципа инверсии зависимостей:
```typescript
const doSomething = () => {
  const logger = new Logger();
  // some code
};
```

Как правильно:
```typescript
interface Logger{
    debug(msg: string)
}
const doSomething = (logger: Logger) => {
  // some code
};
```

Еще пример правильного использования:

```typescript
// Dependency Inversion Principle (DIP) example

interface DataFetcher {
  fetchData(): void;
}

class Database implements DataFetcher {
  public fetchData(): void {
    console.log("Fetching data from the database...");
  }
}

class WebService implements DataFetcher {
  public fetchData(): void {
    console.log("Fetching data from the web service...");
  }
}

class DataAnalyzer {
  private dataFetcher: DataFetcher;

  constructor(dataFetcher: DataFetcher) {
    this.dataFetcher = dataFetcher;
  }

  public analyze(): void {
    this.dataFetcher.fetchData();
    // perform analysis
  }
}

const database = new Database();
const analyzer = new DataAnalyzer(database);
analyzer.analyze();

const webService = new WebService();
const analyzer2 = new DataAnalyzer(webService);
analyzer2.analyze();

```