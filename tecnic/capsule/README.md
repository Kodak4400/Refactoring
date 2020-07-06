## レコードのカプセル化
```JavaScript:Before.js
organization = {name: "Acme Gooseberries", country: "GB"}
```

```JavaScript:After.js
class Organization {
  constructor(data) {
    this._name = data.name;
    this._cuntry = data.country;
  }
  get name() {return this._name;}
  set name(arg) {this._name = arg;}
  get country() {return this._country;}
  set country(arg) {this._country = arg;}
}
```

- オブジェクト化することで、レコードの構造を意識する必要がない。
- そのため、必要なデータを`適切な名前を設定する`だけで取得できる。
- カプセル化することで、フィールド名を変更しても古い名前と新しい名前の両方のメソッドを共存できる。
- 呼び出し側からのデータの変更を禁止する場合は、`clone`したデータを保持しておき、それを返すようにする。

## コレクションのカプセル化
```JavaScript:Before.js
class Person {
  get courses() {return this._courses;}
  set courses(aList) {this._courses = aList;}
}
```

```JavaScript:After.js
class Person {
  get courses() {return this._courses.slice();}
  addCourse(aCourse) {...}
  removeCourse(aCourse) {...}
}
```

- コレクションの変数へのアクセスはカプセル化されていても、getterでコレクションそのものを返すとコレクションを保持するクラスを介さずに、その中身を変更できてしまう。
- そのため、`add`,`remove`メソッドを用意しておく。
- 加えて、コレクションそのものを返さないようにする！
- もしくは、コレクションのコピーを返すことで、読み取り専用化する！

## オブジェクトによるプリミティブの置き換え
```JavaScript:Before.js
orders.filter(o => "high" === o.priority
                || "rush" === o.priority);
```

```JavaScript:After.js
orders.filter(o => opriority.higherThan(new Priority("normal")));
```

- 単純な表示以上のことをする必要があるとわかったら、ちょっとしたデータでも新たなクラスにする。
- そうしたクラスはプリミティブ型を単にラップするだけでもOK
- イメージを付けるためにサンプルを置く。
```JavaScript:Migration1.js
//Migration 1
class Order() {
  constructor(data) {
    this.priority = data.priority;
  }
}

// 呼び出し
highPriorityCount = orders.filter(o => "high" === o.priority|| "rush" === o.priority).length;
```

```JavaScript:Migration2.js
//Migration 2
class Order() {
  constructor(data) {
    this.priority = data.priority;
  }
  get priority() {return this._priority;}
  set priority(aString) {this._priority = aString;}
}

// PriorityClassを用意しておく.
class Priority {
  constructor(value) {this._value = value;}
  toString() {return this._value;}
}

// 呼び出し
highPriorityCount = orders.filter(o => "high" === o.priority|| "rush" === o.priority).length;
```

```JavaScript:Migration3.js
//Migration 3
class Order() {
  constructor(data) {
    this.priority = data.priority;
  }
  get priorityString() {return this._priority.toString;}
  set priority(aString) {this._priority = new Priority(aString);}
}

class Priority {
  constructor(value) {this._value = value;}
  toString() {return this._value;}
}

// 呼び出し
highPriorityCount = orders.filter(o => "high" === o.priorityString|| "rush" === o.priorityString).length;
```

もし、Priorityクラスそのもを使うべきと判断した場合

```JavaScript:Migration3.js
//Migration 3
class Order() {
  constructor(data) {
    this.priority = data.priority;
  }
  get priority() {return this._priority;}
  get priorityString() {return this._priority.toString;}
  set priority(aString) {this._priority = new Priority(aString);}
}

class Priority {
  constructor(value) {
    // コンストラクタの調整
    if (value instanceof Priority) return value;
    this._value = value;
  }
  toString() {return this._value;}
}

// 呼び出し
highPriorityCount = orders.filter(o => "high" === o.priority.toString()|| "rush" === o.priority.toString()).length;
```

Priorityクラスに優先度の値をバリデーションするロジックと比較するロジックを追加した場合の例  
`equals`メソッドを用意することで、変更不可であることを保証するようにした。  

```JavaScript:Migration3.js
//Migration 3
class Order() {
  constructor(data) {
    this.priority = data.priority;
  }
  get priority() {return this._priority;}
  get priorityString() {return this._priority.toString;}
  set priority(aString) {this._priority = new Priority(aString);}
}

class Priority {
  constructor(value) {
    // コンストラクタの調整
    if (value instanceof Priority) return value;
    if (Priority.legalValues().includes(value)) {
      this._value = value;
    } else {
      throw new Error(`<${value}> is invalid for Priority`);
    }
  }
  toString() {return this._value;}
  get _index() {return Priority.legalValues().findIndex(s => s === this._value);}
  static legalValues() {return ['low', 'normal', 'high', 'rush'];}

  equals(other) {return this._index === order._index;}
  higherThan(other) {return this._index > other._index;}
  lowerThan(other) {return this._index < other._index;}
}

// 呼び出し
highPriorityCount = orders.filter(o => "high" === o.priority.higherThan(new Priority("normal"))).length;
```

## 問い合わせによる一時変数の置き換え
```JavaScript:Before.js
const basePrice = this._quantity * this._itemPrice;

if (basePrice > 1000) {
  return basePrice * 0.95;
} else {
  return basePrice * 0.98;
}
```

```JavaScript:After.js
get basePrice() {this._quantity * this._itemPrice;}

if (this.basePrice > 1000) {
  return this.basePrice * 0.95;
} else {
  return this.basePrice * 0.98;
}
```

- 一時変数の目的は、あるコードを保持しておき、それを後ろのほうで参照できるようにすること。
- そういったものは関数にすることで、たとえば、計算ロジックが重複するのを防げたりできる。
- また、リファクタリング途中（たとえば関数の分解中）であっても、容易に抽出しやすい。

## クラスの抽出
```JavaScript:Before.js
class Person {
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber() {return this._officeNumber;}
}
```

```JavaScript:After.js
class Person {
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber() {return this._telephonNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number() {return this._number;}
}
```

- クラスはカリカリの抽象であるべきで、少数の明確な債務だけを受け持つ
- しかし、現実は、クラスは大きくなり、債務が成長して子を産んで手に負えないほど複雑になります。
- そういった場合はクラスを抽出する。データとメソッドの一部を纏めてクラスにできるかどうかを考える。
- 他にも、データの一部が同時に変更されたり、互いに強く依存したりしている関係があれば、それも目安に抽出できないか考える。

## クラスのインライン化
```JavaScript:Before.js
class Person {
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber() {return this._telephoneNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number() {return this._number;}
}
```

```JavaScript:After.js
class Person {
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber() {return this._officeNumber;}
}
```

- 2つのクラスをリファクタリングして特性の配置を換えたいときに使用する。
- あえてクラスのインライン化をした後にクラスの抽出をしたほうが良いケースもある。

## 委譲の隠蔽
```JavaScript:Before.js
manager = aPerson.department.manager;
```

```JavaScript:After.js
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}
}
```

- わかり辛いので、サンプルを先に書く

```JavaScript:Migration1.js
//Migration 1
class Person {
  constructor(name) {
    this._name = name;
  }
  get name() {return this._name;}
  get department() {return this._department;}
  set department(arg) {this._department = arg;}
}

class Department {
  get chargeCode() {return this._chargeCode;}
  set chargeCode(arg) {this._chargeCode = arg;}
  get manager() {return this._manager;}
  set manager(arg) {this._manager = arg;}
}

// 呼び出し
aPerson = new Person(new Department);
manager = aPerson.department.manager;
```

このままでは、Departmentクラスの振る舞いと部署が上司を取得する債務をもっていることが呼び出し側から丸見えである。
Departmentクラスを呼び出し側から隠蔽することで、結合度を下げることができる。

```JavaScript:Migration2.js
//Migration 2
class Person {
  constructor(name) {
    this._name = name;
  }
  get name() {return this._name;}
  get department() {return this._department;}
  set department(arg) {this._department = arg;}
  get manager() {return this._department.manager;}
}

class Department {
  get chargeCode() {return this._chargeCode;}
  set chargeCode(arg) {this._chargeCode = arg;}
  get manager() {return this._manager;}
  set manager(arg) {this._manager = arg;}
}

// 呼び出し
aPerson = new Person(new Department);
manager = aPerson.manager;
```

## 仲介人の除去
```JavaScript:Before.js
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}
}
```

```JavaScript:After.js
manager = aPerson.department.manager;
```

- 委譲の隠蔽をしすぎると、単純な委譲メソッドだらけになる。
- それではただの仲介人になってしまう・・・ので、適宜コードは調整するべし！

## アルゴリズムの置き換え
```JavaScript:Before.js
function foundPerson(people) {
  for(let i = 0; i < people.length; i++) {
    if (people[i] === "Don") {
      return "Don";
    }
    if (people[i] === "John") {
      return "John";
    }
    if (people[i] === "Kent") {
      return "Kent";
    }
  }
  return "";
}
```

```JavaScript:After.js
function foundPerson(people) {
  const candidates = ["Don", "John", "Kent"];
  return people.find(p => candidates.includes(p)) || '';
}
```

- 何かをするために、よりわかりやすい方法が見つかったら、複雑な方法はそれを置き換えるべき。
