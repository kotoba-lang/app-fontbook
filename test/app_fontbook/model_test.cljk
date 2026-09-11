(ns app-fontbook.model-test
  (:require [app-fontbook.model :as model]
            [kotoba.lang.text :as str]
            [app-fontbook.page :as page]
            [app-fontbook.source :as source]
            [clojure.test :refer [deftest is testing]]
            [design-quality.audit :as dq]
            [mokuroku.catalog :as catalog]))

(def entries
  [{:family "Iosevka" :style "Bold" :format "TTF" :glyphs 3400 :path "/f/io-b.ttf"}
   {:family "Iosevka" :style "Book" :format "TTF" :glyphs 3400 :path "/f/io-r.ttf"}
   {:family "Iosevka" :style "Light" :format "TTF" :glyphs 3400 :path "/f/io-l.ttf"}
   {:family "Concourse" :style "" :format "OTF" :glyphs 900 :path "/f/co.otf"}
   {:family "Odd" :style "Sparkle" :format "OTF" :glyphs 12 :path "/f/odd.otf"}])

(defn- cat-of [es]
  (catalog/refresh (catalog/catalog (source/fixture-source "All Fonts" es)
                                    model/default-query)))

(deftest regular-has-many-names-and-they-are-one-face
  ;; Left un-normalised, one family shows three different Regular rows that
  ;; sort apart and look like three faces.
  (doseq [n ["Book" "Roman" "Normal" "Text" "regular" "  " ""]]
    (is (= "Regular" (model/normalize-style n)) n))
  (is (= "Bold Italic" (model/normalize-style "bold italic")))
  (is (= "Sparkle" (model/normalize-style "Sparkle")) "an unknown style is kept"))

(deftest weight-is-a-number-not-a-style-name
  ;; Sorting a family by style name puts Black before Bold before Light,
  ;; which reads as a weight order and is the reverse of one in two places.
  (is (= 700 (model/derive-weight {:style "Bold"})))
  (is (= 300 (model/derive-weight {:style "Light"})))
  (is (= 400 (model/derive-weight {:style "Book"})))
  (is (= 900 (model/derive-weight {:style "Black"})))

  (testing "a declared numeric weight wins over the name"
    (is (= 350 (model/derive-weight {:weight 350 :style "Light"}))))

  (testing "an unknown weight is nil, not 400"
    ;; Guessing 400 files it with the regulars it may not belong to.
    (is (nil? (model/derive-weight {:style "Sparkle"})))
    (is (nil? (model/derive-weight {})))))

(deftest a-face-is-family-plus-style-not-a-path
  ;; The same face is routinely installed twice, once by the system and once
  ;; by a user. Keying on the path shows two rows that cannot be told apart.
  (let [a {:family "Iosevka" :style "Book" :path "/system/io.ttf"}
        b {:family "Iosevka" :style "Regular" :path "/user/io.ttf"}]
    (is (= (model/face-id a) (model/face-id b))))

  (testing "and duplicates are surfaced rather than silently merged"
    (is (= [["Iosevka" "Regular"]]
           (model/duplicates [{:family "Iosevka" :style "Book"}
                              {:family "Iosevka" :style "Roman"}
                              {:family "Iosevka" :style "Bold"}])))))

(deftest a-family-reads-light-to-bold
  (let [ids (mapv :item/label (:result/items (catalog/result (cat-of entries))))]
    (is (= ["Concourse Regular" "Iosevka Light" "Iosevka Regular" "Iosevka Bold"
            "Odd Sparkle"]
           ids)
        "families alphabetical, weights ascending within, unknown weight last")))

(deftest a-font-viewer-cannot-uninstall
  (let [c (catalog/select (cat-of entries) ["Iosevka" "Bold"])]
    (is (= #{:open :quicklook :copy-path :export}
           (set (map :command/id (:view/commands (catalog/view c))))))
    (is (= :source-does-not-accept (:proposal/refused (catalog/propose c :trash)))
        "removing a font the system is currently rendering with is a different program")))

(deftest an-unknown-weight-is-marked-in-the-window
  (is (str/includes? (page/render-html (cat-of entries)) "Weight unknown")))

(deftest window-meets-the-design-quality-floor
  (let [pages {"fonts" (page/render (cat-of entries))
               "family" (page/render (catalog/search (cat-of entries) "Iosevka"))
               "awaiting-grant" (page/render
                                 (catalog/catalog (source/fixture-source "All Fonts" [])
                                                  model/default-query))}
        {:keys [overall pages] :as report} (dq/audit pages {:extra-axes dq/extra-axes})]
    (println "design-quality: aggregate" overall)
    (is (>= overall 98.0) (pr-str (:findings report)))
    (doseq [[nm r] pages] (is (>= (:overall r) 98.0) nm))))
