<template>
<md-list style="flex: 1;" class="md-dense">
    <md-list class="md-dense">
        <md-list-item>
            <md-input-container>
                <label>Add package</label>
                <md-input v-model="newTrackingNumber"></md-input>
            </md-input-container>

            <md-button class="md-icon-button md-list-action" v-on:click="addPackage">
                <md-icon class="md-primary">local_shipping</md-icon>
            </md-button>
        </md-list-item>
        <md-list-item>
            <md-input-container>
                <label>Automatic refresh (minutes)</label>
                <md-input v-model="settings.refreshInterval" type="number" min="30"></md-input>
            </md-input-container>
            <md-switch id="autoRefresh" name="settings.autoRefresh" v-model="settings.autoRefresh"></md-switch>
        </md-list-item>
    </md-list>
</md-list>
</template>

<script>
export default
{
    name: 'settings',
    props: ['trackingNumbers', 'packages', 'settings'],
    data: function ()
    {
        return {
            newTrackingNumber: "",
            autoRefreshTimeout: null
        }
    },
    watch:
    {
        'settings.autoRefresh': function(newVal)
        {
            localStorage["autoRefresh"] = newVal;
            this.startAutoRefresh();
        },
        'settings.refreshInterval': function(newVal)
        {
            localStorage["refreshInterval"] = newVal;
        }
    },
    mounted: function()
    {
        this.settings.refreshInterval = parseInt(localStorage["refreshInterval"]);

        if (this.settings.refreshInterval < 3000 || isNaN(this.settings.refreshInterval))
        {
            this.settings.refreshInterval = 3000;
        }

        this.settings.autoRefresh = localStorage["autoRefresh"] == "true";
    },
    methods:
    {
        addPackage: function()
        {
            if (this.newTrackingNumber.trim() === "")
            {
                alert("Please enter a tracking number.");
            }
            else
            {
                this.trackingNumbers.push(this.newTrackingNumber);
                this.newTrackingNumber = "";
            }
        },
        startAutoRefresh: function()
        {
            var thisVue = this;

            if (this.settings.autoRefresh)
            {
                clearTimeout(this.autoRefreshTimeout);

                this.autoRefreshTimeout = 
                    setInterval(function()
                    {
                        thisVue.$emit('refreshPackages');
                    }, this.settings.refreshInterval * 60000);
            }
            else
            {
                clearInterval(this.autoRefreshTimeout);
            }
        }
    },
    filters:
    {
        
    }
}
</script>

<style lang="scss">

</style>
