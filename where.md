---
layout: page
title: Where we've been
---

<link rel="stylesheet" href="{{ site.baseurl }}public/where/openlayers/ol.css">
<link rel="stylesheet" href="{{ site.baseurl }}public/where/ol3-popup.css" type="text/css">

<div id="map" class="map" style="height: 100%; width: 100%; min-height: 600px;"></div>
<script src="{{ site.baseurl }}public/where/openlayers/ol.js"></script>
<script src="{{ site.baseurl }}public/where/jquery.min.js" type="text/javascript"></script>
<script src="{{ site.baseurl }}public/where/bootstrap.min.js" type="text/javascript"></script>
<script src="{{ site.baseurl }}public/where/ol3-popup.js" type="text/javascript"></script>
<script type="text/javascript" src="{{ site.baseurl }}public/data.js"></script>

<script type="text/javascript">
function createMap() {
  var map = new ol.Map({
    target: 'map',
    layers: [
      new ol.layer.Tile({
        source: new ol.source.Stamen({
          layer: 'watercolor'
        })
      })
    ],
    view: new ol.View({
      center: ol.proj.transform([data.center.lng, data.center.lat], 'EPSG:4326', 'EPSG:3857'),
      zoom: data.zoom
    })
  });

  return map;
}

function createPathLayer() {
  var styles = [
    new ol.style.Style({
      stroke: new ol.style.Stroke({
        color: 'black',
        width: 4
      }),
      fill: new ol.style.Fill({
        color: 'rgba(0, 0, 255, 0.1)'
      })
    }),
    new ol.style.Style({
      image: new ol.style.Circle({
        radius: 3,
        fill: new ol.style.Fill({
          color: 'orange'
        })
      }),
      geometry: function(feature) {
        var coordinates = feature.getGeometry().getCoordinates()[0];
        return new ol.geom.MultiPoint(coordinates);
      }
    })
  ];

  var features = [];

  function createCoordinates(aData) {
    var coordinates = [];

    for (var i = 0; i < aData.points.length; ++i) {
      coordinates.push(ol.proj.transform([aData.points[i].lng, aData.points[i].lat], 'EPSG:4326', 'EPSG:3857'));
    }

    return coordinates;
  }

  for (var i = 0; i < data.destinations.length; ++i) {
    var coordinates = createCoordinates(data.destinations[i]);

    features.push(new ol.Feature({
      geometry: new ol.geom.MultiLineString([coordinates])
    }));
  }

  var vectorLayer = new ol.layer.Vector({
    source: new ol.source.Vector({
        features: features
    }),
    style: styles
  });

  return vectorLayer;
}

function createMarkerLayer() {
  function createMarker(aPoint) {
    var coordinate = ol.proj.transform([aPoint.lng, aPoint.lat], 'EPSG:4326', 'EPSG:3857');

    var iconFeature = new ol.Feature({
      geometry: new ol.geom.Point(coordinate),
      name: aPoint.place,
      country: aPoint.country
    });

    var iconStyle = new ol.style.Style({
      image: new ol.style.Icon(/** @type {olx.style.IconOptions} */ ({
        anchor: [0.85, 46],
        anchorXUnits: 'fraction',
        anchorYUnits: 'pixels',
        opacity: 0.75,
        src: '{{ site.baseurl }}public/where/marker.png'
      }))
    });

    iconFeature.setStyle(iconStyle);
    return iconFeature;
  }

  var features = [];
  for (var i = 0; i < data.destinations.length; ++i) {
    for (var j = 0; j < data.destinations[i].points.length; ++j) {
      features.push(createMarker(data.destinations[i].points[j]));
    }
  }

  var vectorSource = new ol.source.Vector({
    features: features
  });

  var vectorLayer = new ol.layer.Vector({
    source: vectorSource
  });

  return vectorLayer;
}

function setupPopup(map) {
  var popup = new ol.Overlay.Popup();
  map.addOverlay(popup);

  map.on('singleclick', function(evt) {
    var pixel = map.getEventPixel(evt.originalEvent);
    var feature = map.forEachFeatureAtPixel(pixel, function(feature, layer) {
      return feature;
    });

    if (feature && feature.get('name')) {
      var prettyCoord = ol.coordinate.toStringHDMS(ol.proj.transform(feature.getGeometry().getCoordinates(), 'EPSG:3857', 'EPSG:4326'), 2);
      var html = '<div><h2>' + feature.get('name') + '</h2><p>Country: <strong>' + feature.get('country') + '</strong><br />Coords: <strong>' + prettyCoord + '</strong></p></div>';
      popup.show(evt.coordinate, html);
    } else {
      popup.hide();
    }
  });

  map.on('pointermove', function(evt) {
    if (!map.hasFeatureAtPixel(evt.pixel)) {
      map.getTargetElement().style.cursor = '';
      return;
    }

    var pixel = map.getEventPixel(evt.originalEvent);
    var feature = map.forEachFeatureAtPixel(pixel, function(feature, layer) {
      return feature;
    });

    if (feature && feature.get('name')) {
      map.getTargetElement().style.cursor = 'pointer';
      return;
    }

    map.getTargetElement().style.cursor = '';
  });
}

var map = createMap();
map.addLayer(createPathLayer());
map.addLayer(createMarkerLayer());
setupPopup(map);

</script>
