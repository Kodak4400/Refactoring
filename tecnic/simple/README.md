## 条件記述の分解
```JavaScript:Before.js
if (!aDate.isBefore(push.summerStart) && !aDate.isAfer(plan.summerEnd)) {
  charge = quantity * plan.summerRate;
} else {
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
}
```

```JavaScript:After.js
if (summer()) {
  charge summerCharge();
} else {
  charge = regularCharge();
}
```

- 大きなブロックのコードに対しては常に、コードを分解し、それぞれを意図に沿って名付けた関数の呼び出しに置き換えることで、意図を明確にできる
- 条件記述の場合、条件判定と条件ごとの処理にこれを行うのが良い
- 「関数の抽出」の特殊ケースでもある。

## 条件記述の統合
```JavaScript:Before.js
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

```JavaScript:After.js
if (isNotEligibleForDisability()) return 0;

function isNotEligibleorDisability() {
  return ((anEmployee.seniority < 2)
    || (anEmployee.monthsDisabled > 12)
    || (anEmployee.isPartTime));
}
```

- 一連の条件があって、それぞれの条件は異なるのに、結果のアクションが同じ場合は`and`や`or`演算子を使って、単一の結果を返す1つの条件判定に統合すべし
- 条件判定のコードを統合する重要な理由は、「複数の判定をまとめることで、行っている判定が実は1つだという意図を明示できる」「関数の抽出の準備になる場合が多い」の2つ
- もし、複数の判定が本当に別々のもので、単一の判定と考えるべきでないなら、このリファクタリングはしない

## ガード節による入れ子の条件記述の置き換え
```JavaScript:Before.js
function getPayAmount() {
  let result;
  if (isDead) {
    result = deadAmount();
  } else {
    if (isRetired) {
      result = retiredAmount();
    } else {
      result = normalPayAmount();
    }
  }
  return result;
}
```

```JavaScript:After.js
function getPayAmount() {
  if (isead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```

- 条件記述には2つのスタイルがある。s「`than`節と`else`節の両方が正常動作の場合」「どちらか一方の節が正常動作で他方が例外的な動作の場合」
- この2つのスタイルの条件記述は意図が異なるため、その違いをコードで表現すべき！
- 例外的な動作に対しては、その条件を判定し、成立する場合にはリターンします。この種の判定を`ガード節`と呼ぶ。
- 「ガード節による入れ子の条件記述の置き換え」の重要な点は、強調の仕方にある
- `if-else`構文を使うときは、`than`節と`else`節は同じウェイトを置く。こにより、読み手に対して、両方が等しく起きること、重要であることを伝える

## ポリモーフィズムによる条件記述の置き換え
```JavaScript:Before.js
switch (bird.type) {
  case 'EuropeanSwallow':
    return "average";
  case 'AfricanSwallow':
    return (bird.numberOfCoconuts > 2) ? "tired" : "average";
  case 'NorwegianBlueParrot':
    return (bird.voltage > 100) ? "scorched" : "beautiful";
  default:
    return "nuknown";
}
```

```JavaScript:After.js
class EuropeanSwallow {
  get plumage() {
    return "average";
  }
}
class AfricanSwallow {
  get plumage() {
    return (this.numberOfCoconuts > 2) ? "tired" : "average";
  }
}
class NorwegianBlueParrot {
  get plumage() {
    return (this.voltage > 100) ? "scorched" : "beautiful";
  }
}
```

- 条件記述の構造自体を表現するのに、クラスとポリモーフィズムを用いると、分離をより明快に表現できる
- ポリモーフィズムを利用して型固有の振る舞いをさせることで、共通な`switch`文のロジックの重複を排除できる
- また、ロジックがバリエーションを持つ基本ケースとみなせる場合、スーパークラスに定義することで、バリエーションを気にせず、ロジックを把握できる
- それぞれのバリエーションのロジックはサブクラスに定義するべし
- 以下サンプルを書く

```JavaScript:Migration1.js
//Migration 1
function plumages(birds) {
  return new Map(birds.map(b => [b.name, plumage(b)]));
}
function speeds(birds) {
  return new Map(birds.map(b => [b.name, airSpeedVelocity(b)]));
}
function plumage(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return 'average';
    case 'AfricanSwallow':
      return (bird.numberOfCoconunts > 2) ? 'tired' : 'average';
    case 'NorwegianBlueParrot':
      return (bird.voltage > 100) ? 'scorched' : 'beautiful';
    default:
      return 'unknown';
  }
}
function airSpeedVelocity(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return 35;
    case 'AfricanSwallow':
      return 40 - 2 * bird.numberOfCoconuts;
    case 'NorwegianBlueParrot':
      return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
    default:
      return null;
  }
}
```

```JavaScript:Migration2.js
//Migration 2
function plumages(birds) {
  return new Bird(bird).plumage;
}
function speeds(birds) {
  return new Bird(bird).airSpeedVelocity;
}
class Bird {
  constructor(birdObject) {
    Object.assign(this, birdObject);
  }
  get plumage() {
    switch (bird.type) {
      case 'EuropeanSwallow':
        return 'average';
      case 'AfricanSwallow':
        return (bird.numberOfCoconunts > 2) ? 'tired' : 'average';
      case 'NorwegianBlueParrot':
        return (bird.voltage > 100) ? 'scorched' : 'beautiful';
      default:
        return 'unknown';
    }
  }
  get airSpeedVelocity() {
    switch (bird.type) {
      case 'EuropeanSwallow':
        return 35;
      case 'AfricanSwallow':
        return 40 - 2 * bird.numberOfCoconuts;
      case 'NorwegianBlueParrot':
        return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
      default:
        return null;
    }
  }
}
```

```JavaScript:Migration3.js
//Migration 3
function plumages(birds) {
  // return createBird(bird).plumage;
  return new Map(birds
                .map(b => createBird(b))
                .map(bird => [bird.name, bird.plumage]));
}
function speeds(birds) {
  // return createBird(bird).airSpeedVelocity;
  return new Map(birds
                .map(b => createBird(b))
                .map(bird => [bird.name, bird.plumage]));
}
function createBird(bird) {
    switch (bird.type) {
      case 'EuropeanSwallow':
        return new EuropeanSwallow(bird);
      case 'AfricanSwallow':
        return new AfricanSwallow(bird);
      case 'NorwegianBlueParrot':
        return new NorwegianBlueParrot(bird);
      default:
        return new Bird(bird);
  }
}
class Bird {
  constructor(birdObject) {
    Object.assign(this, birdObject);
  }
  get plumage() {
    return 'unknown';
  }
  get airSpeedVelocity() {
    return null;
  }
}

class EuropeanSwallow extends Bird {
  get plumage() {
    return 'average';
  }
  get airSpeedVelocity() {
    return 35;
  }
}
class AfricanSwallow extends Bird {
  get plumage() {
    return (bird.numberOfCoconunts > 2) ? 'tired' : 'average';
  }
  get airSpeedVelocity() {
    return 40 - 2 * bird.numberOfCoconuts;
  }
}
class NorwegianBlueParrot extends Bird {
  get plumage() {
    return (bird.voltage > 100) ? 'scorched' : 'beautiful';
  }
  get airSpeedVelocity() {
    return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
  }
}
```

### バリエーションに対してポリモーフィズムを適用する例

```JavaScript:Migration1.js
//Migration 1
function rating(voyage, history) {
  const vpf = voyageProfitFactor(voyage, history);
  const vr = voyageRisk(voyage);
  const chr = captainHistoryRisk(voyage, history);
  if(vpf * 3 > (vr + chr * 2)) return "A";
  else return "B";
}
function voyageRisk(voyage) {
  let result = 1;
  if (voyage.length > 4) result += 2;
  if (voyage.length > 8) result += voyage.length - 8;
  if (["china", "east-indies"].includes(voyage.zone)) result += 4;
  return Math.max(result, 0);
}
function captainHistoryRisk(voyage, history) {
  let result = 1;
  if (history.length < 5) result += 4;
  result += history.filter(v => v.profit < 0).length;
  if ( voyage.zone === 'china' && hasChina(history)) result -= 2; // どのような条件判定を行っているか確認.
  return Math.max(result, 0);
}
function hasChina(history) {
  return history.some(v => 'china' === v.zone);
}
function voyageProfitFactor(voyage, history) {
  let result = 2;
  if (voyage.zone === 'china') result += 1;
  if (voyage.zone === 'east-indies') result += 1;
  if (voyage.zone === 'china' && hasChina(history)) { // どのような条件判定を行っているか確認.
    result += 3;
    if(history.length > 10) result += 1;
    if(voyage.length > 12) result += 1;
    if(voyage.length > 18) result -= 1;
  }
  else {
    if (history.length > 8) result += 1;
    if (voyage.length > 14) result -= 1;
  }
  return result;
}

// 呼び出し側
const voyage = {zone: "west-indies", length: 10};
const history = [
  {zone: "east-indies", profit: 5},
  {zone: "west-indies", profit: 15},
  {zone: "china", profit: -2},
  {zone: "west-africa", profit: 7},
];
const myRating = rating(voyage, history);
```

```JavaScript:Migration2.js
//Migration 2
function rating(voyage, history) {
  return new Rating(voyage, history).value;
}

class Raging {
  constructor(voyage, history) {
    this.voyage = voyage;
    this.history = history;
  }
  get value() {
    const vpf = voyageProfitFactor(this.voyage, this.history);
    const vr = voyageRisk(this.voyage);
    const chr = captainHistoryRisk(this.voyage, this.history);
    if(vpf * 3 > (vr + chr * 2)) return "A";
    else return "B";
  }
  get voyageRisk() {
    let result = 1;
    if (this.voyage.length > 4) result += 2;
    if (this.voyage.length > 8) result += this.voyage.length - 8;
    if (["china", "east-indies"].includes(this.voyage.zone)) result += 4;
    return Math.max(result, 0);
  }
  get captainHistoryRisk() {
    let result = 1;
    if (this.history.length < 5) result += 4;
      result += this.history.filter(v => v.profit < 0).length;
    if ( this.voyage.zone === 'china' && hasChina(this.history)) result -= 2; // どのような条件判定を行っているか確認.
    return Math.max(result, 0);
  }
  get hasChina() {
    return this.history.some(v => 'china' === v.zone);
  }
  get voyageProfitFactor() {
    let result = 2;
    if (this.voyage.zone === 'china') result += 1;
    if (this.voyage.zone === 'east-indies') result += 1;
    if (this.voyage.zone === 'china' && hasChina(this.history)) {　// どのような条件判定を行っているか確認.
      result += 3;
      if(this.history.length > 10) result += 1;
      if(this.voyage.length > 12) result += 1;
      if(this.voyage.length > 18) result -= 1;
    }
    else {
      if (this.history.length > 8) result += 1;
      if (this.voyage.length > 14) result -= 1;
    }
    return result;
  }
}
```

```JavaScript:Migration3.js
//Migration 3
function rating(voyage, history) {
  return createRating(voyage, history).value;
}
function createRating(voyage, history) {
  if (voyage.zone === "china" && history.some(v => "china" === v.zone))
    return new ExperiencedChinaRating(voyage, history);
  else return new Rating(voyage, history);
}

class Raging {
  constructor(voyage, history) {
    this.voyage = voyage;
    this.history = history;
  }
  get value() {
    const vpf = voyageProfitFactor(this.voyage, this.history);
    const vr = voyageRisk(this.voyage);
    const chr = captainHistoryRisk(this.voyage, this.history);
    if(vpf * 3 > (vr + chr * 2)) return "A";
    else return "B";
  }
  get voyageRisk() {
    let result = 1;
    if (this.voyage.length > 4) result += 2;
    if (this.voyage.length > 8) result += this.voyage.length - 8;
    if (["china", "east-indies"].includes(this.voyage.zone)) result += 4;
    return Math.max(result, 0);
  }
  get captainHistoryRisk() {
    let result = 1;
    if (this.history.length < 5) result += 4;
    result += this.history.filter(v => v.profit < 0).length;
    return Math.max(result, 0);
  }
  get hasChina() {
    return this.history.some(v => 'china' === v.zone);
  }
  get voyageProfitFactor() {
    let result = 2;
    if (this.voyage.zone === 'china') result += 1;
    if (this.voyage.zone === 'east-indies') result += 1;
    result += this.voyageAndHistoryLengthFactor;
    return result;
  }
  get voyageAndHistoryLengthFactor() {
    let result = 0;
    if (this.history.length > 8) result += 1;
    if (this.history.length > 14) result -= 1;
    return result;
  }
}

class ExperiencedChinaRating extends Rating {
  get captainHistoryRisk() {
    const result = super.captainHistoryRisk - 2;
    return Math.max(result, 0);
  }
  get voyageAndHistoryLengthFactor() {
    let result = 0;
    result += 3;
    if(this.history.length > 10) result += 1;
    if(this.voyage.length > 12) result += 1;
    if(this.voyage.length > 18) result -= 1;
    return result;
  }
}
```

※ リファクタリングはここで終了だが、雑なリファクタリングのため、追加でリファクタリングする

```JavaScript:Migration4.js
//Migration 4
function rating(voyage, history) {
  return createRating(voyage, history).value;
}
function createRating(voyage, history) {
  if (voyage.zone === "china" && history.some(v => "china" === v.zone))
    return new ExperiencedChinaRating(voyage, history);
  else return new Rating(voyage, history);
}


class Raging {
  constructor(voyage, history) {
    this.voyage = voyage;
    this.history = history;
  }
  get value() {
    const vpf = voyageProfitFactor(this.voyage, this.history);
    const vr = voyageRisk(this.voyage);
    const chr = captainHistoryRisk(this.voyage, this.history);
    if(vpf * 3 > (vr + chr * 2)) return "A";
    else return "B";
  }
  get voyageRisk() {
    let result = 1;
    if (this.voyage.length > 4) result += 2;
    if (this.voyage.length > 8) result += this.voyage.length - 8;
    if (["china", "east-indies"].includes(this.voyage.zone)) result += 4;
    return Math.max(result, 0);
  }
  get captainHistoryRisk() {
    let result = 1;
    if (this.history.length < 5) result += 4;
    result += this.history.filter(v => v.profit < 0).length;
    return Math.max(result, 0);
  }
  get voyageProfitFactor() {
    let result = 2;
    result += this.historyLengthFactor;
    result += this.voyageLengthFactor;
    return result;
  }
  get historyLengthFactor() {
    return (this.history.length > 8) ? 1 : 0;
  }
  get voyageLengthFactor() {
    return (this.voyage.length > 14) ? -1 : 0;
  }
}

class ExperiencedChinaRating extends Rating {
  get captainHistoryRisk() {
    const result = super.captainHistoryRisk - 2;
    return Math.max(result, 0);
  }
  get voyageAndHistoryLengthFactor() {
    let result = 0;
    result += this.historyLengthFactor();
    if(this.voyage.length > 12) result += 1;
    if(this.voyage.length > 18) result -= 1;
    return result;
  }
  get historyLengthFactor() {
    return (this.history.length > 10) ? 1 : 0;
  }
  get voyageProfitFactor() {
    return super.voyageProfitFactor + 3;
  }
}
```

## 特殊ケースの導入
```JavaScript:Before.js
if (aCustomer === 'unknown') customerName = 'occupant';
```

```JavaScript:After.js
class UnknownCustomer {
  get name() {return 'occupant';}
}
```

- コード重複のよくあるケースとして、データ構造が特定の値かどうかを判定して、該当する場合には同じ処理をしていることがある
- 特定の値に対して同じ処理をするコードがたくさんあると、その処理を1ヵ所にまとめる
- このパターンでは、特殊ケースとして共通な振る舞いをすべて備えた要素をつくる
- これにより、特殊ケースの判定はほとんど簡単な呼び出しに置き換えできる


## アサーションの導入
```JavaScript:Before.js
if (this.discountRate) {
  base = base - (this.discountRate * base);
}
```

```JavaScript:After.js
assert(this.discountRate >= 0)
if (this.discountRate) {
  base = base - (this.discountRate * base);
}
```

- 多くの場合、コードの特定部分はある条件が成り立つ場合のみ機能する。そうした条件は、コメントに書かれている場合が多い
- テクニックとしてアサーションをつかう
- アサーションは常に真であることを前提にした条件文。アサーションの失敗はプログラマのエラーを意味する。
- アサーションはエラーの発見にもなるが、それだけでなく、コミュニケーションにも役立つ
- こういったコード内にアサーションを入れることをセルフテストという

