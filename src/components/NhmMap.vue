<template>

	<div id="map" class="section no-pad map-container main-section">
	    <h4>Map goes here...</h4>
	</div>

</template>


<script>
	import { eventBus, resourceTypes } from './helpers';
	import nhmservice from '../gateways/nhmservice';
	require('materialize-css/dist/js/materialize');
	
	import GMaps from 'gmaps/gmaps.min.js';

	const defaultType = 'resources';

	//console.log(resourceTypes);

	export default {
		name: 'nhm-map',

		data() {
			const eventIds = {};
			const resources = resourceTypes.RESOURCES.name;
			const events = resourceTypes.EVENTS.name;
			const urgentneeds = resourceTypes.URGENTNEEDS.name;

			eventIds[resources] = 'popup-provider';
			eventIds[events] = 'popup-event';
			eventIds[urgentneeds] = 'popup-urgent-need';

			return {
				resourceType: resourceTypes.RESOURCES.name,
				results: [],
				searchParams: {
					search: null,
					keyword: null,
					service: null,
					startDate: null,
					endDate: null,
					urgentNeedDate: null
				},
				resourceMap: null,
				eventIds
			};
		},
		props: {
			markers: {
				type: Array, 
				required: false
			}
		},
		created() {
		},

		mounted() {
			// Initialize ...
			// load GMaps
			if(GMaps) {
				console.log("...found GMaps, loading map...");
				this.resourceMap = new GMaps({
					div: '#map',
					lat: 36.174465,
					lng: -86.767960,
					zoom: 13
				});
			} else {
				console.log("...could not find GMaps on window object.");
			}
			

		    // Register to listen on eventBus
			eventBus.$on('select-resource-type', resourceType => {
				//console.log('bus: received select-resource-type event. Showing...', resourceType);
				
				this.resourceType = resourceType;

				//console.log(this.results, this.resourceType);
			});

			this.$on('update-markers', markers => {
				console.log('updating markers', this.markers);
			})

			console.log("NhmMap mounted", this.markers);
		},

		updated() {
			//update markers if necessary
			console.log('map should update markers...', this.markers);
		},

		methods: {
			doSearch: function(param) {}
		},

		watch: {
			markers: function(oldMarkers) {
				var self = this;
				//console.log('markers changed in map to: ', this.markers);
				var bounds = new google.maps.LatLngBounds();

				this.resourceMap.removeMarkers();
				var renderMarkers = this.markers.map((val, idx) => {
					//console.log(val);
					let marker = {
					  lat: parseFloat(val.latitude),
					  lng: parseFloat(val.longitude),
					  title: val.provider_name,
					  click: (e) => {
					    console.log('You clicked on ' + val.item_name + '!' + this.eventIds[this.resourceType]);
					    this.$emit(this.eventIds[this.resourceType], val.provider_id, val);
					  },
					  mouseover: (e) => {
					  	Materialize.toast(val.item_name, 1000, 'rounded');
					  }
					};

					if(val.icon) {
						marker.icon = {
						  	scaledSize: new google.maps.Size(30, 30),
							url: val.icon
						};
					}

					bounds.extend(marker);

					return marker;
				});
				//console.log(renderMarkers);
				if(renderMarkers.length > 0) this.resourceMap.fitBounds(bounds);
				//this.resourceMap.setCenter(bounds.getCenter());
				//AIzaSyB6L-gd4MueuPig0CtU6He3nf9lebyfYwI
				this.resourceMap.addMarkers(renderMarkers);
			}
		}
	}

</script>


<style scoped>
	/* Map Container */
	.map-container {
		width: 100vw;
		height: 100vh;
		
		background-color: #999;
	}

</style>