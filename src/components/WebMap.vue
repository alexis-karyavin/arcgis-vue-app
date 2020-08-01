<template>
    <div></div>
</template>

<script>
import { loadModules } from 'esri-loader';

export default {
    name: 'web-map',
    mounted() {
        // lazy load the required ArcGIS API for JavaScript modules and CSS
        loadModules(['esri/Map', 'esri/views/MapView', 'esri/widgets/Search'], { css: true })
            .then(([ArcGISMap, MapView, Search]) => {
                const map = new ArcGISMap({
                    basemap: 'topo-vector'
                });

                this.view = new MapView({
                    container: this.$el,
                    map: map,
                    center: [-118, 34],
                    zoom: 8
                });
                this.search = new Search({
                    view: this.view
                });

                this.view.ui.add(this.search, "top-right");

                this.view.on("click", (evt) => {
                    this.search.clear();
                    this.view.popup.clear();
                    if (this.search.activeSource) {
                        var geocoder = this.search.activeSource.locator; // World geocode service
                        var params = {
                            location: evt.mapPoint
                        };
                        geocoder.locationToAddress(params).then(
                         (response) => {
                            // Show the address found
                            var address = response.address;
                            this.showPopup(address, evt.mapPoint);
                        },
                         (err) => {
                            // Show no address found
                            this.showPopup("No address found.", evt.mapPoint);
                        }
                        );
                    }
                    });
            });

            
    },
    beforeDestroy() {
        if (this.view) {
            // destroy the map view
            this.view.container = null;
        }
    },

    methods: {
        showPopup: function(address, pt) {
            this.view.popup.open({
                title:
                +Math.round(pt.longitude * 100000) / 100000 +
                "," +
                Math.round(pt.latitude * 100000) / 100000,
                content: address,
                location: pt
            });
        }
    }
};

</script>

<style scoped>
    div {
        padding: 0;
        margin: 0;
        width: 100%;
        height: 100%;
    }
</style>