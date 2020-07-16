## 変数の分離
```JavaScript:Before.js
let temp = 2 * (height + width);
consolelog(temp);
temp = height * width;
console.log(temp);
```

```JavaScript:After.js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area)
```

- 責務を持つ変数は、それぞれの債務を持つ1つの変数で置き換えるべき
- 1つの変数を2つのことに使うと、コードの読み手が非常に混乱する
- JSであれば、`const`を使うべし！

## フィールド名の変更
```JavaScript:Before.js
class Organization {
  get name {...}
}
```

```JavaScript:After.js
class Organization {
  get title {...}
}
```

- フレッド・ブルックスの名言「フローチャートを見せてくれても、テーブルを隠されたら煙に巻かれたままだろう。テーブルを見せてくれれば通常フローチャートは要らない。それだけですぐわかる」
- データ構造は何が起こっているのかを理解するための鍵

### データ構造が広範囲で使われていると名前の変更は大変。以下にサンプルを書く。
```JavaScript:Migration1.js
//Migration 1
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name() {return this._name;}
  set name(aString) {this._name = aString;}
  get country() {return this._country;}
  set country(aCountryCode) {this._country = aCountryCode;}
}
```

```JavaScript:Migration2.js
//Migration 2
class Organization {
  constructor(data) {
    this._title = data.name;
    this._country = data.country;
  }
  get name() {return this._title;}
  set name(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode) {this._country = aCountryCode;}
}
```

```JavaScript:Migration3.js
//Migration 3
class Organization {
  constructor(data) {
    this._title = (data.title !== undefined) ? data.title : data.name;
    this._country = data.country;
  }
  get name() {return this._title;}
  set name(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode) {this._country = aCountryCode;}
}

// 呼び出し側
const organaization = new Organization({title: "Acme Gooseberries", contry: "GB"});
```

```JavaScript:Migration4.js
//Migration 4
class Organization {
  constructor(data) {
    this._title = data.title;
    this._country = data.country;
  }
  get title() {return this._title;}
  set title(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode) {this._country = aCountryCode;}
}

// 呼び出し側
const organaization = new Organization({title: "Acme Gooseberries", contry: "GB"});
```

## 問い合わせによる導出変数の置き換え
```JavaScript:Before.js
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
  const old = this._discount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber;
}
```

```JavaScript:After.js
get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```

- 変更可能なデータは、ソフトウェアにおける問題の発生源となる。
- そのため、変更可能なデータの影響範囲は小さくすべき!
- 簡単に計算できる変数はすべて削除し、適切な名前を設定することで対処できることが多い

## 参照から値への変更
```JavaScript:Before.js
class Product {
  applyDiscount(arg) {this._price.amount -= arg;}
}
```

```JavaScript:After.js
class Product {
  applyDiscount(arg) {
    this._price = new Mony(this._price.amount - arg, this._price.currency);
  }
}
```

- オブジェクトやデータ構造を入れ子にする場合、内部オブジェクトは参照または値として扱うことができる。
- 参照として扱う場合、内部プロパティを更新し、同じ内部オブジェクトを保持します。
- 値として扱う場合、期待するプロパティを持つ新しい内部オブジェクトにまるごと置き換える。
- あるフィールドを値として置き換える場合、内部オブジェクトのクラスは「値オブジェクト」に変更できる。そのほうが仕様の把握が容易で、変更不可なので良い。
- サンプルを書く

```JavaScript:Migration1.js
//Migration 1
class Person {
  constructor() {
    this._telephoneNumber = new TelephoneNumber();
  }
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  set officeAreaCode(arg) {this._telephoneNumber.areaCode = arg;}
  get officeNumber() {return this._telephoneNumber.number;}
  set officeNumber(arg) {this._telephoneNumber.number = arg;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  set areaCode(arg) {this._areaCode = arg;}
  get number() {return this._number;}
  set number(arg) {this._number = arg;}
}
```

```JavaScript:Migration2.js
//Migration 2
class Person {
  constructor() {
    this._telephoneNumber = new TelephoneNumber();
  }
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  set officeAreaCode(arg) {this._telephoneNumber = new TelephoneNumber(arg, this.officeNumber);}
  get officeNumber() {return this._telephoneNumber.number;}
  set officeNumber(arg) {this._telephoneNumber = new TelephoneNumber(this.officeAreaCode, arg);}
}
class TelephoneNumber {
  constractor(areaCode, number) {
    this._areaCode = areaCode;
    this._number = number;
  }
  get areaCode() {return this._areaCode;}
  set areaCode(arg) {this._areaCode = arg;}
  get number() {return this._number;}
  set number(arg) {this._number = arg;}
}
```

こうすることで電話番号が変更不可となり、値オブジェクトに置き換えが成功した。
以下のようなテストを書くことも重要

```JavaScript:Test.js
class TelephonNumber {
  equals(other) {
    if (!(other instanceof TelephoneNumber)) return false;
    return this.areaCode === other.areaCode &&
      this.number === other.number;
  }
}

it('telephone equals', function() {
  assert(    new TelephoneNumber("321", "555-0142)
    .equals(new TelephoneNumber("321", "555-0142)));
})
```

## 値から参照への変更
```JavaScript:Before.js
let customer = new Customer(customerData);
```

```JavaScript:After.js
let customer = customerRepository.get(customerData.id);
```

- 複数のレコードが意味的に同じデータ構造にリンクする場合、参照・値どちらでも使うことができます。
- しかし、意味的に同じデータ構造にリンクするというのは、複数の物理コピーをもたせることになります。
- そういった場合は参照に変更して、1つの実体に対して存在するオブジェクトを唯一のものにする。

