<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Bus Tracking</title>

<!-- REAL MAP -->
<link
 rel="stylesheet"
 href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
/>

<script
 src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js">
</script>

<style>
*{
  box-sizing:border-box;
  font-family:Arial,sans-serif;
}

body{
  margin:0;
  background:#eef4ff;
}

header{
  background:#1565c0;
  color:white;
  padding:18px;
  text-align:center;
  font-size:24px;
  font-weight:bold;
}

.container{
  max-width:600px;
  margin:auto;
  padding:18px;
}

.card{
  background:white;
  padding:20px;
  border-radius:18px;
  margin-bottom:18px;
  box-shadow:0 4px 15px #0002;
}

input{
  width:100%;
  padding:14px;
  margin:7px 0;
  border:1px solid #bbb;
  border-radius:10px;
  font-size:16px;
}

button{
  width:100%;
  padding:15px;
  margin-top:10px;
  border:0;
  border-radius:10px;
  background:#1565c0;
  color:white;
  font-size:17px;
  font-weight:bold;
}

.stop{
  background:#d32f2f;
}

.status{
  margin-top:15px;
  padding:14px;
  border-radius:10px;
  text-align:center;
  font-weight:bold;
  background:#eee;
}

.live{
  background:#c8f7d2;
  color:#08752a;
}

.offline{
  background:#ffd6d6;
  color:#a00000;
}

#map{
  width:100%;
  height:400px;
  margin-top:18px;
  border-radius:15px;
  overflow:hidden;
}

.hidden{
  display:none;
}

.back{
  background:#555;
}
</style>
</head>

<body>

<header>
🚌 BUS TRACKING
</header>

<div class="container">

<!-- HOME -->

<div id="home" class="card">

<h2>Select User</h2>

<button onclick="showDriver()">
🚍 Driver / Conductor
</button>

<button onclick="showPassenger()">
👤 Passenger
</button>

</div>


<!-- DRIVER -->

<div id="driver" class="card hidden">

<h2>🚍 Driver / Conductor</h2>

<input
 id="busName"
 placeholder="Bus Name"
/>

<input
 id="busNumber"
 placeholder="Bus Number - Example 30A"
/>

<input
 id="boardNumber"
 placeholder="Bus Board Number - TN 99 AB 1234"
/>

<input
 id="driverPhone"
 placeholder="Phone Number"
 type="tel"
/>

<button onclick="startGPS()">
🛰️ SAVE & START GPS
</button>

<button
 class="stop"
 onclick="stopGPS()">
⛔ STOP GPS
</button>

<div
 id="driverStatus"
 class="status">
GPS NOT STARTED
</div>

<button
 class="back"
 onclick="goHome()">
BACK
</button>

</div>


<!-- PASSENGER -->

<div id="passenger" class="card hidden">

<h2>👤 Passenger</h2>

<input
 id="searchBus"
 placeholder="Enter Bus Number - Example 30A"
/>

<button onclick="trackBus()">
🔍 TRACK BUS
</button>

<div id="busInfo"></div>

<div id="map"></div>

<button
 class="back"
 onclick="goHome()">
BACK
</button>

</div>

</div>


<script>

/* =========================================
   HOME
========================================= */

function showDriver(){

 document.getElementById("home")
 .classList.add("hidden");

 document.getElementById("driver")
 .classList.remove("hidden");

}

function showPassenger(){

 document.getElementById("home")
 .classList.add("hidden");

 document.getElementById("passenger")
 .classList.remove("hidden");

 setTimeout(initMap,300);

}

function goHome(){

 location.reload();

}


/* =========================================
   MAP
========================================= */

let map;
let busMarker;

function initMap(){

 if(map) return;

 map = L.map("map").setView(
   [11.0168,76.9558],
   13
 );

 L.tileLayer(
   "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
   {
     maxZoom:19,
     attribution:"© OpenStreetMap"
   }
 ).addTo(map);

}


/* =========================================
   DRIVER GPS
========================================= */

let gpsWatch = null;

function startGPS(){

 const busName =
 document.getElementById("busName")
 .value.trim();

 const busNumber =
 document.getElementById("busNumber")
 .value.trim();

 const boardNumber =
 document.getElementById("boardNumber")
 .value.trim();

 const phone =
 document.getElementById("driverPhone")
 .value.trim();


 if(!busName || !busNumber){

   alert(
    "Bus Name மற்றும் Bus Number enter பண்ணு"
   );

   return;
 }


 if(!navigator.geolocation){

   alert(
    "இந்த browser GPS support செய்யவில்லை"
   );

   return;
 }


 if(!window.isSecureContext){

   alert(
    "GPSக்கு HTTPS தேவை.\n\n" +
    "GitHub Pages HTTPS link-ல் open பண்ணு."
   );

   return;
 }


 navigator.geolocation.getCurrentPosition(

   function(position){

     updateDriver(
       position,
       busName,
       busNumber,
       boardNumber,
       phone
     );

   },

   function(error){

     showGPSError(error);

   },

   {
     enableHighAccuracy:true,
     timeout:30000,
     maximumAge:0
   }

 );


 gpsWatch =
 navigator.geolocation.watchPosition(

   function(position){

     updateDriver(
       position,
       busName,
       busNumber,
       boardNumber,
       phone
     );

   },

   function(error){

     showGPSError(error);

   },

   {
     enableHighAccuracy:true,
     timeout:30000,
     maximumAge:0
   }

 );


 document.getElementById(
  "driverStatus"
 ).innerHTML =
 "🟢 GPS STARTED<br>Waiting for location...";

 document.getElementById(
  "driverStatus"
 ).className =
 "status live";

}


/* =========================================
   DRIVER LOCATION
========================================= */

function updateDriver(
 position,
 busName,
 busNumber,
 boardNumber,
 phone
){

 const lat =
 position.coords.latitude;

 const lng =
 position.coords.longitude;


 console.log(
  "GPS:",
  lat,
  lng
 );


 document.getElementById(
  "driverStatus"
 ).innerHTML =

 "🟢 GPS LIVE<br><br>" +

 "Bus: " + busNumber +
 "<br>" +

 "Latitude: " +
 lat.toFixed(6) +
 "<br>" +

 "Longitude: " +
 lng.toFixed(6);


}


/* =========================================
   GPS ERROR
========================================= */

function showGPSError(error){

 let message =
 "GPS Error";

 if(error.code === 1){

   message =
   "❌ Location permission DENIED";

 }

 else if(error.code === 2){

   message =
   "❌ Location unavailable";

 }

 else if(error.code === 3){

   message =
   "❌ GPS timeout";

 }


 document.getElementById(
  "driverStatus"
 ).innerHTML =
 message;

 document.getElementById(
  "driverStatus"
 ).className =
 "status offline";

}


/* =========================================
   STOP GPS
========================================= */

function stopGPS(){

 if(gpsWatch !== null){

   navigator.geolocation
   .clearWatch(gpsWatch);

   gpsWatch = null;

 }

 document.getElementById(
  "driverStatus"
 ).innerHTML =
 "⚪ GPS STOPPED";

 document.getElementById(
  "driverStatus"
 ).className =
 "status offline";

}


/* =========================================
   PASSENGER
========================================= */

function trackBus(){

 const bus =
 document.getElementById(
  "searchBus"
 ).value.trim();


 if(!bus){

   alert("Bus Number enter பண்ணு");

   return;
 }


 document.getElementById(
  "busInfo"
 ).innerHTML =

 `<div class="status live">
 🚌 Tracking Bus ${bus}
 </div>`;


 /*
  Firebase connection will be added here.

  Driver GPS
       ↓
  Firebase
       ↓
  Passenger
       ↓
  Bus marker moves on map
 */

}


/* =========================================
   DEMO MAP MARKER
========================================= */

function showBusOnMap(lat,lng){

 if(!map){

   initMap();

 }


 if(busMarker){

   busMarker.setLatLng([
     lat,
     lng
   ]);

 }

 else{

   busMarker =
   L.marker([
     lat,
     lng
   ]).addTo(map);

   busMarker.bindPopup(
    "🚌 BUS 30A"
   ).openPopup();

 }


 map.setView(
   [lat,lng],
   16
 );

}

</script>

</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Bus Tracking</title>

<!-- REAL MAP -->
<link
 rel="stylesheet"
 href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
/>

<script
 src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js">
</script>

<style>
*{
  box-sizing:border-box;
  font-family:Arial,sans-serif;
}

body{
  margin:0;
  background:#eef4ff;
}

header{
  background:#1565c0;
  color:white;
  padding:18px;
  text-align:center;
  font-size:24px;
  font-weight:bold;
}

.container{
  max-width:600px;
  margin:auto;
  padding:18px;
}

.card{
  background:white;
  padding:20px;
  border-radius:18px;
  margin-bottom:18px;
  box-shadow:0 4px 15px #0002;
}

input{
  width:100%;
  padding:14px;
  margin:7px 0;
  border:1px solid #bbb;
  border-radius:10px;
  font-size:16px;
}

button{
  width:100%;
  padding:15px;
  margin-top:10px;
  border:0;
  border-radius:10px;
  background:#1565c0;
  color:white;
  font-size:17px;
  font-weight:bold;
}

.stop{
  background:#d32f2f;
}

.status{
  margin-top:15px;
  padding:14px;
  border-radius:10px;
  text-align:center;
  font-weight:bold;
  background:#eee;
}

.live{
  background:#c8f7d2;
  color:#08752a;
}

.offline{
  background:#ffd6d6;
  color:#a00000;
}

#map{
  width:100%;
  height:400px;
  margin-top:18px;
  border-radius:15px;
  overflow:hidden;
}

.hidden{
  display:none;
}

.back{
  background:#555;
}
</style>
</head>

<body>

<header>
🚌 BUS TRACKING
</header>

<div class="container">

<!-- HOME -->

<div id="home" class="card">

<h2>Select User</h2>

<button onclick="showDriver()">
🚍 Driver / Conductor
</button>

<button onclick="showPassenger()">
👤 Passenger
</button>

</div>


<!-- DRIVER -->

<div id="driver" class="card hidden">

<h2>🚍 Driver / Conductor</h2>

<input
 id="busName"
 placeholder="Bus Name"
/>

<input
 id="busNumber"
 placeholder="Bus Number - Example 30A"
/>

<input
 id="boardNumber"
 placeholder="Bus Board Number - TN 99 AB 1234"
/>

<input
 id="driverPhone"
 placeholder="Phone Number"
 type="tel"
/>

<button onclick="startGPS()">
🛰️ SAVE & START GPS
</button>

<button
 class="stop"
 onclick="stopGPS()">
⛔ STOP GPS
</button>

<div
 id="driverStatus"
 class="status">
GPS NOT STARTED
</div>

<button
 class="back"
 onclick="goHome()">
BACK
</button>

</div>


<!-- PASSENGER -->

<div id="passenger" class="card hidden">

<h2>👤 Passenger</h2>

<input
 id="searchBus"
 placeholder="Enter Bus Number - Example 30A"
/>

<button onclick="trackBus()">
🔍 TRACK BUS
</button>

<div id="busInfo"></div>

<div id="map"></div>

<button
 class="back"
 onclick="goHome()">
BACK
</button>

</div>

</div>


<script>

/* =========================================
   HOME
========================================= */

function showDriver(){

 document.getElementById("home")
 .classList.add("hidden");

 document.getElementById("driver")
 .classList.remove("hidden");

}

function showPassenger(){

 document.getElementById("home")
 .classList.add("hidden");

 document.getElementById("passenger")
 .classList.remove("hidden");

 setTimeout(initMap,300);

}

function goHome(){

 location.reload();

}


/* =========================================
   MAP
========================================= */

let map;
let busMarker;

function initMap(){

 if(map) return;

 map = L.map("map").setView(
   [11.0168,76.9558],
   13
 );

 L.tileLayer(
   "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
   {
     maxZoom:19,
     attribution:"© OpenStreetMap"
   }
 ).addTo(map);

}


/* =========================================
   DRIVER GPS
========================================= */

let gpsWatch = null;

function startGPS(){

 const busName =
 document.getElementById("busName")
 .value.trim();

 const busNumber =
 document.getElementById("busNumber")
 .value.trim();

 const boardNumber =
 document.getElementById("boardNumber")
 .value.trim();

 const phone =
 document.getElementById("driverPhone")
 .value.trim();


 if(!busName || !busNumber){

   alert(
    "Bus Name மற்றும் Bus Number enter பண்ணு"
   );

   return;
 }


 if(!navigator.geolocation){

   alert(
    "இந்த browser GPS support செய்யவில்லை"
   );

   return;
 }


 if(!window.isSecureContext){

   alert(
    "GPSக்கு HTTPS தேவை.\n\n" +
    "GitHub Pages HTTPS link-ல் open பண்ணு."
   );

   return;
 }


 navigator.geolocation.getCurrentPosition(

   function(position){

     updateDriver(
       position,
       busName,
       busNumber,
       boardNumber,
       phone
     );

   },

   function(error){

     showGPSError(error);

   },

   {
     enableHighAccuracy:true,
     timeout:30000,
     maximumAge:0
   }

 );


 gpsWatch =
 navigator.geolocation.watchPosition(

   function(position){

     updateDriver(
       position,
       busName,
       busNumber,
       boardNumber,
       phone
     );

   },

   function(error){

     showGPSError(error);

   },

   {
     enableHighAccuracy:true,
     timeout:30000,
     maximumAge:0
   }

 );


 document.getElementById(
  "driverStatus"
 ).innerHTML =
 "🟢 GPS STARTED<br>Waiting for location...";

 document.getElementById(
  "driverStatus"
 ).className =
 "status live";

}


/* =========================================
   DRIVER LOCATION
========================================= */

function updateDriver(
 position,
 busName,
 busNumber,
 boardNumber,
 phone
){

 const lat =
 position.coords.latitude;

 const lng =
 position.coords.longitude;


 console.log(
  "GPS:",
  lat,
  lng
 );


 document.getElementById(
  "driverStatus"
 ).innerHTML =

 "🟢 GPS LIVE<br><br>" +

 "Bus: " + busNumber +
 "<br>" +

 "Latitude: " +
 lat.toFixed(6) +
 "<br>" +

 "Longitude: " +
 lng.toFixed(6);


}


/* =========================================
   GPS ERROR
========================================= */

function showGPSError(error){

 let message =
 "GPS Error";

 if(error.code === 1){

   message =
   "❌ Location permission DENIED";

 }

 else if(error.code === 2){

   message =
   "❌ Location unavailable";

 }

 else if(error.code === 3){

   message =
   "❌ GPS timeout";

 }


 document.getElementById(
  "driverStatus"
 ).innerHTML =
 message;

 document.getElementById(
  "driverStatus"
 ).className =
 "status offline";

}


/* =========================================
   STOP GPS
========================================= */

function stopGPS(){

 if(gpsWatch !== null){

   navigator.geolocation
   .clearWatch(gpsWatch);

   gpsWatch = null;

 }

 document.getElementById(
  "driverStatus"
 ).innerHTML =
 "⚪ GPS STOPPED";

 document.getElementById(
  "driverStatus"
 ).className =
 "status offline";

}


/* =========================================
   PASSENGER
========================================= */

function trackBus(){

 const bus =
 document.getElementById(
  "searchBus"
 ).value.trim();


 if(!bus){

   alert("Bus Number enter பண்ணு");

   return;
 }


 document.getElementById(
  "busInfo"
 ).innerHTML =

 `<div class="status live">
 🚌 Tracking Bus ${bus}
 </div>`;


 /*
  Firebase connection will be added here.

  Driver GPS
       ↓
  Firebase
       ↓
  Passenger
       ↓
  Bus marker moves on map
 */

}


/* =========================================
   DEMO MAP MARKER
========================================= */

function showBusOnMap(lat,lng){

 if(!map){

   initMap();

 }


 if(busMarker){

   busMarker.setLatLng([
     lat,
     lng
   ]);

 }

 else{

   busMarker =
   L.marker([
     lat,
     lng
   ]).addTo(map);

   busMarker.bindPopup(
    "🚌 BUS 30A"
   ).openPopup();

 }


 map.setView(
   [lat,lng],
   16
 );

}

</script>

</body>
</html>
