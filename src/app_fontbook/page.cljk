(ns app-fontbook.page
  (:require [mokuroku.catalog :as catalog]
            [mokuroku-ui.core :as mui]))

(def view-opts
  {:columns [:face :weight :format]
   :formatters {:weight #(or % "--")
                :glyphs #(when % (str % " glyphs"))}
   :noun "faces"
   :search-placeholder "Search fonts"
   :empty-title "No fonts"
   :empty-body "Nothing in this collection matches the current filter."
   ;; A face whose weight could not be determined is marked rather than shown
   ;; as Regular, which would file it with the 400s it may not belong to.
   :badge (fn [it] (when (nil? (:weight (:item/attrs it))) "Weight unknown"))
   :title "Font Book"
   :description "Installed typefaces, by family."})

(defn render [cat] (mui/->page (catalog/view cat) view-opts))
(defn render-html [cat] (mui/->html (catalog/view cat) view-opts))
