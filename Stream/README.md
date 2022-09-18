# Observables 스트림 생성기 만들기

### 배열된 스트림
```javascript
const { of, from, range, generate } = rxjs

const obs1$ = of(1, 2, 3, 4, 5)
const obs2$ = from([6, 7, 8, 9, 10])
const obs3$ = range(11, 5)
const obs4$ = generate(
  15, x => x < 30, x => x + 2
)

obs1$.subscribe(item => console.log(`of: ${item}`)) // 1 2 3 4 5
obs2$.subscribe(item => console.log(`from: ${item}`)) // 6 7 8 9 10
obs3$.subscribe(item => console.log(`range: ${item}`))
obs4$.subscribe(item => console.log(`generate: ${item}`))
```

- 가장 간단한 형태의 스트림
- generate는 초기값, 조건 함수, 연산 함수로 구성되어있습니다.

### 시간에 의한 스트림
```javascript
const { interval, timer } = rxjs;

const obs1$ = interval(1000);
const obs2$ = timer(3000);

obs1$.subscribe(item => console.log(item));
obs2$.subscribe(item => console.log(item));
```
- interval 스트림은 반복적인 시간이고, timer는 특정 시간 이후에 발생하는 스트림입니다.


###  이벤트에 의한 스트림
```javascript
<input id="myInput" type="text"/>
```
```javascript
const { fromEvent } = rxjs;

const obs1$ = fromEvent(document, 'click');
const obs2$ = fromEvent(document.getElementById('myInput'), 'keypress');

obs1$.subscribe(item => console.log(item));
obs2$.subscribe(item => console.log(item));
```
- 엘리먼트에 이벤트에 의한 스트림을 생성하고, 구독 시 이벤트에 맞춰 발생합니다.

###  Ajax를 통한 스트림
```javascript
const { ajax } = rxjs.ajax

const obs$ = ajax(`http://127.0.0.1:3000/people/1`)
obs$.subscribe(result => console.log(result.response))
```
- ajax 요청 스트림을 생성하고, 구독 시 결과값을 받아 올 수 있습니다.

###  직접 만드는 스트림
```javascript
const { Observable } = rxjs;

const obs$ = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);

  // 값을 다 발행한 뒤에는 complete를 실행하여 메모리 해제
  subscriber.complete()
});

obs$.subscribe(item => console.log(item)); // 1 2 3
```

### Observable은 lazy 하다.
```javascript
const { of, interval, fromEvent } = rxjs;

const obs1$ = of('a', 'b', 'c');
const obs2$ = interval(1000);
const obs3$ = fromEvent(document, 'click');

setTimeout(() => {
  console.log('of 구독 시작');
  obs1$.subscribe(console.log)
}, 5000);

setTimeout(() => {
  console.log('interval 구독 시작');
  obs2$.subscribe(console.log)
}, 10000);

setTimeout(() => {
  console.log('fromEvent 구독 시작');
  obs3$.subscribe(() => console.log('click'));
}, 15000);

setTimeout(() => {
  console.log('같은 스트림 2번째 구독 (interval)');
  obs2$.subscribe(console.log)
}, 20000);
```
- 누군가 구독을 해야 발행 시작합니다.
- 각 구독자에게 따로 발행합니다. 같은 스트림을 구독하더라도 값이 유지 되지 않습니다.