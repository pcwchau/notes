# Postman - Index

- [Postman - Index](#postman---index)
- [Add current time in request](#add-current-time-in-request)
- [Test Average response time](#test-average-response-time)

# Add current time in request

1. Set Pre-request Script.
2. Use the variable in the request.

Set Pre-request Script:

```js
var moment = require('moment');

pm.environment.set('currentdate', moment().format(("YYYY-MM-DD hh:mm:ss")));
```

Use the variable `{{currentdate}}` in the request.

# Test Average response time
Runner in Postman: https://learning.postman.com/docs/running-collections/intro-to-collection-runs/

Note that this is not load testing.

Use test script to calculate average resposne time: https://adequatica.medium.com/compute-response-time-in-postman-89ff3edd093e

1. Add global variable `timings`
2. Add the following script to Tests.
3. Add that API to collection, then use Runner and set iterations.
4. Remember clear the global variable before each time running.

```js
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

const currentTiming = pm.response.responseTime;
let totalTiming = pm.globals.get("timings");
let array;

// If postman variable is empty it is an empty string
if (totalTiming.length === 0) {
    // This is the case for the first or single request
    totalTiming = currentTiming;
    array = [totalTiming];
} else {
    totalTiming = `${totalTiming},${currentTiming}`;
    array = totalTiming.split(',');
}

pm.globals.set("timings", totalTiming);

let sumOfArray = 0;

for (let i = 0; i < array.length; i++) {
    sumOfArray += Number(array[i]);
}

const average = sumOfArray / (array.length);

pm.test(`Response timings: Average = ${average.toFixed(2)}, Max = ${Math.max(...array)}, Min = ${Math.min(...array)}`, function () {
    pm.expect(pm.response.responseTime).to.be.above(0);
});
```
![](image/Snipaste_2022-08-17_14-34-22.png)