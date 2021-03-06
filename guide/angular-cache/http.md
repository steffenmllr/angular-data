@doc overview
@id http
@name Angular-cache w/$http
@description

#### Using angular-cache with $http

__Note__ The downside of letting $http handle caching for you is that it caches the responses (in string form) to your requests–not the JavaScript Object parsed from the response body. This means you can't interact with the data in the cache used by `$http`. See below for how to handle the caching yourself–giving you more control and the ability to interact with the cache (use it as a data store).

Configure `$http` to use a cache created by `DSCacheFactory` by default:
```javascript
app.run(function ($http, DSCacheFactory) {

    DSCacheFactory('defaultCache', {
        maxAge: 900000, // Items added to this cache expire after 15 minutes.
        cacheFlushInterval: 6000000, // This cache will clear itself every hour.
        deleteOnExpire: 'aggressive' // Items will be deleted from this cache right when they expire.
    });

    $http.defaults.cache = DSCacheFactory.get('defaultCache');
});

app.service('myService', function ($http, $q) {
    return {
        getDataById: function (id) {
            var deferred = $q.defer(),
                start = new Date().getTime();

            $http.get('api/data/' + id, {
                cache: true
            }).success(function (data) {
                console.log('time taken for request: ' + (new Date().getTime() - start) + 'ms');
                deferred.resolve(data);
            });
            return deferred.promise;
        }
    };
});

app.controller('myCtrl', function (myService) {
    myService.getDataById(1)
        .then(function (data) {
            // e.g. "time taken for request: 2375ms"
            // Data returned by this next call is already cached.
            myService.getDataById(1)
                .then(function (data) {
                    // e.g. "time taken for request: 1ms"
                });
        });
});
```

Tell `$http` to use a cache created by `DSCacheFactory` for a specific request:
```javascript
app.service('myService', function ($q, $http, DSCacheFactory) {

    DSCacheFactory('dataCache', {
        maxAge: 90000, // Items added to this cache expire after 15 minutes.
        cacheFlushInterval: 600000, // This cache will clear itself every hour.
        deleteOnExpire: 'aggressive' // Items will be deleted from this cache right when they expire.
    });

    return {
        getDataById: function (id) {
            var deferred = $q.defer(),
                start = new Date().getTime();

            $http.get('api/data/' + id, {
                cache: DSCacheFactory.get('dataCache')
            }).success(function (data) {
                console.log('time taken for request: ' + (new Date().getTime() - start) + 'ms');
                deferred.resolve(data);
            });
            return deferred.promise;
        }
    };
});

app.controller('myCtrl', function (myService) {
    myService.getDataById(1)
        .then(function (data) {
            // e.g. "time taken for request: 2375ms"
            // Data returned by this next call is already cached.
            myService.getDataById(1)
                .then(function (data) {
                    // e.g. "time taken for request: 1ms"
                });
        });
});
```

Do your own caching while using the $http service:
```javascript
app.service('myService', function ($q, $http, DSCacheFactory) {

    DSCacheFactory('dataCache', {
        maxAge: 900000, // Items added to this cache expire after 15 minutes.
        cacheFlushInterval: 3600000, // This cache will clear itself every hour.
        deleteOnExpire: 'aggressive' // Items will be deleted from this cache right when they expire.
    });

    return {
        getDataById: function (id) {
            var deferred = $q.defer(),
                start = new Date().getTime(),
                dataCache = DSCacheFactory.get('dataCache');

            // Now that control of inserting/removing from the cache is in our hands,
            // we can interact with the data in "dataCache" outside of this context,
            // e.g. Modify the data after it has been returned from the server and
            // save those modifications to the cache.
            if (dataCache.get(id)) {
                deferred.resolve(dataCache.get(id));
            } else {
                $http.get('api/data/' + id).success(function (data) {
                        console.log('time taken for request: ' + (new Date().getTime() - start) + 'ms');
                        dataCache.put(id, data);
                        deferred.resolve(data);
                    });
            }
            return deferred.promise;
        }
    };
});

app.controller('myCtrl', function (myService) {
    myService.getDataById(1)
        .then(function (data) {
            // e.g. "time taken for request: 2375ms"
            // Data returned by this next call is already cached.
            myService.getDataById(1)
                .then(function (data) {
                    // e.g. "time taken for request: 1ms"
                });
        });
});
```
