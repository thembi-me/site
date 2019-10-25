---
layout: page
title: Weather where we are
---

<iframe id="iframe" frameborder="0" width="100%" height="100%" style="width: 100%; height: 100%; min-height: 600px"></iframe>

<script type="text/javascript" src="{{ site.baseurl }}/public/data.js"></script>
<script type="text/javascript">

var points = data.destinations[data.destinations.length - 1].points;
var point = points[points.length - 1];

var url = "https://embed.windy.com/?" + point.lat + "," + point.lng + ",6,menu,marker";

var iframe = document.getElementById("iframe");
iframe.src = url;
</script>
