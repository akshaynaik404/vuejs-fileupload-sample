<template>
  <div>
    <h1>Signup succeeded</h1>
    <button @click='logOut'>Log out</button>
    <hr>
    <img :src="photo" style='height: 120px'> <br>
    <p>{{name}}</p>
    <p>{{email}}</p>
    <p>{{userId}}</p>
    <hr>
    <pre>{{user}}</pre>
    <progress value="0" max="100" id="uploader"></progress>
    <input accept="image/*" type="file" value="upload" capture="environment" @change="fileBtn($event)">
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
      let storageRef = firebase.storage().ref('/user/'+ this.userId + '/' + getFile.name);
      //upload file
      let task = storageRef.put(getFile);
      task.on('state_changed',
      function progress(snapshot){
        let percentage = (snapshot.bytesTransferred / snapshot.totalBytes) *100;
        uploader.value = percentage;
      },
      function error(err){
        console.log(err);
      },
      function complete(){
        console.log('complete upload');
        storageRef.getDownloadURL().then(function (url) {
          let a = document.createElement('a');
          a.setAttribute('href', url);
          a.setAttribute('download', 'download')
          a.innerHTML = url;
          console.log(url, a);
          document.body.appendChild(a)
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
