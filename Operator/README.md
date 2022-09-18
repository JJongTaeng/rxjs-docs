# Operator 사용해보기

### Creation Operators

- `Observable`을 생성하는 연산자
- `of`, `from`, `range`, `fromEvent`, `timeout`, `interval`...
- `ajax` `Observable`만 `rxjs.ajax`에서 가져옴

### Pipable operators

- ReactiveX에서는 Observable에서 생성한 스트림을 파이프를 거치면 그 값들이 가공되어서 구독자에게 전달되어서 실행됩니다.
- 여기서 파이프가 파이프 오퍼레이터입니다.
- pure function 순수함수입니다. 순수함수는 실행되면서 부수적인 효과가 없는 함수입니다.

```javascript
const { range } = rxjs

const { filter } = rxjs.operators
const observable$ = range(1, 10)

const observer = {
  next: x => console.log(x + ' 발행'),
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
}

observable$.pipe(
  filter(x => x % 2 === 0)
).subscribe(observer);

observable$.subscribe(console.log); // range에 대한 값이 변경되지 않았음
```

#### 파이프에는 하나 이상의 operator들이 쉽표로 구분되어 들어갈 수 있습니다.
```javascript
const { range } = rxjs

const { filter, map } = rxjs.operators
const observable$ = range(1, 10)

const observer = {
  next: x => console.log(x + ' 발행'),
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
}

observable$.pipe(
  filter(x => x % 2 === 0),
  map(x => x * 2)
).subscribe(observer);

```

#### 시간에 의한 발행물에 적용해보기
```javascript
const { interval } = rxjs

const { tap, filter, map } = rxjs.operators
const observable$ = interval(1000) 

const { range } = rxjs

const observer = {
  next: x => console.log(x + ' 발행'),
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
}

observable$.pipe(
  tap(console.log),
  filter(x => x % 2 === 0),
  map(x => x * x)
).subscribe(observer)
```
- `tab` 연산자는 pipe 중간 값을 찍어볼 수 있습니다.

#### 이벤트에 의한 발행물 적용해보기
```javascript
const { fromEvent } = rxjs

const { map } = rxjs.operators
const observable$ = fromEvent(document, 'click')

const observer = {
  next: x => console.log(x + ' 발행'),
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
}

observable$.pipe(
  map(e => e.x + ' ' + e.y),
).subscribe(observer)
```