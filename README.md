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
  
  
  });
});



```

```
```

```
```


