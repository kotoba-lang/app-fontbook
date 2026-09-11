(ns app-fontbook.source
  "The `fs/browse` seam. Parsing is glyph's job, reading is the provider's."
  (:require [app-fontbook.model :as model]
            [mokuroku.source :as source]))

(defrecord FontSource [collection read-fn]
  source/ISource
  (-descriptor [_] (model/descriptor collection))
  (-fetch [_] (model/listing->items (read-fn collection))))

(defn font-source [collection read-fn] (->FontSource collection read-fn))
(defn fixture-source [collection entries] (font-source collection (constantly entries)))

(def denied
  {:fonts/state :denied :fonts/capability model/capability :fonts/entries []})

(defn granted [entries]
  {:fonts/state :granted :fonts/capability model/capability
   :fonts/entries (vec entries)})

(defn denied? [r] (= :denied (:fonts/state r)))
