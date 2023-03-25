#!/usr/bin/env bb

(ns serve
  (:require [org.httpkit.server :refer [run-server as-channel send!]]
            [hiccup.core :refer [html]]
            [babashka.pods :as pods]
            [clojure.java.browse :refer [browse-url]]
            [babashka.cli :as cli]
            [babashka.fs :as fs]))


(pods/load-pod 'org.babashka/fswatcher "0.0.3")
(require '[pod.babashka.fswatcher :as fw])


;; default port to serve from.
(def ^:dynamic *port* 8080)


;; default ip to serve from.
(def ^:dynamic *ip* "localhost")


;; target is the file to watch
(def file (atom nil))


;; client side script to listen on websocket and receive new page.
(defn ws-script []
  (str
   "<script type=\"text/javascript\">
    // use vanilla JS because why not
    window.addEventListener(\"load\", function() {
        
        // create websocket instance
        var mySocket = new WebSocket(\"ws://" *ip*  ":" *port* "/ws\");
        
        // add event listener reacting when message is received
        mySocket.onmessage = function (event) {
           var output = document.getElementById(\"output\");
           output.innerHTML = event.data;
        };
    });
    </script>"))


;; --State--
;; clients (websocket channels.
(defonce clients (atom #{}))
;; holds the server - a function that shuts down the server.
(defonce server (atom nil))
;; the file watcher
(defonce watcher (atom nil))


;; server side websocket handler
(defn ws-handler [req]
  (as-channel req
              {:on-open (fn [ch] (swap! clients conj ch))
               :on-close (fn [ch _] (swap! clients disj ch))}))


;; Our app
(defn body [] (slurp @file))


(defn app [req]
  (case (:uri req)
    "/"      {:status  200
              :headers {"Content-Type" "text/html"}
              :body    (html [:html
                              [:head
                               (ws-script)]
                              [:body
                               [:div {:id "output"}
                                (body)]]])}
    "/ws"    (ws-handler req)

    {:status 404
     :body "Not found."
     :headers {"Content-Type" "text/html"}}))


(defn start-server []
  (reset! server (run-server #'app {:ip *ip* :port *port*}))
  (browse-url (str "http://" *ip* ":" *port*)))


(defn stop-server []
  (when-not (nil? @server)
    (@server :timeout 100)
    (reset! server nil)
    (reset! clients #{})))


(defn notify-clients [msg]
  (doseq [cli @clients]
    (send! cli msg)))


(defn now []
  (let [date (java.time.LocalDateTime/now)
        formatter (java.time.format.DateTimeFormatter/ofPattern "HH:mm:ss")]
    (str "[" (.format date formatter) "]")))


(defn start-watch [tgt]
  (reset! watcher
          (fw/watch tgt
                    (fn [event]
                      (when (= :write (:type event))
                        (notify-clients (body))
                        (println (now) "broadcasting update"))))))


(defn stop-watch []
  (fw/unwatch @watcher)
  (reset! watcher nil))


(defn start [tgt]
  (start-watch tgt)
  (start-server)
  nil)


(defn stop []
  (stop-watch)
  (stop-server)
  (println (str (now) " stopped serving.")))


;; UX
(def spec {:port   {:ref           "<port>"
                    :desc          "The port to host at. Should be a long."
                    :coerce        :long
                    :alias         :p
                    :default-desc  "8080"
                    :default       *port*}
           :ip     {:ref           "<ip>"
                    :desc          "The ip address to host at."
                    :alias         :i
                    :default-desc  "localhost"
                    :default       *ip*}
           :help   {:ref           "<help>"
                    :desc          "Prints this help."
                    :coerce        :boolean
                    :alias         :h}
           :file   {:ref           "<file>"
                    :desc          "The file to serve. Can be supplied as the main argument without the --file/-f flag."
                    :alias         :f}})


(def cli-opts {:alias {:p :port :i :ip :h :help :f :file}
            ;; :require [:file]
               :coerce {:port :long :help :boolean}
               :exec-args {:port *port* :ip *ip*}
               :args->opts [:file]})


(defn help [] (cli/format-opts {:spec spec :order [:file :port :ip :help]}))


(def help-text
  "<serve> is a utility that watches for changes in a file whose content can be served over http, e.g. text, json, svg and serves that content in your browser. When the file is changed, <serve> updates.\nUsage:\nserve <file>\nOptions:\n")


(let [opts (cli/parse-opts *command-line-args* cli-opts)]
  (if (:help opts)
    (println (str help-text (help)))
    (if (not (fs/exists? (:file opts)))
      (do (println "File doesn't exist!")
          (System/exit 1))
      (binding [*port* (:port opts)
                *ip* (:ip opts)]

        (reset! file (:file opts))
        (println (str (now)  " serving at http://" *ip* ":" *port*))
        (start (:file opts))
          
        @(promise)))))
