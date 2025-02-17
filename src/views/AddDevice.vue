<template>
  <amb-app-frame>
    <v-container
      fluid
    >
      <v-row
        dense
      >
        <v-col class="ma-0 pa-0">
          <v-breadcrumbs :items="breadcrumbs" />
        </v-col>
      </v-row>
      <v-row
        align="center"
      >
        <v-col cols="12">
          <v-alert
            v-if="this.edgeDeviceError"
            outlined
            type="warning"
            dense
            align-self="center"
            class="text-center"
            transition="scale-transition"
            dismissible
            data-cy="edge-device-error"
            ref="edge-device-error"
          >
            {{ this.edgeDeviceError }}
          </v-alert>
        </v-col>
      </v-row>
      <v-row
        justify="center"
        class="pb-5"
        align="center"
      >
        <v-card flat>
          <v-card-title
            data-cy="titlecard"
          >
            Add Ambianic Edge device
          </v-card-title>
          <v-card-text grid-list-sm>
            <v-row
              align="start"
              justify="space-around"
            >
              <v-col
                style="max-width: 420px;"
                align="center"
                justify="center"
                cols="12"
                class="pa-0 ma-0 fill-height"
              >
                <v-stepper
                  v-model="addDeviceStep"
                  vertical
                >
                  <v-stepper-step
                    :complete="addDeviceStep > 1"
                    step="1"
                    :rules="[() => true]"
                  >
                    Choose connection method
                  </v-stepper-step>
                  <v-stepper-content step="1">
                    <v-card>
                      <v-card-text>
                        <p class="text-left">
                          Discover a device on the local WiFi network or connect remotely.
                        </p>
                      </v-card-text>
                      <v-card-actions>
                        <v-btn
                          data-cy="btn-local"
                          @click="chooseDiscoverLocal"
                        >
                          Local
                        </v-btn>
                        <v-spacer />
                        <v-btn
                          data-cy="btn-remote"
                          @click="chooseRemoteConnection"
                        >
                          Remote
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-stepper-content>
                  <v-stepper-step
                    :complete="addDeviceStep > 2"
                    step="2"
                  >
                    Identify Device
                  </v-stepper-step>
                  <v-stepper-content step="2">
                    <v-card
                      v-if="isChoiceDiscoverLocal"
                    >
                      <v-card-text>
                        <v-list
                          v-if="isPeerDiscovered"
                        >
                          <v-subheader>Local Devices</v-subheader>
                          <v-list-item-group
                            color="primary"
                            v-model="selectedLocalDevice"
                            mandatory
                          >
                            <v-list-item
                              v-for="(item, i) in discoveredPeers"
                              :key="i"
                            >
                              <v-list-item-icon>
                                <v-icon>mdi-identifier</v-icon>
                              </v-list-item-icon>
                              <v-list-item-content>
                                <v-list-item-title v-text="item" />
                              </v-list-item-content>
                            </v-list-item>
                          </v-list-item-group>
                        </v-list>
                        <v-skeleton-loader
                          v-else
                          type="list-item-avatar"
                        />
                      </v-card-text>
                      <v-card-actions>
                        <v-btn
                          :disabled="selectedLocalDevice < 0"
                          @click="clickConnectToDiscoveredDevice"
                          data-cy="btn-connect-discovered"
                        >
                          Connect
                        </v-btn>

                      </v-card-actions>
                      <pre id="log"></pre>
                    </v-card>
                    <v-card
                      v-else
                    >
                      <v-card-title>
                        Remote device ID
                      </v-card-title>
                      <v-card-text>
                        <p class="text-left">
                          Enter the Peer ID of the remote Ambianic Edge device.
                        </p>
                        <v-text-field
                          v-model="edgePeerId"
                          type="text"
                          label="Peer ID of remote Ambianic Edge device*"
                          placeholder="Enter Peer ID"
                          id="remotePeerID"
                          outlined
                          dense
                          class="mt-4"
                          data-cy="input-remotePeerID"
                          :rules="[rules.required, rules.validPeerID]"
                        />
                      </v-card-text>
                      <v-card-actions>
                        <v-btn
                          data-cy="btn-connect-remote"
                          @click="clickConnectToRemoteDevice"
                          :disabled="!isPeerIdValid"
                        >
                          Connect
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-stepper-content>
                  <v-stepper-step
                    :complete="addDeviceStep > 3"
                    step="3"
                    :rules="[() => true]"
                  >
                    Connect
                    <small>Establishing connection with Ambianic Edge device...</small>
                  </v-stepper-step>
                  <v-stepper-content step="3">
                    <v-progress-linear
                      v-if="!this.isPeerConnectionError"
                      color="info"
                      indeterminate
                      :size="50"
                      :width="7"
                    />
                  </v-stepper-content>
                  <v-stepper-step step="4">
                    Done
                  </v-stepper-step>
                  <v-stepper-content step="4">
                    <v-card>
                      <v-card-title
                        data-cy="title-success"
                      >
                        <v-icon
                          x-large
                          left
                          color="green"
                        >
                          playlist_add_check_circle
                        </v-icon>
                        <span>Success!</span>
                      </v-card-title>
                      <v-card-text>
                        <p class="text-left">
                          Device information saved.
                        </p>
                      </v-card-text>
                      <v-card-actions>
                        <v-btn
                          to="timeline"
                        >
                          Timeline
                        </v-btn>
                        <v-spacer />
                        <v-btn
                          to="devicecard"
                          data-cy="btn-settings"
                        >
                          Settings
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-stepper-content>
                </v-stepper>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-row>
    </v-container>
  </amb-app-frame>
</template>
<script id="api-source">// Bugout demo server API code
// Edit this to change the live API this server is running:

// respond to ping calls by sending back "pong"
b.register("ping", function(pk, args, cb) {
  args["pong"] = true;
  cb(args);
}, "Respond to ping with 'pong'.");
</script>
<script>
import { mapActions, mapState } from 'vuex'
import {
  PEER_DISCOVERING_DONE,
  PEER_CONNECTED,
  PEER_CONNECTION_ERROR
} from '@/store/mutation-types'
import {
  CHANGE_REMOTE_PEER_ID,
  PEER_DISCOVER
} from '@/store/action-types.js'

export default {
  components: {
    AmbAppFrame: () => import('@/components/AppFrame.vue')
  },
  data () {
    return {
      edgePeerId: undefined,
      isPeerIdValid: false,
      edgeDeviceError: null,
      syncing: false, // is the UI in the process of syncing with remote device data
      rules: {
        required: value => !!value || 'Required.',
        validPeerID: value => this.validatePeerID(value) || 'Must be a valid peer ID.'
      },
      addDeviceStep: 1, // the sequential step number in the add device stepper flow
      isChoiceDiscoverLocal: false, // user chooses to discover a local device vs remote connection
      selectedLocalDevice: -1, // device number picked by the user from a list of discovered local devices
      breadcrumbs: [
        {
          text: 'Settings',
          disabled: false,
          to: 'settings'
        },
        {
          text: 'Create a Deployment',
          disabled: false,
          to: 'selectdevice'
        },
        {
          text: 'Add Repository',
          disabled: true,
          to: 'adddevice'
        }

      ]
    }
  },
  created () {
  },
  async mounted () {
  },
  methods: {
    // Validate the user input so the ID has the correct format before showing the connect button
    validatePeerID (value) {
      if (value && /^([a-zA-Z0-9]{8})-([a-zA-Z0-9]{4})-([a-zA-Z0-9]{4})-([a-zA-Z0-9]{4})-([a-zA-Z0-9]{12})$/.test(value)) {
        this.isPeerIdValid = true
      } else {
        // if value is not matching regex, hide Connect button
        this.isPeerIdValid = false
      }
      console.debug('isPeerIdValid (value)', { value }, this.isPeerIdValid)
      return this.isPeerIdValid
    },
    ...mapActions({
      switchEdgeDeviceConnection: CHANGE_REMOTE_PEER_ID,
      addDeviceCard: 'myDevices/add',
      updateDeviceCard: 'myDevices/update',
      setCurrentDevice: 'myDevices/setCurrent',
      updateFromRemote: 'myDevices/updateFromRemote',
      peerDiscover: PEER_DISCOVER
    }),
    /**
     * User clicked Connect to a discovered local device
     */
    async clickConnectToDiscoveredDevice () {
      console.debug('clickConnectToDiscoveredDevice() called')
      this.addDeviceStep++
      this.edgePeerId = this.discoveredPeers[this.selectedLocalDevice]
      console.debug('User selected repository:', this.selectedLocalDevice, this.edgePeerId)
      await this.deviceConnect()
    },
    /**
     * User clicked Connect to a remote device
     */
    async clickConnectToRemoteDevice () {
      console.debug('clickConnectToRemoteDevice() called')
      this.addDeviceStep++
      console.debug('User requested remote connection to device ID:', this.edgePeerId)
      await this.deviceConnect()
    },
    async deviceConnect () {
      await this.switchEdgeDeviceConnection(this.edgePeerId)
    },
    /**
     * User wants local device discovery
     */
    async chooseDiscoverLocal () {
      console.debug('chooseDiscoverLocal() called')
      this.addDeviceStep++
      this.isChoiceDiscoverLocal = true
      this.discoverLocalEdgeDevice()
    },
    async discoverLocalEdgeDevice () {
      this.edgePeerId = undefined
      console.debug('discoverLocalEdgeDevice() called')
      console.debug('removing any existing peer connection')
      await this.peerDiscover()
      console.debug('discoverLocalEdgeDevice() ended')
    },
    /**
     * User wants to connect to a remote device
     */
    async chooseRemoteConnection () {
      console.debug('chooseDiscoverLocal() called')
      this.addDeviceStep++
      this.isChoiceDiscoverLocal = false
    },
    /**
     * Connection step completed.
     */
    async connectStepCompleted () {
      console.debug('connectStepCompleted() called')
      // fetch device info and update vuex state
      await this.fetchEdgeDetails()
      // switch current device reference in UI state (vuex store)
      await this.setCurrentDevice(this.edgePeerId)
      this.addDeviceStep++
    },
    async fetchEdgeDetails () {
      try {
        const details = await this.pnp.edgeAPI.getEdgeStatus()
        console.debug('Edge device details fetched:', { details })
        if (!details || !details.version) {
          this.edgeDeviceError = 'Edge device requires update.'
        } else {
          console.debug('this.edgePeerId', this.edgePeerId)
          details.peerID = this.edgePeerId
          await this.updateFromRemote(details)
        }
        return details
      } catch (e) {
        this.edgeDeviceError = 'Edge device API offline or unreachable.'
        console.debug(this.edgeDeviceError, e)
      }
    }
  },
  computed: {
    ...mapState({
      discoveryStatus: state => state.pnp.discoveryStatus,
      isPeerDiscovered: state => state.pnp.discoveryStatus === PEER_DISCOVERING_DONE,
      discoveredPeers: state => state.pnp.discoveredPeers,
      peerConnectionStatus: state => state.pnp.peerConnectionStatus,
      isPeerConnectionError: state => state.pnp.peerConnectionStatus === PEER_CONNECTION_ERROR,
      isEdgeConnected: state =>
        state.pnp.peerConnectionStatus === PEER_CONNECTED,
      pnp: state => state.pnp,
      allDeviceCards: state => state.myDevices.allDeviceCards
    })
  },
  watch: {
    edgePeerId (value) {
      // this.edgePeerId = value
      this.validatePeerID(value)
    },
    isEdgeConnected: async function (isConnected) {
      if (isConnected) {
        await this.connectStepCompleted()
      }
    },
    isPeerConnectionError: async function (isError) {
      console.debug('watch peerConnectionError triggered. New value', { isError })
      if (isError) {
        this.edgeDeviceError = this.$store.state.pnp.userMessage
        console.debug('isPeerConnectionError TRUE. Error message:', this.edgeDeviceError)
      } else {
        // clear the user friendly error message
        this.edgeDeviceError = undefined
        console.debug('isPeerConnectionError FALSE. Error message:', this.edgeDeviceError)
      }
    }
  }
}
</script>
<script>
  log("*BUGOUT*\n");
  log("Server log started.\n");

  // instantiate our bugout instance
  var b = new Bugout({seed: localStorage["bugout-demo-server-seed"]});

  // save the seed for next time
  localStorage["bugout-demo-server-seed"] = b.seed;

  // log this server's address
  log("\nThis browser tab is running a Bugout server.");
  log("This server's address:", b.address());
  log("announcing...");

  /*** logging ***/

  // log when network connectivity changes
  var connected = false;
  b.on("connections", function(c) {
    if (c == 0 && connected == false) {
      connected = true;
      log("ready");
      // link to the messageboard client URL
      var clientURL = document.location.href.replace("server.html", "") + "#" + b.address();
      // qrcode for the client URL
      if (window.innerWidth > 600) {
        log(qr(clientURL));
      } else {
        log();
      }
      log(clientURL + "\n");
      log("Connect back to this server-in-a-tab using the link above.");
    }
    log("connections:", c);
  });

  // log when a client sends a message
  b.on("message", function(address, msg) { log("message:", address, msg); });

  // log when a client makes an rpc call
  b.on("rpc", function(address, call, args) { log("rpc:", address, call, args); });

  // log when we see a new client address
  b.on("seen", function(address) { log("seen:", address); });

  // simple logging function
  function log() {
    var args = Array.prototype.slice.call(arguments);
    console.log.apply(null, args);
    args = args.map(function(a) { if (typeof(a) == "string") return a; else return JSON.stringify(a); });
    document.getElementById("log").textContent += args.join(" ") + "\n";
  }

  // generate a text based qr code

  function qr(txt) {
    var q = QRCode.create(txt);
    var code = "\n\n";
    for (var i=0; i<2; i++) {
      code += "  ████";
      for (var j=0; j<q.modules.size; j++) {
        code += "██";
      }
      code += "████\n";
    }
    for (var i=0; i<q.modules.size; i++) {
      code += "  ████";
      for (var j=0; j<q.modules.size; j++) {
        code += q.modules.data[i * q.modules.size + j] ? "  " : "██";
      }
      code += "████\n";
    }
    for (var i=0; i<2; i++) {
      code += "  ████";
      for (var j=0; j<q.modules.size; j++) {
        code += "██";
      }
      code += "████\n";
    }
    return code + "\n\n";
  }

  /*** API editing ***/

  // reference to the API code textarea
  var api = document.getElementById("api");

  // add a codemirror editor
  var editor = CodeMirror.fromTextArea(document.getElementById("api"), {
    lineNumbers: true,
    matchBrackets: true,
    continueComments: "Enter",
    viewportMargin: Infinity,
    extraKeys: {"Ctrl-Q": "toggleComment"}
  });

  // function to reload the API from the textarea
  function reload(code) {
    if (code) {
      editor.getDoc().setValue(code);
    }
    editor.save();
    b.api = {};
    eval(api.value);
    localStorage["bugout-demo-api"] = api.value;
  }

  // when the user clicks the reload button
  document.getElementById("reload").onclick = function() {
    reload();
  }
</script>
<script id="api-source">// Bugout demo server API code
// Edit this to change the live API this server is running:

// respond to ping calls by sending back "pong"
b.register("ping", function(pk, args, cb) {
  args["pong"] = true;
  cb(args);
}, "Respond to ping with 'pong'.");
</script>
  <script src="bugout.min.js" type="application/javascript"></script>
