<template>
  <div>
    <button class="UppyModalOpenerBtn" style="display: none;">
      upload
    </button>
    <div class="DashboardContainer"></div>
    <div class="upload-form"></div>
  </div>
</template>
<style>
  @import '~@uppy/core/dist/style.css';
  @import '~@uppy/dashboard/dist/style.css';
  .UppyDragDrop-Progress {
    position: relative;
  }
</style>
<script>
  const Webcam = require("@uppy/webcam");
  const XHRUpload = require("@uppy/xhr-upload");
  const Uppy = require('@uppy/core')
  const Dashboard = require('@uppy/dashboard')
  import config from "../../config";
  import Storage from '../../util/storage';

  export default {
    props: {
      uploaded: {
        type: Function,
        required: false,
        default: function () {
          return console.log;
        }
      },
      album: {
        type: String,
        required: false,
        default: 'default',
      }
    },
    data() {
      return {
        images: []
      };
    },
    mounted() {
      const storage = new Storage();
      const token = storage.getItem('token');
      if (!token) {
        return this.$router.push('/login');
      }
      const self = this;
      const uppy = Uppy({
        debug: true,
        autoProceed: false,
        limit: 1,
        bundel: true,
        formData: false,
        onBeforeUpload: (files) => {
          uppy.setMeta({ album: this.album })
        },
        restrictions: {
          maxFileSize: 100000000,
          maxNumberOfFiles: 50,
          minNumberOfFiles: 1,
          allowedFileTypes: ["image/*", "video/*"]
        }
      })
      uppy.use(Dashboard, {
        trigger: ".UppyModalOpenerBtn",
        inline: true,
        target: ".DashboardContainer",
        replaceTargetContent: true,
        note: "Images and video only, 1–50 files, up to 10 MB",
        maxHeight: 450,
        //   { id: "license", name: "License", placeholder: "specify license" },
        // ]
      }).use(Webcam, { target: Dashboard })
      uppy.use(XHRUpload, {
          endpoint: '//' + config.api + '/admin/photos?albumId=' + self.album,
          headers: {
            Authorization: 'Bearer ' + token
          },
          getResponseData(response) {
            console.log('????', response);
            try {
              const result = JSON.parse(response)
              if (result.message === 'ok') {
                const image = result.data
                self.images = image
                self.uploaded(image)
              }
              return result;
            } catch (err) {
              console.error(err)
            }
          },
      });
      uppy.upload().then((result) => {
        if (result.failed.length > 0) {
        result.failed.forEach((file) => {
            console.error(file.error)
          })
        }
      })
      uppy.on("complete", function (result) {
        console.log("successful files:", result.successful);
        console.log("failed files:", result.failed);
      });
    }
  };
</script>
