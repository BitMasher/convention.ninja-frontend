<template>
  <div>
    <span v-if="!organization">No Organization Selected</span>
    <span v-if="organization">
      <button v-on:click="advanced=!advanced">Toggle Advanced View</button>
      <span v-if="advanced">
        <div style="display:table;width:100%;">
          <div style="display: table-row">
            <div style="display: table-cell">
              <fieldset>
                <legend>Categories</legend>
                <table>
                  <thead>
                    <tr>
                      <th>Id</th>
                      <th>Name</th>
                      <th>Action</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in categories" :key="item.id">
                      <td>{{ item.id }}</td>
                      <td>{{ item.name }}</td>
                      <td><button v-on:click="deleteCategory(item)">Delete</button></td>
                    </tr>
                  </tbody>
                  <tfoot>
                    <tr>
                      <td colspan="2"><input type="text" placeholder="name" v-bind:class="{'error':caterr}"
                                             v-model="catname" v-on:input="caterr=false"></td>
                      <td><button v-on:click="addCategory">Save</button></td>
                    </tr>
                  </tfoot>
                </table>
              </fieldset>
            </div>
            <div style="display: table-cell">
              <fieldset>
                <legend>Manufacturers</legend>
                <table>
                  <thead>
                    <tr>
                      <th>Id</th>
                      <th>Name</th>
                      <th>Action</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in manufacturers" :key="item.id">
                      <td>{{ item.id }}</td>
                      <td>{{ item.name }}</td>
                      <td><button v-on:click="deleteManufacturer(item)">Delete</button></td>
                    </tr>
                  </tbody>
                  <tfoot>
                    <tr>
                      <td colspan="2"><input type="text" placeholder="name" v-model="mfgname"
                                             v-bind:class="{'error':mfgerr}" v-on:input="mfgerr=false"></td>
                      <td><button v-on:click="addManufacturer">Save</button></td>
                    </tr>
                  </tfoot>
                </table>
              </fieldset>
            </div>
            <div style="display: table-cell">
              <fieldset>
                <legend>Models</legend>
                <table>
                  <thead>
                    <tr>
                      <th>Id</th>
                      <th>Name</th>
                      <th>Manufacturer</th>
                      <th>Category</th>
                      <th>Action</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in models" :key="item.id">
                      <td>{{ item.id }}</td>
                      <td>{{ item.name }}</td>
                      <td>{{ item.manufacturer.name }}</td>
                      <td>{{ item.category.name }}</td>
                      <td><button v-on:click="deleteModel(item)">Delete</button></td>
                    </tr>
                  </tbody>
                  <tfoot>
                    <tr>
                      <td colspan="2"><input type="text" placeholder="name" v-model="modelname"
                                             v-bind:class="{'error':modelerr}" v-on:input="modelerr=false"></td>
                      <td>
                        <select v-model="modelmfgid" v-on:change="modelerr=false">
                          <option value="">Manufacturer</option>
                          <option v-for="opt in manufacturers" :key="opt.id" v-bind:value="opt.id">{{
                              opt.name
                            }}</option>
                        </select>
                      </td>
                      <td>
                        <select v-model="modelcatid" v-on:change="modelerr=false">
                          <option value="">Category</option>
                          <option v-for="opt in categories" :key="opt.id" v-bind:value="opt.id">{{ opt.name }}</option>
                        </select>
                      </td>
                      <td><button v-on:click="addModel">Save</button></td>
                    </tr>
                  </tfoot>
                </table>
              </fieldset>
            </div>
          </div>
        </div>
        <div>
            <div style="width: 100%">
              <fieldset>
                <legend>Assets</legend>
                <table>
                  <thead>
                    <tr>
                      <th>Id</th>
                      <th>Serial</th>
                      <th>Model</th>
                      <th>Manufacturer</th>
                      <th>Category</th>
                      <th>Tags</th>
                      <th>Action</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in assets" :key="item.id">
                      <td>{{ item.id }}</td>
                      <td>{{ item.serialNumber }}</td>
                      <td>{{ item.model.name }}</td>
                      <td>{{ item.model.manufacturer.name }}</td>
                      <td>{{ item.model.category.name }}</td>
                      <td>{{ item.assetTags.map((e) => e.tagId).join(',') }}</td>
                      <td><button v-on:click="deleteAsset(item)">Delete</button></td>
                    </tr>
                  </tbody>
                  <tfoot>
                    <tr>
                      <td colspan="2"><input type="text" placeholder="serial number" v-model="assetsn"
                                             v-bind:class="{'error':asseterr}" v-on:input="asseterr=false"></td>
                      <td>
                        <select v-model="assetmodelid" v-on:change="asseterr=false">
                          <option value="">Model</option>
                          <option v-for="opt in models" :key="opt.id" v-bind:value="opt.id">{{ opt.name }}</option>
                        </select>
                      </td>
                      <td colspan="3">
                        <input type="text" v-model="assettags" v-bind:class="{'error':asseterr}"
                               v-on:input="asseterr=false">
                      </td>
                      <td><button v-on:click="addAsset">Save</button></td>
                    </tr>
                  </tfoot>
                </table>
              </fieldset>
            </div>
          </div>
        </span>
      <fieldset>
        <legend>Scanner</legend>
        <form @submit="findBarcode">
        Barcode: <input type="text" v-model="sbarcode"><input type="submit" value="Search">
        </form>
        <fieldset v-if="sasset != null">
          <legend>Asset {{ sasset.id ? sasset.barcodesMap ? sasset.barcodesMap : sasset.id : 'not found' }}</legend>
          <form v-on:keydown="disableEnter" @submit="saveAsset">
          Id: {{ sasset.id }}<br>
          Serial: <input type="text" v-model="sasset.serialNumber"><br>
          Model: <select v-model="sasset.modelId">
          <option value="">Select One</option>
          <option v-for="opt in models" :key="opt.id" v-bind:value="opt.id">{{ opt.name }} - {{ opt.manufacturer.name }} - {{
              opt.category.name
            }}</option>
        </select><br>
          Barcodes: <input type="text" v-model="sasset.barcodesMap"><br>
            Location: <select v-model="sasset.roomId">
            <option value="0">Select One</option>
            <option v-for="opt in rooms" :key="opt.id" v-bind:value="opt.id">{{ opt.label }}</option>
          </select><br>
            <input type="submit" value="Save"><br>
            <span v-if="sstatus===1" style="color:#234c23; font-weight: bold;">Successfully Saved</span>
            <span v-if="sstatus===2" style="color: #541a1a; font-weight: bold;">Failed to save</span>
            </form>
        </fieldset>
      </fieldset>
    </span>
  </div>
</template>

<script>
export default {
  name: "Inventory",
  props: ['organization', 'credentials'],
  data: function () {
    return {
      categories: [],
      models: [],
      manufacturers: [],
      assets: [],
      venues: [],
      rooms: [],
      catname: '',
      caterr: false,
      mfgname: '',
      mfgerr: false,
      modelname: '',
      modelcatid: '',
      modelmfgid: '',
      modelerr: false,
      assetsn: '',
      assetmodelid: '',
      assettags: '',
      asseterr: false,
      advanced: false,
      sbarcode: '',
      sasset: null,
      sstatus: 0
    }
  },
  watch: {
    organization: async function () {
      await this.loadManufacturers();
      await this.loadCategories();
      await this.loadModels();
      await this.loadAssets();
      await this.loadVenues();
    }
  },
  methods: {
    disableEnter(e) {
      if (e.key === 'Enter') {
        e.preventDefault();
      }
    },
    async saveAsset(e) {
      e.preventDefault()
      let barcodes = this.sasset.barcodesMap.split(',');
      barcodes = barcodes.filter(e => e.trim().length > 0);
      if (!this.sasset.id) {
        let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets', {
          method: 'POST',
          credentials: 'include',
          headers: {
            Authorization: 'Bearer ' + await this.credentials.getIdToken(),
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            serialNumber: this.sasset.serialNumber,
            modelId: this.sasset.modelId,
            assetTags: barcodes
          })
        });
        if (!(resp.status < 300)) {
          this.sstatus = 2;
          await this.loadAssets();
          return;
        } else {
          let val = await resp.json();
          this.sasset.id = val.id
        }
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets' + this.sasset.id, {
        method: 'PATCH',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          serialNumber: this.sasset.serialNumber,
          modelId: this.sasset.modelId,
          roomId: this.sasset.roomId
        })
      });
      if(resp.status < 300) {
        this.sstatus = 1
      } else {
        this.sstatus = 2
      }
      await await this.loadAssets();
    },
    async findBarcode(e) {
      e.preventDefault()
      this.sstatus = 0;
      if (this.sbarcode.length === 0) {
        return
      }
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets/barcode/' + this.sbarcode, {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.sasset = await resp.json();
        this.sasset.barcodesMap = this.sasset.assetTags.map((e) => e.tagId).join(',')
      } else if (resp.status === 404) {
        this.sasset = {}
      } else {
        this.sasset = {}
        console.error(resp)
      }
    },
    async loadVenues() {
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/venues', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.venues = await resp.json();
        for (let venue of this.venues) {
          resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/venues/' + venue.id + '/rooms', {
            credentials: 'include',
            headers: {
              Authorization: 'Bearer ' + await this.credentials.getIdToken()
            }
          });
          if (resp.status < 300) {
            let rooms = await resp.json();
            for (let i = 0; i < rooms.length; i++) {
              this.rooms.push({
                id: rooms[i].id,
                label: venue.name + " - " + rooms[i].name
              });
            }
          } else {
            console.error(resp);
          }
        }
      } else {
        console.error(resp);
      }
    },
    async loadCategories() {
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/categories', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.categories = await resp.json();
      } else {
        console.error(resp);
      }
    },
    async addCategory() {
      if (this.catname.length === 0) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/categories', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          name: this.catname
        })
      });
      if (resp.status < 300 || this.status === 409) {
        this.catname = '';
        this.caterr = false;
      } else {
        this.caterr = true;
      }
      await this.loadCategories();
    },
    async deleteCategory(cat) {
      if (cat === null || !cat.id) {
        return;
      }
      await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/categories/' + cat.id, {
        method: 'DELETE',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      await this.loadCategories();
    },
    async loadManufacturers() {
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/manufacturers', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.manufacturers = await resp.json();
      } else {
        console.error(resp);
      }
    },
    async addManufacturer() {
      if (this.mfgname.length === 0) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/manufacturers', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          name: this.mfgname
        })
      });
      if (resp.status < 300 || this.status === 409) {
        this.mfgname = '';
        this.mfgerr = false;
      } else {
        this.mfgerr = true;
      }
      await this.loadManufacturers();
    },
    async deleteManufacturer(mfg) {
      if (mfg === null || !mfg.id) {
        return;
      }
      await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/manufacturers/' + mfg.id, {
        method: 'DELETE',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      await this.loadManufacturers();
    },
    async loadModels() {
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/models', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.models = await resp.json();
      } else {
        console.error(resp);
      }
    },
    async addModel() {
      if (this.modelname.length === 0 || this.modelcatid.length === 0 || this.modelmfgid.length === 0) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/models', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          name: this.modelname,
          manufacturerId: this.modelmfgid,
          categoryId: this.modelcatid
        })
      });
      if (resp.status < 300 || this.status === 409) {
        this.modelname = '';
        this.modelmfgid = '';
        this.modelcatid = '';
        this.modelerr = false;
      } else {
        this.modelerr = true;
      }
      await this.loadModels();
    },
    async deleteModel(m) {
      if (m === null || !m.id) {
        return;
      }
      await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/models/' + m.id, {
        method: 'DELETE',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      await this.loadModels();
    },
    async loadAssets() {
      if (this.organization === null) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets', {
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      if (resp.status < 300) {
        this.assets = await resp.json();
      } else {
        console.error(resp);
      }
    },
    async addAsset() {
      if (this.assetmodelid.length === 0) {
        return;
      }
      let resp = await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          serialNumber: this.assetsn,
          modelId: this.assetmodelid,
          assetTags: this.assettags.split(',')
        })
      });
      if (resp.status < 300 || this.status === 409) {
        this.assetsn = '';
        this.assetmodelid = '';
        this.assettags = '';
        this.asseterr = false;
      } else {
        this.asseterr = true;
      }
      await this.loadAssets();
    },
    async deleteAsset(a) {
      if (a === null || !a.id) {
        return;
      }
      await fetch('//convention.ninja/api/orgs/' + this.organization.id + '/inventory/assets/' + a.id, {
        method: 'DELETE',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken()
        }
      });
      await this.loadAssets();
    },
  },
  mounted: function () {
    // this.$nextTick(async () => {
    //   await this.loadCategories();
    //   await this.loadManufacturers();
    //   await this.loadModels();
    // });
  }
}
</script>

<style scoped>
.error {
  border: 2px solid red;
}
</style>