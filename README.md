### idoit
---
https://github.com/nodeca/idoit

```js
// test/chain.js
'use strict';

function delay(ms) { return new Promise(resolve => setTimeout(resolve, ms)); }

async function clear_namespace(ns) {
  const r = require('redis').createClient(REDIS_URL);
  const keys = await r.keysAsync(`${ns}`);
  
  if (keys.length) await r.delAsync(keys);
  await r.quitAsync();
}





```

```
```

```
```


