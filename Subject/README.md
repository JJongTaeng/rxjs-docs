# Subject

```javascript
const { Subject } = rxjs
const subject = new Subject()

subject.subscribe(console.log)

subject.next(1)
subject.next(3)
subject.next(5)
```

## Observable과의 차이점

### Observable
- 누군가 구독을 해야 발행을 시작
- 각 구도작에게 따로 발행
- unicast
- cold 발행
- Netflix처럼 언제든 1화부터 볼 수 있음

### Subject
- 개발자가 원하는 때에 발행
- 모든 구독자에게 똑같이 발행
- multicast
- hot 발행
- TV채널 처럼 모든 시청자에게 같은 화면 송출

```javascript
const { Subject } = rxjs
const subject = new Subject()

let x = 0
setInterval (_ => {
  subject.next(x++)
}, 2000)

subject.subscribe(x => console.log('바로구독: ' + x))
setTimeout(_ => {
    subject.subscribe(x => console.log('3초 후 구독: ' + x))
}, 3000)
setTimeout(_ => {
    subject.subscribe(x => console.log('10초 후 구독: ' + x))
}, 10000)
setTimeout(_ => {
    subject.subscribe(x => console.log('14초 후 구독: ' + x))
}, 14000)
```

## 일반 Observable에 결합하기
```javascript
const { interval } = rxjs

const obs$ = interval(1000)

obs$.subscribe(x => console.log('바로구독: ' + x))
setTimeout(_ => {
    obs$.subscribe(x => console.log('3초 후 구독: ' + x))
}, 3000)
setTimeout(_ => {
    obs$.subscribe(x => console.log('5초 후 구독: ' + x))
}, 5000)
setTimeout(_ => {
    obs$.subscribe(x => console.log('10초 후 구독: ' + x))
}, 10000)
```

- 위 코드에서 아래 코드로 변경하면 Observable로 생성된 스트림을 Subject형태로 사용이 가능합니다.
```javascript
const { interval, Subject } = rxjs

const subject = new Subject()
const obs$ = interval(1000)

obs$.subscribe(subject)

subject.subscribe(x => console.log('바로구독: ' + x))
setTimeout(_ => {
    subject.subscribe(x => console.log('3초 후 구독: ' + x))
}, 3000)
setTimeout(_ => {
    subject.subscribe(x => console.log('5초 후 구독: ' + x))
}, 5000)
setTimeout(_ => {
    subject.subscribe(x => console.log('10초 후 구독: ' + x))
}, 10000)
```

## 추가 기능이 있는 Subject

### BehaviorSubject
- 마지막 값을 저장 후 추가 구독자에게 발행합니다.
```javascript
const { BehaviorSubject } = rxjs
const subject = new BehaviorSubject(0) // 초기값이 있음

subject.subscribe((x) => console.log('A: ' + x))

subject.next(1)
subject.next(2)
subject.next(3)

subject.subscribe((x) => console.log('B: ' + x))

subject.next(4)
const lastValue = subject.getValue() // 서브젝트가 마지막으로 발행한 값을 얻을 수 있습니다.
subject.next(5)

console.log(lastValue);
```

### AsyncSubject
````javascript
const { AsyncSubject } = rxjs
const subject = new AsyncSubject()
 
subject.subscribe((x) => console.log('A: ' + x))
 
subject.next(1)
subject.next(2)
subject.next(3)
 
subject.subscribe((x) => console.log('B: ' + x))
 
subject.next(4)
subject.next(5)

subject.subscribe((x) => console.log('C: ' + x))

subject.next(6)
subject.next(7)
subject.complete()
````
- Complete 후의 마지막 값만 발행합니다.