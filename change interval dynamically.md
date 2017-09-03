
页面上经常会有一个定时任务，用来更新页面上的某些内容吗，那么就有一个常见的需求就是要动态的改变这个时间间隔

其实也不是很复杂，给出几个思路

https://stackoverflow.com/questions/42079800/rxjs-observable-changing-interval
https://stackoverflow.com/questions/1280263/changing-the-interval-of-setinterval-while-its-running

思路1
----

```js
const t = Date.now();

let subject = new Rx.Subject();
subject
  .switchMap(period => Rx.Observable.interval(period))
  .do(() => console.log('some action at time T+' + (Date.now() - t)))
  .take(8)
  .subscribe();

subject.next(50);
setTimeout(() => subject.next(100), 200);
setTimeout(() => subject.next(200), 400);
```

思路2
----

```js
let startButton = document.getElementById('start');
let stopButton  = document.getElementById('stop');
let updateButton = document.getElementById('update');
let i1 = document.getElementById('i1');

const start$ = Rx.Observable.fromEvent(startButton, 'click');
const stop$ = Rx.Observable.fromEvent(stopButton, 'click');
const update$ = Rx.Observable.fromEvent(updateButton, 'click')

const period = () => (parseInt(i1.value));

Rx.Observable.merge(
    start$, 
    update$, 
  )
  .switchMap(() => {
    return Rx.Observable.interval(period()).takeUntil(stop$);
  })
  .subscribe(res => {
    console.log('here in the subscription:' + res);
  })
```

思路3
----
```js
var timer = {
    running: false,
    iv: 5000,
    timeout: false,
    cb : function(){},
    start : function(cb,iv){
        var elm = this;
        clearInterval(this.timeout);
        this.running = true;
        if(cb) this.cb = cb;
        if(iv) this.iv = iv;
        this.timeout = setTimeout(function(){elm.execute(elm)}, this.iv);
    },
    execute : function(e){
        if(!e.running) return false;
        e.cb();
        e.start();
    },
    stop : function(){
        this.running = false;
    },
    set_interval : function(iv){
        clearInterval(this.timeout);
        this.start(false, iv);
    }
};
```

```js
timer.start(function(){
    console.debug('go');
}, 2000);

timer.set_interval(500);

timer.stop();
```

思路4
----
```js
window.setVariableInterval = function(callbackFunc, timing) {
  var variableInterval = {
    interval: timing,
    callback: callbackFunc,
    stopped: false,
    runLoop: function() {
      if (variableInterval.stopped) return;
      var result = variableInterval.callback.call(variableInterval);
      if (typeof result == 'number')
      {
        if (result === 0) return;
        variableInterval.interval = result;
      }
      variableInterval.loop();
    },
    stop: function() {
      this.stopped = true;
      window.clearTimeout(this.timeout);
    },
    start: function() {
      this.stopped = false;
      return this.loop();
    },
    loop: function() {
      this.timeout = window.setTimeout(this.runLoop, this.interval);
      return this;
    }
  };

  return variableInterval.start();
};
```

```js
var vi = setVariableInterval(function() {
  // this is the variableInterval - so we can change/get the interval here:
  var interval = this.interval;

  // print it for the hell of it
  console.log(interval);

  // we can stop ourselves.
  if (interval>4000) this.stop();

  // we could return a new interval after doing something
  return interval + 100;
}, 100);  

// we can change the interval down here too
setTimeout(function() {
  vi.interval = 3500;
}, 1000);

// or tell it to start back up in a minute
setTimeout(function() {
  vi.interval = 100;
  vi.start();
}, 60000);
```
