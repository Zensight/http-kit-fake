# http-kit-fake

A Clojure library for stubbing out calls to the http-kit client in tests.

``` clojure
[http-kit/fake "0.1.0"]
```

## Usage

Use the `with-fake-http` macro to fake some HTTP responses.

``` clojure
(ns your.app
  (:use org.httpkit.fake)
  (:require [org.httpkit.client :as http]))

(with-fake-http {"http://google.com/" "faked"
                 "http://facebook.com/" 500
                 {:url "http://foo.co/" :method :post} {:status 201 :body "ok"}
                 #"https?://localhost/" :allow}

  (:body @(http/get "http://google.com/"))      ; "faked"
  (:status @(http/post "http://google.com/"))   ; 200
  (:status @(http/post "http://facebook.com/")) ; 500
  (:status @(http/post "http://foo.co/"))       ; 201
  (:body @(http/post "http://foo.co/"))         ; "ok"
  (:body @(http/get "http://localhost/x"))      ; "the real response"
  (:body @(http/get "https://localhost/y"))     ; "the real response"
  (http/put "http://foo.co/"))                  ; IllegalArgumentException
```

When a request is sent with http-kit, the list of faked requests is checked.
If any match, based on checking *all* elements in the Map (e.g. :url, :method,
:form-params) then the first match is sent back as a response.

If a request is made to a URL that does not match any of the registered routes,
an IllegalArgumentException is raised and the request is prevented.

For simplicity, you may specify just a URL to match on, and/or just a body or
status code to reply with. Regexes are supported everywhere and will be
applied to the respective values in the request.

If the response is the keyword `:allow`, the request will be sent without being
blocked. This is useful when you need to whitelist certain URLs.

## License

Copyright © 2013 Chris Corbyn. See the LICENSE file for details.
