---
name: check-bus
description: Check bus arrival times for Hong Kong bus routes using hkbus.app. Use when asked about bus schedules, next bus arrival, or bus stop ETAs. Currently supports Route 12A at stop KC272 (欣榮花園).
---

# Check Bus Arrival Times (Hong Kong)

Check real-time bus arrival times using hkbus.app with Puppeteer.

## Route 12A - Stop KC272 (欣榮花園)

### Quick Command

```bash
cd /tmp && node check_bus.js
```

### Script

Save as `/tmp/check_bus.js`:

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ 
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage']
  });
  const page = await browser.newPage();
  
  await page.setUserAgent('Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36');
  
  await page.goto('https://hkbus.app/zh/route/12a-1-whampoa-garden-cheung-sha-wan-(hoi-tat-estate)/A91745D47D69FF05%2C10', { 
    waitUntil: 'domcontentloaded',
    timeout: 30000 
  });
  
  await new Promise(r => setTimeout(r, 5000));
  
  const data = await page.evaluate(() => {
    const lines = document.body.innerText.split('\n');
    let foundKC272 = false;
    let results = [];
    
    for (let i = 0; i < lines.length; i++) {
      if (lines[i].includes('KC272') || lines[i].includes('欣榮花園')) {
        foundKC272 = true;
      }
      if (foundKC272 && lines[i].includes('分鐘')) {
        results.push(lines[i].trim());
        if (results.length >= 3) break;
      }
    }
    return results;
  });
  
  console.log('Route 12A - Stop KC272 (欣榮花園)');
  console.log('Next buses:');
  data.forEach((t, i) => console.log(`  ${i+1}. ${t}`));
  
  await browser.close();
})();
```

### URL
```
https://hkbus.app/zh/route/12a-1-whampoa-garden-cheung-sha-wan-(hoi-tat-estate)/A91745D47D69FF05%2C10
```

### Route 12A Stops

| # | Code | Name | Fare |
|---|------|------|------|
| 1 | HH920 | 黃埔花園總站 | $5.8 |
| 2 | HH300 | 紅樂道 | $5.8 |
| 3 | HH320 | 紅磡南道 | $5.8 |
| 4 | HH330 | 機利士南路 | $5.8 |
| 5 | HH341 | 孖庶街 | $5.8 |
| 6 | HH245 | 北拱街 | $5.8 |
| 7 | HH249 | 石塘街 | $5.8 |
| 8 | KC260 | 浙江街 | $5.8 |
| 9 | KC263 | 上鄉道 | $5.8 |
| 10 | KC270 | 中華煤氣公司 | $5.8 |
| 11 | KC272 | **欣榮花園** | $5.8 |
| 12+ | ... | 深水埗方向 | $5.8 |

### Example Output

```
Route 12A - Stop KC272 (欣榮花園)
Next buses:
  1. 6 分鐘
  2. 24 分鐘 - 原定班次
  3. 39 分鐘 - 原定班次
```
