{:title "Authorization"
 :layout :page
 :page-index 800}


Authorization is managed by two different things.

1. Buddy - which is a security library and provides the middleware to procees authorization information
2. A self implemented, very basic role system

Both is active after the application was created.

You can find the documentation here: <https://funcool.github.io/buddy-auth/latest/#access-rules>.
The rules are defined in _auth.clj_ in `src/clj/foo/example/service`. This is the standard set:

```
(defn admin-access [_] (= "admin" (sess/get :role)))
(defn loggedin-access [_] (some? (sess/get :identity)))
(defn unauthorized-access [_] true)

(def rules [{:pattern #"^/admin.*"
             :handler admin-access}
            {:pattern #"^/user/changepassword"
             :handler loggedin-access}
            {:pattern #"^/user.*"
             :handler unauthorized-access}
            {:pattern #"^/"
             :handler unauthorized-access}])
```

Every user should have a role which is either "admin" or "none". Arbitrary other roles can be implemented at will.
Every admin user has access to the /admin space of the application.
Every logged in user has access to the _changepassword_ section.
And everything else is open for every user, no matter the status.