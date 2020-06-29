## 関数の抽出
```JavaScript:Before.js
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  // 明細の印字(print details)
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
```

```JavaScript:After.js
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  printDetails(invoice, outstanding);
}

function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
```

- きわめて頻繁に行われる。  
- 「関数は一画面に収まること」というガイドラインもあるらしい・・・  
- 「2回以上使われるコードはそれ自体を関数にすべき」と、再利用に基づいている。
- 適切な名前をつけることで、関数の中身を気にする必要がなくなる。
- 関数を抽出していると、戻り値を複数返す必要があるケースも出てくる。そんな場合は、`複数関数を用意する`or`一時変数に手を入れる`を選択する。

## 関数のインライン化
```JavaScript:Before.js
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFivelateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
```

```JavaScript:After.js
function getRating(driver) {
  return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

- コードの本体と名前が同じくらい明白なものはインライン化する。
- うまく分割できていない関数は、一度インライン化して戻したあとに、再度、関数を抽出する。

## 変数の抽出
```JavaScript:Before.js
return order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100);
```

```JavaScript:After.js
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(order.quantity * order.itemPrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

- 式は、きわめて読みにくくなることがあるため、ローカル変数を活用して式を分割する。
- コード内の式に名前を付けたい時は抽出を検討する。
- クラス全体で通用する場合は、変数ではなくメソッドとして抽出することも検討する。

```JavaScript:After.js
class Order {
  constructor(aRecord) {
    this._data = aRecord;
  }
  get quantity() {return this._data.quantity;}
  get itemPrice() {return this._data.itemPrice;}

  get price() {
    return this.basePrice - this.quantityDiscount + this.shipping;
  }
  get basePrice() {return this.quantity * this.itemPrice;}
  get quantityDiscount() {return Math.max(0, this.quantity - 500) * this.itemPrice * 0.05}
  get shipping() {return Math.min(this.quantity * this.itemPrice * 0.1, 100);}
}
```

## 変数のインライン化
```JavaScript:Before.js
let basePrice = anOrder.basePrice;
return (basePrice > 1000);
```

```JavaScript:After.js
return anOrder.basePrice > 1000;
```

- 名前が式以上のことを語らない場合、変数が周辺のリファクタリングの邪魔になる場合はインライン化を検討する。

## 関数宣言の変更
```JavaScript:Before.js
function circum(radius) {...}
```

```JavaScript:After.js
function circumference(radius) {...}
```

- 関数はソフトウェア・システムにおけるつなぎ目の役割を持つ、そのため名前が少しでもわかり辛い場合は変更すべき！
- 関数名の変更方法は、`単純に名前を変更する方法`と`移行的に変更する方法`の2種類ある。
- `移行的に変更する方法`は、関数名変更後の関数を仮実装して移行する。もとの実装を壊さないように移行できる。

### 移行的に変更する方法
```JavaScript:Migration1.js
//Migration 1
// 呼び出し
const newEnglanders = someCustomer.filter(c => inNewEngland(c));

function inNewEngland(aCustomer) {
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

```JavaScript:Migration2.js
//Migration 2
// 呼び出し
const newEnglanders = someCustomer.filter(c => inNewEngland(c));

function inNewEngland(aCustomer) {
  const stateCode = aCustomer.address.state;
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

```JavaScript:Migration3.js
//Migration 3
// 呼び出し
const newEnglanders = someCustomer.filter(c => inNewEngland(c));

function inNewEngland(aCustomer) {
  const stateCode = aCustomer.address.state;
  return xxNEWinNewEngland(stateCode);
}

function xxNEWinNewEngland(stateCode) {
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

```JavaScript:Migration4.js
//Migration 4
// 呼び出し
const newEnglanders = someCustomer.filter(c => inNewEngland(c));

function inNewEngland(aCustomer) {
  return xxNEWinNewEngland(aCustomer.address.state);
}

function xxNEWinNewEngland(stateCode) {
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

```JavaScript:Migration5.js
//Migration 5
// 呼び出し
const newEnglanders = someCustomer.filter(c => xxNEWinNewEngland(c.address.state));

function inNewEngland(aCustomer) {
  return xxNEWinNewEngland(aCustomer.address.state);
}

function xxNEWinNewEngland(stateCode) {
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

```JavaScript:Migration6.js
//Migration 6
// 呼び出し
const newEnglanders = someCustomer.filter(c => inNewEngland(c.address.state));

function inNewEngland(stateCode) {
  return ["MA", "CA", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```

## 変数のカプセル化
```JavaScript:Before.js
let defaultOwner = {firstName: "Martin", lastName: "Fowler"};
```

```JavaScript:After.js
let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
export function defaultOwner() {return defaultOwnerData;}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```

- 広い範囲で利用されるデータは、カプセル化して、変数へのアクセスを関数経由にするのが良い。
- データの移動も`export`を使えば容易。
- `getter`でコピーを返したい場合は以下の方法を使う。
```JavaScript:After.js
let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
export function defaultOwner() {return Object.assigin({}, defaultOwnerData);}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```
- コピーは使うと、それがコピーだと認識されない場合がある。その場合は以下のようにすると良い。
```JavaScript:After.js
let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
export function defaultOwner() {return new Person(defaultOwnerData);}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}

class Person {
  constructor(data) {
    this._lastName = data.lastName;
    this._firstName = data.firstName;
  }
  get lastName() {return this._lastName;}
  get firstName() {return this._firstName;}
}
```

## 変数名の変更
```JavaScript:Before.js
let a = height * width;
```

```JavaScript:After.js
let area = height * width;
```

- 良い名前を付けることは、明快なプログラムを書くための核心。
- JavaScriptのような動的型付け言語は「型」を入れると良い。

## パラメータオブジェクトの導入
```JavaScript:Before.js
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}
```

```JavaScript:After.js
function amountInvoiced(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```

- セットになるデータは、構造化する。
- イメージを付けるためにサンプルを置く。
```JavaScript:Migration1.js
//Migration 1
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, min, max) {
  return station.redings.filter(r => r.temp < min || r.temp > max);
}

// 呼び出し
alerts = redingsOutsideRange(station,
                              operationPlan.temperatureFloor,
                              operatingPlan.temperatureCeiling);
```

```JavaScript:Migration2.js
//Migration 2
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, min, max) {
  return station.redings.filter(r => r.temp < min || r.temp > max);
}

// 呼び出し
alerts = redingsOutsideRange(station,
                              operationPlan.temperatureFloor,
                              operatingPlan.temperatureCeiling);

class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min;}
  get max() { return this._data.max;}
}
```

```JavaScript:Migration3.js
//Migration 3
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, min, max, range) {
  return station.redings.filter(r => r.temp < min || r.temp > max);
}

// 呼び出し
alerts = redingsOutsideRange(station,
                              operationPlan.temperatureFloor,
                              operatingPlan.temperatureCeiling,
                              null);

class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min;}
  get max() { return this._data.max;}
}
```

```JavaScript:Migration4.js
//Migration 4
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, min, max, range) {
  return station.redings.filter(r => r.temp < min || r.temp > max);
}

// 呼び出し
const range = new NumberRange(operationPlan.temperatureFloor, operationPlan.tempertureCeiling);
alerts = redingsOutsideRange(station,
                              operationPlan.temperatureFloor,
                              operatingPlan.temperatureCeiling,
                              range);

class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min;}
  get max() { return this._data.max;}
}
```

```JavaScript:Migration5.js
//Migration 5
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, range) {
  return station.redings.filter(r => r.temp < range.min || r.temp > range.max);
}

// 呼び出し
const range = new NumberRange(operationPlan.temperatureFloor, operationPlan.tempertureCeiling);
alerts = redingsOutsideRange(station, range);

class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min;}
  get max() { return this._data.max;}
}
```

```JavaScript:Migration6.js
//Migration 6
const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time "2016-11-10 09:10"},
                    {temp: 53, time "2016-11-10 09:20"},
                    {temp: 58, time "2016-11-10 09:30"},
                    {temp: 53, time "2016-11-10 09:40"},
                    {temp: 51, time "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, range) {
  return station.redings.filter(r => !range.contains(r.temp););
}

// 呼び出し
const range = new NumberRange(operationPlan.temperatureFloor, operationPlan.tempertureCeiling);
alerts = redingsOutsideRange(station, range);

class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min;}
  get max() { return this._data.max;}
  contains(arg) {return (arg >= this.min && arg <= this.max);}
}
```

## 関数群のクラスへの集約
```JavaScript:Before.js
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
```

```JavaScript:After.js
class {
  base() {...}
  taxableCharge() {...}
  calculateBaseCharge() {...}
}
```

- 共通のデータに対して互いに関わりの深い処理を行う一郡の関数があれば、クラスを定義する。
- イメージを付けるためにサンプルを置く。
```JavaScript:Migration1.js
//Migration 2
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const aReading = acquireReading();
const base = (baseRate(aReading.month, aReading.year) * aReading.quantity);
```

```JavaScript:Migration2.js
//Migration 2
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const aReading = acquireReading();
const baseChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

```JavaScript:Migration3.js
//Migration 3
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = new Reading(rawReading)
const baseChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}

class Reading {
  constructor(data) {
    this._customer = data.customer;
    this._quantity = data.quantity;
    this._month = data.month;
    this._year = data.year;
  }
  get customer() {return this._customer;}
  get quantity() {return this._quantity;}
  get month() {return this._month;}
  get year() {return this._year;}
}
```

```JavaScript:Migration4.js
//Migration 4
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = new Reading(rawReading)
const baseChargeAmount = aReading.calculateBaseCharge;

class Reading {
  constructor(data) {
    this._customer = data.customer;
    this._quantity = data.quantity;
    this._month = data.month;
    this._year = data.year;
  }
  get customer() {return this._customer;}
  get quantity() {return this._quantity;}
  get month() {return this._month;}
  get year() {return this._year;}
  get calculateBaseCharge() {
    return baseRate(this._month, this._year) * this._quantity;
  }
}
```

```JavaScript:Migration5.js
//Migration 5
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = new Reading(rawReading)
const baseChargeAmount = aReading.baseCharge;

class Reading {
  constructor(data) {
    this._customer = data.customer;
    this._quantity = data.quantity;
    this._month = data.month;
    this._year = data.year;
  }
  get customer() {return this._customer;}
  get quantity() {return this._quantity;}
  get month() {return this._month;}
  get year() {return this._year;}
  get baseCharge() {
    return baseRate(this._month, this._year) * aReading._quantity;
  }
}
```

## 関数群の変換への集約
```JavaScript:Before.js
function base(aReading) {...}
function taxableCharge(aReading) {...}
```

```JavaScript:After.js
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading;
}
```

- ソフトウェアの多くはプログラムにデータを投入して、そこからさまざまな計算が行われ派生していく。
- こうした派生値は纏めたほうが良い。
- クラスを使うことで、元のデータを残すことができ、データの不整合を防ぐことができる。
- イメージを付けるためにサンプルを置く。
```JavaScript:Migration1.js
//Migration 2
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const aReading = acquireReading();
const base = (baseRate(aReading.month, aReading.year) * aReading.quantity);
```

```JavaScript:Migration2.js
//Migration 2
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const aReading = acquireReading();
const baseChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

```JavaScript:Migration3.js
//Migration 3
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = new Reading(rawReading)
const baseChargeAmount = calculateBaseCharge(aReading);

function enrichReading(original) {
  // lodash.cloneDeepを使用する.
  const result = _.cloneDeep(original);
  return result;
}

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

```JavaScript:Migration4.js
//Migration 4
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = enrichReading(rawReading)
const baseChargeAmount = aReading.calculateBaseCharge;

function enrichReading(original) {
  // lodash.cloneDeepを使用する.
  const result = _.cloneDeep(original);
  return result;
}

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

```JavaScript:Migration5.js
//Migration 5
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 呼び出し
const rawReading = acquireReading();
const aReading = enrichReading(rawReading)
const baseCharge = aReading.baseCharge;

function enrichReading(original) {
  // lodash.cloneDeepを使用する.
  const result = _.cloneDeep(original);
  result.baseCharge = calculateBaseCharge(result);
  return result;
}

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

## フェーズの分離
```JavaScript:Before.js
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;
```

```JavaScript:After.js
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const alues = aString.split(/\s+/);
  return ({
    productID: values[0].split("-")[1],
    quantity: parseInt(values[1]),
  });
}
function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

- 1つのコードが異なる2つの処理を行っている場合、別々のモジュールに分ける。
- こうすることで、変更が入った時、トピックごとに分けて対処できる。
