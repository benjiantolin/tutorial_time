# Timeseries Chart Tutorial

> Spring 2019 | Geography 4/572 | Geovisual Analytics
>
> **Instructor:** Bo Zhao  **Location:** Wilkinson 210 | **Time:** TR 1600 - 1650 |
> **Tutorial by:** Benjamin Antolin

**Learning Objectives**

- Learn how to format data to effectively
- Learn how to add a dynamic timeseries chart on a leaflet map
- Learn how to filter map features through a time variable

#### JavaScript Libraries
### c3.js
This tutorial will use `c3.js` which is a javascript library that allows you to develop graphs and chart with faily simple syntax. The `c3.js` library is built using the `d3.js` javascript library and is a powerful tool to develop dynamic charts on a website while also having a relativley simple framework to work with.


![](img/chart.png)
### Leaflet

This tutorial will also use the `leaflet` javascript library to render a map and add map interaction with the `c3.js` library.


##### Other Libraries used:
- jQuery
- Google Fonts
- d3.js

### Introduction
This tutorial will use data collected from a survey conducted by [REACH](http://www.reachresourcecentre.info/advanced-search?name_list%5B%5D=JO&field_document_type_tid%5B%5D=6) in the Syrian refugee camp of Zaatari in Jordan. This data contains each individuals date of arrival which makes this dataset a good candidate to be visualized as a timeseries graph.

#### Data
Take a look at the [time.csv](assets/time.csv) file and take note of how the data is formatted. This data has been preprocessed from the raw [survey data](assets/campdata/campdata.csv) of the Zaatari camp to reflect the influx of refugees arriving every month from July 2012 through December 2014.

### Code

##### Initialize a HTML page

As usuall, we will create an empty html page.

```html
<!DOCTYPE html>
    <head>
    </head>
    <body>
    </body>
</html>
```

##### Libraries and CSS Styling
Here in the head of the html will include the following libraries and css styling:

```html
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
<meta name="description" content="">
<meta name="author" content="Benji Antolin">
<title>Zaatari Incoming Refugees</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.4.0/dist/leaflet.css" />
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.8.1/css/all.css">
<link href="https://fonts.googleapis.com/css?family=Titillium+Web" rel="stylesheet">
<link href="https://cdnjs.cloudflare.com/ajax/libs/c3/0.6.14/c3.min.css" rel="stylesheet">
<style>
  html,
  body,
    #map {
      width: 100%;
      height: 100%;
      margin: 0;
      background: #fff;
      font-family: 'Titillium Web', sans-serif;
    }

    #info {
      position: fixed;
      z-index: 1000;
      bottom: 20px;
      width: 768px;
      color: #333333;
      padding: 6px 8px;
      background: rgba(225, 225, 225, 0.5);
      box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
      border-radius: 5px;
    }
    </style>
<script src="https://unpkg.com/d3@5/dist/d3.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/c3/0.6.14/c3.min.js"></script>
<script src="https://unpkg.com/leaflet@1.4.0/dist/leaflet.js"></script>
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>

</head>
```
##### Create placeholders
In the first part of the body section, we will create placeholders for two charts, a play button, and a leaflet map

```html
<div id="info">
  <div id="vis">
    <button id="play-button" type="button">Play</button>
  </div>
  <div id="total" style="min-height: 250px"></div>
  <hr>
  <div id="district" style="min-height: 250px"></div>
</div>
<div id="map" role="main"></div>
```

##### Create a map object and add base map

Here we will create a map object within the `<script>` tags.

```javacript
var mymap = L.map('map', {
    center: [32.293, 36.3125],
    zoom: 15,
    maxZoom: 16,
    minZoom: 11
  });

L.tileLayer('http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}@2x.png').addTo(mymap);
```
##### Add data using d3 promise system

Now we add the data of interest and create variables based on the attributes in the CSV file. Here we create 14 different variables to have access to the time variabe, all twleve districts and the total count. After initializing the variables, we oush each variable to create an array of the embedded data.

```javacript
Promise.all([
      d3.csv('assets/time.csv'),
      d3.json('assets/districts.geojson'),
    ]).then(function(datasets) {
      var t = ["t"];
      var d1 = ["D1"];
      var d2 = ["D2"];
      var d3 = ["D3"];
      var d4 = ["D4"];
      var d5 = ["D5"];
      var d6 = ["D6"];
      var d7 = ["D7"];
      var d8 = ["D8"];
      var d9 = ["D9"];
      var d10 = ["D10"];
      var d11 = ["D11"];
      var d12 = ["D12"];
      var total = ["Total"];

      datasets[0].forEach(function(d) {
        t.push(new Date(d["t"]))
        d1.push(+d["d1"])
        d2.push(+d["d2"])
        d3.push(+d["d3"])
        d4.push(+d["d4"])
        d5.push(+d["d5"])
        d6.push(+d["d6"])
        d7.push(+d["d7"])
        d8.push(+d["d8"])
        d9.push(+d["d9"])
        d10.push(+d["d10"])
        d11.push(+d["d11"])
        d12.push(+d["d12"])
        total.push(+d["total"])

      });
```

##### Charts
Now that the data is loaded into the system, we can use them in the `c3.js` charts. First we create a chart variable and then we can generate the chart. `c3.js` has many functionalities and is very customizable. Take a look at the [`c3.js` documentation](https://c3js.org/reference.html) to see what type of customazation may suit your needs.

###### Chart Total
```javacript
var chart = c3.generate({
    title: {
      text: 'Refugee Arrival (2012/7-2014/12)'
    },
    data: {
        x: 't',
        columns: [t,total],
        type: 'spline',
    },
    subchart: {
    show: true,
    size: {
      height: 15
    },
    onbrush: function(d) {
      chart2.zoom(chart.zoom());
    }
    },
    axis: {
        x: {
            type: 'timeseries',
            tick: {
                format: '%Y-%m'
            }
        },
        y: {
          label: {
              text: '# of Refugees',
              position: 'outer-middle'
          }
        },
    },
    point: {
      r: 0,
      focus: {
        expand: {
          r: 2
        }
      }
    },
    zoom: {
      // rescale: true,+
      enabled: true,
      type: "scroll",
      onzoom: function(d) {
        chart2.zoom(chart.zoom());
        // step();
      }
    },
    stanford: {
    scaleMin: 1,
    scaleMax: 10000,
    scaleFormat: 'pow10',
    padding: {
        top: 15,
        right: 0,
        bottom: 0,
        left: 0
    }
},
      bindto: "#total"
});
```
###### Chart Districts

```javacript
var chart2 = c3.generate({
    title: {
      text: 'Refugee Arrival by District (D)'
    },
    data: {
        x: 't',
        columns: [t, d1, d2, d3, d4, d5, d6,d7, d8, d9, d10, d11, d12],
        type: 'spline',
    },
    axis: {
        x: {
            type: 'timeseries',
            tick: {
                format: '%Y-%m'
            }
        },
        y: {
          tick: {
              format: ''
          },
          label: {
              text: '# of Refugees',
              position: 'outer-middle'
          }
        },
    },
    point: {
      r: 0,
      focus: {
        expand: {
          r: 2
        }
      }
    },
    zoom: {
      enabled: {
        type: "drag"
      },
      onzoomend: function(d) {
        chart.zoom(chart2.zoom());
      }
    },
    tooltip: {
      linked: true
    },
      bindto: "#district"
});
```
##### Add Geojson data onto map
This code allows you to add geojson data to the leaflet map created earlier in the tutorial.

```javacript
districts = L.geoJSON(datasets[1], {
    onEachFeature: function(feature, layer) {
      layer.bindPopup(feature.properties.NAME_EN)
    },
    style: {
      opacity: 0.8,
      fillOpacity: 0,
      weight: 2,
      color: "brown",
      dashArray: '6',
    }
  }).addTo(mymap);
```
