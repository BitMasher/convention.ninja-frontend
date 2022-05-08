<template>
  <div id="app">
    <div v-if="!authorized">
      <div v-if="show===''">
        <button v-on:click="doGoogle">Google</button>
        <button v-on:click="show='login'">Login</button>
        <button v-on:click="show='signup'">Sign up</button>
      </div>
      <div v-if="show==='login'">
        <form @submit="doLogin">
          <label for="lun">Username</label> <input v-on:input="lerr=false" type="email" v-model="lun" id="lun"><br>
          <label for="lpw">Password</label> <input v-on:input="lerr=false" type="password" v-model="lpw" id="lpw"><br>
          <input type="submit" value="Login">
          <button v-on:click="show=lun=lpw='';">Cancel</button>
          <br>
          <span class="error" v-if="lerr">Invalid email or password</span>
        </form>
      </div>
      <div v-if="show==='signup'">
        <form @submit="doSignup">
          <label for="sun">Username</label> <input v-on:input="serr=false" type="email" v-model="sun" id="sun"><br>
          <label for="spw">Password</label> <input v-on:input="serr=false" type="password" v-model="spw" id="spw"><br>
          <input type="submit" value="Sign up">
          <button v-on:click="show=sun=spw='';">Cancel</button>
          <br>
          <span class="error" v-if="serr">{{ serrtext }}</span>
        </form>
      </div>
      <div v-if="show==='onboard'">
        <form @submit="doOnboard">
          <label for="oem">Contact Email</label> <input type="email" id="oem" v-model="oem" v-on:input="oerr=false"><br>
          <label for="onm">Name</label> <input type="text" id="onm" v-model="onm" v-on:input="oerr=false"><br>
          <label for="odn">Display Name</label> <input type="text" id="odn" v-model="odn" v-on:input="oerr=false"><br>
          <input type="submit" value="Onboard"><br>
          <span class="error" v-if="oerr">Bad input</span>
        </form>
      </div>
    </div>
    <div v-if="authorized">
      <AuthorizedHome v-bind:credentials="credentials"></AuthorizedHome>
    </div>
  </div>
</template>

<script>

// Import the functions you need from the SDKs you need
import {initializeApp} from "firebase/app";
import {getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword, signInWithPopup, GoogleAuthProvider} from "firebase/auth"
import AuthorizedHome from "@/components/AuthorizedHome";

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyAGPcx9jOMAK-Z65fgDfykoNw06leABeSM",
  authDomain: "convention-ninja.firebaseapp.com",
  projectId: "convention-ninja",
  storageBucket: "convention-ninja.appspot.com",
  messagingSenderId: "656645627437",
  appId: "1:656645627437:web:1d957f0266da7767fce2a4"
};

// Initialize Firebase
initializeApp(firebaseConfig);

const googleProvider = new GoogleAuthProvider();

export default {
  name: 'App',
  components: {AuthorizedHome},
  data: function () {
    return {
      authorized: false,
      show: '',
      lun: '',
      lpw: '',
      lerr: false,
      sun: '',
      spw: '',
      sn: '',
      sdn: '',
      serr: false,
      serrtext: '',
      credentials: null,
      oem: '',
      onm: '',
      odn: '',
      oerr: false
    };
  },
  methods: {
    async doGoogle(e) {
      e.preventDefault();
      try {
        const auth = getAuth();
        let credentials = await signInWithPopup(auth, googleProvider);
        this.credentials = credentials.user;
        let testresp = await fetch('//convention.ninja/api/users/me', {
          credentials: 'include',
          headers: {
            Authorization: 'Bearer ' + await credentials.user.getIdToken()
          }
        })
        if (testresp.status < 300) {
          this.authorized = true;
        } else {
          this.show = 'onboard';
        }
      } catch (e) {
        console.error(e);
      }
    },
    async doLogin(e) {
      e.preventDefault();
      try {
        const auth = getAuth();
        let credentials = await signInWithEmailAndPassword(auth, this.lun, this.lpw);
        this.credentials = credentials.user;
        let testresp = await fetch('//convention.ninja/api/users/me', {
          credentials: 'include',
          headers: {
            Authorization: 'Bearer ' + await credentials.user.getIdToken()
          }
        })
        if (testresp.status < 300) {
          this.authorized = true;
        } else {
          this.show = 'onboard';
        }
      } catch (e) {
        this.lerr = true;
        console.error(e);
      }
    },
    async doSignup(e) {
      e.preventDefault()
      try {
        if (this.sun.length === 0 || this.spw.length < 12) {
          this.serrtext = 'Please fill in all fields appropriately';
          this.serr = true;
          return;
        }
        const auth = getAuth();
        let credentials = await createUserWithEmailAndPassword(auth, this.sun, this.spw);
        console.log(credentials.user.uid);
        this.credentials = credentials.user;
        this.show = 'onboard'
      } catch (e) {
        this.serrtext = 'Invalid username or password';
        this.serr = true;
        console.error(e);
      }
    },
    async doOnboard(e) {
      e.preventDefault()
      if (this.oem.length === 0 || this.onm.length === 0) {
        this.oerr = true;
        return
      }
      if (this.odn.length === 0) {
        this.odn = this.onm;
      }
      let resp = await fetch('//convention.ninja/api/users', {
        method: 'POST',
        credentials: 'include',
        headers: {
          Authorization: 'Bearer ' + await this.credentials.getIdToken(),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          email: this.oem,
          name: this.onm,
          displayName: this.odn
        })
      });
      if (resp.status < 300) {
        this.authorized = true;
      } else {
        this.oerr = true;
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

input[type=email]:invalid {
  border-color: red;
  border-width: 2px;
}

.error {
  color: darkred;
  font-weight: bold;
}
</style>
