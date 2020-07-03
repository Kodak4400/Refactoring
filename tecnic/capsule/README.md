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
