<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { from, map, tap, catchError } from 'rxjs'
import axios from 'axios'
import { parseISO, add } from 'date-fns'
let showError = ref(false)
let contextTokenOptions = ref('')
let redirectionConfig = {
  successUrl: window.location.origin + '/components/connect?accountConnection=success',
  failureUrl: window.location.origin + '/components/connect?accountConnection=failure',
  popup: false
}
let notificationSettings = {
  manualNotifications: false,
  useHostToastrStyles: false
}
let daasUrl = ref('')
let loaded = ref(false)

let contentOverrides = {
  'certua-ob-provider-permissions': {
    howWeAreUsingData: '<p>[Custom text about how you use data]</p>'
  },
  'certua-ob-manage-providers': {
    confirmDisconnect: {
      implicationOfRevocation: '<p>[Custom text about implications of revoke]</p>',
      whatHappens: '<p>[Custom text about what happens to data if they revoke] </p>',
      relink: '<p>[Custom text about ability to relink]</p>',
      confirm: '<p>[Custom text asking user to confirm]</p>',
      termsOfServiceLink: `<a href='#'>[Custom Link to Terms of service here]</a>`
    },
    disconnectSuccess: {
      implicationOfRevocation: '<p>[Custom text about implications of revoke]</p>',
      whatHappens: '<p>[Custom text about what happens to data now account are disconnected]</p>'
    }
  }
}

function checkExpiry() {
  let token = JSON.parse(localStorage.getItem('apiConfig') ?? '')
  let tokenCreation = parseISO(token.dateCreated)
  if (add(tokenCreation, { minutes: 30 }) <= new Date()) {
    showError.value = true
    localStorage.removeItem('apiConfig')
  }

  let root = document.documentElement
  if (!root.style.getPropertyValue('--primary')) {
    let primary = localStorage.getItem('--primary')
    root.style.setProperty('--primary', primary)

    let secondary = localStorage.getItem('--secondary')
    root.style.setProperty('--secondary', secondary)
  }
}
// lifecycle hooks
onMounted(() => {
  if (!localStorage.getItem('apiConfig')) {
    showError.value = true
  } else {
    checkExpiry()
    contextTokenOptions.value = JSON.parse(localStorage.getItem('apiConfig') ?? '')
    window.opener?.postMessage('close-window-success', {
      targetOrigin: '*'
    })
  }
  if (localStorage.getItem('daasUrl')) {
    daasUrl.value = localStorage.getItem('daasUrl') ?? ''
  }
  loaded.value = true
})
</script>

<template>
  <div class="row" v-if="!showError">
    <h2>Connect Accounts</h2>
    <p>
      This component can be used to facilitate a user connecting their bank account via Open Banking
    </p>
    <certua-ob-connect
      v-if="loaded"
      :contentOverrides="contentOverrides"
      :contextData="contextTokenOptions"
      :daasUrl="daasUrl"
      :redirectionConfig="redirectionConfig"
      :startingPhase="'InstitutionSelection'"
      :notificationSettings="notificationSettings"
    >
    </certua-ob-connect>
  </div>
  <div>
    <h4>Example code</h4>
    <pre><code>
      &lt;certua-ob-connect 
        :contentOverrides="contentOverrides"
        :redirectionConfig="redirectionConfig"
        :contextData="contextData"
        :daasUrl="daasUrl"
        :startingPhase="'InstitutionSelection'"&gt;
        :notificationSettings="notificationSettings"
      &lt;/certua-ob-connect&gt;
      </code>
    </pre>
    <table class="table">
      <thead>
        <th>Property Name</th>
        <th>Mandatory</th>
        <th>Description</th>
      </thead>
      <tbody>
        <tr>
          <td>contextData</td>
          <td>Yes</td>
          <td>
            This is a JSON string which contains your context token and user reference. <br /><code>
              {"contextToken":"FF5D16AAE1ED74E4C8F0E8B6D9E2EB06","ownerId":"1","dateCreated":"2023-03-20T11:17:24.121Z"}
            </code>
          </td>
        </tr>
        <tr>
          <td>redirectionConfig</td>
          <td>Yes</td>
          <td>
            This is a JSON string which contains the success or failure redirection Urls, which are
            used after the connection to the bank. Popup controls whether or not the bank url is
            shown in the same tab or in a new tab, this defaults to false if not present<br /><code
              >{ successUrl: 'http://localhost:5713/components/connect?accountConnection=success',
              failureUrl: 'http://localhost:5713/components/connect?accountConnection=failure',
              popup: false }</code
            >
          </td>
        </tr>
        <tr>
          <td>contentOverrides</td>
          <td>No</td>
          <td>
            This is a JSON string which contains any text overrides that you want to pass to the
            information screens/modals<br /><code>
              { 'certua-ob-provider-permissions': { howWeAreUsingData: ' &lt;p&gt;[Custom text about
              how you use data]&lt;/p&gt; ' }, 'certua-ob-manage-providers': { confirmDisconnect: {
              implicationOfRevocation: ' &lt;p&gt;[Custom text about implications of
              revoke]&lt;/p&gt; ', whatHappens: ' &lt;p&gt;[Custom text about what happens to data
              if they revoke]&lt;/p&gt; ', relink: ' &lt;p&gt;[Custom text about ability to
              relink]&lt;/p&gt; ', confirm: ' &lt;p&gt;[Custom text asking user to
              confirm]&lt;/p&gt; ', termsOfServiceLink: `&lt;a href="#" &gt;[Custom Link to Terms of
              service here]&lt;/a &gt;` }, disconnectSuccess: { implicationOfRevocation: '
              &lt;p&gt;[Custom text about implications of revoke]&lt;/p&gt; ', whatHappens: '
              &lt;p&gt;[Custom text about what happens to data now account are
              disconnected]&lt;/p&gt; ' } } }
            </code>
          </td>
        </tr>
        <tr>
          <td>startingPhase</td>
          <td>No</td>
          <td>
            This is a string which controls the starting screen when connecting accounts, this can
            be <code>AccountList</code> or <code>InstitutionSelection</code>.
          </td>
        </tr>
        <tr>
          <td>daasUrl</td>
          <td>No</td>
          <td>
            This only needs to be passed in if you wish to load the Daas Elements from your own CDN
            rather than Certua's. Must be an absolute URL.
          </td>
        </tr>
        <tr>
          <td>notificationSettings</td>
          <td>No</td>
          <td>
            This is JSON which contains any custom notification settings required for your
            implementation. By default this property is not required if you simply wish to use the
            default style toastrs.<br />
            Manual Notifications: if this is set to true then toastrs are disabled entirely and the
            host will be required to listen to 'certua-ob-notifications' channel on the eventbus.
            <br />Use External Toastr Styles: Intended for Internal Certua use, this leaves toastrs
            outside of the shadow dom and the host app has to manage ngx-toastr stylesheet
            <br />
            <code> { "manualNotifications": "boolean" , "useHostToastrStyles": "boolean" } </code>
          </td>
        </tr>
      </tbody>
    </table>
  </div>

  <div class="row" v-if="showError">
    <div class="col">
      <h3>You do not have a context token</h3>
      <RouterLink to="/">Home</RouterLink>
    </div>
  </div>
</template>
