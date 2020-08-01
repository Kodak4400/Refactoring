## 問い合わせと更新の分離
```JavaScript:Before.js
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();
  return result;
}
```

```JavaScript:After.js
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

- ある関数が値を返すだけで副作用がない関数は有用。好きなだけ呼び出せるし、移動・テストも容易。
- そのため、関数には副作用を可能な限り減らすこと！副作用があれば、可能な限り分離させる！
- 以下サンプルを書く

```JavaScript:Migration1.js
//Migration 1
function alertForMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      setOffAlarms();
      return 'Don'
    }
    if (p === 'John') {
      setOffAlarms();
      return 'Johon'
    }
  }
  return '';
}

// 呼び出し側
const found = alertForMiscreant(people);
```

```JavaScript:Migration2.js
//Migration 2
function alertForMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      setOffAlarms();
      return 'Don'
    }
    if (p === 'John') {
      setOffAlarms();
      return 'Johon'
    }
  }
  return '';
}

function findMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      return 'Don'
    }
    if (p === 'John') {
      return 'Johon'
    }
  }
  return '';
}

// 呼び出し側
const found = alertForMiscreant(people);
```

```JavaScript:Migration3.js
//Migration 3
function alertForMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      setOffAlarms();
      return;
    }
    if (p === 'John') {
      setOffAlarms();
      return;
    }
  }
  return;
}

function findMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      return 'Don'
    }
    if (p === 'John') {
      return 'Johon'
    }
  }
  return '';
}

// 呼び出し側
const found = findMiscreant(people);
alertForMiscreant(people);
```

```JavaScript:Migration4.js
//Migration 4
function alertForMiscreant (people) {
  if ( findMiscreant(people) !== "") setOffAlarms();
  return;
}

function findMiscreant (people) {
  for (const p of people) {
    if (p === 'Don') {
      return 'Don'
    }
    if (p === 'John') {
      return 'Johon'
    }
  }
  return '';
}

// 呼び出し側
const found = alertForMiscreant(people);
```

## パラメーターによる関数の統合
```JavaScript:Before.js
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}
```

```JavaScript:After.js
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

- リテラル値が異なるだけの非常に似たロジックを持つ2つの関数があるなら、異なる値を渡すためのパラメーターを持った1つの関数を用いることで、重複を排除できる

## フラグパラメーターの削除
```JavaScript:Before.js
function setDimension(name, value) {
  if(name === "height") {
    this._height = value;
    return;
  }
  if (name === "width") {
    this._width = value;
    return;
  }
}
```

```JavaScript:After.js
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```

- フラグ引数は、呼び出し元が呼び出し先に対してどのロジックを実行してほしいかを支持するためのパラメーターとして使われる。
- 1つの関数に2つの機能を持ち込むのは良くない。（1つの関数で多くのことをやろうとする兆候でもある！）
- また、フラグ引数は利用可能な関数の呼び出しの多様性を隠してしまう。

## オブジェクトそのものの受け渡し
```JavaScript:Before.js
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))
```

```JavaScript:After.js
if (aPlan.withinRange(aRoom.daysTempRange))
```

- 1つのレコードから数個の値を取り出して関数に渡しているコードを見ると、それらの値の元のレコードに置き換えて、必要とする値を関数本体で取り出すように変更したくなる
- 将来、渡したレコードからより多くのデータを取り出すようになっても、変更がすくなくてすみます。
- また、レコードの1部を引数として呼び出す関数が多くなると、レコードの1部を処理するロジックが重複しがちになる

## 問い合わせパラメーターの置き換え
```JavaScript:Before.js
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // 休暇日数計算...
}
```

```JavaScript:After.js
availableVacation(anEmployee);

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;
}
```

- 関数のパラメーターリストは、関数の可変部分を縮約して、関数の振る舞いの主なバリエーションを表すようにすべき。
- コード内でのステートメントの重複と同様に、パラメーターリスト内の重複も避けるべき
- パラメーターを取り除くと、パラメーターの値を決める債務は移動する。つまり、パラメーターがない場合は、債務が関数側に移動する。
- そのため、パラメーターを取り除く場合は、債務が関数側に相応しいかを考える。

## パラメーターによる問い合わせの置き換え
```JavaScript:Before.js
targetTemperature(aPlan);

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  // 後続ロジック...
}
```

```JavaScript:After.js
targetTemperature(aPlan, thermostat.currentTemperature);

function targetTemperature(aPlan, currentTemperature) {
  // 後続ロジック...
}
```

- 「問い合わせパラメーターの置き換え」の逆パターン
- つまり、債務が関数側に相応しくない場合は、パラメーターを増やす。

## setterの削除
```JavaScript:Before.js
class Person {
  get name() { ... }
  set name(aString) { ... }
}
```

```JavaScript:After.js
class Person {
  get name() { ... }
}
```

- setterが用意されているということは、フィールドが変更される可能性があることを意味する
- オブジェクトを生成したあとで、フィールドを変更したくない場合、seeterは用意しない
- フィールドはコンストラクターでのみ設定され、変更させないという意図が明確になり、フィールドが変更される可能性を排除できる
- 不要なsetterを用意してしまうケースは以下の2つ
- フィールドを操作するときに、コンストラクターの内部も含めて常にアクセサーを使用しているケース
- 単純なコンストラクター呼び出しではなく、クライアントが生成するスクリプトを使ってオブジェクトを生成しているケース

## ファクトリ関数によるコンストラクターの置き換え
```JavaScript:Before.js
leadEngineer = new Employee(document.leadEngineer, 'E');
```

```JavaScript:After.js
leadEngineer = createdEngineer(document.leadEngineer);
```

- コンストラクターには、通常の関数にはない厄介な以下制限がある。が、ファクトリ関数にはないので、場合によってはファクトリ関数を使うほうが良いケースもある
- 必ずインスタンスを返す
- 名前がデフォルトで決まっている
- `new`演算子が必要

## コマンドによる関数の置き換え
```JavaScript:Before.js
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let helthLevel = 0;
  // ...
}
```

```JavaScript:After.js
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }
  execute() {
    this._result = 0;
    this._healthLevel = 0;
    ...
  }
}
```

- 関数自身をオブジェクトとして、カプセル化することが有用な場合もある
- そのようなオブジェクトを「コマンドオブジェクト」or「コマンド」という
- 「コマンド」は、単なる関数の仕組みと比べて、関数の制御と表現において高い柔軟性を持つ
- undoのような補完的な操作を提供できるし、より複雑なライフサイクルに対応したパラメーターを組み立てるためのメソッドも提供できるし、継承・フックを使ったカスタマイズも可能

## 関数によるコマンドの置き換え
```JavaScript:Before.js
class ChargeCalculator {
  constructor (customer, usage) {
    this._customer = customer;
    this._usage = usage;
  }
  execute() {
    return this._customer.rate = this._usage;
  }
}
```

```JavaScript:After.js
function charge(customer, usage) {
  return customer.rate = usage;
}
```

- 「コマンドによる関数の置き換え」の逆パターン
- 関数が複雑でない場合、「コマンド」は骨折り損。もとに戻すべき！

