# RxJS의 다양한 연산자들

## 기본적인 배열 연산자

### 산수 관련 Operator
- count, max, min, reduce
- 이미 충분히 사용해봤던 것들과 동일한 역할을 합니다.
````javascript
const { of } = rxjs
const { count, max, min, reduce } = rxjs.operators

const obs$ = of(4, 2, 6, 10, 8)

obs$.pipe(count()).subscribe(x => console.log('count: ' + x)) // "count: 5"
obs$.pipe(max()).subscribe(x => console.log('max: ' + x)) // "max: 10"
obs$.pipe(min()).subscribe(x => console.log('min: ' + x)) // "min: 2"
obs$.pipe( 
    reduce((acc, x) => { return acc + x }, 0)
).subscribe(x => console.log('reduce: ' + x)) // "reduce: 30"
````

### 선택 관련 Operator
- first, last, elementAt, distinct, filter
- distinct는 중복제거입니다.
- first, last, elementAt는 특정 인덱스의 값을 선택합니다.
- filter는 특정 조건에 부합하는 것들을 선택합니다.
```javascript
const { from } = rxjs
const { first, last, elementAt, filter, distinct } = rxjs.operators

const obs$ = from([
    9, 3, 10, 5, 1, 10, 9, 9, 1, 4, 1, 8, 6, 2, 7, 2, 5, 5, 10, 2
])

obs$.pipe(first()).subscribe(x => console.log('first: ' + x))
obs$.pipe(last()).subscribe(x => console.log('last: ' + x))
obs$.pipe(last()).subscribe(x => console.log('last: ' + x))
obs$.pipe(distinct()).subscribe(x => console.log('distinct: ' + x))
obs$.pipe(
    filter(x => x % 2 === 1)
).subscribe(x => console.log('filter: ' + x))
```
### 활용해보기
#### 짝수들 중에서 가장 큰수
```javascript
const { from } = rxjs
const { first, last, elementAt, filter, distinct, max } = rxjs.operators

const obs$ = from([
    9, 3, 10, 5, 1, 10, 9, 9, 1, 4, 1, 8, 6, 2, 7, 2, 5, 5, 10, 2
])

obs$.pipe(
	filter(x => x % 2 === 0),
  max()
).subscribe(console.log)
```
#### 5보다 큰 3번째 짝수
```javascript
const { from } = rxjs
const { first, last, elementAt, filter, distinct, max } = rxjs.operators

const obs$ = from([
    9, 3, 10, 5, 1, 10, 9, 9, 1, 4, 1, 8, 6, 2, 7, 2, 5, 5, 10, 2
])

obs$.pipe(
	filter(x => x % 2 === 0),
	filter(x => x > 5),
  elementAt(2)
).subscribe(console.log)
```
#### 한 번 이상 나온 홀수들의 갯수, 합
```javascript
const { from } = rxjs
const { first, last, elementAt, filter, distinct, max, reduce } = rxjs.operators

const obs$ = from([
  9, 3, 10, 5, 1, 10, 9, 9, 1, 4, 1, 8, 6, 2, 7, 2, 5, 5, 10, 2
])

obs$.pipe(
  distinct(),
  filter(x => x % 2 === 1),
  reduce((acc, curr) => acc + curr, 0)
).subscribe(console.log)


```

### tab
```javascript
const { from } = rxjs
const { tap, filter, distinct } = rxjs.operators

from([
    9, 3, 10, 5, 1, 10, 9, 9, 1, 4, 1, 8, 6, 2, 7, 2, 5, 5, 10, 2
]).pipe(
    tap(x => console.log('-------------- 처음 탭: ' + x)),
    filter(x => x % 2 === 0),
    tap(x => console.log('--------- 필터 후: ' + x)),
    distinct(),
    tap(x => console.log('중복 제거 후: ' + x)),
).subscribe(x => console.log('발행물: ' + x))
```
- 통과되는 모든 값마다 특정 작업을 수행합니다.
- 발행 결과에 영향을 주지 않습니다. 부수효과를 내는 작업을 진행할 수는 있지만 지양해야할 방식입니다.
- 주로 디버깅 용도로 쓰면 좋습니다.

## Transformation 연산자
- 파이프 라인 안에서 통과되는 각 값들을 내가 원하는 값들로 변환시켜 반환하는 연산자들을 말합니다.
### map Operator
- 기존 Array 메서드 map과 동일하게 사용하면 됩니다.
```javascript
const { of } = rxjs
const { map } = rxjs.operators

of(1, 2, 3, 4, 5).pipe(
    map(x => x * x)
).subscribe(console.log)
```
```javascript
const { from } = rxjs
const { map } = rxjs.operators

from([
    { name: 'apple', price: 1200 },
    { name: 'carrot', price: 800 },
    { name: 'meat', price: 5000 },
    { name: 'milk', price: 2400 }
]).pipe(
    map(item => item.price)
).subscribe(console.log)
```

### pluck Operator
- 특정 프로퍼티 키의 값을 pluck(뽑아내는) 연산자
- map을 사용하면, `map(item => item.price)` 로 표현해야하는 것을  `pluck('price')`로 표현이 가능합니다.
```javascript
const { from } = rxjs
const { pluck } = rxjs.operators

const obs$ = from([
    { name: 'apple', price: 1200, info: { category: 'fruit' } },
    { name: 'carrot', price: 800, info: { category: 'vegetable' } },
    { name: 'pork', price: 5000, info: { category: 'meet' } },
    { name: 'milk', price: 2400, info: { category: 'drink' } }
])

obs$.pipe(
    pluck('price')
).subscribe(console.log)
```
- category 값을 pluck 하고 싶으면 아래처럼 사용하면 됩니다. 
```javascript
obs$.pipe(
    pluck('info'),
    pluck('category'),
).subscribe(console.log)

obs$.pipe(
  pluck('info', 'category')
).subscribe(console.log)
```
- 서버 요청으로 받은 응답 객체는 복잡한 구조를 갖는 경우가 많은데 해당 경우에 편하게 사용할 수 있습니다.
```javascript
const { ajax } = rxjs.ajax
const { pluck } = rxjs.operators

const obs$ = ajax(`https://api.github.com/search/users?q=user:mojombo`).pipe(
  pluck('response', 'items', 0, 'html_url')
)
obs$.subscribe(console.log)
```

### toArray Operator
- toArray를 하면 스트림 정보를 배열로 만듭니다.
```javascript
const { range } = rxjs
const { toArray, filter } = rxjs.operators

range(1, 50).pipe(
    filter(x => x % 3 === 0),
    filter(x => x % 2 === 1),
    toArray()
).subscribe(console.log) // [3, 9, 15, 21, 27, 33, 39, 45]
```

### scan Operator

#### scan
- scan은 reduce와 동작 형태가 유사하며, reduce 동작의 과정을 모두 발행한다고 이해하면 됩니다.
```javascript
const { of } = rxjs
const { reduce, scan } = rxjs.operators

const obs$ = of(1, 2, 3, 4, 5)

obs$.pipe(
    reduce((acc, x) => { return acc + x }, 0)
).subscribe(x => console.log('reduce: ' + x)) // reduce: 15

obs$.pipe(
  scan((acc, x) => { return acc + x }, 0)
).subscribe(x => console.log('scan: ' + x)) 
// "scan: 1"
// "scan: 3"
// "scan: 6"
// "scan: 10"
// "scan: 15"

```

### zip Operator
- 스트림 정보를 합치는 Creation Operator
- 배열 뿐 아니라, 인터벌, 이벤트 등 다양한 스트림을 합칠 수 있습니다.
- 합쳐졌을 때의 동작은 각각 다르며, 배열 기준 가장 길이가 작은 배열에 맞춰서 합쳐집니다.
```javascript
const { from, interval, fromEvent, zip } = rxjs
const { pluck } = rxjs.operators

const obs1$ = from([1, 2, 3, 4, 5])
const obs2$ = from(['a', 'b', 'c', 'd', 'e'])
const obs3$ = from([true, false, 'F', [6, 7, 8], { name: 'zip' }])

zip(obs1$, obs2$).subscribe(console.log)
```
- 아래 코드처럼 이벤트와 interval이 합쳐지면, 인터벌 시간보다 많은 이벤트는 발생하지 않습니다.
```javascript
const obs4$ = interval(1000)
const obs5$ = fromEvent(document, 'click').pipe(pluck('x'))

zip(obs4$, obs5$).subscribe(console.log)
```

**지금은 zip 대신 zipWith를 사용하는 것이 권장됩니다.**

## take와 skip
### take 관련 Operator

#### take
- 앞에서부터 N개 선택
```javascript
const { range, interval, fromEvent } = rxjs
const { take, filter, pluck } = rxjs.operators
```
```javascript
range(1, 20).pipe(
  take(5)
).subscribe(console.log) // 1 2 3 4 5
```
```javascript
range(1, 20).pipe(
  filter(x => x % 2 === 0),
  take(5)
).subscribe(console.log) // 2 4 6 8 10
```
```javascript
range(1, 20).pipe(
  take(5),
  filter(x => x % 2 === 0) // 2 4 
).subscribe(console.log)
```
```javascript
interval(1000).pipe(
  take(5)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE') //  1 2 3 4 5 COMPLETE
)
```

```javascript
fromEvent(document, 'click').pipe(
  pluck('x'),
  filter(x => x < 200),
  take(5),
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
) // x 좌표가 200보다 작은 곳에 클릭하는 이벤트가 5번만 발생함
```
#### takeLast
- 뒤에서부터 N개 선택
```javascript
const { range, interval, fromEvent } = rxjs
const { takeLast, take, pluck } = rxjs.operators
```
```javascript
range(1, 20).pipe(
    takeLast(5)
).subscribe(console.log)
```
```javascript
interval(1000).pipe(
    takeLast(5)
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```
```javascript
interval(1000).pipe(
    take(10),
    takeLast(5)
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
) // take 10 번 할 동안 출력이 없다가 10번 모두 실행된후 5 ~ 9 까지 한번에 출력
```
```javascript
fromEvent(document, 'click').pipe(
  takeLast(5),
  pluck('x')
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)
```
```javascript
fromEvent(document, 'click').pipe(
    take(10),
    takeLast(5),
    pluck('x')
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
) // 마찬가지로 10번 클릭 후 한번애 5개 받아옴
```

#### takeWhile
- ~ 하는 동안 선택
```javascript
const { range, interval, fromEvent } = rxjs
const { takeWhile, takeLast, filter, pluck } = rxjs.operators
```
```javascript
range(1, 20).pipe(
  takeWhile(x => x <= 10)
).subscribe(console.log) // 1 ~ 20 중 10보다 작은 경우 발행
```
```javascript
interval(1000).pipe(
  takeWhile(x => x < 5)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
) // 인터벌 5번 발행 
```
```javascript
fromEvent(document, 'click').pipe(
  pluck('x'),
  takeWhile(x => x < 200),
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
) // x 좌표가 200 보다 작은 경우만 발행
```

#### takeUntil 
- 기준이 되는 스트림이 발행하기까지

```javascript
const { interval, timer, fromEvent } = rxjs
const { ajax } = rxjs.ajax
const { takeUntil, pluck, tap } = rxjs.operators

obs1$ = interval(1000)
obs2$ = fromEvent(document, 'click')

obs1$.pipe(
  takeUntil(obs2$)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)
```
- 두개의 옵저버블이 있음
- interval을 발생시키는 옵저버블을 구독하면 1 씩 증가하면서 console.log 실행
- takeUntil 옵저버블이 실행하면, 기존에 실행중이던 스트림이 중지된다.
- 정리하면 takeUntil로 스트림이 실행될 때 까지 스트림이 발행된다.
- 위의 예제로 예를들면 intaval이 발생하다가~ 클릭이 발생하면! interval을 중지시키고 'COMPLETE' 출력

```javascript
obs1$ = fromEvent(document, 'click')
obs2$ = timer(5000)

obs1$.pipe(
  pluck('x'),
  takeUntil(obs2$)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)
```
- 이 예시로보면, 클릭 이벤트가 발생하다가~ 5초가 지나면 더 구독이 완료된다!
```javascript
interval(50).pipe(
    takeUntil(
        ajax('http://127.0.0.1:3000/people/name/random').pipe(
            pluck('response'),
            tap(console.log)
        )
    )
).subscribe(console.log)
```
- 실제 사용 예시로 위 예시를 들 수 있습니다.
- ajax 요청을 계속 하다가 응답이 왔을 때! 스트림이 완료됩니다. 
### skip 관련 Operator
#### skip
- 앞에서부터 N개씩 건너뛰기
```javascript
const { range, interval, fromEvent } = rxjs
const { skip, filter, pluck } = rxjs.operators
```
```javascript
range(1, 20).pipe(
    skip(5)
).subscribe(console.log)
```
```javascript
interval(1000).pipe(
    skip(5)
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```
```javascript
fromEvent(document, 'click').pipe(
    skip(5),
    pluck('x')
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```

#### skipLast
```javascript
const { range, interval, fromEvent } = rxjs
const { skipLast, pluck } = rxjs.operators

range(1, 20).pipe(
  skipLast(5)
).subscribe(console.log)
```

```javascript
interval(1000).pipe(
    skipLast(5)
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```
- 5초동안 아무 값 출력 없다가 0부터 출력합니다.
- 이유는 스트림에서 마지막 5개를 생략해야하는데, 0 ~ 4가 스트림이 발행되고 5가 발행된다면 일단 0은 뒤에서 5개를 스킵해도 출력되어야 하는 값이기 때문에 0부터 출력을 시작합니다.
- 이후 마찬가지로 6, 7, 8이 발행되면서 0, 1, 2는 뒤의 값이 스킵되어도 발행되어야하기 때문에 출력됩니다.
- 정리하면 현재까지 나온 값들중 마지막 5개를 스킵한 값이 출력됩니다.
```javascript
fromEvent(document, 'click').pipe(
    skipLast(5),
    pluck('x')
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```
- 위의 이벤트도 마찬가지로 5번 클릭할 때까지는 아무값도 출력하지 않다가 6번째 클릭부터 1첫째 값부터 출력을 시작합니다.
- 뒤에서 5개를 생략하더라도 첫번째 값은 출력되어야하기 때문입니다.
- 출력되는 값은 현재 클릭한 값보다 5회 이전의 값을 의미합니다.

#### skipWhile
- ~ 하는 동안 건너뛰기
```javascript
const { range, interval, fromEvent } = rxjs
const { skipWhile, filter, pluck } = rxjs.operators
range(1, 20).pipe(
  skipWhile(x => x <= 10)
).subscribe(console.log)

interval(1000).pipe(
  skipWhile(x => x < 5)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)

fromEvent(document, 'click').pipe(
  pluck('x'),
  skipWhile(x => x < 200),
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)
```
- takeWhile의 반대로 조건에 충족하는 값은 생략합니다.

#### skipUntil
- 기준이 되는 스트림이 발행하고부터
```javascript
const { interval, timer, fromEvent } = rxjs
const { skipUntil, pluck } = rxjs.operators

const obs1$ = interval(1000)
const obs2$ = fromEvent(document, 'click')

obs1$.pipe(
  skipUntil(obs2$)
).subscribe(
  console.log,
  err => console.error(err),
  _ => console.log('COMPLETE')
)
```
- takeUntil의 반대 의미로 takeUntil 옵저버블이 발생하기 전까지는 발행하지 않다가 발생하면 발행 시작
- 예제로 설명하면 아무 동작없다가 클릭 이벤트 발생 시 발행 시작

```javascript
const obs1$ = fromEvent(document, 'click')
const obs2$ = timer(5000)

obs1$.pipe(
    pluck('x'),
    skipUntil(obs2$)
).subscribe(
    console.log,
    err => console.error(err),
    _ => console.log('COMPLETE')
)
```
- 5초간 클릭이 되지 않다가 5초후부터 클릭이 가능해집니다.
- 5초 동안은 skipUntil로 발행이 skip되기 때문에!

## 시간을 다루는 연산자


## 스트림을 결합하는 연산자

## 기타 유용한 연산자
