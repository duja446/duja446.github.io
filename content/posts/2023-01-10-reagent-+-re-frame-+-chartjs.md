---
title: "Reagent + Re-Frame + ChartJs"
date: 2023-01-10T13:34:42+01:00
tags: ["clojure", "clojurescript"]
keywords: ["reagent", "re-frame", "chartjs", "charts"]
---

# Charts in Reagent

I wanted to include some charts in my ClojureScript project, and having worked with the [Chart.js](https://www.chartjs.org/) library I decided to use it.
My stack up to that point included `shadow-cljs`, `reagent` and `re-frame`. I had to piece together the solution by reading to the documentation. I think 
I landed on a good solution.

## Including Chart.js

First we need to bring the `chart.js` dependancy to our project. To do that we first consult the [documentation](https://shadow-cljs.github.io/docs/UsersGuide.html#npm-install) 
on installing npm packages. We just need to run `npm install chart.js` and we can that use `chart.js` in our project by requireing it `(require '["chart.js/auto" :as chartjs])`.

## Reagent and Re-frame

Our chart component is stateful which clashes re-frame immutable structure. Many JS libraries are like this so the re-frame team decided on a way to use these 
components is our application. The strategy is outlined in their [documentation](https://day8.github.io/re-frame/Using-Stateful-JS-Components/). Basically 
we need to create an outer component which sources the data, and an inner component which manages the actual state.

### Inner component

Lets start with first writing the inner component. This component is represented as a `reagent class` which is like a `react class`. You can read more about them [here](https://github.com/reagent-project/reagent/blob/master/doc/CreatingReagentComponents.md).
We need decide on four things:
1. function that gets called on mount
2. function that gets called on update
3. display name
4. render function

The general outline of our component would be
```clojure
(defn chart-component-inner []
  (reagent/create-class
    {:component-did-mount ...
     :component-did-update ...
     :display-name ...
     :reagent-render ...}))
```

#### Render function and display name

Our render function is actually just some Hiccup markup. In this case for `chart.js` to function we need to create a `canvas` where out chart will reside. We also need to 
identify this chart with an id. We can just give it some id like `chart-canvas` and this would work if we want to create only one chart. If we need more charts the ids would clash, 
so we need to make this id dynamically and keep track of it in our code. For this purpose we can generate a random string and pass it around. We write this random string function 
and incorporate it into our function using `let`. 

For the display name I decided to use `chart-component-{chart-id}`. 

Now out component looks like this 
```clojure
(defn rand-str [len]
  (apply str (take len (repeatedly #(char (+ (rand 26) 65))))))

(defn chart-component-inner []
  (let [chart-id (rand-str 10)]
   (r/create-class 
    {:component-did-mount ...
     :component-did-update ...
     :display-name (str "chart-component-" chart-id)
     :reagent-render (fn []
                      [:canvas {:id chart-id}])})))
```

#### Mount function

Our mount function need to initialize the component using `chart.js` lifecycle functions and supply it with the initial data. To initialize a `chart.js` component we need a canvas and its id. 
We took care of that in the previous step so our function is now easy to write. Our first try might look something like this:
```clojure
(defn mount-chart [chart-id]
  (let [context (.getContext (.getElementById js/document chart-id) "2d")
        data {:type "line"
              :data {
                     :labels []
                     :datasets [{
                                 :data []
                                 :borderColor "rgb(75, 192, 192)"
                                 }]
                     }
              :options {
                        }}
        chart (chartjs/Chart. context (clj->js data))]
    ))
```

We just use javascript functions to do the job and convert our data structure from clojure to javascript using `clj->js` function. This approach is fine but has two major problems that present themselves 
later. First, we can't supply the initial data so we would need to create a chart and then update it immediately. Second, we can't even update this chart because for that we need the reference to the 
chart we created which we currently have no way of getting from this function.

To solve the first problem we can return a function that does these computations insted of doing them. By doing this we get access to the `this` variable which we can use to get data passed to 
the component. To get that data we can use `reagent/props this`. This will get us all the arguments passed to this component. For example if we created our component like `[chart-component-inner [1 2 3]]`, 
by running `reagent/props this` we would get the vector `[1 2 3]`. We can leverage this plus Clojure destructuring to create a function that can take initial data but can also be created without it. 
Now our function looks like this:
```clojure
(defn mount-chart [chart-id]
  (fn [this] 
    (let [{:keys [labels data] :or {labels [] data []}} (r/props this)
          context (.getContext (.getElementById js/document chart-id) "2d")
          data {:type "line"
                :data {
                       :labels labels
                       :datasets [{
                                   :data data
                                   :borderColor "rgb(75, 192, 192)"
                                   }]
                       }
                :options {
                          }}
          chart (chartjs/Chart. context (clj->js data))]
    )))
```

To solve the second problem we need to pass our `chart` structure back to the caller. We can't do that directly so we can use `reagent atoms` which work in the same way as `clojure atoms`. In our `create-inner-chart` 
function we create this atom and pass it to our `mount-chart` function which uses `reset!` to change it's value.   
Our functions now:
```clojure
(defn chart-component-inner []
  (let [chart-atom (r/atom nil)
        chart-id (rand-str 10)]
   (r/create-class 
    {:component-did-mount (mount-chart chart-atom chart-id)
     :component-did-update ...
     :display-name (str "chart-component-" chart-id)
     :reagent-render (fn []
                      [:canvas {:id chart-id}])})))

(defn mount-chart [chart-atom chart-id]
  (fn [this] 
    (let [{:keys [labels data] :or {labels [] data []}} (r/props this)
          context (.getContext (.getElementById js/document chart-id) "2d")
          data {:type "line"
                :data {
                       :labels labels
                       :datasets [{
                                   :data data
                                   :borderColor "rgb(75, 192, 192)"
                                   }]
                       }
                :options {
                          }}
          chart (chartjs/Chart. context (clj->js data))]
    (reset! chart-atom chart))))
```

#### Update function

To update our chart we need to take data passed to it which we do using `props` and change our state using javascript functions. I decided to pass all data to chart instead of passing it as it arrives. 
This allows us to easily revert the state and not have to deal with more javascript.
```clojure
(defn update-chart [chart-atom]
  (fn [this]
    (let [{:keys [labels data]} (r/props this)
          labels                (clj->js labels)
          data                  (clj->js data)]
      (set! (.. @chart-atom -data -labels) labels)
      (set! (.. @chart-atom -data -datasets (at 0) -data) data)
      (.update @chart-atom))))
```

#### Final

Our final `chart-component-inner` function looks like: 
```clojure
(defn mount-chart [chart-atom chart-id]
  (fn [this] 
    (let [{:keys [labels data] :or {labels [] data []}} (r/props this)
          context (.getContext (.getElementById js/document chart-id) "2d")
          data {:type "line"
                :data {
                       :labels labels
                       :datasets [{
                                   :data data
                                   :borderColor "rgb(75, 192, 192)"
                                   }]
                       }
                :options {
                          }}
          chart (chartjs/Chart. context (clj->js data))]
    (reset! chart-atom chart))))

(defn update-chart [chart-atom]
  (fn [this]
    (let [{:keys [labels data]} (r/props this)
          labels                (clj->js labels)
          data                  (clj->js data)]
      (set! (.. @chart-atom -data -labels) labels)
      (set! (.. @chart-atom -data -datasets (at 0) -data) data)
      (.update @chart-atom))))

(defn chart-component-inner []
  (let [chart-atom (r/atom nil)
        chart-id (rand-str 10)]
   (r/create-class 
    {:component-did-mount (mount-chart chart-atom chart-id)
     :component-did-update (update-chart chart-atom)
     :display-name (str "chart-component-" chart-id)
     :reagent-render (fn []
                      [:canvas {:id chart-id}])})))
```

### Outer component 

For the outer component we can write to subscribe to `re-frame` our just as a fucntion that takes our initial data and passed it to our inner component.
```clojure
(defn chart-component-outer [initial-data]
  [chart-component-inner initial-data])
```
