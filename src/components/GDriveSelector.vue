<template>
  <div class="container">
    <div class="file-selector">
      <figure>
        <UploadIcon />
      </figure>
      Select Files from Google Drive
      <p>
        <span>Authenticate with Google Drive</span>
      </p>
      <button type="button" @click="driveIconClicked()">
        Connect to Google Drive
      </button>
    </div>
    <AttachmentList :tempAttachments="getTempAttachments" />
  </div>
</template>

<script>
import AttachmentList from "./AttachmentList";
import UploadIcon from "./UploadIcon";
export default {
  name: "Attachment",
  data() {
    return {
      tempAttachments: [],
      attachments: [],
      tokenClient: null,
      accessToken: null,
      gisInited: null,
      add_id: "",
      developerKey: "AIzaSyBwH-SfDealP62hgDgJPMQWSlqiNzOV3Gg",
      clientId:
        "362666266411-9dce3l8g14ggqp703jl2npk0b2v76rho.apps.googleusercontent.com",
      scope: "https://www.googleapis.com/auth/drive.readonly",
      oauthToken: null,
    };
  },
  components: {
    AttachmentList: AttachmentList,
    UploadIcon,
  },
  mounted() {
    let gDrive = document.createElement("script");
    gDrive.setAttribute("type", "text/javascript");
    gDrive.setAttribute("src", "https://apis.google.com/js/api.js");
    document.head.appendChild(gDrive);
    let client = document.createElement("script");
    client.setAttribute("type", "text/javascript");
    client.setAttribute("src", "https://accounts.google.com/gsi/client");
    document.head.appendChild(client);
  },
  methods: {
    gapiLoaded() {
      gapi.load("client:picker", this.intializePicker);
    },
    async intializePicker() {
      await gapi.client.load(
        "https://www.googleapis.com/discovery/v1/apis/drive/v3/rest"
      );
      this.pickerInited = true;
      this.maybeEnableButtons();
    },
    gisLoaded() {
      this.tokenClient = google.accounts.oauth2.initTokenClient({
        client_id: this.clientId,
        scope: this.scope,
        callback: "",
      });
      this.gisInited = true;
      this.maybeEnableButtons();
    },
    maybeEnableButtons() {
      if (this.pickerInited && this.gisInited) {
      }
    },
    handleAuthClick() {
      this.tokenClient.callback = async (response) => {
        if (response.error !== undefined) {
          throw response;
        }
        this.accessToken = response.access_token;
        await createPicker();
      };

      if (this.accessToken === null) {
        // Prompt the user to select a Google Account and ask for consent to share their data
        // when establishing a new session.
        this.tokenClient.requestAccessToken({ prompt: "consent" });
      } else {
        // Skip display of account chooser and consent dialog for an existing session.
        this.tokenClient.requestAccessToken({ prompt: "" });
      }
    },
    createPicker() {
      const view = new google.picker.View(google.picker.ViewId.DOCS);
      view.setMimeTypes("image/png,image/jpeg,image/jpg");
      const picker = new google.picker.PickerBuilder()
        .enableFeature(google.picker.Feature.NAV_HIDDEN)
        .enableFeature(google.picker.Feature.MULTISELECT_ENABLED)
        .setDeveloperKey(this.developerKey)
        .setAppId(this.add_id)
        .setOAuthToken(this.accessToken)
        .addView(view)
        .addView(new google.picker.DocsUploadView())
        .setCallback(this.pickerCallback)
        .build();
      picker.setVisible(true);
    },
    async pickerCallback(data) {
      if (data.action === google.picker.Action.PICKED) {
        let text = `Picker response: \n${JSON.stringify(data, null, 2)}\n`;
        const document = data[google.picker.Response.DOCUMENTS][0];
        const fileId = document[google.picker.Document.ID];
        console.log(fileId);
        const res = await gapi.client.drive.files.get({
          fileId: fileId,
          fields: "*",
        });
        text += `Drive API response for first document: \n${JSON.stringify(
          res.result,
          null,
          2
        )}\n`;
        console.log(text);
      }
    },
  },
  computed: {
    getTempAttachments() {
      return this.tempAttachments;
    },
    getAttachments() {
      return this.attachments;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.file-selector {
  padding: 55px;
  font-weight: 600;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 4px;
  color: #4e5b69;
  z-index: -9;
}
figure {
  margin: 0px;
}

.dropzone-container {
  display: flex;
  flex-direction: column;
  border: 1px dashed #ccc;
  border-radius: 15px;
}
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
button {
  background: #031629;
  box-shadow: 0 0 2px 0 rgba(3, 22, 41, 0.11),
    0 6px 16px 0 rgba(3, 22, 41, 0.08);
  font-family: SFProDisplay-Regular;
  font-size: 14px;
  color: #ffffff;
  letter-spacing: 0.4px;
  padding: 12px 30px;
  border-radius: 4px;
  outline: none;
  cursor: pointer;
  transition: all 0.25s;
}

button:hover {
  background-color: rgba(65, 184, 131, 1);
  border-color: transparent;
}
.separator {
  position: relative;
}
.separator:after {
  position: absolute;
  content: "";
  height: 1px;
  width: 200px;
  background: #d8d8d8;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
span {
  position: relative;
  background: #f9f9f9;
  padding: 0 4px;
  z-index: 9;
  font-size: 12px;
  color: #4e5b69;
}
</style>
