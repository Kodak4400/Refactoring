## 関数の移動
```JavaScript:Before.js
class Account {
  get overdraftCharge() {...}
}
```

```JavaScript:After.js
class AccountType {
  get overdraftCharge() {...}
}
```

- コードを書いていく中で、要素をどのようにグループ化すべきか理解してくる。
- そんなときは要素を移動させることを検討すべし。
- ただし移動は困難を伴う。
- 以下サンプルを書く。

### 入れ子関数をトップレベルに移動する。
```JavaScript:Migration1.js
//Migration 1
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;
  return {
    time: totalTime,
    distance: totalDistance,
    pace: pace
  }
}

function calculateDistance() {
  let result = 0;
  for (let i = 1; i < points.length; i++) {
    result += distance(points[i-1], points[i]);
  }
  return result;
}

function distance(p1, p2) {...}
function radians(degrees) {...}
function calculateTime() {...}
```

```JavaScript:Migration2.js
//Migration 2
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;
  return {
    time: totalTime,
    distance: totalDistance,
    pace: pace
  }

  function calculateDistance() {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i-1], points[i]);
    }
    return result;
  }
  ・・・
  function distance(p1, p2) {...}
  function radians(degrees) {...}
  function calculateTime() {...}
}
```

```JavaScript:Migration3.js
//Migration 3
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = top_calculateDistance(points);
  const pace = totalTime / 60 / totalDistance;
  return {
    time: totalTime,
    distance: totalDistance,
    pace: pace
  }

  function top_calculateDistance(points) {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i-1], points[i]);
    }
    return result;

    // distance関数はradians関数を使用しているので、セットで移動
    function distance(p1, p2) {...}
    function radians(degrees) {...}
  }
  ・・・
  function calculateTime() {...}
}
```

```JavaScript:Migration4.js
//Migration 4
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = totalDistance(points);
  const pace = totalTime / 60 / totalDistance(points);
  return {
    time: totalTime,
    distance: totalDistance(points),
    pace: pace
  };

  function totalDistance(points) {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i-1], points[i]);
    }
    return result;

    // distance関数はradians関数を使用しているので、セットで移動
    function distance(p1, p2) {...}
    function radians(degrees) {...}
  }
  ・・・
  function calculateTime() {...}
}
```

### クラス間で移動する。
```JavaScript:Migration1.js
//Migration 1
class Account {
  get bankCharge() {
    let result = 4.5;
    if (this._daysOverdrawn > 0) result += this.overdraftCharge;
    return result;
  }

  get overdraftCharge() {
    if (this.type.isPremium) {
      const baseCharge = 10;
      if (this.daysOverdrawn <= 7) {
        return baseCharge;
      } else {
        return baseCharge + (this.daysOverdrawn - 7) * 0.85;
      }
    } else {
      return this.daysOverdrawn * 1.75;
    }
  }
}
```

```JavaScript:Migration2.js
//Migration 2
class Account {
  get bankCharge() {
    let result = 4.5;
    if (this._daysOverdrawn > 0) result += this.overdraftCharge;
    return result;
  }

  get overdraftCharge() {
    if (this.type.isPremium) {
      const baseCharge = 10;
      if (this.daysOverdrawn <= 7) {
        return baseCharge;
      } else {
        return baseCharge + (this.daysOverdrawn - 7) * 0.85;
      }
    } else {
      return this.daysOverdrawn * 1.75;
    }
  }
}

class AccounType {
  get overdraftCharge(daysOverdrawn) {
    if (this.isPremium) {
      const baseCharge = 10;
      if (daysOverdrawn <= 7) {
        return baseCharge;
      } else {
        return baseCharge + (daysOverdrawn - 7) * 0.85;
      }
    } else {
      return daysOverdrawn * 1.75;
    }
  }
}
```

```JavaScript:Migration3.js
//Migration 3
class Account {
  get bankCharge() {
    let result = 4.5;
    if (this._daysOverdrawn > 0) result += this.type.overdraftCharge(this.daysOverdrawn);
    return result;
  }

  get overdraftCharge() {
    return this.type.overdraftCharge(this.daysOverdrawn);
  }
}

class AccounType {
  get overdraftCharge(daysOverdrawn) {
    if (this.isPremium) {
      const baseCharge = 10;
      if (daysOverdrawn <= 7) {
        return baseCharge;
      } else {
        return baseCharge + (daysOverdrawn - 7) * 0.85;
      }
    } else {
      return daysOverdrawn * 1.75;
    }
  }
}
```

```JavaScript:Migration4.js
//Migration 4
class Account {
  get bankCharge() {
    let result = 4.5;
    if (this._daysOverdrawn > 0) result += this.overdraftCharge;
    return result;
  }

  get overdraftCharge() {
    return this.type.overdraftCharge(this);
  }
}

class AccounType {
  get overdraftCharge(account) {
    if (this.isPremium) {
      const baseCharge = 10;
      if (account.daysOverdrawn <= 7) {
        return baseCharge;
      } else {
        return baseCharge + (account.daysOverdrawn - 7) * 0.85;
      }
    } else {
      return account.daysOverdrawn * 1.75;
    }
  }
}
```

## フィールドの移動
```JavaScript:Before.js
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this._discountRate;}
}
```

```JavaScript:After.js
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this.plan.discountRate;}
}
```

- プログラミングでは、振る舞いを実装するコードをたくさん書くが、プログラムの力の源は実はデータ構造にある。
- 問題に適合する優れたデータ構造であれば、振る舞いのコードは少なくなる。
- そのため、データ構造が正しくない場合は、すぐに変更が必要
- 以下サンプルを書く。

```JavaScript:Migration1.js
//Migration 1
class Customer {
  constrructor(name, discountRate) {
    this._name = name;
    this._discountRate = discountRate;
    this._contract = new CustomerContract(dateToday());
  }
  get discountRate() {rate this._discountRate;}
  becomePreferred() {
    this._discountRate += 0.03;
  }
  applyDiscount(amount) {
    return amount.subtract(amount.multiply(this._discountRate));
  }
}

class CustomerContract {
  constructor(startDate) {
    this._startDate = startDate;
  }
}
```

```JavaScript:Migration2.js
//Migration 2
class Customer {
  constrructor(name, discountRate) {
    this._name = name;
    this._setDiscountRate(discountRate);
    this._contract = new CustomerContract(dateToday());
  }
  get discountRate() {rate this._discountRate;}
  _setDiscountRate(aNumber) {this._discountRate = aNumber;}
  becomePreferred() {
    this._setDiscountRate(this._discountRate + 0.03);
  }
  applyDiscount(amount) {
    return amount.subtract(amount.multiply(this.discountRate));
  }
}

class CustomerContract {
  constructor(startDate) {
    this._startDate = startDate;
  }
}
```

```JavaScript:Migration3.js
//Migration 3
class Customer {
  constrructor(name, discountRate) {
    this._name = name;
    this._setDiscountRate(discountRate);
    this._contract = new CustomerContract(dateToday());
  }
  get discountRate() {rate this._contract.discountRate;}
  _setDiscountRate(aNumber) {this._contract.discountRate = aNumber;}
  becomePreferred() {
    this._setDiscountRate(this._discountRate + 0.03);
  }
  applyDiscount(amount) {
    return amount.subtract(amount.multiply(this.discountRate));
  }
}

class CustomerContract {
  constructor(startDate, discountRate) {
    this._startDate = startDate;
    this._discountRate = discountRate;
  }
  get discountRate() {return this._discountRate;}
  set discountRate(arg) {this._discountRate = arg;}
}
```

## ステートメントの関数内への移動
```JavaScript:Before.js
result.push(`<p>title: ${person.photo.title}</p>`);
result.concate(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>location: ${aPhoto.location}</p>`,
    `<p>date: ${aPhoto.date.toDateString()}</p>`,
  ];
}
```

```JavaScript:After.js
result.concate(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>title: ${aPhoto.title}</p>`,
    `<p>location: ${aPhoto.location}</p>`,
    `<p>date: ${aPhoto.date.toDateString()}</p>`,
  ]
}
```

- 重複の削除は健全なコードを導くための1つ
- 特定の関数を呼び出すたびに同じコードが実行されていたら、反復コードを関数自体に取り込むことを検討する！

## ステートメント呼び出し側への移動
```JavaScript:Before.js
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`)
  outStream.write(`<p>location: ${photo.location}</p>`)
}
```

```JavaScript:After.js
emitPhotoData(outStream, person.photo);
outStream.write(`<p>location: ${person.photo.location}</p>`)

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`)
}
```

- 関数の抽象化の境界線があいまになってきた場合、ステートメントを移動する必要がでてくる・・・
- 境界線があいまいに感じるのは、複数箇所で利用していた共通の振る舞いを、一部の呼び出しに対してだけ変更する必要がでてきた場合だ
- これも呼び出し側が1つか2つしかない単純な場合と、複数ある場合の2パターンがあり、複数ある場合は一時的な名前を付ける必要がでてくる。
- 以下サンプルを書く。

```JavaScript:Migration1.js
//Migration 1
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>`);
  renderPhoto(outStream, person.photo);
  emitPhotoData(outStream, person.photo);
}

function listRecentPhotos(outStream, photos) {
  photos
        .filter(p => p.date > recentDateCutoff())
        .forEach(p => {
          outStream.write("<div>\n");
          emitPhotoData(outStream, p);
          outStream.write("</div>\n");
        });
}

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`);
  outStream.write(`<p>date: ${photo.date.toDateString()}</p>`);
  outStream.write(`<p>location: ${photo.location}</p>`);
}
```

```JavaScript:Migration2.js
//Migration 2
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>`);
  renderPhoto(outStream, person.photo);
  emitPhotoData(outStream, person.photo);
}

function listRecentPhotos(outStream, photos) {
  photos
        .filter(p => p.date > recentDateCutoff())
        .forEach(p => {
          outStream.write("<div>\n");
          emitPhotoData(outStream, p);
          outStream.write("</div>\n");
        });
}

function emitPhotoData(outStream, photo) {
  zztmp(outStream, photo);
  outStream.write(`<p>location: ${photo.location}</p>`);
}

function zztmp(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`);
  outStream.write(`<p>date: ${photo.date.toDateString()}</p>`);
}
```

```JavaScript:Migration3.js
//Migration 3
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>`);
  renderPhoto(outStream, person.photo);
  zztmp(outStream, person.photo);
  outStream.write(`<p>location: ${person.photo.location}</p>`);
}

function listRecentPhotos(outStream, photos) {
  photos
        .filter(p => p.date > recentDateCutoff())
        .forEach(p => {
          outStream.write("<div>\n");
          zztmp(outStream, p);
          outStream.write(`<p>location: ${p.location}</p>`);
          outStream.write("</div>\n");
        });
}

function emitPhotoData(outStream, photo) {
  zztmp(outStream, photo);
  outStream.write(`<p>location: ${photo.location}</p>`);
}

function zztmp(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`);
  outStream.write(`<p>date: ${photo.date.toDateString()}</p>`);
}
```

```JavaScript:Migration4.js
//Migration 4
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>`);
  renderPhoto(outStream, person.photo);
  zztmp(outStream, person.photo);
  outStream.write(`<p>location: ${person.photo.location}</p>`);
}

function listRecentPhotos(outStream, photos) {
  photos
        .filter(p => p.date > recentDateCutoff())
        .forEach(p => {
          outStream.write("<div>\n");
          zztmp(outStream, p);
          outStream.write(`<p>location: ${p.location}</p>`);
          outStream.write("</div>\n");
        });
}

function zztmp(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`);
  outStream.write(`<p>date: ${photo.date.toDateString()}</p>`);
}
```

```JavaScript:Migration5.js
//Migration 4
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>`);
  renderPhoto(outStream, person.photo);
  emitPhotoData(outStream, person.photo);
  outStream.write(`<p>location: ${person.photo.location}</p>`);
}

function listRecentPhotos(outStream, photos) {
  photos
        .filter(p => p.date > recentDateCutoff())
        .forEach(p => {
          outStream.write("<div>\n");
          emitPhotoData(outStream, p);
          outStream.write(`<p>location: ${p.location}</p>`);
          outStream.write("</div>\n");
        });
}

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>`);
  outStream.write(`<p>date: ${photo.date.toDateString()}</p>`);
}
```

## 関数呼び出しによるインラインコードの置き換え
```JavaScript:Before.js
let appliesToMass = false;
for(const s of states) {
  if(s === "MA") appliesToMass = true;
}
```

```JavaScript:After.js
appliesToMass = states.inclues("MA");
```

## ステートメントのスライド
```JavaScript:Before.js
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```

```JavaScript:After.js
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

- 互いに関係するコードが並んでいるとわかりやすい
- 単純な移動ができない場合はあきらめることも必要

## ループの分離
```JavaScript:Before.js
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge = averageAge / people.length;
```

```JavaScript:After.js
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
averageAge = averageAge / people.length;
```

- 1回のループで処理したいという理由だけで、2つの異なる処理を同時に行っているループをよくみる。
- しかし、同じループの中で2つの異なる処理をしているとループを修正する際には、必ず両方の処理内容を理解しないといけない。
- そのため、異なる処理の場合はループをわけるのが良い
- パフォーマンスは、後からやれば良く、もしパフォーマンスに問題がでてくるようであっても、簡単に戻すことができる

## パイプラインによるループの置き換え
```JavaScript:Before.js
const names = [];

for (const i of input) {
  if(i.job === "programmer") {
    names.push(i.name);
  }
}
```

```JavaScript:After.js
const names = input
  .filter(i => i.job === "programmer")
  .map(i => i.name)
;
```

- コレクションのパイプラインを使用すると、コレクションを消費したり、出力したりする各処理を一連の操作として記述できる
- ロジックを追うのがはるかに簡単になり、パイプラインを通過するオブジェクト群の流れを上から下に向かって読むことができる

## デットコードの削除
```JavaScript:Before.js
if (false) {
  doSomethingThatUsedToMatter();
}
```

```JavaScript:After.js
```

- 未使用のコードは、存在するだけで把握するものの対象となる。
- 不要なコードは削除するべし！

