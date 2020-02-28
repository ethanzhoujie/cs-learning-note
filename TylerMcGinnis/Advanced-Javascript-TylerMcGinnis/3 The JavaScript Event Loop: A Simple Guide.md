## The JavaScript Event Loop: A Simple Guide

JavaScript is single threaded. What that means is that only one task can be executed at a time. You call a function, a new “frame” is added to the call stack and doesn’t get popped off until that function is finished executing.

If that function calls another function, another frame is added to the call stack.

Only when the call stack is empty will the JavaScript engine look to see if there’s anything in the Task Queue. What that means is that if there’s a frame on the call stack that’s taking forever, nothing on the Task Queue will get processed. This whole process is called the “Event Loop”. 

We’re going to use a tool called [Loupe](http://latentflip.com/loupe) which allows you to visualize the JavaScript run time, at run time.

![loupe-timeout](./assets/loupe-timeout.gif)

This flow should look familiar. It’s the exact same concept that Wadsworth used earlier for figuring out how to most effectively manage his day, but instead of “Current Task”, “Services”, and “Text Inbox” we have the “Call Stack”, “Web APIs”, and “Task Queue”.

Watch the GIF a few times so you can see what’s going on. First `log` is invoked. That’s a synchronous task so JavaScript doesn’t need to do anything fancy. It just pushes `log` onto the call stack then it’s popped off once it’s executed. Next `setTimeout` is invoked. This one is async so JavaScript needs to put on its big boy pants. It first pushes `setTimeout` onto the call stack. From there, the anonymous function that is given to `setTimeout` is added to the `Web APIs` section. It’s this section which is responsible for counting to 1000 milliseconds. From there `setTimeout` is popped off the call stack and JavaScript continues to evaluate the code line by line. Next another synchronous `console.log` invocation is made so it’s added to the call stack and executed. At some point during the execution of the second `console.log`, the Web API saw that 1000 milliseconds had passed so it pushed the anonymous function into the `Task Queue` section. As we discussed earlier, whenever the call stack is empty the Event Loop checks the Task Queue and pushes the first item in there onto the call stack.

That was a big wall of text. Here’s the thing. If you’re with me still, you’re good. The complexity of the examples from here increases, but fundamentally everything will be the same as the example above. Even if the code becomes more complex, the process is still the exact same.

---------------------------------------------------------------------------------------------------------------

You may be familiar with native Promises that were introduced into JavaScript as of ES6. To accommodate that addition, there was a change to the Event Loop as well called the “Job Queue”. The way I like to think of it is it’s similar to the Task Queue except items in the Job Queue have a higher priority than items in the Task Queue. That means the Event Loop will clear out the Job Queue before it starts clearing out the Task Queue. The code below demonstrates this.

```js
console.log('First')

setTimeout(function () {
  console.log('Second')
}, 0)

new Promise(function (res) {
  res('Third')
}).then(console.log)

console.log('Fourth')
First
Fourth
Third
Second
```

Even though the `setTimeout` was before the `Promise.then`, because Promise jobs are put in the Job Queue which has a higher priority than the Task Queue, `Third` is logged to the console before `Second`.