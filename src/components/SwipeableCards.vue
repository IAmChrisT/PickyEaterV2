<template>
  <section class="container">
    <div class="fixed header">
      <i class="material-icons color-pink" @click="index = 0">refresh</i>
      <img class="logo" src="../assets/images/Logo_Small_t.png" />
      <i class="material-icons color-pink">tune</i>
    </div>
    <div
      v-if="current"
      class="fixed fixed--center"
      style="z-index: 3"
      :class="{ 'transition': isVisible }">
      <Vue2InteractDraggable
        v-if="isVisible"
        :interact-out-of-sight-x-coordinate="500"
        :interact-max-rotation="15"
        :interact-x-threshold="200"
        :interact-y-threshold="200"
        :interact-event-bus-events="interactEventBus"
        interact-block-drag-down
        @draggedRight="emitAndNext('match')"
        @draggedLeft="emitAndNext('reject')"
        @draggedUp="emitAndNext('skip')"
        class="rounded-borders card card--one">
        <div style="height: 100%">
          <img 
          :srcset="this.checkurl"
          :src="this.checkurl"
          class="rounded-borders" />
          <div class="text">
            <h2>{{current.name}}, <span>{{current.rating}}</span></h2>
            <h4>{{current.location}}
          </div>
        </div>
      </Vue2InteractDraggable>
    </div>
    <div
      v-if="next"
      class="rounded-borders card card--two fixed fixed--center"
      style="z-index: 2">
      <div style="height: 100%">
        <img class="rounded-borders" :srcset="this.checknexturl"
        :src="this.checknexturl"/>
        <div class="text">
            <h2>{{next.name}}, <span>{{next.rating}}</span></h2>
            <h4>{{next.location}}
          </div>
      </div>
    </div>
    <div
      v-if="index + 2 < cards.length"
      class="rounded-borders card card--three fixed fixed--center"
      style="z-index: 1">
      <div style="height: 100%">
      </div>
    </div>
    <div class="footer fixed">
      <div class="btn btn--decline" @click="reject">
          <i class="material-icons">close</i>
      </div>
      <div class="btn btn--skip" @click="skip">
          <i class="material-icons">call_missed</i>
      </div>
      <div class="btn btn--like" @click="match">

          <i style="top: 44%; font-size: 26px; font-style: normal;">🙌</i>
      </div>
      
    </div>
    <div id="map"></div>
  </section>
</template>
 
<script>

import { Vue2InteractDraggable, InteractEventBus } from 'vue2-interact';





 
    var loc = new google.maps.LatLng(52.5639745, -0.1409372);
    var map;
    var service;
    var restdata = [];

    map = new google.maps.Map(
    document.getElementById('map'), {center: loc, zoom: 15});
    var request = {
      location: loc,
      radius: '5000',
      type: ['restaurant']
    };
    service = new google.maps.places.PlacesService(map);
    service.nearbySearch(request, callback);
  

  function callback(results, status) {
    if (status == google.maps.places.PlacesServiceStatus.OK) {
      for (var i = 0; i < results.length; i++) {
        
        if(results[i]["business_status"] === "OPERATIONAL") {
        //  Restaurant is operational
          
          if(results[i].hasOwnProperty("photos")) {

          //GETS ALL PHOTOS
          //results[i].photos.forEach(photo => {
          //  console.log(photo.getUrl({maxHeight: 300})) // will log a url but no photo_reference
          //})

            let imageurl = results[i].photos[0].getUrl({maxWidth: 5000, maxHeight: 5000})
            
            restdata.push({ src: imageurl, name: results[i]["name"], location: results[i]["vicinity"], rating: (results[i]["rating"] == null ? "No Rating" : results[i]["rating"] + "⭐")})
       
          } else {
            restdata.push({ src: 'missing_image.png', name: results[i]["name"], location: results[i]["vicinity"], rating: (results[i]["rating"] == null ? "No Rating" : results[i]["rating"] + "⭐")})
          }
        }
      }
      //console.log(restdata);
    } 
  }

const EVENTS = {
  MATCH: 'match',
  SKIP: 'skip',
  REJECT: 'reject'
}

export default {
  name: 'SwipeableCards',
  components: { Vue2InteractDraggable },
  data() {
      /* let carddata = [{ src: 'karina.jpg', name: 'Chiquitos', location: "Hampton", rating: "⭐⭐⭐" },
      { src: 'alexander.jpg', name: 'Bella Italia', location: "Hampton", rating: "⭐⭐⭐⭐⭐" },
      { src: 'bona.jpg', name: 'Pizza Cafe', location: "Peterborough", rating: "⭐⭐⭐" },
      { src: 'ichi.jpg', name: 'Prezzo\'s', location: "Peterborough", rating: "⭐⭐" },
      { src: 'lloyd.jpg', name: 'Chimmichangas', location: "Peterborough", rating: "⭐⭐⭐" },
      { src: 'luiza.jpg', name: 'Turtle Bay', location: "Peterborough", rating: "⭐⭐⭐⭐" },
      { src: 'max.jpg', name: 'The Queen\'s Head', location: "Peterborough", rating: "⭐⭐⭐⭐⭐" },
      { src: 'mona.jpg', name: 'Five Guys', location: "Peterborough", rating: "⭐" },
      { src: 'naru.jpg', name: 'Tavan', location: "Peterborough", rating: "⭐⭐" },
      { src: 'ramdan.jpg', name: 'Cote Brassarie', location: "Peterborough", rating: "⭐" },
      { src: 'rikki-austin.jpg', name: 'East', location: "Peteborough", rating: "⭐" },
      { src: 'tucker.jpg', name: 'Bills', location: "Peterborough", rating: "⭐" },
      { src: 'uriel.jpg', name: 'The Pizza Parlour', location: "Peterborough", rating: "⭐⭐⭐" },
      { src: 'zoe.jpg', name: 'The Banyan Tree', location: "Peterborough", rating: "⭐⭐⭐⭐⭐⭐" }]
 */
    return {
      isVisible: true,
      index: 0,
      interactEventBus: {
        draggedRight: EVENTS.MATCH,
        draggedLeft: EVENTS.REJECT,
        draggedUp: EVENTS.SKIP
      },
      cards: restdata
    }

  },
  computed: {
    current() {
      return this.cards[this.index]
    },
    next() {
      return this.cards[this.index + 1]
    },
    checkurl() {
      var url = this.cards[this.index].src
      if(url.includes("google", 0)){
        //console.log(url)
        url = url.replace("http://localhost/assets/images/","")
        //console.log(url)
        return url
      } else {
        return require("../assets/images/" + url)
      }
    },
    checknexturl(){
      var card = this.cards[this.index + 1]
      var url = card.src
      if(url.includes("google", 0)){
        //console.log(url)
        url = url.replace("http://localhost/assets/images/","")
        //console.log(url)
        return url
      } else {
        return require("../assets/images/" + url)
      }
    },
    defaultimage(){
      return require("../assets/images/missing_image.png")
    }
  },
  methods: {
    match() {
      InteractEventBus.$emit(EVENTS.MATCH)
      
    },
    reject() {
      InteractEventBus.$emit(EVENTS.REJECT)
      
    },
    skip() {
      InteractEventBus.$emit(EVENTS.SKIP)
      
    },
    emitAndNext(event) {
      this.$emit(event, this.index)
      setTimeout(() => this.isVisible = false, 200)
      setTimeout(() => {
        this.index++
        this.isVisible = true
      }, 200)

      console.log(this.name)

      if(event === "match"){
        console.log("Sick little match there, going to add it to the database now");
      }

      if(event === "reject"){
        console.log("Not feeling that one, let me tell the others");
      }

      if(event === "skip"){
        console.log("No opinion, no matter. Onto the next one");
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  background: #eceff1;
  width: 100%;
  height: 100vh;
}

.color-pink{
  color: rgb(223,17,101);
}

.header {
  width: 100%;
  height: 60vh;
  z-index: 0;
  top: 0;
  left: 0;
  color: white;
  text-align: center;
  background: #fff;
  clip-path: polygon(0 0%, 100% 0%, 100% 76%, 0 89%);
  display: flex;
  justify-content: space-between;
  span {
    display: block;
    font-size: 4rem;
    padding-top: 2rem;
    text-shadow: 1px 1px 1px red;
  }
  i {
    padding: 24px;
  }
}

.logo{
  height: 150px;
  margin-top: 15px;
}

.footer {
  width: 77vw;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  padding-bottom: 30px;
  justify-content: space-around;
  align-items: center;
}

.btn {
  position: relative;
  width: 50px;
  height: 50px;
  padding: .2rem;
  border-radius: 50%;
  background-color: white;
  box-shadow: 0 6px 6px -3px rgba(0,0,0,0.02), 0 10px 14px 1px rgba(0,0,0,0.02), 0 4px 18px 3px rgba(0,0,0,0.02);
  cursor: pointer;
  transition: all .3s ease;
  user-select: none;
  -webkit-tap-highlight-color:transparent;
  &:active {
    transform: translateY(4px);
  }
  i {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    &::before {
      content: '';
    }
  }
  &--like {
    background-color: rgb(223,17,101);
    padding: .5rem;
    color: white;
    box-shadow: 0 10px 13px -6px rgba(0,0,0,.2), 0 20px 31px 3px rgba(0,0,0,.14), 0 8px 38px 7px rgba(0,0,0,.12);
    i {
      font-size: 32px;
    }
  }
  &--decline {
    color: red;
  }
  &--skip {
    color: green;
  }
}

.flex {
  display: flex;
  &--center {
    align-items: center;
    justify-content: center;
  }
}

.fixed {
  position: fixed;
  &--center {
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}
.rounded-borders {
  border-radius: 12px;
}
.card {
  width: 80vw;
  height: 60vh;
  color: white;
  img {
    object-fit: cover;
    display: block;
    width: 100%;
    height: 100%;
  }
  &--one {
    box-shadow: 0 1px 3px rgba(0,0,0,.2), 0 1px 1px rgba(0,0,0,.14), 0 2px 1px -1px rgba(0,0,0,.12);
  }
  &--two {
    transform: translate(-48%, -48%);
    box-shadow: 0 6px 6px -3px rgba(0,0,0,.2), 0 10px 14px 1px rgba(0,0,0,.14), 0 4px 18px 3px rgba(0,0,0,.12);
  }
  &--three {
    background: rgba(black, .8);
    transform: translate(-46%, -46%);
    box-shadow: 0 10px 13px -6px rgba(0,0,0,.2), 0 20px 31px 3px rgba(0,0,0,.14), 0 8px 38px 7px rgba(0,0,0,.12);
  }
  .text {
    position: absolute;
    bottom: 0;
    width: 100%;
    background:black;
    background:rgba(0,0,0,0.5);
    border-bottom-right-radius: 12px;
    border-bottom-left-radius: 12px;
    text-indent: 20px;
    line-height: 20px;
    height: 100px;
    span {
      font-weight: normal;
    }
  }
}

.transition {
  animation: appear 200ms ease-in;
}

@keyframes appear {
  from {
    transform: translate(-48%, -48%);
  }
  to {
    transform: translate(-50%, -50%);
  }
}

.h2{
    font-size: 1.5em;
    margin-block-start: 0.83em;
    margin-block-end: 0.83em;
    margin-inline-start: 0px;
    margin-inline-end: 0px;
    font-weight: bold;
}

.h4 {
    margin-block-start: 1.33em;
    margin-block-end: 1.33em;
    margin-inline-start: 0px;
    margin-inline-end: 0px;
    font-weight: bold;
}
</style>
