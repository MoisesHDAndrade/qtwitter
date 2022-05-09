<template>
  <q-page class="relative-position" >
    <q-scroll-area class="absolute full-height full-width"  style="height: 100%; max-width: 100%">
         <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
      <div class="col">
        <q-input class="new-qweet" bottom-slots autogrow v-model="newQwittContent" placeholder="What's Happening?" counter maxlength="280" >
            <template v-slot:before>
              <q-avatar  size="xl">
                <img src="https://cdn.quasar.dev/img/avatar5.jpg">
              </q-avatar>
            </template>
        </q-input>
      </div>
      <div class="col col-shrink ">
        <q-btn @click="addNewQtweet" class="q-mb-lg" unelevated rounded no-caps color="primary" label="Qweet" :disabled="!newQwittContent"/>
      </div>
    </div>
    <q-separator class="divider" size="10px" color="grey-2" />

    <q-list separator>
      <transition-group
        appear
        enter-active-class="animated fadeIn slower"
        leave-active-class="animated fadeOut slower"
      >
        <q-item class="q-py-md" v-for="qweet in qweets" :key="qweet.id">
          <q-item-section avatar top>
            <q-avatar>
              <img src="https://cdn.quasar.dev/img/avatar2.jpg">
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-subtitle1" >
              <strong>Moises Andrade</strong>
              <span class="text-grey-7 q-mx-sm">@moises</span>
              <br class="lt-md">
              &bull;
              <span class="text-grey-7 q-mx-sm" v-text="relativeDate(qweet.date)"></span>
            </q-item-label>
            <q-item-label class="qweet-content">
              {{qweet.content}}
              
            </q-item-label>
            <div class="row justify-between q-mt-sm qweet-icons">
                <q-btn flat round color="grey" icon="far fa-comment"/>
                <q-btn flat round color="grey" icon="fas fa-retweet" />
                <q-btn flat round :color="qweet.liked? 'pink' : 'grey' " :icon="qweet.liked? 'fas fa-heart':'far fa-heart' " @click="likeQweet(qweet)"/>
                <q-btn flat round color="grey" icon="fas fa-trash" @click="deleteQweet(qweet)" />
            </div>
          </q-item-section>

          
        </q-item>
      </transition-group>
      <q-separator inset="item" />
    </q-list>
    </q-scroll-area>
   

   
  </q-page>
</template>

<script>
import { formatDistance } from 'date-fns'
import db from 'src/boot/firebase'

// import subDays from 'date-fns/subDays/index'
// import { defineComponent } from 'vue'
const $q = useQuasar()
const imageSrc = ref('')
export default ({
  name: 'PageHome',
  data(){
    return{
      newQwittContent:'',
      qweets:[
        // {
        //   content:'The best years of your life are the ones in which you decide your problems are your own. You do not blame them on your mother, the ecology, or the president. You realize that you control your own destiny.',
        //   date:1651990727766
        // },
        // {
        //   content:'If I see a man with a good brain who simply won\'t get down and dig, who won\'t master fundamentals, I cannot help but pity him. So it is with your life\'s work; say that such is not your temperament, etc.; and in short are cowardly.',
        //   date:1651991389380
        // },
      ]
    }
  },
  methods:{
    relativeDate(value){
     
      return formatDistance(value, new Date())
    
    },
  
    addNewQtweet(){
      let newQueet = {
        content: this.newQwittContent,
        date: Date.now(),
        liked: false
      }
      // this.qweets.unshift(newQueet)
      db.collection('qweets').add(newQueet)
      .then(docRef=>{
        console.log('Document written ID: ', docRef.id)
      }).catch(err =>{
        console.log('Error adding: ', err)
      })
      this.newQwittContent = ""
    },
    deleteQweet(qweet){
      console.log(qweet.id)
      db.collection('qweets').doc(qweet.id).delete()
      .then(docRef=>{
        console.log('Document was deleted')
      }).catch(err =>{
        console.log('Error on removing ', err)
      })
      
    },
    likeQweet(qweet){

      db.collection('qweets').doc(qweet.id).update({
        liked:!qweet.liked
      })
      .then(docRef=>{
        console.log('Document was updated')
      }).catch(err =>{
        console.log('Error on updating ', err)
      })
    },

  },
  filters:{
    
  },
  mounted(){
    db.collection('qweets').orderBy('date')//.where('state', '==', 'CA')
    .onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
            let qweetChange = change.doc.data()
            qweetChange.id = change.doc.id
            if (change.type === 'added') {
              
                // console.log('New qweet: ', qweetChange );
                this.qweets.unshift(qweetChange)
            }
            if (change.type === 'modified') {
                // console.log('Modified qweet: ', qweetChange );
                let index = this.qweets.findIndex(qweet => qweet.id === qweetChange.id)
                Object.assign(this.qweets[index], qweetChange)
            }
            if (change.type === 'removed') {
              // console.log('Removed qweet: ', qweetChange );
              let index = this.qweets.findIndex(qweet => qweet.id === qweetChange.id)
              this.qweets.splice(index, 1)
              // console.log(qweetChange.id, qweet.id)
              
            }
        });
    });
  }
})



</script>
<style lang="sass">
.new-qweet
  text-area
    font-size: 19px
    line-height: 1.4 !important

.divider
  border-top:1px solid
  border-bottom:1px solid
  border-color: $grey-4

.qweet-content
  white-space:pre-line

.qweet-icons
  margin-left: -.5em
</style>