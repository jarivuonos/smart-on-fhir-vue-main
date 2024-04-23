<template>
  <!-- <h1>{{ msg }}</h1> -->
  <div class="container">
    <div style="text-align: center">
      <h1>Smart on FHIR - Patient Info</h1>
      <p><strong>Username:</strong> fhircamilla/EKOTESTI</p>
      <p><strong>Password:</strong> epicepic1</p>
      <a class="btn btn-info" v-if="code == undefined" style="text-decoration: none" target="_blank"
        :href="authorizeLink">
        Sign in
      </a>
      <hr />
    </div>

    <div v-if="accesstoken">
      <p><strong>Patient Id:</strong> {{ patient }}</p>

<!-- <strong>Name: </strong>{{ patientdata.name }}<br /> -->
      <!-- <strong>Name: </strong>{{ patientdata.name[1].text }}<br /> -->
      <strong>Birth Date: </strong>{{ patientdata.birthDate }} <br />
      <strong>Gender: </strong>{{ patientdata.gender }} <br />
      <strong>Vital Status: </strong>{{ patientdata.deceasedBoolean ? "Dead" : "Alive" }} <br />
      <!-- <strong>Marital Status: </strong>{{ patientdata.maritalStatus.text }} -->
      <br />
      <strong>Telecom: </strong> <br />
      <div v-for="telecom in patientdata.telecom" :key="telecom.value">
        <div class="ml-2">
          <strong>{{ telecom.system }}</strong> -
          {{ telecom.use }}
          {{ telecom.value }}
        </div>
      </div>

      <strong>Address: </strong> <br />
      <div v-for="address in patientdata.address" :key="address.use">
        <div class="ml-2">
          <strong>{{ address.use }} -</strong>
          {{ address.line.toString() }}, {{ address.city }},
          {{ address.district }}, {{ address.state }}, {{ address.postalCode }},
          {{ address.country }}
          <span v-if="address.period?.start"><strong>From</strong></span>
          {{ address.period?.start }}
        </div>
      </div>

      <!-- <strong>Language: </strong>{{ patientdata.communication[0].language.coding[0].display }} <br /> -->

      <!-- <strong>General Practitioner </strong>{{ patientdata.generalPractitioner[0].display }}<br /> -->
      <!-- <strong>Managing Organization </strong>{{ patientdata.managingOrganization.display }}<br /> -->
      <hr />
      <strong>Access code:</strong>
      <p class="ml-2" style="word-break: break-all">{{ accesstoken }}</p>
      <strong>Patient Resource:</strong>
      <pre>{{ patientdata }}</pre>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      code: "",
      accesstoken: "",

      //patient: "eOPSKZbz6YIgoilQprzPy0Q3", // Epic Sandbox patient
      //patient: "eGLLOWTSYzLLEHY9a8HHiPQ", // EKO01 patient

      patientdata: {},
      //clientId: "fe82befd-ebca-4433-b6f7-14b7efa9f7f3", // landofspring
      //clientId: "c18451f3-81d1-4bd2-bc6f-274519a3f25e", // Epic Sandbox client id

      clientId: "0e15d705-6049-449d-95db-05418946fe19",  // Epic Sandbox App2
      
      
      //clientId: "d45049c3-3441-40ef-ab4d-b9cd86a17225", // Epic Sandbox client id from Epic demo app
      
      //clientId: "813e058d-4934-4498-b412-46c8504772da", // Apotti EKO01 patient facing client id
      
      //clientId: "17c6b143-039e-4570-b176-a841a5e7cabe",
      redirect: import.meta.env.PROD
        ? "http://localhost:3000"
        : "http://localhost:3000",
      aud: "https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4",
    };
  
  },
  computed: {
    
    authorizeLink() {
      
      //return `https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/oauth2/authorize?response_type=code&redirect_uri=http://localhost:3000&client_id=17c6b143-039e-4570-b176-a841a5e7cabe&state=abc123&aud=https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/api/FHIR/R4`;
      // return `https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=launch, patient.read, patient.search&aud=https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/api/FHIR/R4`; // EKO01 ei toimi
      return `https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=launch, patient.read, patient.search&aud=${this.aud}`; // &aud=${this.aud} Epic sandbox, toimii
    },
  },
  async mounted() {
    //console.log("auth:" + `https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=launch, patient.read, patient.search&aud=https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/api/FHIR/R4`);

    console.log('redirect: ' + this.redirect);
    console.log('clientId: ' + this.clientId);
    console.log('code: ' + this.$route.query.code);
    console.log('aud: ' + this.aud);
    console.log('----');
    console.log("URL " + `https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=launch patient.read patient.search`);
    //const queryString = window.Location;
//console.log('queryString ' + queryString);

    this.code = this.$route.query.code; 

    if (this.code != undefined) {
      const params = new URLSearchParams();
      params.append("grant_type", "authorization_code");
      params.append("redirect_uri", this.redirect);
      params.append("code", this.code);
      params.append("client_id", this.clientId);
      params.append("state", "1234");
      params.append("patient", this.patient);
      params.append("iss", this.iss);
      params.append("aud", this.aud);

      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };
      await axios
        .post(
         // "https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/oauth2/token",
           "https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token",
          params,
          config
        )
        .then((response) => {
          console.log('response.data ' + JSON.stringify(response.data));
          this.accesstoken = response.data.access_token;
          this.patient = response.data.patient;

          console.log('accesstoken: ' + this.accesstoken);
          console.log('patient: ' + this.patient);
        })
        .catch(function (error) {
          alert('ei OK: ' + error);
          console.log(error);
        });
    }

    if (this.accesstoken != "") {
      await axios
        .get(
         // `https://gw.apottiekosysteemi.fi/Interconnect-FHIR-EKO01/api/FHIR/R4/Patient/${this.patient}`,
          `https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Patient/${this.patient}`,
          { headers: { Authorization: `Bearer ${this.accesstoken}` } }
        )
        .then((response) => {
          this.patientdata = response.data;
          console.log('patname: ' + this.patientdata.name[0].text);
          console.log('GP: ' + this.patientdata.generalPractitioner[0].display);

          console.log(response);
        })
        .catch((error) => {
          console.log(error);
        });
    }
  },
};
// const state = reactive({ count: 0 })
</script>

<style scoped>
/* a {
  color: #42b983;
} */
</style>
