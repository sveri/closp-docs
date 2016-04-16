{:title "Using Websockets"
 :layout :page
 :page-index 1200}

In this chapter I will show you how to add websockets to a closp application and how to use them
bidirectional.

## Libraries

First we will add two new libraries to enable websockets support. Put these into the _:dependencies_ section of your _project.clj_

- [com.taoensso/sente "1.8.1"]
- [org.clojure/core.match "0.3.0-alpha4"]

core.match is used for dispatching on the event types later.

Exit any REPL that you had open and also stop figwheel. After adding the dependencies you have to start both again in a command line:

> lein repl
> lein figwheel

This is necessary to pick up new dependencies.

## Component

To make use of websockets we will have to add a new component on the backend side. It is responsible for starting and stopping them.
Create a new file _ws.clj_ in `src/foo/example/components` with the following content:

```
(ns foo.example.components.ws
  (:require [taoensso.sente :as sente]
            [taoensso.sente.server-adapters.http-kit :refer [sente-web-server-adapter]]
            [com.stuartsierra.component :as component]))

(defrecord Websockets []
  component/Lifecycle
  (start [component]
      (assoc component :websockets (sente/make-channel-socket! sente-web-server-adapter {})))
  (stop [component] component))

(defn new-websockets []
  (map->Websockets {}))
  ```

Next we will have to make it available to the other components that want to use websockets. For this open _components.clj_ in `src/foo/example/components`. You will see other components defined already, add this line:

> :websockets (ws/new-websockets)

above the _:db_ component.
Then change the handler component and add the websockets reference:

> :handler (component/using (new-handler) [:config :locale :websockets])

And finally add add the require statement:

> [foo.example.components.ws :as ws]

Now we have to go through the call chain and add the _ws_ everywhere it needs to be:

> _handler.clj_ in `src/foo/example/components` - add the `websockets` argument to the _defrecord_
> add the `websockets` parameter to the `get-handler` call in the _defrecord_
> add the `websockets` as parameter to the `get-handler` method as parameter

It should look like this now:

```
(defn get-handler [config locale websockets]
  (-> (app-handler
        (into [] (concat (when (:registration-allowed? config) [(registration-routes config)])
                         ;; add your application routes here
                         [(blog-routes) (cc-routes config) home-routes (user-routes config) base-routes]))
        ;; add custom middleware here
        :middleware (load-middleware config (:tconfig locale))
        :ring-defaults (mk-defaults false)
        ;; add access rules here
        :access-rules []
        ;; serialize/deserialize the following data formats
        ;; available formats:
        ;; :json :json-kw :yaml :yaml-kw :edn :yaml-in-html
        :formats [:json-kw :edn :transit-json])
      ; Makes static assets in $PROJECT_DIR/resources/public/ available.
      (wrap-file "resources")
      ; Content-Type, Content-Length, and Last Modified headers for files in body
      (wrap-file-info)))

(defrecord Handler [config locale websockets]
  comp/Lifecycle
  (start [comp]
    (assoc comp :handler (get-handler (:config config) locale websockets)))
  (stop [comp]
    (assoc comp :handler nil)))

(defn new-handler []
  (map->Handler {}))
```

## Defining Routes

We need to add two routes as websocket endpoints. For this create a new source file _ws.clj_ in
`src/foo/example/routes` with the following content:

```
(ns foo.example.routes.ws
  (:require [compojure.core :refer [routes GET POST]]))

(defn ws-routes [{:keys [websockets]}]
  (routes (GET "/ws" req ((:ajax-get-or-ws-handshake-fn websockets) req))
          (POST "/ws" req ((:ajax-post-fn websockets) req))))
```

Also we have to register the routes in the handler. Reopen the _handler.clj_ file from the last paragraph. Require some more namespaces:

> [compojure.core :refer [defroutes routes]]
> [foo.example.routes.ws :refer [ws-routes]]
> [ring.middleware.params :refer [wrap-params]]
> [ring.middleware.keyword-params :refer [wrap-keyword-params]]

Also add these two functions above the `get-handler` function:

```
(defn- pre-init [middleware]
  (let [proxy (middleware (fn [req] ((:route-handler req) req)))]
    (fn [handler]
      (fn [request]
        (proxy (assoc request :route-handler handler))))))

(defn wrap-routes
  "Apply a middleware function to routes after they have been matched."
  ([handler middleware]
   (let [middleware (pre-init middleware)]
     (fn [request]
       (let [mw (:route-middleware request identity)]
         (handler (assoc request :route-middleware (comp middleware mw)))))))
  ([handler middleware & args]
   (wrap-routes handler #(apply middleware % args))))
```

And finally change the `get-handler` function to look like that:

```
(defn get-handler [config locale websockets]
  (routes
    (-> (ws-routes websockets)
        (wrap-routes wrap-params)
        (wrap-routes wrap-keyword-params))
    (-> (app-handler
         (into [] (concat (when (:registration-allowed? config) [(registration-routes config)])
                          ;; add your application routes here
                          [(ws-routes websockets) (blog-routes) (cc-routes config) home-routes (user-routes config) base-routes]))
         ;; add custom middleware here
         :middleware (load-middleware config (:tconfig locale))
         :ring-defaults (mk-defaults false)
         ;; add access rules here
         :access-rules []
         ;; serialize/deserialize the following data formats
         ;; available formats:
         ;; :json :json-kw :yaml :yaml-kw :edn :yaml-in-html
         :formats [:json-kw :edn :transit-json])
       ; Makes static assets in $PROJECT_DIR/resources/public/ available.
       (wrap-file "resources")
       ; Content-Type, Content-Length, and Last Modified headers for files in body
       (wrap-file-info))))
```
As you may have noticed we separated the websocket routes from the other existing routes. This is because the websockets need a different set of middleware applied than the _standard_ routes.

Switch to the REPL and execute:

> (reload)

to notify closp of the new routes. If you followed the steps there should be no compile error here.
## Client Side

Now we will establish a connection from client side to the websocket routes.

Open _core.cljs_ in `cljs/foo/example`. Delete all the source code and replace it with this one:

```
(ns foo.example.core
  (:require [reagent.core :as reagent :refer [atom]]
            [taoensso.sente :as sente]
            [foo.example.helper :as h]))


(defn entry []
  [:div "Websocket Example"])

(defn ^:export main []
  (reagent/render-component (fn [] [entry]) (h/get-elem "app")))
```

Now browse to <http://localhost:3000/example> it should look like this:

![WS example](/img/websocket-01.png)

Next add the websocket initialization and event handling for sente to that file:

```
(def router_ (atom nil))

(defmulti event-msg-handler (fn [event] (:id event)))

(defmethod event-msg-handler :default
  [{:keys [event]}]
  (println "Unhandled event: %s" event))

(defmethod event-msg-handler :chsk/recv
  [{:keys [?data]}]
  (println ?data))

(defn start-router! []
  ;ws-map contains these keys: chsk ch-recv send-fn state
  (let [ws-map (sente/make-channel-socket! "/ws"
                                    {:type :auto})] ; e/o #{:auto :ajax :ws}

    (reset! router_ (sente/start-chsk-router! (:ch-recv ws-map) (fn [event] (event-msg-handler event))))
    ws-map))
```

Also call the `start-router!` function from the `main` function. If you switch to the webpage and open the developer console you should see output like this:

> Unhandled event: %s [:chsk/state {:type :ws, :open? true, :uid :taoensso.sente/nil-uid, :csrf-token nil, :first-open? true}]
> core.cljs:147 Unhandled event: %s [:chsk/handshake [:taoensso.sente/nil-uid nil]]

after refreshing the page.

These are the messages you get after establishing a websocket connection from client to the server.


And finally we will do the same on backend side.  Reopen _ws.clj_ from `src/foo/example/routes` and change it to look like this:

```
(ns foo.example.routes.ws
  (:require [compojure.core :refer [routes GET POST]]
            [taoensso.sente :as sente]))


(defmulti event-msg-handler (fn [event] (:id event)))

(defmethod event-msg-handler :default
  [{:keys [event]}]
  (println "Unhandled event: %s" event))

(defmethod event-msg-handler :chsk/recv
  [{:keys [?data]}]
  (println ?data))


(defn start-listening [websockets]
  (sente/start-chsk-router! (:ch-recv websockets) (fn [event] (event-msg-handler event))))

(defn ws-routes [{:keys [websockets]}]
  (start-listening websockets)
  (routes (GET "/ws" req ((:ajax-get-or-ws-handshake-fn websockets) req))
          (POST "/ws" req ((:ajax-post-fn websockets) req))))
```

We added the same event handling method as on client side and can act as well as send messages.

To send messages you have to use the `send-fn` that was returned from the `start-chsk-router!` like this:

```
(doseq [uid (:any @connected-uids)] (send-fn uid [:some/namespace "some message"]))
```

Where `connected-uids` is another key value pair avaible in the returned map.