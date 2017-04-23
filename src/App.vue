<template>
  <div id='app'>
    <div v-if="ready">
      <div v-if="authorized">
        <img :src="profile.photoURL" alt="">
        <h1>{{ profile.displayName }}</h1>
        <button type="button" @click="getLocation()">Check in</button>
        <button type="button" @click="logout()">Logout</button>
        <hr>
        <ul>
          <li v-for="todo in todos">
            {{ todo.text }}
          </li>
        </ul>
        <input type="text" v-model="todoInput">
        <button @click="addTodo()">Add</button>
      </div>
      <div v-else>
        <h1>Not Authorized</h1>
        <button type="button" @click="login()">Login</button>
      </div>
    </div>
    <div v-else>
      <h1>Loading...</h1>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase'
import GeoFire from 'geofire'
var firebaseApp = firebase.initializeApp({
  apiKey: 'AIzaSyBmoAs87kv7vDcJ_SJeXvtggCtG0_UEpP8',
  authDomain: 'learn-firebase-a0a71.firebaseapp.com',
  databaseURL: 'https://learn-firebase-a0a71.firebaseio.com',
  storageBucket: 'learn-firebase-a0a71.appspot.com',
  messagingSenderId: '629674258793'
})
var db = firebaseApp.database()

var provider = new firebase.auth.FacebookAuthProvider()

// GeoFire //
var firebaseRef = firebase.database().ref('locations')
var geoFire = new GeoFire(firebaseRef)

export default {
  name: 'app',
  data () {
    return {
      todoInput: '',
      xxx: 'xxx',
      profile: {},
      ready: false,
      authorized: false,
      todos: []
    }
  },
  methods: {
    getLocation () {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.addLocation)
      } else {
        console.log('Geolocation is not supported by this browser.')
      }
    },
    addLocation (position) {
      var location = [position.coords.latitude, position.coords.longitude]
      var vm = this
      geoFire.set(vm.profile.uid, location).then(function () {
        console.log(vm.profile.displayName + ' initially set to [' + location + ']')
      })
    },
    addTodo () {
      this.$firebaseRefs.todos.push({
        text: this.todoInput
      })
    },
    login () {
      firebase.auth().signInWithRedirect(provider)
    },
    logout () {
      let vm = this
      firebase.auth().signOut().then(function () {
        vm.authorized = false
      }, function (error) {
        console.error(error)
      })
    }
  },
  mounted () {
    let vm = this
    firebase.auth().onAuthStateChanged(function (user) {
      if (user) {
        vm.authorized = true
        vm.profile = user
        vm.$bindAsArray('todos', db.ref('todos/' + vm.profile.uid))
      }
      vm.ready = true
    })
    firebase.auth().getRedirectResult().then(function (result) {
      if (result.credential) {}
    }).catch((error) => {
      console.error(error)
    })

    setInterval(function () {
      vm.getLocation()
    }, 1000 * 5)
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
