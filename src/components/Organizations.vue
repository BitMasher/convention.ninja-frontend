<template>
  <table>
    <thead>
    <tr>
      <td>Name</td>
      <td>Select</td>
      <td>Delete</td>
    </tr>
    </thead>
    <tbody>
    <tr v-for="org in organizations" :key="org.id" v-bind:class="{'selected':selectedOrg != null && selectedOrg.id === org.id}">
      <td>{{ org.name }}</td>
      <td>
        <button v-on:click="selectOrganization(org)">Select</button>
      </td>
      <td>
        <button v-on:click="deleteOrganization(org)">Delete</button>
      </td>
    </tr>
    </tbody>
    <tfoot>
    <tr>
      <td><input type="text" v-model="newOrgName"></td>
      <td colspan="2">
        <button v-on:click="createOrg">New</button>
      </td>
    </tr>
    </tfoot>
  </table>
</template>

<script>
export default {
  name: 'Organizations',
  props: ['credentials'],
  data: function () {
    return {
      organizations: [],
      newOrgName: '',
      selectedOrg: null,
    }
  },
  methods: {
    selectOrganization(org) {
      this.selectedOrg = org;
      this.$emit('onOrgSelected', org);
    },
    async deleteOrganization(org) {
      let resp = await fetch('//convention.ninja/api/orgs/' + org.id, {
        method: 'DELETE',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        if (this.selectedOrg != null && this.selectedOrg.id === org.id) {
          this.selectedOrg = null;
        }
        this.$emit('onOrgSelected', null)
      }
      await this.loadOrgs();
    },
    async createOrg(e) {
      e.preventDefault();
      if (this.newOrgName.length === 0) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: this.newOrgName
        })
      });
      if (resp.status < 300) {
        this.newOrgName = '';
        await this.loadOrgs()
      } else {
        this.newOrgName = 'error'
      }
    },
    async loadOrgs() {
      let resp = await fetch('//convention.ninja/api/orgs', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.organizations = await resp.json();
      }
    }
  },
  mounted: function () {
    this.$nextTick(() => this.loadOrgs());
  }
}
</script>

<style scoped>
table {
  border-collapse: collapse;
}
.selected {
  border: 1pt solid blue;
}
</style>