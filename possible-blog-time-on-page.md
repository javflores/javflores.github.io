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

Some links:
////In here create a script that will do the following:
//    var start;
//
//    $(document).ready(function() {
//        start = Date.getTime();
//
//        $(window).unload(function() {
//            end = Date.getTime();
//            $.ajax({ 
//                url: "log.php",
//                data: {'timeSpent': end - start}
//            })
//        });
//    }   from http://stackoverflow.com/questions/4667068/how-to-measure-a-time-spent-on-a-page

    //rather than doing ajax, use ga to send an event similar to this: 'event' : 'unloadEvent', 'timeonpage' : calculated like mentioned above. http://www.simoahava.com/analytics/leverage-usebeacon-beforeunload-google-analytics/
    //use transport beacon https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#transport
    //why? https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon
	
	
http://www.simoahava.com/analytics/leverage-usebeacon-beforeunload-google-analytics/
https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon
http://www.simoahava.com/analytics/use-page-visibility-api-google-tag-manager/
http://www.analytics-ninja.com/blog/2015/02/real-time-page-google-analytics.html
https://www.thyngster.com/google-analytics-added-sendbeacon-functionality-universal-analytics-javascript-api/
https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#transport


eventAction: unloadEvent
eventCategory: timeonpage
eventLabel:   /goto
eventValue: 34871
