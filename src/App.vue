<template>
  <div style="height: 100vh">
    <div style="width: 100px;">
      <input style="z-index: 2; position: absolute;" type="file" ref="doc" @change="readFile()" />
    </div>
    <label style="position: absolute; top: 0.5%; left: 20%" for="start">Enter start location (0 - {{ coordinates.length-1 }}):</label><br>
    <input style="position: absolute; top: 2.75%; left: 20%; z-index: 1;" type="number" id="start" name="start" v-model="startingPlace"><br>
    <label style="position: absolute; top: 0.5%; left: 40%" for="iteration">Enter linear sweep iteration count (1-11): </label><br>
    <input style="position: absolute; top: 2.75%; left: 40%; z-index: 1;" type="number" id="iteration" name="iteration" v-model="linearSweepIterations"><br>
    <label style="position: absolute; top: 0.5%; left: 60%" for="iteration">Enter angular sweep iteration count (1-8): </label><br>
    <input style="position: absolute; top: 2.75%; left: 60%; z-index: 1;" type="number" id="iteration" name="iteration" v-model="angularSweepIterations"><br>
    <button style="position: absolute; top: 2.75%; left: 30%; z-index: 1;" @click="route">Route</button>
    <div v-for="(coordinate, index) in coordinates" :key="coordinate" style="position: absolute; bottom: 0%; left: 0%; height: 10000px; width: 10000px;">
      <span class="dot" style="position: absolute; color: white; text-align: center" v-bind:style="{bottom: `${(coordinate[0])/8-100}px`, left: `${(coordinate[1])/8+100}px`}">{{index}}</span>
    </div>
    <canvas id="canvas" v-for="(cluster, index) in nodeCluster" :key="cluster">
      <span class="dot" style="position: absolute; text-align: center" v-bind:style="{bottom: `${(coordinates[index][0])/(10000)*100}%`, left: `${(coordinates[index][1])/(10000)*100}%`, backgroundColor: `${colors[cluster]}`}"></span>
    </canvas>
    <canvas id="canvas" style="position: absolute; bottom: -10%; left: 6%; transform: rotateX(180deg)"  height="10000px" width="10000px"></canvas>
    <div style="position: absolute; top: 75px; left: 20%">Tour Length: {{tourLength*ratio}}</div>
  </div>
</template>

<script lang="js">
/* eslint-disable */
export default {
  /* eslint-disable */
  data(){
    return {
      coordinates: [[6734, 1453],
                    [2233, 10],
                    [5530, 1424],
                    [401, 841],
                    [3082, 1644],
                    [7608, 4458],
                    [7573, 3716],
                    [7265, 1268],
                    [6898, 1885],
                    [1112, 2049],
                    [5468, 2606],
                    [5989, 2873],
                    [4706, 2674],
                    [4612, 2035],
                    [6347, 2683],
                    [6107, 669],
                    [7611, 5184],
                    [7462, 3590],
                    [7732, 4723],
                    [5900, 3561],
                    [4483, 3369],
                    [6101, 1110],
                    [5199, 2182],
                    [1633, 2809],
                    [4307, 2322],
                    [675, 1006],
                    [7555, 4819],
                    [7541, 3981],
                    [3177, 756],
                    [7352, 4506],
                    [7545, 2801],
                    [3245, 3305],
                    [6426, 3173],
                    [4608, 1198],
                    [23, 2216],
                    [7248, 3779],
                    [7762, 4595],
                    [7392, 2244],
                    [3484, 2829],
                    [6271, 2135],
                    [4985, 140],
                    [1916, 1569],
                    [7280, 4899],
                    [7509, 3239],
                    [10, 2676],
                    [6807, 2993],
                    [5185, 3258],
                    [3023, 1942],
                  ],
      inputtedCoords: "",
      locationDifferences: [], // locationDifferences[start point][0(differences)][end point] / locationdifferences[start point][1(closest point)][0/1(distance to point/point index)] / locationdifferences[start point][2(furthest point)][0/1(distance to point/point index)]
      visitedPlaces: [],
      startingPlace: 0,
      nodeCluster: [],
      clusters: [],
      clusterCount: 0,
      colors: ["#AAA", "#F00", "#0F0", "#00F", "#FB0", "#0BF", "#B0F", "#EEE", "#F12", "#C07", "#09A"],
      canvas: null,
      tourLength: 0,
      startTime: 0,
      endTime: 0,
      ratio: 1,
      linearSweepIterations: 1,
      angularSweepIterations: 1,
      highestX: 0,
      highestY: 0,
      step: 1,
      step2: 100,
    }
  },
  methods: {
    setDistances(){
      for(let i = 0; i < this.locationDifferences.length; i++){
        let shortestDistance = 99999
        let shortestDistanceI = -1
        for(let o = 0; o < this.locationDifferences[i][0].length; o++){
          if(i!=o && this.locationDifferences[i][0][o]<shortestDistance){shortestDistance=this.locationDifferences[i][0][o]; shortestDistanceI=o}
        }
        this.locationDifferences[i][1] = [shortestDistance, shortestDistanceI]
      }
    },
    router(x, y, algorithm, step, step2, shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep, isSweepingAway){
      let angularSweepSteps = [11.25, 15, 9, 15, 5, 22.5, 3, 45, 1]
      let linearSweepSteps = [100, 300, 70, 500, 50, 700, 30, 1000, 3000, 5000, 10]
      step = angularSweepSteps[y]
      step2 = linearSweepSteps[x]
      if(algorithm==0){
        let genericRoute = [];
        let genericRouteLength = 0;
        for(let i = 0; i < this.coordinates.length; i++){
          genericRoute.push(i)
        }
        for(let i = 0; i < genericRoute.length-1; i++){genericRouteLength+=this.locationDifferences[i][0][i+1];}
        //this.endTime = performance.now(); 
        //console.log("Two Opt only time: " + (this.endTime - this.startTime));
        console.log("genetic route: " + genericRoute + ", genetic route length: " + genericRouteLength)
        return [genericRoute, genericRouteLength]
      }
      if(this.startingPlace >= this.coordinates.length || this.startingPlace < 0){return;}
      this.visitedPlaces.splice(0)
      console.log(this.visitedPlaces)
      let currentSpot = this.startingPlace;

      for (let i = 180-step; i >= 90; i-=step){
        isSweepingAway = !isSweepingAway;
        let currentNodes = []
        for(let o = 0; o < this.coordinates.length; o++){
          if(this.visitedPlaces.includes(o)==false && (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0) && ((i%90==0 && i%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || ((i+step)%90==0 && (i+step)%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || (i%180==0 && this.coordinates[o][0]-this.coordinates[this.startingPlace][0]>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) || 
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) ||
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])))){
            currentNodes.push(o);                                                                                
          }}
        console.log("i: " + i + " Curr nodes: " + currentNodes + "tl")
        if(currentNodes.length){
          let route = []
          if(algorithm==1){
            if(i >= 135){
              if(isSweepingAway){
                for(let o = step2; o <= this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] <= this.coordinates[this.startingPlace][1]-(o-step2) && this.coordinates[currentNodes[p]][1] >= this.coordinates[this.startingPlace][1]-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] >= (o-step2) && this.coordinates[currentNodes[p]][1] <= o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
            else if(i <= 135){
              if(isSweepingAway){
                for(let o = step2; o <= this.highestY-this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] >= this.coordinates[this.startingPlace][0]+(o-step2) && this.coordinates[currentNodes[p]][0] <= this.coordinates[this.startingPlace][0]+o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.highestY-this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] <= this.highestY-(o-step2) && this.coordinates[currentNodes[p]][0] >= this.highestY-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
          }
          else if (algorithm==2){
            let shortestRoute = [currentNodes[0]];
            let shortestRouteLength = 99999;
            let startPoint = -1;
            let startPointDistance = 99999
            let endPoint = -1;
            let endPointDistance = 0;
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[currentSpot][0][currentNodes[o]] < startPointDistance){startPointDistance = this.locationDifferences[currentSpot][0][currentNodes[o]]; startPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[startPoint][0][currentNodes[o]] > endPointDistance){endPointDistance = this.locationDifferences[startPoint][0][currentNodes[o]]; endPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length && currentNodes.length>1; o++){        
              let routeCopy = currentNodes;
              let currentRouteLength = 0;
              routeCopy.splice(routeCopy.indexOf(startPoint), 1)
              routeCopy.splice(routeCopy.indexOf(endPoint), 1)
              routeCopy.sort(() => Math.random() - 0.5);
              routeCopy.unshift(startPoint);
              routeCopy.push(endPoint);
              for (let p = 1; p < routeCopy.length; p++){
                currentRouteLength += this.locationDifferences[routeCopy[p-1]][0][routeCopy[p]];  
              }    
              if(currentRouteLength < shortestRouteLength){shortestRouteLength = currentRouteLength; shortestRoute = routeCopy}
            }
            route = shortestRoute;
          }
          for(let o = 0; o < route.length; o++){
            this.visitedPlaces.push(route[o]);
            if(o==route.length-1){currentSpot = route[o];}
          }
          console.log("i: " + i + ", route: " + route + "isSweepingAway" + isSweepingAway)
          }
        this.setDistances();
        let allVisited = true
        for(let i = 0; i < this.coordinates.length; i++){
          if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
        }
      } // top-left quarter     
      for (let i = 90-step; i >= 0; i-=step){
        isSweepingAway = !isSweepingAway;
        let currentNodes = []
        for(let o = 0; o < this.coordinates.length; o++){
          if(this.visitedPlaces.includes(o)==false && (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]>=0) && ((i%90==0 && i%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || ((i+step)%90==0 && (i+step)%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || (i%180==0 && this.coordinates[o][0]-this.coordinates[this.startingPlace][0]>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) || 
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) ||
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])))){
            currentNodes.push(o);                                                                                
          }}
        console.log("i: " + i + " Curr nodes: " + currentNodes + "tr")
        if(currentNodes.length){
          let route = []
          if(algorithm==1){
            if(i >= 45){
              if(isSweepingAway){
                for(let o = step2; o <= this.highestY-this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] >= this.coordinates[this.startingPlace][0]+(o-step2) && this.coordinates[currentNodes[p]][0] <= this.coordinates[this.startingPlace][0]+o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.highestY-this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] <= this.highestY-(o-step2) && this.coordinates[currentNodes[p]][0] >= this.highestY-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
            else if(i <= 45){
              if(isSweepingAway){
                for(let o = step2; o <= this.highestX-this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] >= this.coordinates[this.startingPlace][1]+(o-step2) && this.coordinates[currentNodes[p]][1] <= this.coordinates[this.startingPlace][1]+o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.highestX-this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] <= this.highestX-(o-step2) && this.coordinates[currentNodes[p]][1] >= this.highestX-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
          }
          else if (algorithm==2){
            let shortestRoute = [currentNodes[0]];
            let shortestRouteLength = 99999;
            let startPoint = -1;
            let startPointDistance = 99999
            let endPoint = -1;
            let endPointDistance = 0;
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[currentSpot][0][currentNodes[o]] < startPointDistance){startPointDistance = this.locationDifferences[currentSpot][0][currentNodes[o]]; startPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[startPoint][0][currentNodes[o]] > endPointDistance){endPointDistance = this.locationDifferences[startPoint][0][currentNodes[o]]; endPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length && currentNodes.length>1; o++){        
              let routeCopy = currentNodes;
              let currentRouteLength = 0;
              routeCopy.splice(routeCopy.indexOf(startPoint), 1)
              routeCopy.splice(routeCopy.indexOf(endPoint), 1)
              routeCopy.sort(() => Math.random() - 0.5);
              routeCopy.unshift(startPoint);
              routeCopy.push(endPoint);
              for (let p = 1; p < routeCopy.length; p++){
                currentRouteLength += this.locationDifferences[routeCopy[p-1]][0][routeCopy[p]];  
              }    
              if(currentRouteLength < shortestRouteLength){shortestRouteLength = currentRouteLength; shortestRoute = routeCopy}
              route = shortestRoute;
            }
          }
          for(let i = 0; i < route.length; i++){
            this.visitedPlaces.push(route[i]);
            if(i==route.length-1){currentSpot = route[i];}
          }
          console.log("i: " + i + ", route: " + route + " isSweepingAway: " + isSweepingAway)
          }
        this.setDistances();
        let allVisited = true
        for(let i = 0; i < this.coordinates.length; i++){
          if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
        }
      } // top-right quarter
      for (let i = 180-step; i >= 90; i-=step){
        isSweepingAway = !isSweepingAway;
        let currentNodes = []
        for(let o = 0; o < this.coordinates.length; o++){
          if(this.visitedPlaces.includes(o)==false && (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]>=0) && ((i%90==0 && i%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]>=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || ((i+step)%90==0 && (i+step)%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || (i%180==0 && this.coordinates[o][0]-this.coordinates[this.startingPlace][0]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) || 
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) ||
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])))){
            currentNodes.push(o);                                                                                
          }}
        console.log("i: " + i + " Curr nodes: " + currentNodes + " br")
        if(currentNodes.length){
          let route = []
          if(algorithm==1){
            if(i >= 135){
              if(isSweepingAway){
                for(let o = step2; o <= this.highestX-this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] >= this.coordinates[this.startingPlace][1]+(o-step2) && this.coordinates[currentNodes[p]][1] <= this.coordinates[this.startingPlace][1]+o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.highestX-this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] <= this.highestX-(o-step2) && this.coordinates[currentNodes[p]][1] >= this.highestX-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
            else if(i <= 135){
              if(isSweepingAway){
                for(let o = step2; o <= this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] <= this.coordinates[this.startingPlace][0]-(o-step2) && this.coordinates[currentNodes[p]][0] >= this.coordinates[this.startingPlace][0]-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] >= (o-step2) && this.coordinates[currentNodes[p]][0] <= o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
          }
          else if (algorithm==2){
            let shortestRoute = [currentNodes[0]];
            let shortestRouteLength = 99999;
            let startPoint = -1;
            let startPointDistance = 99999
            let endPoint = -1;
            let endPointDistance = 0;
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[currentSpot][0][currentNodes[o]] < startPointDistance){startPointDistance = this.locationDifferences[currentSpot][0][currentNodes[o]]; startPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[startPoint][0][currentNodes[o]] > endPointDistance){endPointDistance = this.locationDifferences[startPoint][0][currentNodes[o]]; endPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length && currentNodes.length>1; o++){        
              let routeCopy = currentNodes;
              let currentRouteLength = 0;
              routeCopy.splice(routeCopy.indexOf(startPoint), 1)
              routeCopy.splice(routeCopy.indexOf(endPoint), 1)
              routeCopy.sort(() => Math.random() - 0.5);
              routeCopy.unshift(startPoint);
              routeCopy.push(endPoint);
              for (let p = 1; p < routeCopy.length; p++){
                currentRouteLength += this.locationDifferences[routeCopy[p-1]][0][routeCopy[p]];  
              }    
              if(currentRouteLength < shortestRouteLength){shortestRouteLength = currentRouteLength; shortestRoute = routeCopy}
              route = shortestRoute;
            }
          }
          for(let i = 0; i < route.length; i++){
            this.visitedPlaces.push(route[i]);
            if(i==route.length-1){currentSpot = route[i];}
          }
          console.log("i: " + i + ", route: " + route)
          }
        this.setDistances();
        let allVisited = true
        for(let i = 0; i < this.coordinates.length; i++){
          if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
        }
      } // bottom-right quarter
      for (let i = 90-step; i >= 0; i-=step){
        isSweepingAway = !isSweepingAway;
        let currentNodes = []
        for(let o = 0; o < this.coordinates.length; o++){
          if(this.visitedPlaces.includes(o)==false && (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0) && ((i%90==0 && i%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || ((i+step)%90==0 && (i+step)%180!=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]))) || (i%180==0 && this.coordinates[o][0]-this.coordinates[this.startingPlace][0]<=0 && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) || 
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])) ||
            ((this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan(i * Math.PI/180)) <= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0]) && (this.coordinates[o][1]-this.coordinates[this.startingPlace][1]) * (Math.tan((i+step) * Math.PI/180)) >= (this.coordinates[o][0]-this.coordinates[this.startingPlace][0])))){
            currentNodes.push(o);                                                                                
          }}
        console.log("i: " + i + " Curr nodes: " + currentNodes + " bl")
        if(currentNodes.length){
          let route = []
          if(algorithm==1){
            if(i >= 45){
              if(isSweepingAway){
                for(let o = step2; o <= this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] <= this.coordinates[this.startingPlace][0]-(o-step2) && this.coordinates[currentNodes[p]][0] >= this.coordinates[this.startingPlace][0]-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.coordinates[this.startingPlace][0]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][0] >= (o-step2) && this.coordinates[currentNodes[p]][0] <= o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
            else if(i <= 45){
              if(isSweepingAway){
                for(let o = step2; o <= this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] <= this.coordinates[this.startingPlace][1]-(o-step2) && this.coordinates[currentNodes[p]][1] >= this.coordinates[this.startingPlace][1]-o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
              else {
                for(let o = step2; o <= this.coordinates[this.startingPlace][1]+step2; o+=step2){
                  for(let p = 0; p < currentNodes.length; p++){
                    if(this.coordinates[currentNodes[p]][1] >= (o-step2) && this.coordinates[currentNodes[p]][1] <= o){
                      route.push(currentNodes[p]);
                    }
                  }
                }
              }
            }
          }
          else if (algorithm==2){
            let shortestRoute = [currentNodes[0]];
            let shortestRouteLength = 99999;
            let startPoint = -1;
            let startPointDistance = 99999
            let endPoint = -1;
            let endPointDistance = 0;
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[currentSpot][0][currentNodes[o]] < startPointDistance){startPointDistance = this.locationDifferences[currentSpot][0][currentNodes[o]]; startPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length; o++){
              if(!this.visitedPlaces.includes(currentNodes[o]) && this.locationDifferences[startPoint][0][currentNodes[o]] > endPointDistance){endPointDistance = this.locationDifferences[startPoint][0][currentNodes[o]]; endPoint = currentNodes[o]}
            }
            for (let o = 0; o < currentNodes.length && currentNodes.length>1; o++){        
              let routeCopy = currentNodes;
              let currentRouteLength = 0;
              routeCopy.splice(routeCopy.indexOf(startPoint), 1)
              routeCopy.splice(routeCopy.indexOf(endPoint), 1)
              routeCopy.sort(() => Math.random() - 0.5);
              routeCopy.unshift(startPoint);
              routeCopy.push(endPoint);
              for (let p = 1; p < routeCopy.length; p++){
                currentRouteLength += this.locationDifferences[routeCopy[p-1]][0][routeCopy[p]];  
              }    
              if(currentRouteLength < shortestRouteLength){shortestRouteLength = currentRouteLength; shortestRoute = routeCopy}
              route = shortestRoute;
            }
          }
          for(let i = 0; i < route.length; i++){
            this.visitedPlaces.push(route[i]);
            if(i==route.length-1){currentSpot = route[i];}
          }
          console.log("i: " + i + ", route: " + route)
          }
        this.setDistances();
        
        let allVisited = true
        for(let i = 0; i < this.coordinates.length; i++){
          if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
        }
      } // bottom-left quarter
      this.tourLength = 0
      let allVisited = true
      for(let i = 0; i < this.coordinates.length; i++){
        if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
      }
      
      console.log(allVisited)
      for(let i = 0; i < this.visitedPlaces.length-1; i++){this.tourLength+=this.locationDifferences[this.visitedPlaces[i]][0][this.visitedPlaces[i+1]];} 
      shortestFoundRoute = this.visitedPlaces; shortestFoundRouteLength = this.tourLength; this.visitedPlaces = []; bestLinearStep = x; bestAngularStep = y;
      console.log([shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep])
      return [shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep]
    },
    route(){
      let algorithm;
      let bestAlgorithm;
      this.startTime = performance.now();
      this.calculateDifferences(); 
      console.log("Routing")
      let step = this.step;
      let step2 = this.step2;
      this.startingPlace = parseInt(this.startingPlace)
      let shortestOverallRoute = [];
      let shortestOverallRouteLength = 99999
      let shortestFoundRoute = [];
      let shortestFoundRouteLength = 99999;
      let bestAngularStep = -1;
      let bestLinearStep = -1;
      let isSweepingAway = false;
      algorithm = 0;
      console.log('%c Local Search Only Algorithm', 'font-size: 36px; font-weight: bold');
      let arr1 = this.router(0, 0, algorithm, step, step2, shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep, isSweepingAway); shortestOverallRoute = arr1[0]; shortestOverallRouteLength = arr1[1]; bestAlgorithm = "local search only"
      algorithm = 1;
      console.log('%c Angular + Linear Sweep Algorithm', 'font-size: 36px; font-weight: bold');
      for(let x = 0; x < this.linearSweepIterations && this.linearSweepIterations > 0 && this.linearSweepIterations < 12; x++){let arr = this.router(x, 0, algorithm, step, step2, shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep, isSweepingAway); 
                                                        shortestFoundRoute = arr[0]; shortestFoundRouteLength = arr[1]; bestAngularStep = arr[2]; bestLinearStep = arr[3];}
      for(let y = 0; y < this.angularSweepIterations && this.angularSweepIterations > 0 && this.angularSweepIterations < 9; y++){let arr = this.router(bestLinearStep, y, algorithm, step, step2, shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep, isSweepingAway); 
                                                        shortestFoundRoute = arr[0]; shortestFoundRouteLength = arr[1]; bestAngularStep = arr[2]; bestLinearStep = arr[3]}
      console.log("found length: " + shortestFoundRouteLength + ", overall length: " + shortestOverallRouteLength)
      if(shortestOverallRouteLength > shortestFoundRouteLength){shortestOverallRoute = shortestFoundRoute; shortestOverallRouteLength = shortestFoundRouteLength; bestAlgorithm = "Angular + Linear Sweep"}
      /*
      algorithm = 2;
      console.log('%c Linear Sweep + Random Pathing Algorithm', 'font-size: 36px; font-weight: bold');
      let arr = this.router(bestLinearStep, bestAngularStep, algorithm, step, step2, shortestFoundRoute, shortestFoundRouteLength, bestAngularStep, bestLinearStep, isSweepingAway); 
      shortestFoundRoute = arr[0]; shortestFoundRouteLength = arr[1];
      if(shortestOverallRouteLength > shortestFoundRouteLength){shortestOverallRoute = shortestFoundRoute; shortestOverallRouteLength = shortestFoundRouteLength; bestAlgorithm = "Angular Sweep + Random Pathing"}
      */
      console.log("Shortest found route length: " + shortestFoundRouteLength + ", Best angular step: " + bestAngularStep + ", Best linear step : " + bestLinearStep + ", Used Algorithm: " + bestAlgorithm)
      console.log(shortestOverallRoute)
      this.visitedPlaces = shortestOverallRoute;
      let allVisited = true
      for(let i = 0; i < this.coordinates.length; i++){
        if(!this.visitedPlaces.includes(i)){console.log("Missing i: " + i); allVisited=false}
      }
      if(allVisited){this.visitedPlaces.push(this.visitedPlaces[0]); this.twoOpt(); console.log(this.visitedPlaces); this.showRoute(); this.endTime = performance.now(); console.log("Time: " + (this.endTime - this.startTime));  return;}
        this.showRoute()
    },
    twoOpt(){
      let tempPath = this.visitedPlaces;
      let tempPathLength = this.tourLength;
      let i = 0;
      do {
        this.visitedPlaces = tempPath;
        this.tourLength = tempPathLength;
        for(let i = 0; i < tempPath.length-2; i++){
          for(let o = i+2; o < tempPath.length; o++){
            let distance1 = this.locationDifferences[tempPath[i]][0][tempPath[i+1]] + this.locationDifferences[tempPath[o]][0][tempPath[o+1]]
            let distance2 = this.locationDifferences[tempPath[i]][0][tempPath[o]] + this.locationDifferences[tempPath[i+1]][0][tempPath[o+1]]
            if(distance1 > distance2){
              let temp = tempPath[o];
              tempPath[o] = tempPath[i+1];
              tempPath[i+1] = temp;
              tempPathLength -= distance1 - distance2;
            }
          }
        } i++; console.log("tempPathLength: " + tempPathLength + ", tourLength: " + this.tourLength)
      } while(tempPathLength < this.tourLength)
      this.visitedPlaces = tempPath;
      this.tourLength = tempPathLength; 
    },
    showRoute(){
    this.tourLength = 0
    for(let i = 0; i < this.visitedPlaces.length-1; i++){
      this.tourLength+=this.locationDifferences[this.visitedPlaces[i]][0][this.visitedPlaces[i+1]]; 
      this.drawLine(this.coordinates[this.visitedPlaces[i]], this.coordinates[this.visitedPlaces[i+1]]);
    }
    },
    drawLine(coord1, coord2){
    let ctx = this.canvas;
    ctx.lineWidth = 1;
    ctx.strokeStyle = 'black';
    ctx.beginPath();
    let multiplier = 0.125
    ctx.moveTo(coord1[1]*multiplier,coord1[0]*multiplier);
    ctx.lineTo(coord2[1]*multiplier,coord2[0]*multiplier);
    ctx.stroke();
    },
    assignCoords(){
      let coordSet = this.inputtedCoords.split(/\n/)
      let startIdx = 0
      while(coordSet[coordSet.length-1]==""){coordSet.pop()}
      if(coordSet[coordSet.length-1] == "EOF"){coordSet.pop()}
      for(let i = 0; i < coordSet.length; i++){coordSet[i] = coordSet[i].split(" ")}
      for(let i = 0; i < coordSet.length; i++){for(let o = 0; o < coordSet[i].length; o++){coordSet[i][o].trim(); coordSet[i][o] = parseFloat(coordSet[i][o])}}
      for(let i = 0; i < coordSet.length; i++){if(coordSet[i][0]!="1"){startIdx++;} else{break;}}
      for(let i = 0; i < startIdx; i++){coordSet.shift()}
      for(let i = 0; i < coordSet.length; i++){coordSet[i].shift()}
      let highestCoord = 0
      for(let i = 0; i < coordSet.length; i++){if(coordSet[i][0]>highestCoord){highestCoord=coordSet[i][0]} else if(coordSet[i][1]>highestCoord){highestCoord=coordSet[i][1]}}
      if(highestCoord){this.ratio = highestCoord/10000}
      this.coordinates=[];
      while(coordSet[coordSet.length-1].length==0){coordSet.pop()}
      for(let i = 0; i < coordSet.length; i++){this.coordinates.push([coordSet[i][0]/this.ratio, coordSet[i][1]/this.ratio])}
    },
    readFile() {
      this.file = this.$refs.doc.files[0];
      const reader = new FileReader();
      if (this.file.name.includes(".tsp") || this.file.name.includes(".txt")) {
        reader.onload = (res) => {
          this.inputtedCoords = res.target.result;
          this.assignCoords();
        };
        reader.onerror = (err) => console.log(err);
        reader.readAsText(this.file);
      } 
    },
    calculateDifferences(){
      for(let i = 0; i < this.coordinates.length; i++){
        if(this.coordinates[i][0]>this.highestY){this.highestY = this.coordinates[i][0]}
        if(this.coordinates[i][1]>this.highestX){this.highestX = this.coordinates[i][1]}
        this.locationDifferences.push([[], []])
        for(let o = 0; o < this.coordinates.length; o++){
          this.locationDifferences[i][0].push(Math.sqrt((this.coordinates[i][0] - this.coordinates[o][0])**2 + (this.coordinates[i][1] - this.coordinates[o][1])**2))
        }
        let shortestDistance = 99999
        let shortestDistanceI = -1
        for(let o = 0; o < this.locationDifferences[i][0].length; o++){
          if(i!=o && this.locationDifferences[i][0][o]<shortestDistance){shortestDistance=this.locationDifferences[i][0][o]; shortestDistanceI=o}
        }
        this.locationDifferences[i][1] = [shortestDistance, shortestDistanceI]
      }
    },
},
  mounted() {
    var c = document.getElementById("canvas");
    this.canvas = c.getContext('2d');
  }
}
</script>

<style>
.dot {
  height: 20px;
  background-color: #000;
  width: 20px;
  border-radius: 50%;
  display: inline-block;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
