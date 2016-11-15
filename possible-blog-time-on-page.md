Pushing to GA an event in the form:

eventAction: unloadEvent
eventCategory: timeonpage
eventLabel:   /goto
eventValue: 34.871 (seconds)
This way we can get some figures in terms of when to display something in the page, eg an email capture widget.

Time on page is calculated as the time that happens between user lands on the page and they he navigates to a different page/closes tab.
Since we send a request to GA upon page unloads, we have used beacon transport which doesn't block the browser.


```
namespace('DTL').TimeOnPageTracker = function () {
    var start;

    var init = function () {
        start = new Date().getTime();
        $(window).unload(function() {
            var end = new Date().getTime();
            var timeOnPage = (end - start) / 100;
            _dtlq.push(['_trackEventWithBeacon', 'timeonpage', 'unloadEvent', timeOnPage]);
        });
    };

    return {
        init: init
    }
}();

$(function () {
    $(document).ready(function () {
        namespace('DTL').TimeOnPageTracker.init();
    });
```

```
_trackEventWithBeacon = function (category, action, value) {
     var url = document.location.pathname;
     if (url == '/') url = '/homepage';
 
     log("DTLTracking >> _dtlq._trackEventWithBeacon", url, category, action, value);
     ga(nsGa + '.send', {
         'hitType': 'event',
         'eventCategory': category,
         'eventAction': action,
         'eventLabel': url,
         'eventValue': value,
         'page': url,
         'nonInteraction': 1
     }, { transport: 'beacon' });
 },

push = function (item) {
  execute(item);
}

execute = function (item) {
    try {
        eval(item[0]).apply(_dtlq, item.slice(1));
        log("DTLTracking >> _dtlq execute successful: ", item);
    } catch (error) {
        log("DTLTracking >> _dtlq execute failure: ", item);
        log(error);
    }
},
```

