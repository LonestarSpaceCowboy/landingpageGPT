<template>
  <!--App and Nav bar-->
  <v-app-bar :elevation="2">
    <template v-slot:prepend>
      <v-app-bar-nav-icon />
      <v-app-bar-title>LandingPage-GPT</v-app-bar-title>
    </template>
  </v-app-bar>

  <v-row>
    <v-col>
      <!--Custom card with refresh-->
      <custom-feed
        ref="feed"
        class="feed-cards"
        :userData="userProfile"
        :productData="product"
      />
    </v-col>
    <v-col>
      <!--Profile Sandbox to enter custom values-->
      <profile-sandbox
        @submit-profile-info="setProfileInfo"
        :userProfile="userProfile"
      />
    </v-col>
  </v-row>
</template>

<script>
export default {
  mounted() {
    //Get user's IP location
    this.getIPLocation();
  },
  data() {
    return {
      userProfile: {
        name: "Steven Pinker",
        recentSites: [
          "https://google.com",
          "https://facebook.com",
          "https://kleinisdschools.com",
          "https://fiestamart.com",
          "https://EWTN.com",
          "https://oilandgasjobs.com",
        ],
      },
      product: {
        type: "Life Insurance",
        productName: "Whole life insurance",
        company: "Merryl Lynch",
        offer: `Premiums are consistent, unless you want to raise the cash value of your plan.
                The death benefit will be paid to the beneficiary when the coverage ends.
                Your policy builds cash at a constant rate, tax-free in a secure account.
                You do not need to choose a term length – your life insurance coverage lasts your whole life.
                You may be able to access the cash value of your plan before it expires.`,
        offerURL: "https://www.merryllynch.com/whole-life&EYJ192374KL/",
      },
    };
  },
  methods: {
    async setProfileInfo(profileInfo) {
      this.userProfile = {
        name: profileInfo.name,
        recentSites: profileInfo.recentSites.split(","),
        location: profileInfo.location,
      };
      await this.$nextTick();
      this.$refs.feed.regenHandler();
    },
    //Get the user's IP location
    async getIPLocation() {
      let lat = 0;
      let lon = 0;

      //Check if geolocation is supported
      if (navigator.geolocation) {
        const getCoords = () => {
          return new Promise((resolve, reject) => {
            navigator.geolocation.getCurrentPosition(
              (position) => {
                lat = position.coords.latitude;
                lon = position.coords.longitude;
                resolve({ lat, lon });
              },
              (error) => reject(error)
            );
          });
        };

        //Get the user's location
        try {
          //Assign to userProfile
          const coords = await getCoords();
          this.userProfile.location = await this.coordsToLocation(coords);
        } catch (error) {
          console.log("Error getting coordinates:", error);
        }
      } else {
        console.log("Geolocation is not supported by this browser.");
      }
    },
    //Convert Latitude and Longitude coordinates to a human readable location
    async coordsToLocation(coords) {
      const location = await fetch(
        `https://nominatim.openstreetmap.org/reverse?lat=${coords.lat}&lon=${coords.lon}&format=json`
      )
        .then((response) => response.json())
        .then((location) => {
          return {
            city: location.address.city || location.address.county,
            country: location.address.country,
            lat: coords.lat,
            lon: coords.lon,
          };
        })
        .catch((error) => console.error("Error:", error));

      return location;
    },
  },
};
</script>

<style>
.feed-cards {
  padding: 1em;
  margin: 1em;
  width: 80%;
  max-width: 600px;
}
</style>
