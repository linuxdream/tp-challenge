# TP Interview Test

## Prompt
An ice cream stand opens up at 11am on a sunny day. How long will a customer who arrives around
6pm need to wait for the ice cream maker to begin making a cone for them?

It takes the ice cream maker on average seven minutes to make each cone, regardless of time of day.
You can assume that cone-making time is normally distributed, with a standard deviation of one minute,
so a library call like https://docs.python.org/3/library/random.html#random.normalvariate should do
the trick.

A new customer shows up on average every seven minutes, regardless of time of day. You can assume
that arrivals are exponentially distributed, so
https://docs.python.org/3/library/random.html#random.expovariate would be a good way to simulate
the length of time between each arrival.

Please run your simulation 1,001 times.

To show us your answer, please build a web page where a user can adjust the default parameters spelled out above; i.e.:
 * that the simulated time window is seven hours,
 * that cone-making time is on average seven minutes with a standard deviation of one minute,
 * that arrivals are spaced on average seven minutes apart, and that the simulation is repeated 1,001 times.

and can then see how many minutes the final-arriving customer needs to wait for the ice cream maker to begin making their cone.

Please use Python, JavaScript, or both, in whatever framework(s) you prefer. Your UI can be as simple as you’d like it to be. Please don’t feel obligated to host your web app for us if that isn’t convenient and cost-free for you. If you would like simply to send us code, please just include a note indicating how we can get your web app running so we can interact with it. Please make sure your code isn’t publicly visible.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```