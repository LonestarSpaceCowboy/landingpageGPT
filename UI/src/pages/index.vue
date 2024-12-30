<template>
  <!--App and Nav bar-->
  <v-app-bar :elevation="2">
    <template v-slot:prepend>
      <v-app-bar-nav-icon />
      <v-app-bar-title>LandingPage-GPT</v-app-bar-title>
    </template>
  </v-app-bar>

  <!--Custom card with refresh-->
  <custom-feed
    class="feed-cards"
    :userData="userProfile"
    :productData="product"
  />
</template>

<script>
export default {
  mounted() {
    //Get user's IP location
    this.getIPLocation();
  },
  data() {
    return {
      // userProfile: {
      //   name: "Ian Fenwick",
      //   recentSites: ["https://google.com", "https://facebook.com", "https://crypto.com", "https://coinbase.com", "https://github.com", "https://cryptominers.com"],
      // },
      // product: {
      //   type: "savings account",
      //   productName: "High yield savings account",
      //   company: "Bank of America",
      //   offer: "3% APY on all deposits.  Fraud protection and no fees.  Cash back on purchases of electronics and groceries.",
      //   offerURL: "https://www.bankofamerica.com/high-yield-savings&EYJ192374KL/",
      // },
      userProfile: {
        name: "Steven",
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
        offerURL:
          "https://www.merryllynch.com/whole-life&EYJ192374KL/",
      },
    };
  },
  methods: {
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
  width: 50%;
  max-width: 600px;
}
</style>
