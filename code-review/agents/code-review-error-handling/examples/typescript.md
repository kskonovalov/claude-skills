# Examples — TypeScript

```ts
// FAIL: проглоченная ошибка
try { await save(x); } catch (e) {}
// FIX
try { await save(x); } catch (e) { logger.error('save failed', { e }); throw e; }

// FAIL: ресурс не освобождён при ошибке
const c = await pool.connect(); const r = await c.query(sql); c.release(); return r;
// FIX
const c = await pool.connect(); try { return await c.query(sql); } finally { c.release(); }
```
