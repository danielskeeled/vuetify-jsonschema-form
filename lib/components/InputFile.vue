<template>
    <v-layout row wrap>
        <input :id="inputId" ref="file" :accept="accept" type="file" style="display: none;" @change="onChange"/>
        <v-flex xs12>
            <v-btn id="uploadFileButton"
                   :loading="loading"
                   :disabled="disabled"
                   color="primary"
                   block
                   @click="pickFile()">
                <span>{{label ? label : 'Upload'}}</span>
                <v-icon right dark>cloud_upload</v-icon>
            </v-btn>
        </v-flex>

        <v-flex v-if="removable" xs12 d-flex>
            <v-icon v-if="fileName" small color="primary" @click="removeFile()">close</v-icon>
            <span>{{fileName}}</span>
        </v-flex>
        <v-flex v-else-if="fileName" xs12 d-flex>
            <span v-if="fileName">Filename: {{fileName}}</span>
        </v-flex>
        <v-flex v-if="value.url" :key="value.url" xs12 d-flex>
            <span>URL: {{value.url}}</span>
            <img v-if="isImage" :src="value.url" style="width: 50px; height: 50px;"/>
        </v-flex>
    </v-layout>
</template>

<script>
const imageFormats = ['jpg', 'jpeg', 'png', 'bmp', 'JPG', 'JPEG', 'PNG', 'BMP'];

export default {
    name: 'input-file',
    props: {
        value: { type: Object, default: () => {} },
        disabled: { type: Boolean, default: false },
        removable: { type: Boolean, default: true },
        // required because all instances of this component on the DOM will catch the input event altering
        // the v-model variable instead of changing only the wanted variable
        inputId: { type: String, required: true },
        label: { type: String, default: '' },
        accept: {
            type: String,
            default: 'jpg,jpeg,png,bmp,JPG,JPEG,PNG,BMP,.otf,.OTF,.ttf,.TTF,.woff,.WOFF,.woff2,.WOFF2' +
                ',image/*,application/pdf,application/msword,text/plain,.docx,application/font-woff' +
                ',application/font-woff2,application/x-font-ttf,application/x-font-truetype' +
                ',application/x-font-opentype,application/vnd.ms-fontobject'
        },
        maxSize: { type: Number, default: 10 }, // megabytes
    },
    computed: {
        isImage () {
            let extension = '';
            if (this.value && this.value.url) extension = this.value.url.split('.').pop();
            if (imageFormats.indexOf(extension) !== -1) return true;
            return false;
        },
    },
    data () {
        return {
            fileName: '',
            loading: false,
        };
    },
    created () {
        if (this.value && this.value.name) this.fileName = this.value.name;
    },
    methods: {
        /**
         * Checks the file max size and the extension, converts the file to base 64 and emits the file object
         *
         * @param {Object} e - the event
         */
        onChange (e) {
            this.loading = true;
            const files = e.target.files || e.dataTransfer.files;

            if (!files.length) {
                return;
            }

            const file = files[0];
            const size = file.size && (file.size / (1000 ** 2));

            // check file max size
            if (size > this.maxSize) {
                this.$emit('size-exceeded', size);
                this.$_toastHandler_toastAPI('genericLoadDataError');
                this.removeFile();
                return;
            }

            if (this.accept.indexOf(file.name.split('.').pop()) < 0) {
                this.$_toastHandler_toastAPI('invalidFileExtension');
                this.removeFile();
                return;
            }

            // update file
            this.file = file;

            const reader = new FileReader();

            reader.onload = () => {
                let binary = '';
                const bytes = new Uint8Array(reader.result);
                const length = bytes.byteLength;
                for (let i = 0; i < length; i += 1) {
                    binary += String.fromCharCode(bytes[i]);
                }

                // Gets the binary string and converts it to base64 using the btoa method
                file.base64 = btoa(binary);

                const fileObject = {
                    base64: file.base64,
                    name: file.name,
                    type: file.type,
                    extension: file.name.split('.').pop(),
                    url: '',
                };

                this.fileName = file.name;
                this.$emit('input', fileObject);
                this.$emit('change', fileObject);
                this.loading = false;
            };

            // read blob url from file data
            reader.readAsArrayBuffer(file);
        },
        /**
         * Triggers the input file click
         */
        pickFile () {
            this.$refs.file.click();
        },
        /**
         * Removes the current file
         */
        removeFile () {
            this.fileName = '';
            this.$refs.file.value = '';
            this.$emit('input', {});
        },
    },
};
</script>
