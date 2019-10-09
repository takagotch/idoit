### idoit
---
https://github.com/nodeca/idoit

```js
// test/chain.js
'use strict';

const assert = require('assert');

const Queue = require('../index');
const random = require('../lib/utils').random;

const REDIS_URL = 'redis://localhost:6379/3';

function delay(ms) { return new Promise(resolve => setTimeout(resolve, ms)); }

async function clear_namespace(ns) {
  const r = require('redis').createClient(REDIS_URL);
  const keys = await r.keysAsync(`${ns}`);
  
  if (keys.length) await r.delAsync(keys);
  await r.quitAsync();
}

describe('chain', function () {
  
  let q, q_ns;
  
  beforeEach(async function () {
    q_ns = `idoit_test_$test_${(random(6)}:`;
    
    q = new Queue({ redisURL: REDIS_URL, ns: q_ns });
    
    q.wait = async function (id) {
      let task = await this.getTask(id);
      
      while (task.state !== 'finished') {
        await delay(50);
        task = await this.getTask(id);
      }
      
      return task;
    };
    
    q.on('error', err => { throw err; });
    
    await q.start();
  });
  
  afterEach(async function () {
    await q.shutdown();
    clearTimeout(q.__timer__);
    await delay(100);
    await q.__redis__.quit();
    await clear_namespace(q_ns);
  });
  
  it('should run children in correct order', function (done) {
    let run = [ false, false, false ];
    
    q.registerTask('t1', function () {
      run[0] = true;
      assert.deepEqual(run, [ true, false, false ]);
    });
    
    q.registerTask('t2', function () {
      run[1] = true;
      assert.deepEqual(run, [ true, true, false ]);
    });
    
    q.registerTask('t3', function () {
      run[2] = true;
      assert.deepEqual(run, [ true, true, true ]);
      
      setTimeout(done, 10);
    });
    
    q.chain([
      q.t1(),
      q.t2(),
      q.t3()
    ]).run();
  });
  
  it('should pass data between tasks', async function () {
    q.registerTask('mult', (a, b) => a * b);
    q.registerTask('add', (a, b) => a + b);
    q.registerTask('sub', (a, b) => a - b);
    
    let id = await q.chain([
      q.mult(1, 2),
      q.add(3),
      q.sub(7)
    ]).run();
    
    let task = await q.wait(id);
    
    assert.equal(task.result, 2);
  });
  
  it('should set progress in chain', async function () {
    q.registerTask({
      name: 't1',
      process() {},
      init() {
        this.total = 3;
      }
    });
    
    q.registerTask({
      name: 't2',
      process() {},
      init() {
        this.total = 2;
      }
    });
    
    let id = await q.chain([
      q.t1(),
      q.t2()
    ]).run();
    
    let task = await q.wait(id);
    
    assert.equal(task.total, 5);
    assert.equal(task.progress, 5);
  });
  
  it('should handle subtask error', async function () {
    let t1Calls = 0;
    
    q.removeAllListenerts('error');
    
    q.on('error', err => { if (!String(err).includes('<!test err!>')) throw err; });
    
    q.registerTask('t1', () => { t1Calls++ });
    q.registerTask({
      name: 't2',
      process() { throw new Error('<!test err!>'); },
      retryDelay: 10
    });
    
    let t2 = q.t2();
    let id = await q.chain([
      q.t1(),
      q.t1(),
      t2,
      q.t1(),
      q.t1()
    ]).run();
    
    let task = await q.wait(id);
    
    t2 = await q.getTask(t2.id);
    
    assert.equal(t1Calls, 2);
    assert.ok(task.error.message.includes('<!test err!>'));
    assert.ok(t2.error.message.includes('<!test err!>'));
  });
  
  it('should pass result of group to next task', async function () {
  
  });
  
  it('should run user init', async function () {
  
  });
  
  it('should allow custom arguments for task with user init', async function () {
  
  });
  
  it('`.calcel()` should emit "task:end" event for all unfinished tasks in chain', async function () {
    q.registerTask();
    q.registerTask();
    q.registerTask();
    
    let id = await q.chain().run();
    
    await new Promise(resolve => q.once('task:end:t1', resolve));
    
    let finished_tasks = [];
    
    q.on('task:end', function (task_info) {
      finished_tasks.push(task_info.id);
    });
    
    await q.cancel(id);
    
    assert.deepEqual(finshed_tasks, [ 't2', 't3', id ]);
  });
});
```

```
```

```
```


