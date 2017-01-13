# PAC Script Environment (API) for Chrome v55

In PAC script there is no `window` or `global`.  
There is `this` which serves as global context.  
I extracted properties of `this` and share it for your benefit.

## PAC-specific Properties (w/o Implementations)

```
this = {
// Global properties specific for PAC script.

convert_addr: function convert_addr(ipchars) {...},
dateRange: function dateRange() {...},
dnsDomainIs: function dnsDomainIs(host, domain) {...},
dnsDomainLevels: function dnsDomainLevels(host) {...},
dnsResolve: function dnsResolve() {...},
dnsResolveEx: function dnsResolveEx() {...},
isInNet: function isInNet(ipaddr, pattern, maskstr) {...},
isInNetEx: function isInNetEx() {...},
isPlainHostName: function isPlainHostName() {...},
isResolvable: function isResolvable(host) {...},
isResolvableEx: function isResolvableEx(host) {...},
localHostOrDomainIs: function localHostOrDomainIs(host, hostdom) {...},
months: Object,
myIpAddress: function myIpAddress() {...},
myIpAddressEx: function myIpAddressEx() {...},
shExpMatch: function shExpMatch(url, pattern) {...},
sortIpAddressList: function sortIpAddressList() {...},
timeRange: function timeRange() {...},
wdays: Object,
weekdayRange: function weekdayRange() {...},

};
```

## Properties Shared With `window`

```js
this = {

Infinity: null,
Array: function Array() { [native code] },
ArrayBuffer: function ArrayBuffer() { [native code] },
Boolean: function Boolean() { [native code] },
ByteLengthQueuingStrategy: function ByteLengthQueuingStrategy() { [native code] },
CountQueuingStrategy: function CountQueuingStrategy() { [native code] },
DataView: function DataView() { [native code] },
Date: function Date() { [native code] },
Error: function Error() { [native code] },
EvalError: function EvalError() { [native code] },
Float32Array: function Float32Array() { [native code] },
Float64Array: function Float64Array() { [native code] },
Function: function Function() { [native code] },
Int8Array: function Int8Array() { [native code] },
Int16Array: function Int16Array() { [native code] },
Int32Array: function Int32Array() { [native code] },
Intl: Object,
JSON: Object,
Map: function Map() { [native code] },
Math: Object,
NaN: null,
Number: function Number() { [native code] },
Object: function Object() { [native code] },
Promise: function Promise() { [native code] },
Proxy: function Proxy() { [native code] },
RangeError: function RangeError() { [native code] },
ReadableStream: function ReadableStream() { [native code] },
ReferenceError: function ReferenceError() { [native code] },
Reflect: Object,
RegExp: function RegExp() { [native code] },
Set: function Set() { [native code] },
String: function String() { [native code] },
Symbol: function Symbol() { [native code] },
SyntaxError: function SyntaxError() { [native code] },
TypeError: function TypeError() { [native code] },
URIError: function URIError() { [native code] },
Uint8Array: function Uint8Array() { [native code] },
Uint8ClampedArray: function Uint8ClampedArray() { [native code] },
Uint16Array: function Uint16Array() { [native code] },
Uint32Array: function Uint32Array() { [native code] },
WeakMap: function WeakMap() { [native code] },
WeakSet: function WeakSet() { [native code] },
alert: function alert() { [native code] },
decodeURI: function decodeURI() { [native code] },
decodeURIComponent: function decodeURIComponent() { [native code] },
encodeURI: function encodeURI() { [native code] },
encodeURIComponent: function encodeURIComponent() { [native code] },
escape: function escape() { [native code] },
eval: function eval() { [native code] },
isFinite: function isFinite() { [native code] },
isNaN: function isNaN() { [native code] },
parseFloat: function parseFloat() { [native code] },
parseInt: function parseInt() { [native code] },
unescape: function unescape() { [native code] },

}
```

## Implementations of Some PAC-specific Properties

```js
this = {

convert_addr: function convert_addr(ipchars) {
    var bytes = ipchars.split('.');
    var result = ((bytes[0] & 0xff) << 24) |
                 ((bytes[1] & 0xff) << 16) |
                 ((bytes[2] & 0xff) <<  8) |
                  (bytes[3] & 0xff);
    return result;
},
dateRange: function dateRange() {
    function getMonth(name) {
        if (name in months) {
            return months[name];
        }
        return -1;
    }
    var date = new Date();
    var argc = arguments.length;
    if (argc < 1) {
        return false;
    }
    var isGMT = (arguments[argc - 1] == 'GMT');

    if (isGMT) {
        argc--;
    }
    // function will work even without explict handling of this case
    if (argc == 1) {
        var tmp = parseInt(arguments[0]);
        if (isNaN(tmp)) {
            return ((isGMT ? date.getUTCMonth() : date.getMonth()) ==
getMonth(arguments[0]));
        } else if (tmp < 32) {
            return ((isGMT ? date.getUTCDate() : date.getDate()) == tmp);
        } else { 
            return ((isGMT ? date.getUTCFullYear() : date.getFullYear()) ==
tmp);
        }
    }
    var year = date.getFullYear();
    var date1, date2;
    date1 = new Date(year,  0,  1,  0,  0,  0);
    date2 = new Date(year, 11, 31, 23, 59, 59);
    var adjustMonth = false;
    for (var i = 0; i < (argc >> 1); i++) {
        var tmp = parseInt(arguments[i]);
        if (isNaN(tmp)) {
            var mon = getMonth(arguments[i]);
            date1.setMonth(mon);
        } else if (tmp < 32) {
            adjustMonth = (argc <= 2);
            date1.setDate(tmp);
        } else {
            date1.setFullYear(tmp);
        }
    }
    for (var i = (argc >> 1); i < argc; i++) {
        var tmp = parseInt(arguments[i]);
        if (isNaN(tmp)) {
            var mon = getMonth(arguments[i]);
            date2.setMonth(mon);
        } else if (tmp < 32) {
            date2.setDate(tmp);
        } else {
            date2.setFullYear(tmp);
        }
    }
    if (adjustMonth) {
        date1.setMonth(date.getMonth());
        date2.setMonth(date.getMonth());
    }
    if (isGMT) {
    var tmp = date;
        tmp.setFullYear(date.getUTCFullYear());
        tmp.setMonth(date.getUTCMonth());
        tmp.setDate(date.getUTCDate());
        tmp.setHours(date.getUTCHours());
        tmp.setMinutes(date.getUTCMinutes());
        tmp.setSeconds(date.getUTCSeconds());
        date = tmp;
    }
    return ((date1 <= date) && (date <= date2));
},
dnsDomainIs: function dnsDomainIs(host, domain) {
    return (host.length >= domain.length &&
            host.substring(host.length - domain.length) == domain);
},
dnsDomainLevels: function dnsDomainLevels(host) {
    return host.split('.').length-1;
},
dnsResolve: function dnsResolve() { [native code] },
dnsResolveEx: function dnsResolveEx() { [native code] },
isInNet: function isInNet(ipaddr, pattern, maskstr) {
    var test = /^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$/.exec(ipaddr);
    if (test == null) {
        ipaddr = dnsResolve(ipaddr);
        if (ipaddr == null)
            return false;
    } else if (test[1] > 255 || test[2] > 255 || 
               test[3] > 255 || test[4] > 255) {
        return false;    // not an IP address
    }
    var host = convert_addr(ipaddr);
    var pat  = convert_addr(pattern);
    var mask = convert_addr(maskstr);
    return ((host & mask) == (pat & mask));
    
},
isInNetEx: function isInNetEx() { [native code] },
isPlainHostName: function isPlainHostName() { [native code] },
isResolvable: function isResolvable(host) {
    var ip = dnsResolve(host);
    return (ip != null);
},
isResolvableEx: function isResolvableEx(host) {
    var ipList = dnsResolveEx(host);
    return (ipList != '');
},
localHostOrDomainIs: function localHostOrDomainIs(host, hostdom) {
    return (host == hostdom) ||
           (hostdom.lastIndexOf(host + '.', 0) == 0);
},
months: Object,
myIpAddress: function myIpAddress() { [native code] },
myIpAddressEx: function myIpAddressEx() { [native code] },
shExpMatch: function shExpMatch(url, pattern) {
   pattern = pattern.replace(/\./g, '\\.');
   pattern = pattern.replace(/\*/g, '.*');
   pattern = pattern.replace(/\?/g, '.');
   var newRe = new RegExp('^'+pattern+'$');
   return newRe.test(url);
},
sortIpAddressList: function sortIpAddressList() { [native code] },
timeRange: function timeRange() {
    var argc = arguments.length;
    var date = new Date();
    var isGMT= false;

    if (argc < 1) {
        return false;
    }
    if (arguments[argc - 1] == 'GMT') {
        isGMT = true;
        argc--;
    }

    var hour = isGMT ? date.getUTCHours() : date.getHours();
    var date1, date2;
    date1 = new Date();
    date2 = new Date();

    if (argc == 1) {
        return (hour == arguments[0]);
    } else if (argc == 2) {
        return ((arguments[0] <= hour) && (hour <= arguments[1]));
    } else {
        switch (argc) {
        case 6:
            date1.setSeconds(arguments[2]);
            date2.setSeconds(arguments[5]);
        case 4:
            var middle = argc >> 1;
            date1.setHours(arguments[0]);
            date1.setMinutes(arguments[1]);
            date2.setHours(arguments[middle]);
            date2.setMinutes(arguments[middle + 1]);
            if (middle == 2) {
                date2.setSeconds(59);
            }
            break;
        default:
          throw 'timeRange: bad number of arguments'
        }
    }

    if (isGMT) {
        date.setFullYear(date.getUTCFullYear());
        date.setMonth(date.getUTCMonth());
        date.setDate(date.getUTCDate());
        date.setHours(date.getUTCHours());
        date.setMinutes(date.getUTCMinutes());
        date.setSeconds(date.getUTCSeconds());
    }
    return ((date1 <= date) && (date <= date2));
},
wdays: Object,
weekdayRange: function weekdayRange() {
    function getDay(weekday) {
        if (weekday in wdays) {
            return wdays[weekday];
        }
        return -1;
    }
    var date = new Date();
    var argc = arguments.length;
    var wday;
    if (argc < 1)
        return false;
    if (arguments[argc - 1] == 'GMT') {
        argc--;
        wday = date.getUTCDay();
    } else {
        wday = date.getDay();
    }
    var wd1 = getDay(arguments[0]);
    var wd2 = (argc == 2) ? getDay(arguments[1]) : wd1;
    return (wd1 == -1 || wd2 == -1) ? false
                                    : (wd1 <= wday && wday <= wd2);
},

};
```

## How to Get Similar Data

1. In a Web Extension set error listener on PAC-script:
```js
chrome.proxy.onProxyError.addListener((fault) => window.ERROR = fault.details);
```

2. Inside PAC script execute the following code:
```js
var alt = {};

Object.getOwnPropertyNames(this).forEach(function(key) {

    alt[key] = this[key];

}, this);

for(const prop in this) {
  alt[prop] = this[prop];
}

var cache = [];
throw JSON.stringify(alt, function(key, value) {
    if (typeof value === 'object' && value !== null) {
        if (cache.indexOf(value) !== -1) {
            // Circular reference found, discard key
            return;
        }
        // Store value in our collection
        cache.push(value);
    }
    if (typeof value === 'function' && value.name !== 'FindProxyForURL') { return value + ''; }
    return value;
});
```
3. Examine `window.ERROR` inside devTools console of Web Extension's background page.
