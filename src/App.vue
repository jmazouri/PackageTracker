<template>
<div id="app">

    <md-toolbar>
        <md-button class="md-icon-button" v-on:click="showSidebar = !showSidebar">
            <md-icon>menu</md-icon>
        </md-button>

        <h2 class="md-title" style="flex: 1">Package Tracker</h2>
    </md-toolbar>

    <md-layout>
        <md-layout v-show="showSidebar" md-flex-xsmall="100" md-flex-small="50" md-flex-medium="35" md-flex-large="25" md-flex-xlarge="20">
            <md-tabs>
                <md-tab md-icon="local_shipping" class="md-dense">
                    <PackageList v-bind:packages="packages" v-on:removePackage="removePackage(index)"></PackageList>
                </md-tab>
                <md-tab md-icon="settings" class="md-dense">
                    <Settings v-bind:packages="packages" v-bind:settings="settings" 
                              v-bind:trackingNumbers="trackingNumbers" v-on:refreshPackages="refreshPackages"></Settings>
                </md-tab>
            </md-tabs>
        </md-layout>

        <md-layout> 
            <GoogleMap v-bind:trackingNumbers="trackingNumbers" v-bind:packages="packages"></GoogleMap>
        </md-layout>
    </md-layout>

</div>
</template>

<script>
var Vue = require('vue');
var VueMaterial = require('vue-material');
var tu = require('tracking-url');

import GoogleMap from './GoogleMap.vue';
import PackageList from './PackageList.vue';
import Settings from './Settings.vue';

Vue.use(VueMaterial);

export default
{
    name: 'app',
    data: function ()
    {
        return {
            packages: [],

            currentStop: null,
            showSidebar: true,
            trackingNumbers: [],

            settings: {
                autoRefresh: false,
                refreshInterval: 0
            }
        }
    },
    mounted: function()
    {
        this.loadFromCache();
    },
    methods:
    {
        refreshPackages: function()
        {
            this.loadFromCache();
        },
        removePackage: function(index)
        {
            this.trackingNumbers.splice(index, 1);
        },
        loadFromCache: function()
        {
            this.packages = [];
            this.trackingNumbers = [];

            var tracking = localStorage["trackingNumbers"];
            if (tracking != "undefined" && tracking !== undefined)
            {
                this.trackingNumbers = JSON.parse(tracking);
            }
        },
        loadData: function()
        {
            var thisVue = this;

            this.trackingNumbers.forEach(track =>
            {
                var carrier = tu(track).name;

                fetch(`https://api.goshippo.com/tracks/${carrier}/${track}`, {
                    method: 'get'
                }).then(function(response)
                {
                    if (!response.ok)
                    {
                        alert("There was error loading package data.");
                        throw Error("No data");
                    }

                    return response.json();
                }).then(function(data)
                {
                    thisVue.packages.push(data);
                });
            });
        },
        stopSelect: function(stop)
        {
            this.currentStop = stop;
        }
    },
    watch:
    {
        trackingNumbers: function(newVal)
        {
            localStorage["trackingNumbers"] = JSON.stringify(newVal);

            this.packages = [];
            this.loadData();
        }
    },
    components:
    {
        GoogleMap, PackageList, Settings
    }
}
</script>

<style lang="scss">
.md-tab.md-dense
{
    padding: 0;
}
</style>
