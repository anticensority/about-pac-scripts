# About PAC Scripts

1. Settign PAC Scripts
1.1-3. Windows, Chromium, FireFox
2. Caching of PAC Scripts
2.1-3. Windows, Chromium, FireFox
3. Developing PAC Scripts
3.1. Alerts and Debugging
3.2. Chromium Bugs/Features
3.3. Implementations and Sources
3.4. Resources


## 1. Setting PAC Scripts

### 1.1. Windows 

On Windows you may set PAC script via `Win+R > inetcpl.cpl > Connections > LAN settings > Use automatic configuration script`

### 1.2. Chromium

1. In Chromium settings there is a button for opening standard Linux/Windows dialog of setting PAC script.
2. Also Web Extensions of Chromium may set PAC script as a string and as URL via `chrome.proxy.settings`. This is implemented independently of the OS and has its own bugs.

### 1.3. FireFox

`Browser > Options > Advanced > Network > Settings > Automatic proxy configuration URL`

## 2. Caching of PAC Scripts

### 2.1. Windows

Windows (winHttp, etc.) does very poor caching if any. Setting PAC script via URL in system settings results in server bombardment with requests.

### 2.2. Chromium

If web extension set PAC script via URL then the script is loaded only once and cached. Cache overloading happens only on browser restarts. (checked on Chrome 55/Windows)

### 2.3. Firefox

After setting PAC URL in browser settings it is loaded only once or twice and cached for a long period. (checked on Firefox 50/Windows)

## 3. Developing PAC Scripts

1. PAC script is re-evaluated (and reparsed) each time for each URL-resource loaded.
2. There are no `window`, `global`, `setTimeout/setInterval` global variables. There is `Math` however.
2. IE may be detected via [Conditional Compilation](http://stackoverflow.com/questions/10072816/how-does-this-ie-check-work) of comment content: `const isIE = /*@cc_on!@*/false;`. I haven't tested this.
3. 'google.com.' with dot at the end is also a correct notation. I don't know any library that passes `host` this way, but I always defense myself with: `host.replace(/\.?$/, '')`.

### 3.1. Alerts and Debugging

1. Alert messages may be seen in Chromium network events: chrome://net-internals/#events
2. In Chromium you may check active proxy settings via chrome://net-internals/#proxy
3. In Chromium you may add PAC script's error listener via `chrome.proxy.onProxyError.addListener` and pass debug information in error.

### 3.2. Chromium Bugs/Features

1. Web Extensions can't set PAC scripts larger than 1MB via URL, [bug](https://bugs.chromium.org/p/chromium/issues/detail?id=678022).
2. Web Extensions can't set scripts as a string with non-ASCII chars, unicode URLs must be punycoded.
3. Once I deleted all extensions from Chrome, but PAC script was still active and couldn't be purged via settings or browser restarts.

### 3.3. Implementations and Sources

- Chromium: https://cs.chromium.org/chromium/src/net/proxy/proxy_resolver_script.h
- FireFox: https://dxr.mozilla.org/mozilla-central/source/netwerk/base/ProxyAutoConfig.cpp

### 3.4. Resources

1. http://findproxyforurl.com
2. https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Necko/Proxy_Auto-Configuration_(PAC)_file
