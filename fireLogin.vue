<template>
  <div>
    <div v-show="user.uid">
      <button @click="logout">logout</button>
    </div>
    <div class="login" v-show="!user.uid">
      <div class="firebaseui"></div>
    </div>
  </div>
</template>
<script>
import * as firebase from "firebase/app";
import "firebase/auth";
import * as firebaseui from "firebaseui";
import "firebaseui/dist/firebaseui.css";
let user = {
  uid: "",
  email: "",
  photoURL: "",
  displayName: "",
  token: {claims: {}},
  emailVerified: false,
};

if (!firebase.apps.length) {
  firebase.initializeApp({
    projectId: "clipqr",
    messagingSenderId: "418149143423",
    storageBucket: "clipqr.appspot.com",
    authDomain: "clipqr.firebaseapp.com",
    databaseURL: "https://clipqr.firebaseio.com",
    apiKey: "AIzaSyCfuOeKS1yNk179hoYrpvvoXORJbkj0hXQ",
  });
}

let url = window.location.protocol + "//" + window.location.hostname + ":" + window.location.port;
export default {
  name: "fireLogin",
  data: function() {

    const Providers = Object.keys(firebase.auth).reduce((a, p) => {
      if (p.indexOf("Provider") !== -1) {
        const provider = firebase.auth[p].PROVIDER_ID
        if (provider && (this.providers.indexOf(provider) !== -1 ||
            this.providers.indexOf(provider + ".com") !== -1) ) {
          a.push(firebase.auth[p].PROVIDER_ID);
        } else {
          console.log(provider, " is also available.")
        }
      }
      return a;
    }, []);
    let uiConfig = {
      signInSuccessUrl: url + "/" + this.signInSuccessUrl,
      signInOptions: Providers,
      tosUrl: "/" + "tos",
      privacyPolicyUrl: "/" + "privacyPolicy"
    };
    
    return {
      user, 
      tosUrl: "/" + "tos",
      privacyPolicyUrl: "/" + "privacyPolicy", 
      uiConfig
    };
  },
  props: {
    providers: {
      type: String,
      default: "",
    },
    signInSuccessUrl: String,
  },
  mounted: function() {
    firebase.auth().onAuthStateChanged(
      async theUser => {
        if (theUser) {
          user.displayName = theUser.displayName;
          user.email = theUser.email;
          user.emailVerified = theUser.emailVerified;
          user.photoURL = theUser.photoURL;
          user.uid = theUser.uid;
          user.token = await firebase.auth().currentUser.getIdTokenResult()
          .catch(err => {
            console.error(err);
          });

          // Object.assign(user, theUser)
        } else {
          // User is signed out.
          // console.log('signed out')
          user.displayName = "";
          user.email = "";
          user.emailVerified = false;
          user.photoURL = "";
          user.uid = "";
          this.initAuthUI();
        }
        this.$emit("authStateChanged", user);
      },
      error => {
        console.log(error);
      }
    );
  },
  methods: {
    initAuthUI() {
      // Initialize the FirebaseUI Widget using Firebase.
      const ui = new firebaseui.auth.AuthUI(firebase.auth());
      // The start method will wait until the DOM is loaded.
      ui.start(".firebaseui", this.uiConfig);
    },
    logout() {
      firebase.auth().signOut().then(() => {
        this.$emit("signOut");
        // Sign-out successful.
        // window.location.reload();
      },
      error => {
        // An error happened.
        console.log(error);
        // window.location.reload();
      });
    }
  }
};
</script>
