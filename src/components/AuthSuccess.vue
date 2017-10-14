<template>
  <div>
    <h1>Signup succeeded</h1>
    <progress value="0" max="100" id="uploader"></progress>
    <input accept="image/*" type="file" value="upload" capture="environment" @change="fileBtn($event)">
    <button @click='logOut'>Log out</button>
    <hr>
    <img :src="photo" style='height: 120px'> <br>
    <p>{{name}}</p>
    <p>{{email}}</p>
    <p>{{userId}}</p>
    <hr>
    <pre>{{user}}</pre>
  </div>
</template>

<script>
import firebase from 'firebase'
export default {
 data(){
     return {
       photo: '',
       userId: '',
       name: '',
       email: '',
       user: {}
     }
   },
   created() {
    var vm = this
    firebase.auth().onAuthStateChanged(function(user) {
    if (user) {
      vm.user = user;
      vm.name = vm.user.displayName;
      vm.email = vm.user.email;
      vm.photo = vm.user.photoURL;
      vm.userId = vm.user.uid;

      // ref where data for user.uid will be stored
      let usersRef = firebase.database().ref('/users/' + user.uid);

      // check once whether user data exist
      usersRef.once('value').then(function (snapshot) {
        if(!snapshot.val()) {
          // if data doesn't exist then its a new user add user data to db
          usersRef.set({
            'displayName': user.displayName,
            'email': user.email,
            'photo': user.photoURL
          })
        }
      });
    }
    });
  },
  methods: {
    fileBtn:function(e){
      e.preventDefault();
      const uploader = document.getElementById('uploader');
      //get file
      let getFile = e.target.files[0];
      //set storage ref
      let userId = this.userId;
      let storageRef = firebase.storage().ref('/user/'+ userId + '/' + getFile.name);
      //upload file
      let task = storageRef.put(getFile);
      task.on('state_changed',
      function progress(snapshot){
        let percentage = (snapshot.bytesTransferred / snapshot.totalBytes) *100;
        uploader.value = percentage;
      },
      function error(err){
        console.error(err);
      },
      function complete(){
        let aadharNumber =  123456789;
        storageRef.getDownloadURL().then(function (url) {
          firebase.database().ref('/users/' + userId + '/jobseekers/' + aadharNumber).set({
            photo: {
              url: url
            }
          }).then(function (a) {
            let photo = document.createElement('img');
            photo.setAttribute('src', url);
            document.body.prepend(photo)
          }).catch(function (err) {
            console.log(err);
          })
          let a = document.createElement('a');
          a.setAttribute('href', url);
          a.innerHTML = 'Download Link'
          a.setAttribute('download', 'download')
          document.body.prepend(a)
        }).catch(function (err) {
          console.log(err);
        })
      }
    );
  },
    logOut() {
      firebase.auth().signOut();
    }
  },
};
</script>
