---
layout: map
title: Spoons Map
description: Every Wetherspoon pub in the UK, mapped.
---

<script>
(function () {

  var i = 'https://cdn.jsdelivr.net/gh/Sam12604/wsmd@main/1';

  function makeIcon() {
    return L.divIcon({
      className: '',
      html: '<div class="spoons-marker"></div>',
      iconSize:   [22, 22],
      iconAnchor: [11, 22],
      popupAnchor:[0, -20]
    });
  }

function buildPopup(pub, index) {
  var isOpen    = pub.status && pub.status.toLowerCase().includes('open');
  var statusCls = isOpen ? 'open' : 'closed';
  var statusTxt = isOpen ? 'Open' : 'Closed';
  if (pub.closes) {
    statusTxt += ' · ' + pub.closes;
  }

  var imgHtml = pub.image
    ? '<img class="popup-img" src="' + pub.image + '" alt="' + pub.name + '" loading="lazy">'
    : '<div class="popup-img-placeholder"><div class="plate"></div></div>';

  var mapsUrl  = 'https://www.google.com/maps/dir/?api=1&destination='
                 + pub.lat + ',' + pub.lng;
  var shareUrl = window.location.origin + window.location.pathname + '#pub-' + index;

  return '<div>'
    + imgHtml
    + '<div class="popup-body">'
    +   '<div class="popup-name">'    + pub.name             + '</div>'
    +   '<div class="popup-address">' + (pub.address || '')  + '</div>'
    +   '<div class="popup-status '   + statusCls + '">'     + statusTxt + '</div>'
    +   '<div class="popup-actions">'
    +     '<a class="popup-btn" href="' + mapsUrl + '" target="_blank" rel="noopener">Directions</a>'
    +     '<button class="popup-btn share-btn" onclick="navigator.share ? navigator.share({ title: \'' + pub.name.replace(/'/g, "\\'") + '\', url: \'' + shareUrl + '\' }) : navigator.clipboard.writeText(\'' + shareUrl + '\').then(() => alert(\'Link copied!\'))">Share</button>'
    +   '</div>'
    + '</div>'
    + (pub.url
        ? '<a class="popup-link" href="' + pub.url + '" target="_blank" rel="noopener">View Pub Page ›</a>'
        : '')
    + '</div>';
}

  var ZONE_STYLES = {
    '1':   { color: '#d6336c', weight: 2, fillOpacity: 0.08 },
    '2':   { color: '#1c7ed6', weight: 2, fillOpacity: 0.08 },
    '2/3': { color: '#f59f00', weight: 2, fillOpacity: 0.08 }
  };

  var zonesGeoJSON = {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "properties": { "zone": "2" },
        "geometry": {
          "type": "Polygon",
          "coordinates": [[
            [-0.0090073,51.5187317],[-0.0205118,51.5247033],[-0.0179485,51.533272],
            [-0.0232524,51.5454067],[-0.05707,51.56162],[-0.07317,51.56914],
            [-0.09583,51.57077],[-0.13514,51.56525],[-0.17827,51.5566],
            [-0.22221,51.54919],[-0.259981,51.523538],[-0.25453,51.49514],
            [-0.21663,51.46144],[-0.21114,51.45884],[-0.14801,51.45264],
            [-0.10179,51.45328],[-0.08789,51.45446],[-0.01364,51.46543],
            [-0.01669,51.46882],[-0.02257,51.47446],[-0.0148,51.47822],
            [-0.01133,51.482131],[0.002868,51.4980226],[-0.0041017,51.5086754],
            [-0.0090073,51.5187317]
          ]]
        }
      },
      {
        "type": "Feature",
        "properties": { "zone": "1" },
        "geometry": {
          "type": "Polygon",
          "coordinates": [[
            [-0.202732,51.513657],[-0.19801,51.48956],[-0.1481165,51.4839925],
            [-0.1380859,51.472717],[-0.1225786,51.4788263],[-0.1056539,51.4882749],
            [-0.06866,51.50639],[-0.068579,51.517556],[-0.099821,51.534964],
            [-0.126858,51.534484],[-0.202732,51.513657]
          ]]
        }
      },
      {
        "type": "Feature",
        "properties": { "zone": "2/3" },
        "geometry": {
          "type": "Polygon",
          "coordinates": [[
            [-0.0090073,51.5187317],[-0.0041017,51.5086754],[0.0027907,51.4981738],
            [0.0088083,51.5011829],[0.0057774,51.5053567],[0.0162958,51.5132437],
            [0.0159286,51.519367],[0.0103463,51.5298774],[0.0046804,51.537929],
            [-0.0034616,51.5458229],[-0.0232524,51.5454067],[-0.0179485,51.533272],
            [-0.0205118,51.5247033],[-0.0090073,51.5187317]
          ]]
        }
      }
    ]
  };

  var map = L.map('map', {
    center: [52.4, -1.5],
    zoom: 6,
    zoomControl: true
  });

  map.zoomControl.setPosition('bottomright');

  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a> &copy; <a href="https://carto.com/">CARTO</a>',
    subdomains: 'abcd',
    maxZoom: 20
  }).addTo(map);

var locateBtn = L.control({ position: 'bottomright' });

locateBtn.onAdd = function () {
  var btn = L.DomUtil.create('button', 'locate-btn');
  btn.innerHTML = '📍';
  btn.title = 'Show my location';
  btn.onclick = function () {
    map.locate({ setView: true, maxZoom: 14 });
  };
  return btn;
};

locateBtn.addTo(map);

map.on('locationfound', function (e) {
  L.circleMarker(e.latlng, {
    radius: 8,
    color: '#0e3a6e',
    fillColor: '#c8922a',
    fillOpacity: 1,
    weight: 2
  }).addTo(map).bindPopup('You are here').openPopup();
});

map.on('locationerror', function (e) {
  alert('Location access denied or unavailable.');
});

  L.geoJSON(zonesGeoJSON, {
    style: function (f) {
      return ZONE_STYLES[f.properties.zone] || {};
    },
    onEachFeature: function (f, layer) {
      layer.bindPopup('<strong style="font-family:\'Playfair Display\',serif;color:#0e3a6e">Zone ' + f.properties.zone + '</strong>');
    }
  }).addTo(map);

 
 
 
fetch(i)
  .then(r => {
    if (!r.ok) throw new Error('HTTP ' + r.status);
    return r.text();
  })
  .then(function (raw) {
const x = Uint8Array.from(atob(raw.trim().slice(1)), c => c.charCodeAt(0));
const pubs  = JSON.parse(new TextDecoder('utf-8').decode(x));
    var icon  = makeIcon();
    var count = 0;

    pubs.forEach(function (pub) {
      if (!pub.lat || !pub.lng) return;
      count++;
      L.marker([pub.lat, pub.lng], { icon: icon })
        .addTo(map)
        .bindPopup(buildPopup(pub), { maxWidth: 240 });
    });

    document.getElementById('pub-count').textContent =
      count.toLocaleString() + ' pubs';
    document.getElementById('loading').classList.add('hidden');
  })
  .catch(function (err) {
    console.error(err);
    document.getElementById('loading').classList.add('hidden');
    var el = document.getElementById('error-msg');
    el.style.display = 'block';
    el.textContent   = 'Could not load pub data.';
    document.getElementById('pub-count').textContent = 'Error';
  });

})();
</script>
