---
link: https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html
byline: Brice Bernard
excerpt: PostgREST is a lightning fast RESTful API on top of any PostgreSQL database. Ng-admin can plug to any RESTful API. What if we connected the two?
slurped: 2024-06-06T16:30:13.225Z
title: Using Ng-admin with postgREST
tags:
  - postgrest
---

PostgREST is a lightning fast RESTful API on top of any PostgreSQL database. Ng-admin can plug to any RESTful API. What if we connected the two?

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#context)Context

[Ng-admin](https://github.com/marmelab/ng-admin), as you may already know, adds an AngularJS admin GUI to any RESTful API. We've been working on it for quite some time already - it's a huge time saver.

That's really nice, but you still have to spend some time to code and expose an API on top of a relational database. But hey, I'm a little lazy and I don't want to code that API server... here comes [postgREST](https://github.com/begriffs/postgrest) to the rescue.

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#what-is-postgrest)What is PostgREST?

PostgREST serves a fully RESTful API from any existing PostgreSQL database. It provides a cleaner, more standards-compliant, faster API than you are likely to write from scratch. Sounds great!

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#combining-ng-admin-and-postgrest)Combining Ng-admin and PostgREST

An example RESTful server exposed by postgREST is available at [sessions, `speakers`, and `sponsors`. As you can guess, the purpose is to manage events. We can already interact with this API using curl commands but it would be even nicer to plug ng-admin in.](https://marmelab.com/blog/2015/03/23/%22%3Ehttps://postgrest.herokuapp.com/%3C/a%3E.%20The%20underlying%20database%20contains%203%20tables:%20%3Ccode%20class=)

[The natural behaviours of ng-admin and postgREST differ a little bit, so let's make those compatible.](https://marmelab.com/blog/2015/03/23/%22%3Ehttps://postgrest.herokuapp.com/%3C/a%3E.%20The%20underlying%20database%20contains%203%20tables:%20%3Ccode%20class=)

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#adapting-the-code-classlanguage-textgetlistcode-operation)Adapting The `getList` Operation

### [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#the-request-part)The Request Part

Ng-admin uses `_page` and `_perPage` parameters by default to handle pagination. PostgREST expects these informations to be part of a request header (`Range` and `Range-Unit`). The natural `_sortDir` and `_sortField` parameters of ng-admin should also expressed differently to be understood by postgREST.

```
# what ng-admin would sent
curl 'https://postgrest.herokuapp.com/speakers?_page=1&_perPage=10&_sortDir=DESC&_sortField=id'

# what postgrest expects to receive
curl 'https://postgrest.herokuapp.com/speakers?order=id.desc' \
    -H 'Range-Unit: speakers' \
    -H 'Range: 0-9'
```

So we need to alter the corresponding request before letting ng-admin send it. Here is what needs to be done in the ng-admin config.

```
app.config(function(RestangularProvider) {
    RestangularProvider.addFullRequestInterceptor(function(element, operation, what, url, headers, params, httpConfig) {
        if (operation === 'getList') {
            headers = headers || {};
            headers['Range-Unit'] = what;
            headers['Range'] = ((params._page - 1) * params._perPage) + '-' + (params._page * params._perPage - 1);
            delete params._page;
            delete params._perPage;

            if (params._sortField) {
                params.order = params._sortField + '.' + params._sortDir.toLowerCase();
                delete params._sortField;
                delete params._sortDir;
            }
        }
    });
});
```

### [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#the-response-part)The Response Part

In order to display pagination properly, the API server returns the total number of entities in a `Content-Range` header. Ng-admin doesn't understand this header by default.

```
# expected response header by ng-admin
X-Total-Count: 12

# actual response header of postgREST
Content-Range: 0-9/12
```

To translate this header, we need to update the ng-admin config to extract the total count from `Content-Range` header.

```
app.config(function(RestangularProvider) {
    RestangularProvider.addResponseInterceptor(function(data, operation, what, url, response, deferred) {
        if (operation === 'getList') {
            response.totalCount = response.headers('Content-Range').split('/')[1];
        }
    });

    return data;
});
```

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#adapting-the-code-classlanguage-textgetcode-operation)Adapting the `get` Operation

When displaying a detailed view of an entity, ng-admin calls [http://my.api/my_entities/123](http://my.api/my_entities/123) by default. But postgREST uses another kind of pattern: [http://my.api/my_entities?id=eq.123](http://my.api/my_entities?id=eq.123).

```
# default call done by ng-admin
curl 'https://postgrest.herokuapp.com/speakers/1'

# call expected by postgREST
curl 'https://postgrest.herokuapp.com/speakers?id=eq.1'
```

To mimic this behaviour, we alter (again) the request in the ng-admin config. You may notice that we update `$httpProvider` directly - that's because of a bug in RestangularProvider which prevents to alter the url.

```
// @see https://github.com/mgonto/restangular/issues/603
app.config(function($httpProvider) {
    $httpProvider.interceptors.push(function() {
        return {
            request: function(config) {
                var pattern = /\/(\d+)$/;

                if (pattern.test(config.url)) {
                    config.params = config.params || {};
                    config.params['id'] = 'eq.' + pattern.exec(config.url)[1];
                    config.url = config.url.replace(pattern, '');
                }

                return config;
            },
        };
    });
});
```

Furthermore, postgREST always returns arrays instead of a single entity (as ng-admin would expect for a `get` operation). So let's turn this into a single entity using a Response Interceptor.

```
app.config(function(RestangularProvider) {
    RestangularProvider.addResponseInterceptor(function(data, operation, what, url, response, deferred) {
        if (operation === 'get') {
            return data[0];
        }

        return data;
    });
});
```

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#adapting-the-code-classlanguage-textpostcode-operation)Adapting the `post` Operation

Ng-admin expects the response to contain the newly created entity. On the orther hand, postgREST differs by sending an empty response with a `Location` header, redirecting to the newly created entity. Since version 2.6 of postgREST, it is possible to enforce the desired behaviour by setting the `Prefer` header to `return=representation`.

```
app.config(function(RestangularProvider, $httpProvider) {
    RestangularProvider.addFullRequestInterceptor(function(element, operation, what, url, headers, params, httpConfig) {
        headers = headers || {};
        headers['Prefer'] = 'return=representation';
    });
});
```

## [](https://marmelab.com/blog/2015/03/23/using-ng-admin-with-postgrest.html#conclusion)Conclusion

That's it! With this configuration, you can plug ng-admin to any PostgREST-powered API in minutes. You can check the available demo based on the sample RESTful server here: [http://marmelab.com/ng-admin-postgrest/](http://marmelab.com/ng-admin-postgrest/), and read [source on Github](https://github.com/marmelab/ng-admin-postgrest).

PostgREST is a really nice tool, which perfectly fits with ng-admin to increase your productivity. The PostgREST project evolves really fast, and keeps focusing on simplicity and API standards. You definitively should take a look at it.