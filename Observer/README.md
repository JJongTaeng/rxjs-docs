# Observer(구독자)

### 구독자 오브젝트

```javascript
const observer = {
  next: console.log,
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
}
```

- next, error, complete 프로퍼티를 전달합니다.
- next는 스트림에서 들어오는 값들을 하나씩 콜백 함수로 처리하는 것을 뜻합니다.
- error는 스트림 처리 중 에러가 발생할 경우 실행됩니다.
- complete는 모든 스트림이 처리된 후 실행됩니다.

```javascript
const { from } = rxjs;
const observable$ = from([1, 2, 3, 4, 5]);

const observer = {
  next: console.log,
  error: err => console.error('발행중 오류', err),
  complete: () => console.log('발행물 완결'),
};

observable$.subscribe(observer);
```

#### 부분적인 프로퍼티만 전달

```javascript
const { from } = rxjs;
const observable$ = from([1, 2, 3, 4, 5]);

const observer = {
  next: console.log,
  error: err => console.error('발행중 오류', err),
};

observable$.subscribe(observer);
```

- complete 없이 전달할 수 있고, next만 전달할 수도 있습니다.

#### 생략 문법

```javascript
const { from } = rxjs;
const observable$ = from([1, 2, 3, 4, 5]);

observable$.subscribe(
  console.log,
  err => console.error('발행중 오류', err),
  () => console.log('발행물 완결'),
);
```

- subscribe 파라미터 순서대로 next, error, complete 순으로 전달할 수 있습니다.

### Error와 Complete

#### Error
```javascript
const { Observable } = rxjs;

const obs$ = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  (null)[0]; // error
  subscriber.next(4);
});

obs$.subscribe(
  console.log,
  err => console.error('발행중 오류', err),
  () => console.log('완료')
)

// 1 2 3 error
```
- 에러 발생 시 다음 스트림은 실행되지 않으며, complete 또한 실행되지 않습니다.

#### Complete
```javascript
const { Observable } = rxjs

const obs$ = new Observable(subscriber => {
  subscriber.next(1)
  subscriber.next(2)
  subscriber.next(3)
  subscriber.complete()
  subscriber.next(4)
})

obs$.subscribe(
  console.log,
  err => console.error('발행중 오류', err),
  _ => console.log('발행물 완결')
)
```
- complete 이후 next는 실행되지 않습니다.
- 다시 구독 시에는 처음부터 실행됩니다. 각각 구독을 발행하기 때문입니다.

### 구독 해제하기
```javascript
const { interval } = rxjs

const obs$ = interval(1000)
const subscription = obs$.subscribe(console.log)

setTimeout(_ => subscription.unsubscribe(), 5500)
```
- 5초뒤에 구독이 중지되며 스트림 정보가 로그에 찍히지 않습니다.
- 이 경우에도 다른 구독을 발행하면 다시 스트림 정보가 로그에 찍힙니다.(초기화 되서)