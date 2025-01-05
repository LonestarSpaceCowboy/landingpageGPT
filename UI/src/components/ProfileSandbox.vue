<template>
  <v-card>
    <!--TITLE-->
    <v-card-title>Profile Sandbox</v-card-title>
    <v-card-text
      >Play with customer variables and get your custom ad banner
      text</v-card-text
    >
    <v-divider />

    <!--CUSTOM PROFILE FORM-->
    <v-sheet class="mx-auto" max-width="600">
      <v-form validate-on="submit lazy" @submit.prevent="submit">
        <v-text-field
          :placeholder="name"
          v-model="name"
          label="Full Name"
        />
        <v-text-field
          placeholder="e.g. http://www.autotrader.com, http://nasdaq.com..."
          v-model="recentSites"
          label="Recent Websites visited, separated by comas"
        />
        <v-text-field
          placeholder="Enter a location, "
          v-model="location"
          label="Location e.g. City, State..."
        />

        <v-btn text="Submit" type="submit" block></v-btn>
      </v-form>
    </v-sheet>
  </v-card>
</template>

<script>
export default {
  props: {
    userProfile: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      name: this.userProfile?.name || "",
      recentSites: this.userProfile?.recentSites?.join(", ") || "",
      location: this.userProfile?.location || "",
    };
  },
  methods: {
    submit() {
      //Send profile data to parent component.
      this.$emit("submit-profile-info", {
        name: this.name,
        recentSites: this.recentSites,
        location: this.location,
      });
    },
  },
};
</script>
