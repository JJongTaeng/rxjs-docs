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

## 시간을 다루는 연산자

## 스트림을 결합하는 연산자

## 기타 유용한 연산자
