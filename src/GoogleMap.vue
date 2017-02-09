<template>
<md-layout md-flex="100" id="map"></md-layout>
</template>

<script>
var _ = require("lodash");

export default
{
    name: 'googlemap',
    props: ['trackingNumbers', 'packages'],
    data: function()
    {
        return {
            bounds: null,
            markers: [],
            resultsMap: null,
            zoomChangeBoundsListener: null
        }
    },
    mounted: function()
    {
        this.resultsMap = new google.maps.Map(this.$el,
        {
            zoom: 4,
            rotateControl: false,
            streetViewControl: false,
            fullscreenControl: false
        });

        this.updatePoints();
    },
    methods:
    {
        getCoordsForStop: function(stop)
        {
            return localStorage["stopAddr-" + JSON.stringify(stop.location)];
        },
        addMapPoint: function(latLng)
        {
            if (this.bounds == null)
            {
                this.bounds = new google.maps.LatLngBounds();
            }

            this.bounds = this.bounds.extend(latLng);
            
            this.resultsMap.fitBounds(this.bounds);

            if (this.markers.length == 1)
            {
                var thisVue = this;
                this.zoomChangeBoundsListener = google.maps.event.addListenerOnce(thisVue.resultsMap, "idle", function()
                { 
                    thisVue.resultsMap.setZoom(9); 
                });
            }
            else
            {
                if (this.zoomChangeBoundsListener != null)
                {
                    google.maps.event.removeListener(this.zoomChangeBoundsListener);
                }
            }        
        },
        addMapMarker: function(latLng, text)
        {
            var marker = new google.maps.Marker({
                position: latLng,
                map: this.resultsMap,
                icon: {
                    url: 'static/circle.png',
                    size: new google.maps.Size(42, 42),
                    anchor: new google.maps.Point(21, 21),
                    labelOrigin: new google.maps.Point(20, 20)
                },
                label: {
                    text: text,
                    fontSize: (27 - (text.length * 3)) + "px",
                    color: "blue",
                    fontWeight: (text.length == 1 ? "bold" : "normal")
                }
            });

            this.markers.push(marker);
            this.addMapPoint(latLng);
        },
        clearMarkers: function()
        {
            this.markers.forEach(marker =>
            {
                marker.setMap(null);
            });

            this.markers = [];
        },
        updatePoints: function()
        {
            this.clearMarkers();

            var lastStops = _.map(this.packages, (pack, i) => { return {stop: _.last(pack.tracking_history), ind: (i + 1) }});
            
            var groupedStops = _.groupBy(lastStops, stop => JSON.stringify(stop.stop.location));

            var thisVue = this;
            _.each(groupedStops, grp =>
            {
                var thisStop = `${grp[0].stop.location.city}, ${grp[0].stop.location.state} ${grp[0].stop.location.zip}`;
                var stopKey = "stopAddr-" + thisStop;

                if (localStorage[stopKey] == undefined)
                {
                    var geocoder = new google.maps.Geocoder();

                    geocoder.geocode({'address': thisStop}, 
                    function(results, status)
                    {
                        if (status === 'OK')
                        {
                            localStorage[stopKey] = JSON.stringify(results[0].geometry.location);
                            thisVue.addMapMarker(JSON.parse(localStorage[stopKey]), _.map(grp, d => d.ind).join("&"));
                        }
                        else
                        {
                            alert("Uh oh - Google doesn't like you. \"" + status + "\"");
                        }
                    });
                }
                else
                {
                    thisVue.addMapMarker(JSON.parse(localStorage[stopKey]), _.map(grp, d => d.ind).join("&"));
                }
            });
        }
    },
    watch:
    {
        packages: function()
        {
            this.bounds = null;

            this.clearMarkers();
            this.updatePoints();
        }
    }
}
</script>

<style lang="scss">
#map
{
    flex: 1;
    height: 512px;
}
</style>
