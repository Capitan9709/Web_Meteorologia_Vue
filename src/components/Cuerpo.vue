<script setup>
import { ref } from "vue";

window.onload = () => {
    getLocation();

    // se lanzan el resto de peticiones cada minuto 
    // para actualizar los datos
    window.setInterval(lanzarPeticion, 60000);

}


var peticionEnCurso = false;
var contador = 0;
var url;
var httpRequest;
var localizacion = ref("");
var direccionViento = ref("");
var imagen = ref("");
var imagenAlt = ref("");
var descripcion = ref("");
var viento = ref("");
var temperatura = ref("");
var nivelMar = ref("");


// muestra la posicion del usuario
function getLocation() {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition, showError);
        console.log("Geolocation is supported by this browser.");
    } else {
        div.innerHTML = "Geolocation is not supported by this browser.";
    }
  }

// muestra la posicion del usuario
function showPosition(position) {

    latitud.innerHTML =  position.coords.latitude;
    longitud.innerHTML =  position.coords.longitude;
    
    getCity(position)

    crearUrl(latitud.innerHTML, longitud.innerHTML).then(lanzarPeticion);
    
}

// obtener la ciudad del usuario
function getCity(position) {
    let lat = position.coords.latitude;
    let lon = position.coords.longitude;
    let url = "https://maps.googleapis.com/maps/api/geocode/json?key=AIzaSyBt351dMXnP7LskzYpyeugrcX-OebFqkMM&latlng="+ lat + "," + lon +"&sensor=false";
    let httpRequest = new XMLHttpRequest();
    httpRequest.open("GET", url);
    httpRequest.onreadystatechange = function() {
        if (httpRequest.readyState === XMLHttpRequest.DONE) {
            if (httpRequest.status === 200) {
                let respuesta = JSON.parse(httpRequest.responseText);
                console.log(respuesta);
                var ciudad = respuesta.results[0].address_components[2].long_name;
                ciudad += ", " + respuesta.results[0].address_components[3].long_name;

                localizacion.value = ciudad
            }
        }
    }
    httpRequest.send();
}

// muestra el error si no se puede obtener la posicion
function showError(error) {
    let div = document.getElementById("datosTiempo");
    switch(error.code) {
      case error.PERMISSION_DENIED:
        div.innerHTML = "<h2 class='text-danger fw-bolder'>Has denegado el uso de la Geolocalización.</h2>"
        break;
      case error.POSITION_UNAVAILABLE:
        div.innerHTML = "<h2 class='text-danger fw-bolder'>No se puede acceder a la localización.</h2>"
        break;
      case error.TIMEOUT:
        div.innerHTML = "<h2 class='text-danger fw-bolder'>Se ha excedido el tiempo necesario para obtener la localización del usuario.</h2>"
        break;
      case error.UNKNOWN_ERROR:
        div.innerHTML = "<h2 class='text-danger fw-bolder'>A ocurrido un error inesperado.</h2>"
        break;
    }
} 

// crar promesa para crear la url
function crearUrl(latitud, longitud){
    return new Promise((resolve, reject) => {
        url = `https://api.open-meteo.com/v1/forecast?latitude=${latitud}&longitude=${longitud}&current_weather=true&hourly=temperature_2m,relativehumidity_2m,windspeed_10m`;
        resolve();
    });
}

// lanza peticion de peliculas
function lanzarPeticion() {
    
    if(peticionEnCurso == false){

        contador++;
        peticionEnCurso = true;

        httpRequest = new XMLHttpRequest();
    
        httpRequest.open("GET", url);
        httpRequest.onreadystatechange = trataRespuesta;
        httpRequest.send();

    }
}

// muestra los datos de la peticion
function trataRespuesta() {
    if (httpRequest.readyState === XMLHttpRequest.DONE) {
        if (httpRequest.status === 200) {
            let respuesta = JSON.parse(httpRequest.responseText);
            console.log(respuesta);
            console.log("Numero de peticiones: "+contador);
            
            temperatura.value = respuesta.current_weather.temperature;
            nivelMar.value = respuesta.elevation;

            if (respuesta.current_weather.winddirection >= 331 || respuesta.current_weather.winddirection <= 29) {
                direccionViento.value = "Norte";
            }
            else if (respuesta.current_weather.winddirection >= 30 || respuesta.current_weather.winddirection <= 69) {
                direccionViento.value = "Noreste";
            }
            else if (respuesta.current_weather.winddirection >= 70 || respuesta.current_weather.winddirection <= 109) {
                direccionViento.value = "Este";
            }
            else if (respuesta.current_weather.winddirection >= 110 || respuesta.current_weather.winddirection <= 149) {
                direccionViento.value = "Sureste";
            }
            else if(respuesta.current_weather.winddirection >= 150 || respuesta.current_weather.winddirection <= 210) {
                direccionViento.value = "Sur";
            }
            else if (respuesta.current_weather.winddirection >= 211 || respuesta.current_weather.winddirection <= 240) {
                direccionViento.value = "Suroeste";
            }
            else if (respuesta.current_weather.winddirection >= 241 || respuesta.current_weather.winddirection <= 300) {
                direccionViento.value = "Oeste";
            }
            else if (respuesta.current_weather.winddirection >= 301 || respuesta.current_weather.winddirection <= 330) {
                direccionViento.value = "Noroeste";
            }

            viento = respuesta.current_weather.windspeed;

            if (respuesta.current_weather.weathercode == 0) {
                imagen.value = "/img/soleado.svg";
                imagenAlt.value = "sol";
                descripcion.value = "Despejado";
            }
            else if (respuesta.current_weather.weathercode == 1) {
                imagen.value = "/img/soleado.svg";
                imagenAlt.value = "sol";
                descripcion.value = "Mayormente Soleado";
            }
            else if (respuesta.current_weather.weathercode == 2) {
                imagen.value = "/img/poconuboso.svg";
                imagenAlt.value = "poconuboso";
                descripcion.value = "Poco Nuboso";
            }
            else if (respuesta.current_weather.weathercode == 3) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado";
            }
            else if (respuesta.current_weather.weathercode == 4) {
                imagen.value = "/img/tormenta.svg";
                imagenAlt.value = "tormenta";
                descripcion.value = "Tormentoso";
            }
            else if (respuesta.current_weather.weathercode == 5) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado con Lluvias";
            }
            else if (respuesta.current_weather.weathercode == 6) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado con Nevadas";
            }
            else if (respuesta.current_weather.weathercode == 7) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado con Tormentas";
            }
            else if (respuesta.current_weather.weathercode == 8) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado con Tormenta y Lluvias";
            }
            else if (respuesta.current_weather.weathercode == 9) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes";
                descripcion.value = "Nublado con Tormenta y Nevadas";
            }
            else if (respuesta.current_weather.weathercode == 10) {
                imagen.value = "/img/nublado.svg";
                imagenAlt.value = "nubes"; 
                descripcion.value = "Nublado con Tormenta y Granizadas";
            }
            else {
                descripcion.value = "El tiempo es desconocido";
            }
            
            peticionEnCurso = false;
        } 
        else {
            alert("Hubo un problema con la petición.");
        }
    }
}
</script>

<template>
    <div id="contenedor" class="mx-3">
        <h2 hidden>Latitud: <span id="latitud"></span> Longitud: <span id="longitud"></span></h2><br>
        <h2 class="text-center">Localización: {{ localizacion }}</h2><br>
        <h2>Datos En Tiempo Real:</h2><br>
        <div id="Tiempo" class="d-flex flex-column">
            <div id="datosTiempo" class="mx-5">
                <h4><i class="fa-solid fa-temperature-quarter"></i> {{ temperatura }} ºC</h4>

                <img :src="imagen" :alt="imagenAlt"/>
                <h4>{{ descripcion }}</h4>
            </div>
            <div id="detallesTiempo" class="mx-5 mt-3 mb-3">
                <h4><i class="fa-solid fa-water"></i> {{nivelMar}} metros sobre el nivel del mar</h4><br>
                
                <h4><i class='fa-solid fa-wind'></i> {{ viento }} Km/h | Proviene del: {{ direccionViento }}</h4><br>
            </div>
        </div>
         
    </div>
</template>

<style scoped>

</style>
