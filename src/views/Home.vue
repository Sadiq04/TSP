<template>
  <div>
    <img src="@/assets/Map of Turkey.png">
    <button @click="start">Start</button>
    <p>KMratio: {{ KMratio }}</p>
  </div>
</template>

<script lang="js">

export default {
  /* eslint-disable */
  data(){
    return {
      locations: ["Istanbul", "Bursa", "Izmir", "Antalya", "Konya", "Ankara"],
      coordinates: [[41.0052363,28.8717539], [40.2216568,28.9620783], [38.4175913,26.9392913], [36.8979169,30.6478938], [37.8784231,32.3660567], [39.9032919,32.6223391]],
      locationDifferences: [],
      KMratio: 0,
    }
  },
  methods: {
    start(){
      this.KMratio = 90/(Math.sqrt((this.coordinates[0][0] - this.coordinates[1][0])^2 + (this.coordinates[0][1] - this.coordinates[1][1])^2))
     // console.log(Math.abs(this.coordinates[0][0] - this.coordinates[1][0])^2 + Math.abs(this.coordinates[0][1] - this.coordinates[1][1])^2)
      for(let i = 0; i < this.coordinates.length; i++){
        this.locationDifferences.push([])
        for(let o = 0; o < this.coordinates.length; o++){
          if(i!=o){this.locationDifferences[i].push(Math.sqrt((this.coordinates[i][0] - this.coordinates[o][0])^2 + (this.coordinates[i][1] - this.coordinates[o][1])^2)*this.KMratio)}
        }
      }
      console.log(this.locationDifferences)
    },
  }
}
</script>
