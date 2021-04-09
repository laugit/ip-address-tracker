<template>
  <div v-bind:class="{ wholepage: !isMobile, 'mobile-wholepage': isMobile }">
    <div v-bind:class="{ home: !isMobile, 'mobile-home': isMobile }">
      <p v-bind:class="{ title: !isMobile, 'mobile-title': isMobile }">
        IP Address Tracker
      </p>
      <div v-bind:class="{ addrdiv: !isMobile, 'mobile-addrdiv': isMobile }">
        <input
          class="addrinput"
          type="text"
          placeholder="Search for any IP address or domain"
          v-model="ipOrDomain"
        />
        <div class="addrsearch">
          <img class="arrow" src="../assets/icon-arrow.svg" />
        </div>
      </div>
      <div v-bind:class="{ addrinfo: !isMobile, 'mobile-addrinfo': isMobile }">
        <div v-bind:class="{ ip: !isMobile, 'mobile-ip': isMobile }">
          <p v-bind:class="{ iptitle: !isMobile, 'mobile-iptitle': isMobile }">
            IP ADDRESS
          </p>
          <p v-bind:class="{ ipaddr: !isMobile, 'mobile-ipaddr': isMobile }">
            {{ currentip }}
          </p>
        </div>
        <div v-if="!isMobile" class="separator"></div>
        <div
          v-bind:class="{ location: !isMobile, 'mobile-location': isMobile }"
        >
          <p
            v-bind:class="{ loctitle: !isMobile, 'mobile-loctitle': isMobile }"
          >
            LOCATION
          </p>
          <p
            v-bind:class="{
              loccontent: !isMobile,
              'mobile-loccontent': isMobile,
            }"
          >
            {{ currentloc.city }}, {{ currentloc.region
            }}<br v-if="!isMobile" />
            {{ currentloc.postalCode }}
          </p>
        </div>
        <div v-if="!isMobile" class="separator"></div>
        <div
          v-bind:class="{ timezone: !isMobile, 'mobile-timezone': isMobile }"
        >
          <p
            v-bind:class="{
              timetitle: !isMobile,
              'mobile-timetitle': isMobile,
            }"
          >
            TIMEZONE
          </p>
          <p
            v-bind:class="{
              timecontent: !isMobile,
              'mobile-timecontent': isMobile,
            }"
          >
            UTC {{ currenttimezone }}
          </p>
        </div>
        <div v-if="!isMobile" class="separator"></div>
        <div
          v-bind:class="{
            isp: !isMobile,
            'mobile-isp': isMobile,
          }"
        >
          <p
            v-bind:class="{
              isptitle: !isMobile,
              'mobile-isptitle': isMobile,
            }"
          >
            ISP
          </p>
          <p
            v-bind:class="{
              ispcontent: !isMobile,
              'mobile-ispcontent': isMobile,
            }"
          >
            {{ currentISP }}
          </p>
        </div>
      </div>
    </div>
    <div
      id="result"
      v-bind:style="{ width: isMobile ? '375px' : '1440px' }"
    ></div>
  </div>
</template>

<style>
@import "Home.css";
@import url("https://fonts.googleapis.com/css2?family=Rubik:wght@400;500;700&display=swap");
</style>

<script lang="js">
import { Options, Vue } from "vue-class-component";
import axios from "axios";
import "leaflet/dist/leaflet.css";
import L from "leaflet/dist/leaflet";

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
   iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
   iconUrl: require('leaflet/dist/images/marker-icon.png'),
   shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {

  data(){
    return {
      ipOrDomain: "",
      currentip: "",
      currentloc: "",
      currenttimezone: "",
      currentISP: "",
      isMobile: window.innerWidth < 376,
      isFixed: false
    };
  },
  methods:{

  },
  mounted() {
    const mapOptions = {
      center: [51.505,-0.09],
      zoom: 13,
      zoomControl: false
    }
    const map = L.map("result", mapOptions);
    window.onresize = () => {
      this.isMobile = window.innerWidth < 376
    }

    //map set to 51.505,-0.09 by default
    map.setView([51.505,-0.09],13);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);
    document.querySelector(".addrsearch").onclick = () => {
      let type = "domain";
      if (this.ipOrDomain.match(/(\d{1-3}\.){4}/) !== null) {
        type = "ipAddress";
      }
      axios
        .get(
          "https://geo.ipify.org/api/v1?apiKey=at_auwNPoeqroKAanL5dGG6iNHV1UyCw&" +
            type +
            "=" +
            this.ipOrDomain
        )
        .then((res) => {
          this.currentip = res.data.ip;
          this.currentloc = res.data.location;
          map.setView([res.data.location.lat, res.data.location.lng],13);
          if(!this.isFixed){
            //document.querySelector('.mobile-addrdiv').classList.add('fixed');
            //document.querySelector('.addrinfo').classList.add('fixed');
            this.isFixed = true;
          }
          //TODO store true in a variable if fixed then on resize check if fixed
          L.marker([res.data.location.lat, res.data.location.lng]).addTo(map);
          this.currenttimezone = res.data.location.timezone;
          this.currentISP = res.data.isp;
        })
        .catch((err) => {
          console.log(err);
        });
    };
  }
}
</script>
