<template>
  <div>
    <fieldset>
      <legend>Organization</legend>
      <Organizations v-bind:credentials="credentials" v-on:onOrgSelected="onOrgSelected"></Organizations>
    </fieldset>
    <fieldset>
      <legend>Inventory</legend>
      <Inventory v-bind:credentials="credentials" v-bind:organization="selectedOrg"></Inventory>
    </fieldset>
  </div>
</template>

<script>
import Organizations from "@/components/Organizations";
import Inventory from "@/components/Inventory";

export default {
  name: "AuthorizedHome",
  components: {Organizations,Inventory},
  props: ['credentials'],
  data: function () {
    return {
      selectedOrg: null
    }
  },
  methods: {
    onOrgSelected(org) {
      this.selectedOrg = org
    }
  }
}
</script>

<style scoped>

</style>