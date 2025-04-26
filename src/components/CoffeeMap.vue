<template>
    <div class="max-w-4xl mx-auto p-4 space-y-8">
      <!-- Map Container -->
      <div ref="map" class="w-full h-80 rounded-lg shadow-md overflow-hidden"></div>
  
      <div  class="space-y-4">
        <h3 class="text-2xl font-semibold text-gray-800">Coffee Shops Nearby:</h3>
        <ul class="space-y-4">
          <li
            v-for="(coffeeShop, index) in coffeeShops"
            :key="index"
            class="bg-white rounded-lg shadow-lg p-4 hover:shadow-xl transition-shadow cursor-pointer"
            @click="zoomToCoffeeShop(coffeeShop)"
          >
            <h4 class="text-xl font-medium text-gray-900">{{ coffeeShop.name }}</h4>
            <p class="text-gray-600"><strong>Address:</strong> {{ coffeeShop.address }}</p>
            <p class="text-gray-600"><strong>Rating:</strong> {{ coffeeShop.rating || 'N/A' }}</p>
          </li>
        </ul>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        coffeeShops: [],
        map: null,
        markers: [],
      };
    },
    mounted() {
      this.loadGoogleMapsScript();
    },
    methods: {
      loadGoogleMapsScript() {
        if (window.google && window.google.maps) {
          this.initMap();
          return;
        }
  
        const script = document.createElement('script');
        script.src = `https://maps.googleapis.com/maps/api/js?key=AIzaSyBobc7CkConP41TTnFs-F7RTkGrRdT2s7Y&libraries=places&callback=initMap`;
        script.async = true;
        window.initMap = this.initMap;
        document.head.appendChild(script);
      },
  
      initMap() {
        if (this.map) return;
  
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(
            (position) => {
              const userLatLng = {
                lat: position.coords.latitude,
                lng: position.coords.longitude,
              };
              this.createMap(userLatLng);
              this.searchNearbyCoffee(userLatLng);
            },
            () => {
              const fallback = { lat: 40.7128, lng: -74.0060 }; // NYC fallback
              this.createMap(fallback);
              this.searchNearbyCoffee(fallback);
              alert('Geolocation failed. Showing default location.');
            }
          );
        } else {
          const fallback = { lat: 40.7128, lng: -74.0060 };
          this.createMap(fallback);
          this.searchNearbyCoffee(fallback);
          alert('Geolocation not supported.');
        }
      },
  
      createMap(center) {
        this.map = new google.maps.Map(this.$refs.map, {
          center,
          zoom: 14,
        });
  
        new google.maps.Marker({
          position: center,
          map: this.map,
          draggable: true,
          title: 'You are here',
        });
      },
  
      searchNearbyCoffee(location) {
        const service = new google.maps.places.PlacesService(this.map);
  
        const request = {
          location,
          radius: 5000,
          keyword: 'coffee',
        };
  
        service.nearbySearch(request, (results, status) => {
          if (status === google.maps.places.PlacesServiceStatus.OK) {
            this.coffeeShops = [];
            this.markers = [];
  
            results.forEach((place) => {
              const marker = new google.maps.Marker({
                position: place.geometry.location,
                map: this.map,
                title: place.name,
              });
  
              this.markers.push(marker);
  
              this.coffeeShops.push({
                name: place.name,
                address: place.vicinity,
                rating: place.rating,
                position: place.geometry.location,
              });
            });
          } else {
            this.coffeeShops = [];
            alert('No coffee shops found nearby.');
          }
        });
      },
  
      zoomToCoffeeShop(coffeeShop) {
        const targetLat = coffeeShop.position.lat();
        const targetLng = coffeeShop.position.lng();
  
        const marker = this.markers.find((m) => {
          const pos = m.getPosition();
          return pos.lat() === targetLat && pos.lng() === targetLng;
        });
  
        if (marker) {
          this.map.setCenter(marker.getPosition());
          this.map.setZoom(16);
          marker.setAnimation(google.maps.Animation.BOUNCE);
          setTimeout(() => marker.setAnimation(null), 1400);
        }
      },
    },
  };
  </script>
  
  <style scoped>
  /* Add any custom styles here */
  </style>
  