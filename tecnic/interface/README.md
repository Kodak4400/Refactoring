## メソッドの引き下げ
```JavaScript:Before.js
class Employee { ... }

class Salesman extends Employee {
  get name() { ... }
}

class Engineer extends Employee {
  get name() { ... }
}
```

```JavaScript:After.js
class Employee {
  get name() { ... }
}

class Salesman extends Employee { ... }
class Engineer extends Employee { ... }
```

- 重複排除に有効な手段の1つ
- 2つ以上のメソッドが同じ内容を持ち、コピー＆ペーストが行われたことが暗示できる場合は要注意！

## フィールドの引き上げ
```TypeScript:Before.js
class Employee { ... } 

class Salesman extends Employee {
  private name(): string { ... }
}

class Engineer extends Employee {
  private name(): string { ... }
}
```

```TypeScript:After.js
class Employee {
  protected name(): string { ... }
}

class Salesman extends Employee { ... }
class Engineer extends Employee { ... }
```

- サブクラスが別々に開発されたり、リファクタリングによって統合されたりした場合、特性が重複することがある
- とくにフィールドは重複しやすい
- フィールドの処理が、似ているようで似ていない場合もあるので、それらのフィールドを調べて、どう使われているかを理解すること

## コンストラクター本体の引き上げ
```JavaScript:Before.js
class Party { ... } 

class Employee extends Party {
  cnstructor(name, id, monthlyCost) {
    super();
    this._id = id;
    this._name = name;
    this._monthlyCost = monthlyCost;
  }
}
```

```JavaScript:After.js
class Party {
  constructor(name) {
    this._name = name;
  }
}

class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super(name);
    this._id = id;
    this._monthlyCost = monthlyCost;
  }
}
```

- コンストラクターは「くせ者」できることが限られる
- 以下のようなケースは、「ファクトリ関数によるコンストラクターの置き換え」を使用する！
```JavaScript:Migration1.js
//Migration 1
class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
    if (this.isPrivileged) this.assignCar();
  }
  get isPrivileged() {
    return this._grade > 4;
  }
}
```

```JavaScript:Migration2.js
//Migration 2
class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
    this.finishConstructiono();
  }
  finishConstruction() {
    if (this.isPrivileged) this.assignCar();
  }
  get isPrivileged() {
    return this._grade > 4;
  }
}
```

```JavaScript:Migration3.js
//Migration 3
class Employee {
  finishConstruction() {
    if (this.isPrivileged) this.assignCar();
  }
}

class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
    this.finishConstructiono();
  }
  get isPrivileged() {
    return this._grade > 4;
  }
}
```

## メソッドの押し下げ
```JavaScript:Before.js
class Employee {
  get quota { ... }
}

class Engineer extends Employee { ... }
class Salesman extends Employee { ... }
```

```JavaScript:After.js
class Employee { ... }
class Engineer extends Employee { ... }
class Salesman extends Employee {
  get quota { ... }
}
```

- あるメソッドが1つのサブクラスにだけ関わる処理を行っている場合、そのメソッドをスーパークラスから取り除く
- そうすることで、クラス構造が明確になる

## フィールドの押し下げ
```TypeScript:Before.js
class Employee {
  protected quota: string;
}

class Engineer extends Employee { ... }
class Salesman extends Employee { ... }
```

```TypeScript:After.js
class Employee { ... }
class Engineer extends Employee { ... }
class Salesman extends Employee {
  private quota: string;
}
```

- あるフィールドが1つのサブクラスにだけ関わる処理を行っている場合、そのフィールドをスーパークラスから取り除く
- そうすることで、クラス構造が明確になる

## サブクラスによるタイプコードの置き換え
```JavaScript:Before.js
function createEmployee(name, type) {
  return new Employee(name, type);
}
```

```JavaScript:After.js
function createEmployee(name, type) {
  switch(type) {
    case "engineer": return new Engineer(name);
    case "salesman": return new Salesman(name);
    case "manager": return new Manager(name);
  }
}
```

- 類似しているが少し違うものを表現した場合がある。
- これに対処する第一の手段は、なんらかのタイプコード用フィールドです。
- 多くの場合、タイプコード用フィールドを使うことで、なんとかなるが、サブクラスも加えるとより使いやすくなる

#### 関節的な継承の使用
```JavaScript:Migration1.js
//Migration 1
class Employee {
  constractor(name, type) {
    this.validateType(type);
    this._name = name;
    this._type = type;
  }
  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
  }
  get type() { return this._type; }
  set type(arg) {this._type = arg; }

  get capitalizaedType() {
    return this._type.charAt(0).toUpperCase() + this._type.substr(1).toLowerCase();
  }
  toString() {
    return `${this._name} (${this.capitalizedType})`;
  }
}
```

```JavaScript:Migration2.js
//Migration 2
class EmployeeType {
  constractor(aString) {
    this._value = aString;
  }
  toString() {
    return this._value;
  }
}
class Employee {
  constractor(name, type) {
    this.validateType(type);
    this._name = name;
    this._type = type;
  }
  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
  }
  get typeString() { return this._type; }
  set type(arg) {this._type = arg; }

  get capitalizaedType() {
    return this.typeString.charAt(0).toUpperCase() + this.typeString.substr(1).toLowerCase();
  }
  toString() {
    return `${this._name} (${this.capitalizedType})`;
  }
}
```

```JavaScript:Migration3.js
//Migration 3
class EmployeeType {
  constractor(aString) {
    this._value = aString;
  }
  toString() {
    return this._value;
  }
}
class Engineer extends EmployeeType {
  toString() {return "engineer";}
}
class Manager extends EmployeeType {
  toString() {return "manager";}
}
class Salesman extends EmployeeType {
  toString() {return "salesman";}
}

class Employee {
  constractor(name, type) {
    this.validateType(type);
    this._name = name;
    this._type = type;
  }
  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
  }
  get typeString() { return this._type; }
  set type(arg) {this._type = Employee.createEmployeeType(arg); }

  static createEmployeeType(aString) {
    switch(aString) {
      case: "engineer": return new Engineer();
      case: "manager": return new Manager();
      case: "salesman": return new Salesman();
      default throw new Error(`従業員のタイプコードが不正: ${aString}`)
    }
  }

  get capitalizaedType() {
    return this.typeString.charAt(0).toUpperCase() + this.typeString.substr(1).toLowerCase();
  }
  toString() {
    return `${this._name} (${this.capitalizedType})`;
  }
}
```

## サブクラスの削除
```JavaScript:Before.js
class Person {
  get genderCode() { return "X"; }
}
class Male extends Person {
  get genderCode() { return "M"; }
}
class Female extends Person {
  get genderCode() { return "F"; }
}
```

```JavaScript:After.js
class Person {
  get genderCode() { return this._genderCode; }
}
```

- サブクラスは便利だが、サブクラスをサポートするバリエーションが別の場所に移動されたり、まるごと削除されたりして、サブクラスの存在価値がなくなってしまうことがある
- また、将来必要になることを予想して追加したサブクラスが結局使われなかったり、別のやり方で実現されたりもする
- そんなときは、サブクラスを取り除いてスーパークラスのフィールドに置き換えるのが最善。


## スーパークラスの抽出
```JavaScript:Before.js
class Department {
  get totalAnnualCost() { ... }
  get name() { ... }
  get headCount() { ... }
}
class Employee {
  get annualCost() { ... }
  get name() { ... }
  get id() { ... }
}
```

```JavaScript:After.js
class Party {
  get name() { ... }
  get annualCost() { ... }
}
class Department extends Party {
  get annualCost() { ... }
  get headCount() { ... }
}
class Employee extends Party {
  get annualCost() { ... }
  get id() { ... }
}
```

- 2つのクラスが同じようなことをしていたら、継承の基本的なメカニズムを使って、類似点をスーパークラスにまとめてしまう

## クラス階層の平坦化
```JavaScript:Before.js
class Employee { ... }
class Salesman extends Employee { ... }
```

```JavaScript:After.js
class Employee { ... }
```

- クラス階層が進化するほど、あるクラスと親クラスの間に、いつの間にか、別クラスに分けるほどの差がなくなっていることがある
- そうした場合は、それらを1つにマージする

## 委譲によるサブクラスの置き換え
```JavaScript:Before.js
class Order {
  get daysToShip() {
    return this._warehouse.daysToShip;
  }
}
class PriorityOrder extends Order {
  get daysToShip() {
    return this._priorityPlan.daysToShip;
  }
}
```

```JavaScript:After.js
class Order {
  get daysToShip() {
    return (this._priorityDelegate)
      ? this._priorityDelegate.daysToShip
      : this._warehouse.daysToShip;
  }
}
class PriorityOrderDelegate {
  get daysToShip() {
    return this._priorityPlan.daysToShip;
  }
}
```

- 継承には欠点がある。1度しか使えない切り札であること。つまり、バリエーションを持たせたい理由が複数ある場合でも、継承を使えるのは1軸に対してのみ。
- もし年齢と所得水準の2軸で「人」の振る舞いを変えたいのであれば、若者と高齢者をサブクラスにするか、高所得と低所得をサブクラスにするか、どちらか一方を選択するしかない
- また、継承がクラス間に密接な関係を導入することも問題。親クラスに施そうとする変更が、簡単に子クラスを壊してしまう。
- 委譲であれば、これら問題に対処できる。
- 以下サンプルを書く
```JavaScript:Migration1.js
//Migration 1
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
}
class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback');
  }
  // オーバーライド&スーパークラス呼び出し
  get basePrice() {
    return Math.round(super.basePrice + this._extras.premiumFee);
  }
  // スーパークラスにない振る舞い
  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}

// 呼び出し
// booking client
aBooking = new Booking(show, date);
// premium client
aBooking = new PremiumBooking(show, date, extras);
```

```JavaScript:Migration2.js
//Migration 2
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
}
class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback');
  }
  // オーバーライド&スーパークラス呼び出し
  get basePrice() {
    return Math.round(super.basePrice + this._extras.premiumFee);
  }
  // スーパークラスにない振る舞い
  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}

// 呼び出し
function createBooking(show, date) {
  return new Booking(show, date);
}
function createPremiumBooking(show, date, extras) {
  return new PremiumBooking(show, date, extras)
}

// booking client
aBooking = createBooking(show, date);
// premium client
aBooking = createPremiumBooking(show, date, extras);
```

```JavaScript:Migration3.js
//Migration 3
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  _bePremium(extras) {
    this._premiumDelegate = new PremiumBookingDelegate(this, extras);
  }
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
}
class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._show.hasOwnProperty('talkback');
  }
  // オーバーライド&スーパークラス呼び出し
  get basePrice() {
    return Math.round(super.basePrice + this._extras.premiumFee);
  }
  // スーパークラスにない振る舞い
  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}
class PremiumBookingDelegate {
  constractor(hostBooking, extras) {
    this._host = hostBooking;
    this._extras = extras;
  }
}

// 呼び出し
function createBooking(show, date) {
  return new Booking(show, date);
}
function createPremiumBooking(show, date, extras) {
  const result = new PremiumBooking(show, date, extras)
  result._bePremium(extras);
  return result;
}

// booking client
aBooking = createBooking(show, date);
// premium client
aBooking = createPremiumBooking(show, date, extras);
```

```JavaScript:Migration4.js
//Migration 4
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  _bePremium(extras) {
    this._premiumDelegate = new PremiumBookingDelegate(this, extras);
  }
  get hasTalkback() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasTalkback
      : this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
}
class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }
  // オーバーライド&スーパークラス呼び出し
  get basePrice() {
    return Math.round(super.basePrice + this._extras.premiumFee);
  }
  // スーパークラスにない振る舞い
  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}
class PremiumBookingDelegate {
  constractor(hostBooking, extras) {
    this._host = hostBooking;
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._host._show.hasOwnProperty('talkback');
  }
}

// 呼び出し
function createBooking(show, date) {
  return new Booking(show, date);
}
function createPremiumBooking(show, date, extras) {
  const result = new PremiumBooking(show, date, extras)
  result._bePremium(extras);
  return result;
}

// booking client
aBooking = createBooking(show, date);
// premium client
aBooking = createPremiumBooking(show, date, extras);
```

```JavaScript:Migration5.js
//Migration 5
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  _bePremium(extras) {
    this._premiumDelegate = new PremiumBookingDelegate(this, extras);
  }
  get hasTalkback() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasTalkback
      : this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  // 方法1
  get basePrice() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.basePrice
      : this._privateBasePrice;
  }
  get _privateBasePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
  // 方法2
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return (this._premiumDelegate)
      ? this._premiumDelegate.extendBasePrice(result)
      : result;
  }
}
class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }
  // スーパークラスにない振る舞い
  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}
class PremiumBookingDelegate {
  constractor(hostBooking, extras) {
    this._host = hostBooking;
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._host._show.hasOwnProperty('talkback');
  }
  // オーバーライド&スーパークラス呼び出し 方法1
  get basePrice() {
    return Math.round(this._host._privateBasePrice + this._extras.premiumFee);
  }
  // オーバーライド&スーパークラス呼び出し 方法2
  extendBasePrice(base) {
    return Math.round(base + this._extras.premiumFee);
  }

}

// 呼び出し
function createBooking(show, date) {
  return new Booking(show, date);
}
function createPremiumBooking(show, date, extras) {
  const result = new PremiumBooking(show, date, extras)
  result._bePremium(extras);
  return result;
}

// booking client
aBooking = createBooking(show, date);
// premium client
aBooking = createPremiumBooking(show, date, extras);
```

```JavaScript:Migration6.js
//Migration 6
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }
  _bePremium(extras) {
    this._premiumDelegate = new PremiumBookingDelegate(this, extras);
  }
  get hasTalkback() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasTalkback
      : this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }
  // 方法1
  get basePrice() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.basePrice
      : this._privateBasePrice;
  }
  get _privateBasePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }
  // 方法2
  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return (this._premiumDelegate)
      ? this._premiumDelegate.extendBasePrice(result)
      : result;
  }
  get hasDinner() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasDinner
      : undefiend;
  }
}
class PremiumBookingDelegate {
  constractor(hostBooking, extras) {
    this._host = hostBooking;
    this._extras = extras;
  }
  // 単純オーバーライドケース
  get hasTalkback() {
    return this._host._show.hasOwnProperty('talkback');
  }
  // オーバーライド&スーパークラス呼び出し 方法1
  get basePrice() {
    return Math.round(this._host._privateBasePrice + this._extras.premiumFee);
  }
  // オーバーライド&スーパークラス呼び出し 方法2
  extendBasePrice(base) {
    return Math.round(base + this._extras.premiumFee);
  }

}

// 呼び出し
function createBooking(show, date) {
  return new Booking(show, date);
}
function createPremiumBooking(show, date, extras) {
  const result = new Booking(show, date, extras)
  result._bePremium(extras);
  return result;
}

// booking client
aBooking = createBooking(show, date);
// premium client
aBooking = createPremiumBooking(show, date, extras);
```

#### 階層の置き換え

## 委譲によるスーパークラスの置き換え
```JavaScript:Before.js
class List { ... }
class Stack extends List { ... }
```

```JavaScript:After.js
class Stack {
  constructor() {
    this._storage = new List();
  }
}
class List { ... }
```

- スーパークラスの関数がサブクラスで意味をなさないのなら、スーパークラスの機能を継承によって利用すべきではない
- 逆に、スーパークラスのすべての関数がサブクラスで意味をなすと同時に、サブクラスのインスタンスはスーパークラスを使用するすべてのケースで有効であるべき
- 委譲をつかえば、上2つの極端なことは解決できる。「いくつかの関数を使いまわしたいだけの別の存在である」ことが明確にできる！
