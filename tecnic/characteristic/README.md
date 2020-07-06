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