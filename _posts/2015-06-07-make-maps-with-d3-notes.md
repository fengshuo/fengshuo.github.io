---
title: 'Notes: Make Maps with D3'
categories:
- Frontend
- Web
tags:
- d3
- map
description: make a map with d3.js
date: 2015-06-07 22:48
author_profile: true
classes: wide
---

So I went through 2 amazing tutorials from Mike on make maps w/ d3, [Let's Make a Map](http://bost.ocks.org/mike/map/) and [Let's Make a Bubble Map](http://bost.ocks.org/mike/bubble-map/), then I started to realize how many steps it takes to draw a map, which was quite eye-opening in some way. Here in this blog I'd like to write down my thoughts on new tools, functions, properties as well as what I think I should dig deeper later.

For Starters, as I am getting to know more layouts or charts in d3, I've realized that *find data, clean data, reorganize data* is very critical, be it a chord diagram or a map. I have to say once you get the data in right format, rendering them on browser won't take too much time. That's why I think below material should be helpful:

- [Learn JS Data](http://learnjsdata.com/)
- [Classic Algorithms Implemented in JavaScript](http://algorithmsjs.org/)
- [Data Science from Scratch](http://shop.oreilly.com/product/0636920033400.do)  - [The Elements of Data Analytic Style](https://leanpub.com/datastyle)


# Map Data

Apart from explanations on official api documents, this book - [Learning D3.js Mapping](https://www.packtpub.com/web-development/learning-d3js-mapping) is a nice complementary material, especially the contents in chapter1, chapter4, and chapter6.

So what is the map data exactly? Basically in d3.js charts, we will be mainly dealing with shapefiles, GeoJSON, and TopoJSON. In short, shapefiles can be considered as the 'raw' data contains polygons and lines that represent geographic boundaries, however, shapefiles are in binary format and can be very large. Fortunately, we can convert them into GeoJSON and TopoJSON, there two formats, by their nature, are easy to integrated with web development since they are compatible with JavaScript.

Having said that, although we won't be using shapefiles directly in data viz, we still need to understand the structure of the shp file so that we can convert them properly. That is why I think it is necessary to install a shp file viewer app, in my case, I use [QGIS](https://www.qgis.org/en/site/forusers/download.html) to view shapefiles, you can follow the steps here to install it, but just a reminder, it requires a few other dmg files installed, such as [GDAL](http://www.kyngchaos.com/software/frameworks#gdal_complete) and [Matplotlib](http://www.kyngchaos.com/software/python) (you need to download the dmg file).

Let's get back to GeoJSON and TopoJSON, the difference between them can be seen [here](https://github.com/mbostock/topojson/wiki). Now let's install the tools:

`brew install gdal //you might need to update brew to successfully install gdal`
`npm install -g topojson`

There are many commands of topojson to handle data, which will be frequently used, check it out [here](https://github.com/mbostock/topojson/wiki/Command-Line-Reference).

# Render Data

To render a map in browser, a `projection` and a `path generator` are required. A `projection` converts spherical coordinates(3D) to cartesian cooridnates(2D), there are a few standard projections available in d3, such as `d3.geo.albersUsa()`, `d3.geo.metcator()`. On the other hand, a `path generator` is similar to arcGenerators and chordGenerators, it generate `d` values.

A typical snippet of drawing map looks like this:

```js
svg.append("path")
      .datum(topojson.feature(uk, uk.objects.subunits))
      .attr("d", d3.geo.path().projection(d3.geo.mercator()));
      // as you can see, topojson plays a key role in manipulate data.
```

# Better Prepare Data Beforehand

While it is okay to process data in terms of the projection of it, it seems better to handle these kind of issue beforehand with topojson, for example:

`topojson -o us.json --projection='width=960, height=600, d3.geo.albersUsa().scale(1280).translate([width/2,height/2])' --simplify=0.5 -- counties='./us/gz_2010_us_050_00_20m.shp'`

In this manner, the data is converted, quantized, projected before loaded into d3.

# Boundaries

Boundaries between places are indispensable to any map. We can use `topojson.mesh` to compute boundaries, one thing to point out is that, in GeoJSON, a boundary between two places is described twice. So how to use `topojson.mesh`? What you need to know is that this method requires 2 arguments, the topology and a constituent geometry object, and an optional filter, taking argument a and b representing the two features on either side of the boundary, for exterior boundaries such as coastlines, a and b are the same, therefore by filtering on `a===b` or `a !===b`, we can get exterior or interior boundaries:

```js
svg.append("path")
    .datum(topojson.mesh(uk, uk.objects.subunits, function(a, b) {
      return a !== b && a.id !== "IRL";
    }))
    .attr("d", path)
    .attr("class", "nonIRL-boundary");
```

# Labeling Starters

There is a method `centroid` in arc and polygons which can be used to generate labels for arcs etc. A similar function in map is `path.centroid`, you can use it to label state or country, or display a symbol map.

# Merge Data

In the bubble map tutorial, we need to merge a geographic data with population data so that we can display the population bubbles accordingly, that's where the TopoJSON's `--external-properties` feature came to help, it works like a data join in a relational database - using a primary key to combine two sets of data.

# Merge Boundaries with topojson-merge

Once you have topojson installed, there is another command available, that is `topojson-merge`, in this case, it can merge counties within the same state by using the unique code for each datum:

`topojson-merge -o states.json --in-object=counties --out-object=states --key='d.id.substring(0,2)' -- './counties.json'`

Similarly, you can use `topojson-merge` to generate a national boundary by merging states.

If you look at the final data in browser, the object looks like this:
```raw
objects:{
  counties:Object,
  states:Object,
  nation:Object
}
```

As you can see, now we can use different layers of data for different purpose, specifically:

```js
svg.append("path")
    .datum(topojson.feature(us, us.objects.nation))
    // to draw the nation
    .attr("class", "land")
    .attr("d", path);

svg.append("path")
    .datum(topojson.mesh(us, us.objects.states, function(a, b) { return a !== b; }))
    // only draw the borders between states
    .attr("class", "border border--state")
    .attr("d", path);
```

# Makefile

As Mike mentioned in his blog - [Why Use Make], improving/documenting workflow with Make can be life saver sometimes. As far as I'm concerned, the syntax of make is a little bit difficult to master, I am thinking to find alternatives to make when I start focusing on improving the workflow. But for now, I am going to keep focusing on d3.js itself.

To recap, drawing a map w/ d3 isn't as simple as other types of chart, retrieving and modifying geo data is crucial and time-consuming as well, but to be honest, data is messier in reality, so you'd better be prepared for data cleaning before rendering them into browser.
