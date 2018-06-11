---
layout: post
title: "Rename to understand"
author: jflores
date: 2018-06-11 09:00:00
categories: [refactor]
summary: "Use refactor/renaming to understand code"
---

As I decided to change jobs, I paused working in "PeakToRace", my pet project, for a while.
Now that I'm starting to settle at Trainline I have come back to that code.
Even though it is my own code, it was so challenging understanding again what I was trying to do.
My last commit was something in progress that I don't even know what it was.

I found a piece of code like this:

```
const progressiveTest = require('./progressive-test');

const runWorkouts = (day, microcycle, alreadyPlannedHours, w) => {
    if(w === 0 && day === 0){
        return progressiveTest();
    }
    const totalHoursInWeek = microcycle.run.hours;
    const hours = totalHoursInWeek - alreadyPlannedHours;

    return _pick(day, hours, microcycle.run.abilities);
};

function _pick(i, h){
    let found = false;
    while(!found){
        let wk = _next(i);
    }
}

function _next(dayIndex, abilities){
  console.log(allActivities)
}

module.exports = runWorkouts;

```

All I know is that I'm trying to find a run workout for an athlete's training plan. It seems that I was in a total rush and I left everything pending.
It must not be me who wrote that code as I never write bad code... right? :D

As a good side project of mine, I don't have any unit test either.

### Unit tests

Since I don't have any safety in the form of unit tests and hitting that code by running the real app is not that simple, let's use the classic technique, add unit tests (*Working effectively with Legacy code*, Michael Feathers).

It seems that the first day of the training plan was about running a progressive test in the treadmill.

```
const runWorkouts = require('./run-workouts');
const progressiveTest = require('./progressive-test');

test('run progressive test when it is first day of first week', () => {
  const dayIndex = 0;
  const weekNumber = 0;
  expect(runWorkouts(dayIndex, {}, 0, weekNumber).workoutName).toBe(progressiveTest().workoutName);
});
```

Since the rest of the functionality is totally pending I'm not going to write any test for now. Just yet, I'm going to refactor the path I've just covered with Unit tests.

### Rename

I'm not suggesting that everytime you get to a new code you go mad and rename everything. I'm talking about the obvious things.
Those w, day, n,... variables are not really helping me undestand that code. Rename to what they are:

```
const progressiveTest = require('./progressive-test');

const runWorkouts = (dayNumber, microcycle, alreadyPlannedHours, weekNumber) => {
    if(weekNumber === 0 && dayNumber === 0){
        return progressiveTest();
    }
    const totalHoursInWeek = microcycle.run.hours;
    const availableHoursForWorkout = totalHoursInWeek - alreadyPlannedHours;

    return _pickWorkout(dayNumber, availableHoursForWorkout, microcycle.run.abilities);
};

function _pickWorkout(dayNumber, availableHoursForWorkout){
    let workoutFound = false;
    while(!workoutFound){
        let workout = _next(dayNumber);
    }
}

function _next(dayNumber, abilities){
  console.log(allActivities)
}

module.exports = runWorkouts;
```

The majority of us struggle to name things. Me included. 
When I find myself in doubt about how to name a variable I tend to postpone it for later. 
Sometimes I never come back to it and rename it.
Next time you find yourself writing something like `index` or `foo` please make sure you can back to it and name that correctly.

You may be helping yourself of the future.
