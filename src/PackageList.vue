<template>
<md-list style="flex: 1;" class="md-double-line">
    <template v-for="(package, index) in packages">
        <md-list-item class="md-primary">
            <md-avatar>
                <md-chip><strong>{{index + 1}}</strong></md-chip>
            </md-avatar>

            <div class="md-list-text-container">
                <span v-if="packageEdit[index]">
                    <md-input-container>
                        <label>Package Nickname</label>
                        <md-input v-on:input="editTitle(index, $event)"></md-input>
                    </md-input-container>
                </span>

                <span v-else>
                    <strong>{{package.carrier | prettyCarrier}}</strong>

                    <a :href="'https://google.com/search?q=' + package.tracking_number">
                        <template v-if="packageTitles[index] == undefined">{{package.tracking_number}}</template>
                        <template v-else>{{packageTitles[index]}}</template>
                    </a>

                    <br />

                    <span v-if="package.eta != null && Date.parse(package.eta) > new Date()">ETA: {{package.eta | dateFormat}}</span>
                    <strong v-else>Delivered</strong>
                </span>


            </div>

            <md-button class="md-icon-button md-list-action" v-on:click="deletePackage(index)">
                <md-icon>delete</md-icon>
            </md-button>
            <md-button class="md-icon-button md-list-action editButton" v-on:click="toggleEdit(index)">
                <md-icon>mode_edit</md-icon>
            </md-button>
        </md-list-item>

        <span class="packageStops">
            <md-list-item v-for="(stop, index) in grouped(package.tracking_history)" v-if="stop.location.city != ''" class="md-primary locBox">
                <md-avatar>
                    <md-icon>place</md-icon>
                </md-avatar>

                <div class="md-list-text-container">
                    <span class="locName">{{stop.location | prettyLoc}}</span>
                    <span>{{stop.status}} <template v-if="stop.status_details != ''">- {{stop.status_details}}</template><br />{{stop.status_date | dateTimeFormat}}</span>
                </div>
            </md-list-item>
        </span>

        <md-divider></md-divider>
    </template>
    
</md-list>
</template>

<script>
var _ = require("lodash");

export default
{
    name: 'packagelist',
    props: ['packages'],
    data: function ()
    {
        return {
            packageEdit: [],
            packageTitles: []
        }
    },
    mounted: function()
    {
        var storedTitles = localStorage["packageTitles"];

        if (storedTitles != undefined)
        {
            this.packageTitles = JSON.parse(storedTitles);
        }
    },
    watch:
    {
        packages: function()
        {
            this.packageEdit = [];

            _.forEach(this.packages, (pack, index) =>
            {
                this.packageEdit[index] = false;
            });
        }
    },
    methods:
    {
        editTitle: function(index, event)
        {
            this.packageTitles.splice(index, 1, event);

            localStorage["packageTitles"] = JSON.stringify(this.packageTitles);
        },
        deletePackage: function(index)
        {
            this.$emit('removePackage', index);
        },
        grouped: function(history)
        {
            return _.orderBy(_.map(_.groupBy(history, stop => stop.location.zip), grp => _.last(grp)), stop => stop.status_date);
        },
        toggleEdit: function(packIndex)
        {
            this.packageEdit.splice(packIndex, 1, !this.packageEdit[packIndex]);
        }
    },
    filters:
    {
        dateFormat: function (value)
        {
            var d = new Date(value);
            return (d == "Invalid Date" ? "Unknown" : d.toLocaleDateString());
        },
        dateTimeFormat: function (value)
        {
            var d = new Date(value);
            return (d == "Invalid Date" ? "Unknown" : d.toLocaleString());
        },
        prettyCarrier: function (carrier)
        {
            var lookup = 
            {
                ups: "UPS",
                fedex: "FedEx",
                usps: "USPS"
            };

            return lookup[carrier];
        },
        prettyLoc: function(location)
        {
            return `${location.city}, ${location.state} ${location.zip}`;
        }
    }
}
</script>

<style lang="scss">

.locBox:last-child .locName
{
    font-weight: bold;
}

</style>
