(ns app-fontbook.model
  "Font Book's domain: installed typefaces.

  One capability, `fs/browse`, to find the font files. Parsing them is
  `kotoba-lang/glyph`'s job and reading them is the provider's; this namespace
  decides what a face *is* and how faces of one family relate."
  (:require [kotoba.lang.text :as str]
            [mokuroku.item :as item]
            [mokuroku.source :as source]))

(def capability "fs/browse")

(def columns
  [(source/attribute :face "Face" :string)
   (source/attribute :family "Family" :string)
   (source/attribute :weight "Weight" :number)
   (source/attribute :format "Format" :string)
   (source/attribute :glyphs "Glyphs" :number)
   (source/attribute :path "Location" :string false)])

(def commands
  "No install and no remove: those mutate a system-wide font set, and no
  provider implements either. A viewer that could uninstall a font the system
  is currently rendering with is a different and much riskier program."
  #{:open :quicklook :copy-path :export})

(defn descriptor
  ([] (descriptor "All Fonts"))
  ([collection]
   (source/descriptor
    {:id :app-fontbook/faces
     :item-kind :font-face
     :label collection
     :capability capability
     :commands commands
     :attributes columns})))

(def canonical-regular
  "Names foundries use for the same thing. Left un-normalised, one family
  shows three different `Regular` rows that sort apart and look like three
  faces."
  #{"regular" "book" "roman" "normal" "text"})

(defn normalize-style [style]
  (let [s (str/lower (str/trim (str style)))]
    (cond
      (str/blank? s) "Regular"
      (contains? canonical-regular s) "Regular"
      :else (str/join " " (map str/capitalize (str/split s #"\s+"))))))

(def weight-names
  "CSS numeric weights. A face's weight is a number, not its style string:
  sorting a family by style name puts Black before Bold before Light, which
  reads as a weight order and is the reverse of one in two places."
  {"thin" 100 "extralight" 200 "ultralight" 200 "light" 300
   "regular" 400 "normal" 400 "book" 400 "roman" 400 "text" 400
   "medium" 500 "semibold" 600 "demibold" 600
   "bold" 700 "extrabold" 800 "ultrabold" 800
   "black" 900 "heavy" 900})

(defn derive-weight
  "Prefer the weight the font file declares; fall back to reading the style
  name. Returns nil when neither is available rather than guessing 400 — an
  unknown weight is not a regular weight."
  [{:keys [weight style]}]
  (or (when (number? weight) weight)
      (some->> (str/lower (str (or style "")))
               (re-find #"thin|extralight|ultralight|light|regular|normal|book|roman|text|medium|semibold|demibold|bold|extrabold|ultrabold|black|heavy")
               (get weight-names))))

(defn face-id
  "A face is identified by family and style, not by file path.

  The same face is routinely installed twice — once by the system and once by
  a user — and keying on the path shows it as two faces that cannot be told
  apart in the list."
  [{:keys [family style]}]
  [(str family) (normalize-style style)])

(defn entry->item
  [{:keys [family style format glyphs path] :as entry}]
  (let [st (normalize-style style)]
    (item/item (face-id entry)
               :font-face
               (str family " " st)
               {:face (str family " " st)
                :family family
                :style st
                :weight (derive-weight entry)
                :format format
                :glyphs glyphs
                :path path})))

(defn listing->items [entries]
  (mapv entry->item entries))

(def by-family
  "Family, then weight within it — how a specimen sheet reads."
  [[:family :asc] [:weight :asc]])

(def default-query
  {:query/sort by-family :query/text "" :query/filters []})

(defn duplicates
  "Face ids that appeared more than once in the provider's output.

  Surfaced rather than silently deduplicated: two files claiming to be the
  same face is a real condition a font manager exists to show."
  [entries]
  (->> entries
       (map face-id)
       frequencies
       (filter (fn [[_ n]] (> n 1)))
       (mapv key)))
